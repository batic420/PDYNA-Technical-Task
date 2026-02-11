### PaaS
Für das Hosten der Applikation würde ich beim Backend auf **Azure Web Apps** setzen, diese ermöglichen ein schnelle Bereitstellung und wenig administrativen Aufwand - ganz im Gegensatz zu anderen Lösungen wie z.B. VMs. Diese Web Apps laufen dann über einen App Service Plan, der für jede Applikation separat erstellt wird, um den Blast Radius gering zu halten.

Das Frontend wird, da es sich um eine einfache Single Page Web Applikation handelt, auf **Azure Static Web Apps** deployed. Eine simple und effektive Lösung, die wenig Aufwand benötigt und gleichzeitig locker für ein solches Frontend ohne großartig spezielle Anforderungen ausreicht.

Für die NoSQL-Datenbank würde ich **Cosmos DB** verwenden. Hier erhält man eine native Azure-PaaS-Lösung, die für die Applikation (Backend) selbst wie eine echte MongoDB-Instanz wirkt. Das Backend nutzt den normalen MongoDB Connection String, um sich mit der DB zu verbinden und auch für den Applikationscode ist das kein Problem.

Für Logs, Backups oder applikationsspezifische Dateien (z.B. PDF-Dokumente, die man in eine solche Web-Applikation laden könnte), würde ich einen **Storage Account** als PaaS-Ressource verwenden, dieser stellt eine einfache und gleichzeitig hochverfügbare Speicherlösung bereit, die sich sehr vielseitig verwenden lässt.

Für Secrets innerhalb des Projekts, Zertifikate, die z.B. für Custom Domains von den App Services benutzt werden oder z.B. den Connection String der Cosmos DB Instanz stelle ich einen **Key Vault** bereit. Dieser sorgt dafür, dass Secrets nirgendwo im Code gespeichert werden müssen oder anderweitig im Projekt herumfliegen - er dient als zentraler Speicherpunkt.

Was zwar kein richtiger PaaS-Service ist aber eine entscheidende Rolle in meinem Setup ist, ist eine **Managed Identity**. Diese kann man z.B. dem Backend-App-Service zuweisen, über die Managed Identity, die dann eine entsprechende RBAC-Rolle erhält, kann man dann auf Ressourcen wie den Key Vault oder Storage Account zugreifen.

**Dieser Block gehört nicht direkt zur Applikation sondern kann für mehrere Apps verwendet werden:**

Für das Ausführen von Pipelines in Azure DevOps benötigen wir einen **Self-Hosted Runner**. Unser State File für die Terraform-Infrastruktur soll nämlich in einem privaten Storage Account aufbewahrt werden.
Eine VM kommt hier aufgrund des Management-Overheads und da PaaS-Services bevorzugt werden sollen nicht in Frage. Es bleibt hier also die Option **Managed DevOps Pools** übrig - diese Ressource kann in ein VNet deployed werden und so auf interne Ressourcen zugreifen, außerdem fallen Punkte wie die Wartung etc. für VMs vollständig weg. 
### SaaS
Da für die Infrastruktur als Cloud Provider Azure zum Einsatz kommt, würde ich für die Code-Verwaltung und CI/CD auf **Azure DevOps** zurückgreifen. Da beides aus dem Hause Microsoft kommt, lassen sich beide Welt sehr einfach miteinander verbinden.

So kann man bspw. mit einer Service Connection direkt eine Verbindung zu einer Azure Subscription bereitstellen und seine Infrastruktur sehr einfach dort deployen. 

Wer möchte, kann auch auf **HCP Terraform / Terraform Cloud** setzen, um seinen State zu managen. Da hier die Kosten für State-Management allerdings ziemlich schnell in die Höhe schießen können, würde ich eher auf einen Storage Account setzen, der zentral für mehrere Projekte oder Environments (Dev, Stage, Prod) genutzt wird.

**Folgender Block gehört ebenfalls zu HCP Terraform / Terraform Cloud, wurde aber in meinem Terraform-Code nicht so umgesetzt:**

Ein weiterer Pluspunkt, der für HCP spricht, ist die Private Module Registry. Durch sie kann man ganz einfach Module bereitstellen, ohne Terraform-Code für jedes Projekt nochmal komplett neu zu schreiben. Das spart Zeit und sorgt dafür, dass Ressourcen, die basierend auf diesen Modulen bereitgestellt werden, auch immer gleich aufgebaut sind.

**Folgende Bestandteile zählen ebenfalls nicht direkt zum Applikationsdeployment, werden aber im Daily Business standardmäßig verwendet:**

Ein weiterer Bestandteil, den ich nutzen würde, wären **AI-based Coding Agents**. Produkte wie z.B. Claude Code sind mittlerweile weit verbreitet und können bei richtiger Anwendung die Produktivität extrem steigern, sie könnten z.B. IaC-Reviews verfassen oder Troubleshooting betreiben.

Um meinen Fortschritt zu überwachen, Stakeholdern den aktuellen Stand mitzuteilen oder Meilensteine festzulegen würde ich **Jira** als Ticketing-Tool benutzen. Dieser spezielle Anwendungsfall würde dann als Ticket in einem Sprint untergebracht werden und während dessen Laufzeit von mir abgearbeitet werden.

Für eine nachträgliche Dokumentation meiner Umgebung oder weiteren Dingen wie der Anwendung meiner Pipelines würde ich **Confluence** verwenden. So können später basierend auf meiner Arbeit weitere, ähnliche Projekte umgesetzt werden und Pipelines (wenn universal für mehrere Use Cases einsetzbar) wiederverwendet werden. Außerdem ist transparent dokumentiert wie bestimmte Anwendungen aufgebaut sind, was es zu beachten gibt etc.

Für die Kommunikation im Team oder mit anderen Stakeholdern außerhalb des Tickets kommen gängige Kommunikationstools wie **Microsoft Teams** und **Outlook** zum Einsatz.