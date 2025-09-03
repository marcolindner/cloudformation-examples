# CloudFormation Lernprojekt - Stufe 2

Willkommen zum CloudFormation Lernprojekt! Diese Sammlung von Templates führt dich Schritt für Schritt durch die Grundlagen von AWS Infrastructure as Code.

## 📋 Überblick der Templates

### 1. Simple S3 Bucket (`simple-s3/`)
**Was lernen wir hier?**
- Grundaufbau eines CloudFormation Templates
- Einfache Ressourcen erstellen
- Eigenschaften (Properties) definieren

**Wichtige Konzepte:**
- `AWSTemplateFormatVersion`: Version des CloudFormation-Formats
- `Resources`: Hier werden alle AWS-Ressourcen definiert
- `Type`: Gibt an, welche Art von Ressource erstellt wird
- `Properties`: Konfiguriert die Ressource

### 2. Static Website Hosting (`static-website-hosting/`)
**Was lernen wir hier?**
- Parameter verwenden für Flexibilität
- Outputs definieren für wichtige Informationen
- AWS-Funktionen nutzen (`!Ref`, `!Sub`, `!GetAtt`)
- S3 Bucket Policies für öffentlichen Zugriff

**Wichtige Konzepte:**
- `Parameters`: Eingabewerte, die beim Deployment gesetzt werden
- `Outputs`: Wichtige Informationen nach dem Deployment
- `!Ref`: Referenziert andere Ressourcen oder Parameter
- `!Sub`: String-Substitution mit Variablen
- `!GetAtt`: Holt Attribute von anderen Ressourcen

### 3. VPC mit Netzwerk (`vpc/`)
**Was lernen wir hier?**
- Komplexere Netzwerk-Infrastruktur
- Abhängigkeiten zwischen Ressourcen
- Öffentliche vs. private Subnetze
- Internet Gateway und Route Tables

**Wichtige Konzepte:**
- `DependsOn`: Explizite Abhängigkeiten definieren
- Netzwerk-Segmentierung (Public/Private Subnets)
- Routing in AWS VPCs
- Security durch Network-Isolation

### 4. Security Groups (`vpc/security-groups.yaml`)
**Was lernen wir hier?**
- Firewall-Regeln in AWS
- Security Groups als virtuelle Firewalls
- Referenzen zwischen Security Groups
- Best Practices für Netzwerk-Sicherheit

## 🚀 Deployment-Anleitungen

### Voraussetzungen
- AWS CLI installiert und konfiguriert
- AWS-Account mit entsprechenden Berechtigungen
- Grundlegendes Verständnis der AWS-Konsole

### 1. Simple S3 Bucket deployen

```bash
cd simple-s3
aws cloudformation create-stack \
  --stack-name mein-s3-stack \
  --template-body file://template.yaml \
  --region eu-central-1
```

**Was passiert hier?**
- Ein privater S3 Bucket wird erstellt
- Versionierung ist aktiviert
- Der Bucket hat einen eindeutigen Namen mit Datum

### 2. Static Website deployen

```bash
cd static-website-hosting
aws cloudformation create-stack \
  --stack-name meine-website-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=BucketName,ParameterValue=meine-coole-website-2024 \
  --region eu-central-1
```

**Nach dem Deployment:**
1. Lade HTML-Dateien in den Bucket hoch
2. Nutze die Website-URL aus den Outputs
3. Teste den Zugriff im Browser

### 3. VPC-Netzwerk deployen

```bash
cd vpc
aws cloudformation create-stack \
  --stack-name mein-vpc-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=ProjectName,ParameterValue=lernprojekt \
               ParameterKey=Environment,ParameterValue=dev \
  --region eu-central-1
```

**Was wird erstellt?**
- Eine VPC (Virtual Private Cloud) mit CIDR 10.0.0.0/16
- Ein öffentliches Subnetz (10.0.1.0/24) mit Internet-Zugang
- Ein privates Subnetz (10.0.2.0/24) ohne direkten Internet-Zugang
- Internet Gateway für Internet-Verbindung
- Route Tables für Traffic-Routing

