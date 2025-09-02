# AWS CLI Installation und SSO-Konfiguration für Windows

## 1. AWS CLI Installation

### Option A: MSI Installer (Empfohlen)

1. Lade den AWS CLI MSI Installer herunter:
   - Für 64-bit Windows: https://awscli.amazonaws.com/AWSCLIV2.msi
   - Für 32-bit Windows: https://awscli.amazonaws.com/AWSCLIV2-32.msi

2. Führe die heruntergeladene MSI-Datei aus
3. Folge dem Installationsassistenten
4. Nach der Installation, öffne eine neue Eingabeaufforderung (CMD) oder PowerShell

### Option B: Installation über PowerShell

```powershell
# PowerShell als Administrator öffnen und ausführen:
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

### Installation überprüfen

```cmd
# In CMD oder PowerShell ausführen:
aws --version

# Sollte etwa folgendes ausgeben:
# aws-cli/2.x.x Python/3.x.x Windows/10 exe/AMD64
```

## 2. SSO-Konfiguration

### Schritt 1: SSO-Profil erstellen

Öffne CMD oder PowerShell und führe folgenden Befehl aus:

```cmd
aws configure sso
```

### Schritt 2: Konfigurationswerte eingeben

Du wirst nach folgenden Werten gefragt. Gib sie genau wie unten angegeben ein:

```
SSO session name (Recommended): techstarter-sandbox
SSO start URL: https://techstarter-sandboxes.awsapps.com/start/#
SSO region: eu-central-1
SSO registration scopes [sso:account:access]: 
```
(Bei "SSO registration scopes" einfach Enter drücken für den Default-Wert)

### Schritt 3: Browser-Authentifizierung

1. Es öffnet sich automatisch dein Browser
2. Melde dich mit deinen SSO-Credentials an
3. Bestätige die Autorisierung ("Allow")
4. Du kannst den Browser-Tab schließen und zur Konsole zurückkehren

### Schritt 4: Account und Rolle auswählen

Nach erfolgreicher Authentifizierung siehst du eine Liste verfügbarer Accounts:

```
There are X AWS accounts available to you.
> DemoAccount-123456789012
  OtherAccount-987654321098
```

Wähle deinen gewünschten Account mit den Pfeiltasten aus und drücke Enter.

Dann wähle die IAM-Rolle aus (normalerweise gibt es nur eine Option).

### Schritt 5: Profil benennen und Region festlegen

```
CLI default client Region [eu-central-1]: eu-central-1
CLI default output format [None]: json
CLI profile name [DemoAccount-123456789012]: techstarter-sandbox
```

## 3. SSO-Login und Nutzung

### Anmelden

Vor der Nutzung der AWS CLI musst du dich anmelden:

```cmd
aws sso login --profile techstarter-sandbox
```

Dies öffnet wieder deinen Browser zur Authentifizierung. Nach erfolgreicher Anmeldung bist du für mehrere Stunden eingeloggt.

### AWS CLI Befehle mit SSO-Profil verwenden

```cmd
# Beispiel: S3 Buckets auflisten
aws s3 ls --profile techstarter-sandbox

# Beispiel: EC2 Instanzen anzeigen
aws ec2 describe-instances --profile techstarter-sandbox --region eu-central-1
```

### Profil als Standard setzen (Optional)

Um nicht bei jedem Befehl `--profile` angeben zu müssen:

```cmd
# Windows CMD:
set AWS_PROFILE=techstarter-sandbox

# Windows PowerShell:
$env:AWS_PROFILE="techstarter-sandbox"

# Dauerhaft für PowerShell (in PowerShell-Profil speichern):
Add-Content $PROFILE "`n`$env:AWS_PROFILE='techstarter-sandbox'"
```

Nach dem Setzen der Umgebungsvariable kannst du AWS CLI Befehle ohne `--profile` ausführen:

```cmd
aws s3 ls
aws ec2 describe-instances --region eu-central-1
```

## 4. CloudFormation Stacks mit SSO deployen

### Mit explizitem Profil

```cmd
# Stack erstellen
aws cloudformation create-stack ^
  --stack-name mein-stack ^
  --template-body file://template.yaml ^
  --profile techstarter-sandbox ^
  --region eu-central-1

# Stack löschen
aws cloudformation delete-stack ^
  --stack-name mein-stack ^
  --profile techstarter-sandbox ^
  --region eu-central-1
```

### Mit gesetzter Umgebungsvariable

```cmd
# Wenn AWS_PROFILE gesetzt ist:
aws cloudformation create-stack ^
  --stack-name mein-stack ^
  --template-body file://template.yaml ^
  --region eu-central-1
```

## 5. Troubleshooting

### Session abgelaufen

Wenn du die Fehlermeldung "The SSO session associated with this profile has expired" erhältst:

```cmd
aws sso login --profile techstarter-sandbox
```

### Profil-Konfiguration anzeigen

```cmd
# Zeigt alle konfigurierten Profile
aws configure list-profiles

# Zeigt die Konfiguration eines spezifischen Profils
aws configure list --profile techstarter-sandbox
```

### Konfigurationsdatei manuell bearbeiten

Die AWS CLI Konfiguration wird in folgenden Dateien gespeichert:
- `C:\Users\%USERNAME%\.aws\config`
- `C:\Users\%USERNAME%\.aws\credentials`

Du kannst diese mit einem Texteditor bearbeiten, falls nötig.

### SSO Cache löschen (bei Problemen)

```cmd
# SSO Cache Verzeichnis löschen
rmdir /s /q C:\Users\%USERNAME%\.aws\sso\cache
```

## 6. Nützliche Aliase für PowerShell

Füge diese zu deinem PowerShell-Profil hinzu (`$PROFILE`):

```powershell
# AWS SSO Login Shortcut
function awslogin {
    aws sso login --profile techstarter-sandbox
}

# AWS mit Profil
function awst {
    aws @args --profile techstarter-sandbox
}

# Verwendung:
# awslogin           # Für SSO Login
# awst s3 ls         # Statt: aws s3 ls --profile techstarter-sandbox
```

## 7. Best Practices

1. **Regelmäßig ausloggen**: Nach der Arbeit `aws sso logout` ausführen
2. **Profile verwenden**: Verschiedene Profile für verschiedene Accounts/Umgebungen
3. **Region explizit angeben**: Immer `--region` verwenden für Konsistenz
4. **Output-Format**: JSON ist gut für Scripting, Table für Lesbarkeit

## Zusammenfassung

Nach erfolgreicher Konfiguration ist dein typischer Workflow:

1. Terminal öffnen
2. `aws sso login --profile techstarter-sandbox` (einmal pro Session)
3. AWS CLI Befehle mit `--profile techstarter-sandbox` ausführen
4. Nach der Arbeit: `aws sso logout`

Die SSO-Session bleibt normalerweise 8-12 Stunden aktiv, abhängig von den SSO-Einstellungen.