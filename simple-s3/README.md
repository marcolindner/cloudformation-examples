# Simple S3 Bucket CloudFormation Stack

Dieses CloudFormation Template erstellt einen einfachen privaten S3 Bucket mit aktivierter Versionierung.

## Voraussetzungen

- AWS CLI installiert und konfiguriert
- Ausreichende AWS Berechtigungen zum Erstellen von S3 Buckets

## Deployment

### Option 1: AWS CLI

```bash
# Stack erstellen
aws cloudformation create-stack \
  --stack-name simple-s3-stack \
  --template-body file://template.yaml \
  --region eu-central-1

# Status überprüfen
aws cloudformation describe-stacks \
  --stack-name simple-s3-stack \
  --region eu-central-1 \
  --query 'Stacks[0].StackStatus'

# Warten bis Stack fertig ist
aws cloudformation wait stack-create-complete \
  --stack-name simple-s3-stack \
  --region eu-central-1
```

### Option 2: AWS Management Console

1. Öffne die AWS CloudFormation Konsole
2. Klicke auf "Create Stack"
3. Wähle "Upload a template file"
4. Lade die `template.yaml` Datei hoch
5. Gib dem Stack einen Namen (z.B. `simple-s3-stack`)
6. Klicke durch die nächsten Schritte und erstelle den Stack

## Stack Löschen

### Option 1: AWS CLI

```bash
# Bucket zuerst leeren (notwendig, da S3 Buckets nicht gelöscht werden können, wenn sie nicht leer sind)
aws s3 rm s3://marcos-simple-s3-bucket-02092025 --recursive

# Stack löschen
aws cloudformation delete-stack \
  --stack-name simple-s3-stack \
  --region eu-central-1

# Warten bis Stack gelöscht ist
aws cloudformation wait stack-delete-complete \
  --stack-name simple-s3-stack \
  --region eu-central-1
```

### Option 2: AWS Management Console

1. Öffne die AWS CloudFormation Konsole
2. Wähle deinen Stack aus
3. Klicke auf "Delete"
4. Bestätige die Löschung

**Wichtig:** Der S3 Bucket muss leer sein, bevor er gelöscht werden kann. Lösche alle Objekte im Bucket vor dem Stack-Delete.

## Ressourcen

Dieser Stack erstellt folgende Ressourcen:
- **S3 Bucket**: `marcos-simple-s3-bucket-02092025` mit privater Zugriffskontrolle und aktivierter Versionierung

## Hinweise

- Der Bucket-Name muss global eindeutig sein. Falls der Name bereits vergeben ist, ändere ihn in der `template.yaml`
- Die Versionierung ist aktiviert, wodurch alle Änderungen an Objekten nachvollziehbar sind
- Der Bucket ist privat konfiguriert und hat keinen öffentlichen Zugriff