### 4. Security Groups deployen

**WICHTIG:** Erst nach dem VPC-Stack!

```bash
# VPC-ID aus dem ersten Stack holen
VPC_ID=$(aws cloudformation describe-stacks \
  --stack-name mein-vpc-stack \
  --query 'Stacks[0].Outputs[?OutputKey==`VPCId`].OutputValue' \
  --output text)

# Security Groups deployen
aws cloudformation create-stack \
  --stack-name meine-security-groups-stack \
  --template-body file://security-groups.yaml \
  --parameters ParameterKey=ProjectName,ParameterValue=lernprojekt \
               ParameterKey=Environment,ParameterValue=dev \
               ParameterKey=VPCId,ParameterValue=$VPC_ID \
  --region eu-central-1
```

## 🔍 Stack Status überprüfen

```bash
# Status aller Stacks anzeigen
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Details eines bestimmten Stacks
aws cloudformation describe-stacks --stack-name mein-vpc-stack

# Events/Logs eines Stacks
aws cloudformation describe-stack-events --stack-name mein-vpc-stack
```

## 🧹 Aufräumen (Cleanup)

**WICHTIG:** Stacks in umgekehrter Reihenfolge löschen!

```bash
# 1. Security Groups zuerst
aws cloudformation delete-stack --stack-name meine-security-groups-stack

# 2. Warten bis gelöscht, dann VPC
aws cloudformation wait stack-delete-complete --stack-name meine-security-groups-stack
aws cloudformation delete-stack --stack-name mein-vpc-stack

# 3. Andere Stacks
aws cloudformation delete-stack --stack-name meine-website-stack
aws cloudformation delete-stack --stack-name mein-s3-stack
```

## 📚 Wichtige CloudFormation-Konzepte erklärt

### Parameter vs. Hard-coded Values
```yaml
# Schlecht: Hard-coded
BucketName: "mein-fester-bucket-name"

# Gut: Parameter
BucketName: !Ref BucketNameParameter
```

### Intrinsic Functions (AWS-Funktionen) - Ausführlich erklärt

CloudFormation bietet spezielle Funktionen, die zur Laufzeit ausgewertet werden. Diese beginnen immer mit `!` und sind essentiell für dynamische Templates.

#### `!Ref` - Referenziert Ressourcen oder Parameter
**Was macht es?** Gibt die ID oder den Wert einer anderen Ressource/Parameter zurück.

**Beispiele:**
```yaml
Parameters:
  BucketName:
    Type: String
    Default: "mein-bucket"

Resources:
  MeinBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName  # Referenziert den Parameter
      
  MeinePolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref MeinBucket     # Referenziert die Bucket-Ressource
```

**Was passiert hier?**
- `!Ref BucketName` → Gibt den Wert des Parameters zurück ("mein-bucket")
- `!Ref MeinBucket` → Gibt die Bucket-ID zurück (z.B. "mein-bucket-xyz123")

#### `!GetAtt` - Holt spezifische Attribute von Ressourcen
**Was macht es?** Jede AWS-Ressource hat verschiedene Attribute. `!GetAtt` holt ein spezifisches Attribut.

**Beispiele:**
```yaml
Resources:
  WebsiteBucket:
    Type: AWS::S3::Bucket
    Properties:
      WebsiteConfiguration:
        IndexDocument: index.html

Outputs:
  WebsiteURL:
    Value: !GetAtt WebsiteBucket.WebsiteURL  # Holt die Website-URL
  BucketArn:
    Value: !GetAtt WebsiteBucket.Arn         # Holt den ARN
  DomainName:
    Value: !GetAtt WebsiteBucket.DomainName  # Holt den Domain-Namen
```

**Was passiert hier?**
- `WebsiteURL` könnte sein: "http://mein-bucket.s3-website-eu-central-1.amazonaws.com"
- `Arn` könnte sein: "arn:aws:s3:::mein-bucket"
- `DomainName` könnte sein: "mein-bucket.s3.amazonaws.com"

#### `!Sub` - String-Substitution mit Variablen
**Was macht es?** Ersetzt Platzhalter in Strings mit tatsächlichen Werten.

