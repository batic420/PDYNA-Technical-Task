**terraform-pr-validation.yml:**
```pseudo
TRIGGER: Pull Request gegen "main" Branch
RUNNER: Self-Hosted-Agent (Managed DevOps Pool)

1. Formatierung und Syntax überprüfen:
   - Formatierung mit "terraform fmt" checken
   - Syntax mit "terraform validate" checken
   - Bei Fehler, Pipeline abbrechen und PR blockieren
   - Wenn erfolgreich, Nächster Schritt

2. Plan für jedes Environment generieren (Dev, Stage, Prod):
   - Init mit jeweiliger Backend-Config
   - Plan mit jeweiligem .tfvars-file
   - Plan-Output als Kommentar in den PR packen

ERGEBNIS:
   - Alle drei Plan-Outputs im PR ersichtlich (Bewertung)
   - Kein Apply - nur lesen, bewerten + genehmigen o. ablehnen
```

**terraform-deploy-template.yml:**
```pseudo
PARAMETER: "projectPath" - Pfad zum jeweiligen Projekt
RUNNER: Self-Hosted-Agent (Managed DevOps Pool)

1. Deployment "Dev"
   - "terraform init" mit backend-config als Parameter
   - "terraform plan" und Output in Dev-Outputs schreiben
   - "terraform apply" fürs Deployment der Infrastruktur
   - Bei erfolgreichem Deployment:
	     - Dev-Outputs als Artifact bereitstellen
	     - Automatisch zum nächsten Environment
   - Bei Fehler:
		 - Abbruch der Pipeline
		 - Ausgabe des Fehlers

2. Deployment "Stage"
   - "terraform init" mit backend-config als Parameter
   - "terraform plan" und Output in Stage-Outputs schreiben
   - "terraform apply" fürs Deployment der Infrastruktur
   - Bei erfolgreichem Deployment:
	     - Stage-Outputs als Artifact bereitstellen
	     - Approval Gate für Prod triggern
   - Bei Fehler:
		 - Abbruch der Pipeline
		 - Ausgabe des Fehlers

3. Approval Gate
   - Pausiert die Pipeline und erfordert eine manuelle Bestätigung für die weitere Ausführung
   - Bei Bestätigung -> Anstoßen des Prod-Deployments
   - Bei Ablehnung -> Beenden der Pipeline

4. Deployment "Prod"
   - "terraform init" mit backend-config als Parameter
   - "terraform plan" und Output in Prod-Outputs schreiben
   - "terraform apply" fürs Deployment der Infrastruktur
   - Bei erfolgreichem Deployment:
	     - Prod-Outputs als Artifact bereitstellen
	     - Beenden der Pipeline
   - Bei Fehler:
		 - Abbruch der Pipeline
		 - Ausgabe des Fehlers

ERGEBNIS:
- Template kann für jedes Projekt in der Codebase genutzt werden
- DRY-Ansatz bei dem die Projekt-Pipelines selbst minimal gehalten werden
```

**deploy-{terraform-project}.yml:**
```pseudo
TRIGGER: Änderung auf Branch "main" im Projekt-Pfad

1. Template mit Parameter ausführen
   - Template aufrufen und den Projekt-Pfad und verwendeten Module-Pfaden als Parameter mitgeben

ERGEBNIS:
- Template i.V.m. Projekt-Pfad definiert das Deployment
- Projekt-Pipeline wird nur angestoßen bei Veränderungen im Projekt
```

**app-frontend-deployment.yml:**
```pseudo
TRIGGER: Änderung auf dem Main-Branch im Frontend-Repo (oder Frontend-Pfad)
RUNNER: Self-Hosted-Agent (Managed DevOps Pool)

1. Build und Test 
   - Dependencies installieren (npm install) 
   - Linting ausführen (npm run lint) 
	     - Code-Qualität prüfen, Style-Verstöße erkennen 
   - Unit Tests ausführen (npm run test) 
	     - Bei Fehler: 
			   - Abbruch der Pipeline 
   - Produktions-Build erstellen (npm run build) 
	     - Erzeugt den dist/ Ordner mit statischen Dateien 
   - Build-Artifact (dist/ Ordner) sichern für die nächsten Stages 

2. Deploy auf "Dev" 
   - Build-Artifact laden 
   - Auf den Dev App Service deployen (oder Static Web App / Storage Account) 
   - Smoke Test: HTTP-Request an die Dev-URL 
	     - Antwort 200? Weiter zur nächsten Stage
	     - Keine Antwort oder Fehler? Abbruch der Pipeline. 

3. Deploy auf "Stage" 
   - Build-Artifact laden 
   - Auf den Stage App Service deployen 
   - Smoke Test: HTTP-Request an die Stage-URL 
	     - Bei Fehler: Abbruch der Pipeline 

4. Environment Approval 
   - Warten auf manuelle Genehmigung für Prod 
   - Bei Ablehnung: Pipeline beenden 

1. Deploy auf "Prod" 
   - Build-Artifact laden 
   - Auf den Prod App Service deployen 
   - Smoke Test: HTTP-Request an die Prod-URL 
	     - Bei Fehler: Abbruch, Rollback erwägen

ERGEBNIS:
- Frontend für Applikation deployed und getestet
``` 

**app-backend-deployment.yml:**
```pseudo
TRIGGER: Änderung auf Main-Branch im Backend-Repo (oder Backend-Pfad)
RUNNER: Self-Hosted Agent (Managed DevOps Pool)

1. Build und Test
   - Dependencies installieren (npm install)
   - Linting ausführen (npm run lint)
   - Unit Tests ausführen (npm run test)
	     - Bei Fehler: Abbruch der Pipeline
   - Anwendung paketieren
		 - ZIP-Paket erstellen (für ZIP-Deploy auf App Service)
   - Build-Artifact sichern

2. Deploy auf "Dev"
   - Build-Artifact laden (ZIP)
   - Auf den Dev App Service deployen
   - Smoke Test: HTTP-Request an den Health-Endpoint (z.B. GET /api/health)
	     - Prüft: Server antwortet UND Datenbankverbindung steht
		 - Bei Fehler: Abbruch der Pipeline

3. Deploy auf "Stage"
   - Build-Artifact laden
   - Auf den Stage App Service deployen
   - Smoke Test: Health-Endpoint prüfen
	     - Bei Fehler: Abbruch der Pipeline

4. Environment Approval
   - Warten auf manuelle Genehmigung für Prod
   - Bei Ablehnung: Pipeline beenden

5. Deploy auf "Prod"
   - Build-Artifact laden
   - Auf den Prod App Service deployen
   - Smoke Test: Health-Endpoint prüfen
	     - Bei Fehler: Abbruch, Rollback erwägen

ERGEBNIS:
- Backend für Applikation deployed und getestet
```