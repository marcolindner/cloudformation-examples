# Static Website Hosting mit S3 CloudFormation Stack

Dieses CloudFormation Template erstellt einen S3 Bucket für das Hosting einer statischen Website mit öffentlichem Lesezugriff.

## Voraussetzungen

- AWS CLI installiert und konfiguriert
- Ausreichende AWS Berechtigungen zum Erstellen von S3 Buckets und Bucket Policies
- HTML-Dateien für die Website (index.html, error.html)

## Deployment

### Option 1: AWS CLI

```bash
# Stack mit Standard-Bucket-Namen erstellen
aws cloudformation create-stack \
  --stack-name static-website-stack \
  --template-body file://template.yaml \
  --region eu-central-1

# ODER mit eigenem Bucket-Namen
aws cloudformation create-stack \
  --stack-name static-website-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=BucketName,ParameterValue=mein-unique-bucket-name \
  --region eu-central-1

# Status überprüfen
aws cloudformation describe-stacks \
  --stack-name static-website-stack \
  --region eu-central-1 \
  --query 'Stacks[0].StackStatus'

# Warten bis Stack fertig ist
aws cloudformation wait stack-create-complete \
  --stack-name static-website-stack \
  --region eu-central-1

# Website URL abrufen
aws cloudformation describe-stacks \
  --stack-name static-website-stack \
  --region eu-central-1 \
  --query 'Stacks[0].Outputs[?OutputKey==`WebsiteURL`].OutputValue' \
  --output text
```

### Option 2: AWS Management Console

1. Öffne die AWS CloudFormation Konsole
2. Klicke auf "Create Stack"
3. Wähle "Upload a template file"
4. Lade die `template.yaml` Datei hoch
5. Gib dem Stack einen Namen (z.B. `static-website-stack`)
6. Optional: Ändere den Parameter `BucketName` zu einem eindeutigen Namen
7. Klicke durch die nächsten Schritte und erstelle den Stack
8. Nach erfolgreichem Deployment findest du die Website URL in den Stack Outputs

## HTML-Dateien hochladen

Nach dem Erstellen des Stacks müssen die HTML-Dateien in den Bucket hochgeladen werden:

```bash
# Bucket-Namen aus Stack-Outputs abrufen
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name static-website-stack \
  --region eu-central-1 \
  --query 'Stacks[0].Outputs[?OutputKey==`BucketName`].OutputValue' \
  --output text)

# HTML-Dateien hochladen
aws s3 cp index.html s3://$BUCKET_NAME/index.html
aws s3 cp error.html s3://$BUCKET_NAME/error.html
aws s3 cp hello.html s3://$BUCKET_NAME/hello.html

# Oder alle HTML-Dateien auf einmal
aws s3 sync . s3://$BUCKET_NAME/ --exclude "*" --include "*.html"
```

## Website testen

```bash
# Website URL abrufen und öffnen
WEBSITE_URL=$(aws cloudformation describe-stacks \
  --stack-name static-website-stack \
  --region eu-central-1 \
  --query 'Stacks[0].Outputs[?OutputKey==`WebsiteURL`].OutputValue' \
  --output text)

echo "Website URL: $WEBSITE_URL"

# Optional: Website im Browser öffnen (macOS)
open $WEBSITE_URL
```

## Stack Löschen

### Option 1: AWS CLI

```bash
# Bucket-Namen abrufen
BUCKET_NAME=$(aws cloudformation describe-stacks \
  --stack-name static-website-stack \
  --region eu-central-1 \
  --query 'Stacks[0].Outputs[?OutputKey==`BucketName`].OutputValue' \
  --output text)

# Bucket leeren (notwendig vor dem Löschen)
aws s3 rm s3://$BUCKET_NAME --recursive

# Stack löschen
aws cloudformation delete-stack \
  --stack-name static-website-stack \
  --region eu-central-1

# Warten bis Stack gelöscht ist
aws cloudformation wait stack-delete-complete \
  --stack-name static-website-stack \
  --region eu-central-1
```

### Option 2: AWS Management Console

1. Öffne die AWS S3 Konsole
2. Lösche alle Objekte im Website-Bucket
3. Öffne die AWS CloudFormation Konsole
4. Wähle deinen Stack aus
5. Klicke auf "Delete"
6. Bestätige die Löschung

**Wichtig:** Der S3 Bucket muss leer sein, bevor er gelöscht werden kann!

## Ressourcen

Dieser Stack erstellt folgende Ressourcen:
- **S3 Bucket**: Für das Hosting der statischen Website
- **Bucket Policy**: Erlaubt öffentlichen Lesezugriff auf alle Objekte im Bucket

## Stack Outputs

- **WebsiteURL**: Die URL der gehosteten Website
- **BucketName**: Der Name des erstellten S3 Buckets

## Hinweise

- Der Bucket-Name muss global eindeutig sein
- Die Website ist öffentlich zugänglich - alle Dateien im Bucket können von jedem gelesen werden
- Lade nur Inhalte hoch, die öffentlich zugänglich sein sollen
- Die Bucket Policy erlaubt nur Lesezugriff (GetObject), kein Schreibzugriff