**Einfache Syntax:**
```yaml
Resources:
  MeinBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${ProjectName}-${Environment}-bucket"
      # Wird zu: "meinprojekt-dev-bucket"
```

**Komplexe Syntax mit mehreren Variablen:**
```yaml
Resources:
  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      PolicyDocument:
        Statement:
          - Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Sub 
              - 'arn:aws:s3:::${BucketName}/*'
              - BucketName: !Ref MeinBucket
```

**Praktisches Beispiel mit Tags:**
```yaml
Tags:
  - Key: Name
    Value: !Sub "${ProjectName}-${Environment}-vpc"
  - Key: Environment  
    Value: !Ref Environment
  - Key: FullName
    Value: !Sub "Projekt: ${ProjectName} in ${Environment} erstellt am ${AWS::StackName}"
```

#### `!Select` - Wählt Element aus einer Liste
**Was macht es?** Wählt ein Element an einer bestimmten Position aus einer Liste.

**Beispiele:**
```yaml
# Wählt die erste Availability Zone
Resources:
  MeinSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs '']  # Erste AZ
      # Ergebnis: z.B. "eu-central-1a"

  ZweitesSubnet:
    Type: AWS::EC2::Subnet  
    Properties:
      AvailabilityZone: !Select [1, !GetAZs '']  # Zweite AZ
      # Ergebnis: z.B. "eu-central-1b"
```

**Mit eigenen Listen:**
```yaml
Parameters:
  Environment:
    Type: String
    Default: "dev"

Mappings:
  EnvironmentMap:
    dev:
      InstanceType: t2.micro
      StorageSize: 20
    prod:
      InstanceType: t3.medium  
      StorageSize: 100

Resources:
  MeineInstanz:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Select [0, !FindInMap [EnvironmentMap, !Ref Environment, [InstanceType, StorageSize]]]
```

#### `!GetAZs` - Holt alle Availability Zones
**Was macht es?** Gibt eine Liste aller verfügbaren Availability Zones für die aktuelle Region zurück.

**Beispiele:**
```yaml
# Automatisch alle AZs der aktuellen Region holen
Resources:
  ErsteSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs '']  # Erste verfügbare AZ
      
  ZweiteSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [1, !GetAZs '']  # Zweite verfügbare AZ

# Spezifische Region angeben
  DrittesSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [0, !GetAZs 'us-west-2']  # Erste AZ in us-west-2
```

**Warum ist das nützlich?**
- Dein Template funktioniert in jeder AWS-Region
- Du musst nicht wissen, welche AZs verfügbar sind
- Automatische Verteilung auf verschiedene AZs für Hochverfügbarkeit

#### Kombinierte Beispiele - So werden sie in der Praxis verwendet:

**Beispiel 1: VPC mit dynamischen Namen und AZs**
```yaml
Parameters:
  ProjectName:
    Type: String
    Default: "webshop"
  Environment:
    Type: String
    Default: "dev"

Resources:
  MeineVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: "10.0.0.0/16"
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-vpc"  # "webshop-dev-vpc"

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MeineVPC                               # Referenziert die VPC
      CidrBlock: "10.0.1.0/24"
      AvailabilityZone: !Select [0, !GetAZs '']         # Erste verfügbare AZ
      Tags:
        - Key: Name
          Value: !Sub "${ProjectName}-${Environment}-public-subnet"

Outputs:
  VPCId:
    Description: "ID der erstellten VPC"
    Value: !Ref MeineVPC                                 # VPC-ID zurückgeben
    
  VPCArn:
    Description: "ARN der VPC"
    Value: !GetAtt MeineVPC.Arn                         # VPC-ARN zurückgeben
    
  SubnetAZ:
    Description: "AZ des Subnetzes"  
    Value: !GetAtt PublicSubnet.AvailabilityZone        # AZ des Subnetzes
```

