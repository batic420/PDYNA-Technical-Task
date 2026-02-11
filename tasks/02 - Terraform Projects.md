### Module
Die Definition der Ressourcen, die für die Applikation genutzt werden, können entweder lokal über einen separaten `modules/` Ordner lokal definiert werden oder man setzt hier auf remote Modules, hierfür braucht man dann eine Hosting-Lösung - ein gutes Beispiel hierfür ist die Terraform Private Registry (SaaS-Lösung). Das Anlegen von Modulen, das Verknüpfen mit einem entsprechenden Azure DevOps Repo etc. ist alles kostenlos verfügbar (darüber hinaus sind Features dann aber wieder kostenpflichtig).

Generell folge ich der These, dass alles was für mehr als ein Projekt genutzt werden könnte, tendenziell als remote Module bereitgestellt werden sollte. So spart man sich doppelten Terraform-Code in jedem Projekt und hat einen zentralen Punkt, der (je nach Version) auch immer die gleiche Infrastruktur deployed.
### Terraform Projekte
Als Basis für die Applikation dient der Netzwerk-Teil, deshalb wäre mein erstes Projekt **"01-connectivity"**. Hier wird die initiale Netzwerk-Infrastruktur deployed - z.B. VNet, Subnets, NSGs, Route Tables etc. - dieser Teil wird jedoch nur einmal pro Environment (Dev, Stage, Prod) deployed und kann für mehrere Apps verwendet werden.

Neben dem Netzwerk gibt es noch weitere Ressourcen, die sich Applikationen teilen, z.B. einen Log Analytics Workspace pro Environment für das Monitoring - mein nächstes Projekt wäre also **"02-shared-services"**. Auch dieser Teil wird nur einmal pro Environment erstellt.

Abschließend kommt mein drittes Projekt, hierbei handelt es sich um das Deployment für die eigentliche Platform unser App. Also Frontend, Backend, Database, Storage, Key Vault und Managed Identity. Dieses Projekt könnte z.B. **"03-spa"** heißen, allerdings können Subfolder für mehrere Applikationen angelegt werden, dieser Teil kann also im Gegensatz zu den anderen Projekten mehrfach pro Environment deployed werden, je nachdem wie viele Apps ich pro Environment erstellen möchte.

Man könnte also z.B. noch einen vierten Subfolder für eine weitere App mit dem Namen **"04-hr-doc-translator"** erstellen.

Eine vollständige Übersicht über ein leere Variante der Terraform-Codebase ist unter dem `terraform/` Ordner ersichtlich.

Da diese Architektur aber ziemlich schlecht skaliert, gibt es verschiedene Möglichkeiten ein solches Szenario zu vermeiden.

**Ein möglicher Deployment-Workflow wäre Folgender:**

Die Plattform-Ressourcen (Projekt 1 und Projekt 2 in meinem Fall) werden in ein separates Repo gepackt. Diese werden unabhängig von den Applikation einmal für jedes Environment deployed.

Jedes Applikationsteam (z.B. ein Enwickler-Team) erhält sein eigenes Repo. In diesem wird nur die eine Applikation (oder mehrere, je nachdem wie viel und wer für sie verantwortlich ist) deployed. Diese nutzen zwar die Plattform-Ressourcen, erstellen bzw. verwalten diese aber nicht.

Die Entwickler können dann also ihre Applikationen deployen (oder jemand anderes deployed die Infrastruktur für sie) - wichtig ist hier nur das separate Repo und kein Mono-Repo Ansatz.

So kann man ein viel zu großes Repo vermeiden und erspart sich einiges an Kopfschmerzen.
Für den aktuellen Use Case sollte allerdings der Mono-Repo Ansatz noch vertretbar sein.