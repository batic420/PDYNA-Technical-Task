### Development-Strategie
Um alle drei Stages auf der gleichen Version zu halten würde ich für die Entwicklung der Infrastruktur Trunk-based-Development verwenden. So haben wir immer nur einen branch "main" auf dem alle Stages basieren.

Wenn man neue Features, z.B. eine neue Ressource hinzufügen möchte, erstellt man sich einfach einen feature-branch und sobald man fertig ist, merged man das feature auf main. Diese Feature-Branches sind kurzlebig und sollen nicht genutzt werden, um Deployments zu starten! "main" als Branch soll hier die einzige Quelle sein, die für Deployments genutzt wird. So kann man immer sicherstellen, dass alle 3 Stages den gleichen Stand aufweisen. 

### Deployment-Stratgie
Bevor wir überhaupt deployen, setzen wir auf Pull Requests. Sowohl alleine als auch im Team macht das Sinn, damit nichts ohne Review auf "main" darf. Hier würde ich zusätzlich auch noch eine PR-Validation-Pipeline einsetzen, die direkt beim Erstellen eines Pull Requests getriggert wird.

Für den Trigger nutze ich Branch Policies. Hierfür auf den "main"-Branch gehen, eine Branch Policy erstellen und eine Build Validation hinzufügen. Dort wähle ich dann die Validation-Pipeline aus.

Diese Pipeline wird dann bei jedem PR getriggert und führt direkt Checks wie ein "terraform fmt", "terraform validate" und "terraform plan" aus, so kann man direkt die Auswirkungen des neuen Features auf die Infrastruktur erkennen, ohne ein Deployment je gestartet zu haben.

Sobald der Merge bzw. der neue Entwicklungsstand auf "main" (PR war erfolgreich) ist, wird die Deployment-Pipeline getriggert, sie deployed automatisch basierend auf "main" die Infrastruktur. Gestartet wird hier bei der "unkritischsten" Umgebung - "Dev". Auf der nächsten Umgebung "Stage" startet das Deployment automatisch, sobald es auf "Dev" ohne Fehler abgeschlossen wurde.

Wurde das Deployment erfolgreich auf "Stage" abgeschlossen, wird die Pipeline pausiert und es folgt ein Environment Approval (Azure DevOps). Hier muss das Platform-Team oder eine Einzelperson manuell bestätigen, dass die Pipeline laufen soll, andernfalls wird nicht auf "Prod" deployed.

Für jedes Environment packe ich den Output des Terraform Plans in ein Artefact und stelle dieses dann nachträglich bereit, so kann man zu jeder Zeit genau nachverfolgen, welcher Pipeline-Run welche Veränderungen an der Infrastruktur bewirkt hat.