**Beispiel 2: S3 Bucket mit dynamischer Policy**
```yaml
Parameters:
  BucketName:
    Type: String
    Default: "meine-website"

Resources:
  WebsiteBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${BucketName}-${AWS::Region}-${AWS::AccountId}"  # Eindeutiger Name
      
  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref WebsiteBucket                        # Referenziert den Bucket
      PolicyDocument:
        Statement:
          - Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Sub 
              - "${BucketArn}/*"                         # String-Substitution
              - BucketArn: !GetAtt WebsiteBucket.Arn     # Bucket-ARN holen

Outputs:
  WebsiteURL:
    Value: !Sub 
      - "http://${BucketName}.s3-website.${AWS::Region}.amazonaws.com"
      - BucketName: !Ref WebsiteBucket
```

#### Häufige Fehler und wie man sie vermeidet:

**❌ Falscher Syntax:**
```yaml
# FALSCH - Vergessene Anführungszeichen
Value: !Sub ${ProjectName}-${Environment}-bucket

# RICHTIG
Value: !Sub "${ProjectName}-${Environment}-bucket"
```

**❌ Falsches Attribut:**
```yaml
# FALSCH - WebsiteURL existiert nur bei Website-konfigurierten Buckets
Value: !GetAtt MeinNormalerBucket.WebsiteURL

# RICHTIG - Prüfe AWS-Dokumentation für verfügbare Attribute
Value: !GetAtt MeinNormalerBucket.Arn
```

**❌ Index außerhalb des Bereichs:**
```yaml
# FALSCH - Nicht alle Regionen haben 3+ AZs
AvailabilityZone: !Select [3, !GetAZs '']

# RICHTIG - Bleib bei 0, 1, 2 für maximale Kompatibilität  
AvailabilityZone: !Select [0, !GetAZs '']
```

### Outputs - Warum wichtig?
Outputs zeigen wichtige Informationen nach dem Deployment:
- Ressourcen-IDs für andere Stacks
- URLs von erstellten Websites
- ARNs für Berechtigungen

### Tags - Best Practice
Immer Tags verwenden für:
- Kostenverfolgung
- Umgebungsidentifikation
- Automatisierte Verwaltung

## 🔧 Troubleshooting

### Häufige Fehler

**"Stack already exists"**
```bash
# Stack-Status prüfen
aws cloudformation describe-stacks --stack-name STACK_NAME
# Falls nötig, erst löschen
aws cloudformation delete-stack --stack-name STACK_NAME
```

**"Insufficient IAM permissions"**
- Prüfe deine AWS-Berechtigungen
- CloudFormation braucht Rechte für alle verwendeten Services

**"Parameter validation failed"**
- Prüfe Parameter-Namen und -Werte
- Sind alle Required-Parameter gesetzt?

**"Resource already exists"**
- Manche Ressourcen müssen eindeutige Namen haben
- Ändere Parameter oder füge Zufallswerte hinzu

### Debugging-Tipps

1. **CloudFormation-Console nutzen:**
   - Gehe zu AWS CloudFormation Console
   - Schaue unter "Events" für detaillierte Fehler

2. **Template validieren:**
   ```bash
   aws cloudformation validate-template --template-body file://template.yaml
   ```

3. **Dry-Run mit Change Sets:**
   ```bash
   aws cloudformation create-change-set \
     --stack-name mein-stack \
     --template-body file://template.yaml \
     --change-set-name test-changes
   ```

## 🎯 Nächste Schritte

Wenn du alle Templates erfolgreich deployed hast:

1. **Experimentiere mit Parametern:** Ändere Werte und deploye erneut
2. **Schaue dir die AWS-Konsole an:** Sieh dir die erstellten Ressourcen an
3. **Versuche eigene Modifikationen:** Füge neue Tags hinzu, ändere CIDR-Blöcke
4. **Bereit für Stufe 3?** Als nächstes kommen EC2-Instanzen in deine VPC!

## 💡 Pro-Tipps

- **Immer mit kleinen Templates anfangen** und dann erweitern
- **Parameters nutzen** macht Templates wiederverwendbar
- **Outputs definieren** für alle wichtigen Ressourcen-IDs
- **Tags sind dein Freund** für Organisation und Kosten
- **Stack-Namen** sollten aussagekräftig sein

Viel Erfolg beim Lernen! 🚀