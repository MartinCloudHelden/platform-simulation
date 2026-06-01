# Uebungen – Mini-IDP mit GitHub bauen

In diesem Dokument findet ihr alle Anleitungen fuer die praktische Uebung. Waehlt
**einen** Use Case (A, B, C oder D) und arbeitet ihn durch.

---

## Vorab: Workflow-Permissions aktivieren

**Dieser Schritt ist Voraussetzung fuer ALLE Use Cases. Macht das als erstes,
sonst laeuft kein Workflow.**

1. Geht in eurem Repo auf **Settings** (oben rechts).
2. Linke Sidebar: **Actions** → **General**.
3. Scrollt nach unten zu "Workflow permissions".
4. Waehlt **"Read and write permissions"**.
5. Aktiviert zusaetzlich die Checkbox **"Allow GitHub Actions to create and approve pull requests"**.
6. Klickt **Save**.

Ohne diesen Schritt kann der Workflow keine Dateien committen und keine Issues kommentieren.

---

## Beispiel: Use Case D ist schon umgesetzt

Bevor ihr loslegt, lauft das mitgelieferte Beispiel einmal durch. Damit versteht
ihr das Muster, das ihr fuer eure eigene Implementierung uebernehmt.

**Was passiert:**
Ein Issue Form fragt nach Service-Name, Owner, Beschreibung und Umgebung. Nach
Submitten erzeugt ein Workflow eine fertige Markdown-Doku-Datei aus einer
Vorlage, committet sie ins Repo und kommentiert das Issue.

**Beteiligte Dateien:**

| Datei | Funktion |
|---|---|
| `.github/ISSUE_TEMPLATE/example-doc-request.yml` | Das Formular |
| `.github/workflows/example-doc-handler.yml` | Der Workflow |
| `templates/doc-template.md` | Die Markdown-Vorlage mit Platzhaltern |
| `generated-docs/` | Hier landet das Ergebnis |

**Testlauf:**
1. **Issues** → **New Issue** → "Doku-Vorlage anfordern" waehlen.
2. Felder ausfuellen (Service-Name z.B. `mein-test-service`).
3. **Submit new issue**.
4. **Actions**-Tab oeffnen → Workflow laeuft.
5. Wenn fertig: in `generated-docs/` neue Datei pruefen, Kommentar im Issue pruefen.

Wenn das nicht funktioniert: Workflow-Permissions wirklich aktiviert?
`docs/TROUBLESHOOTING.md` konsultieren.

---

## Use Case A: Neuen Service registrieren

### Was soll passieren

Ein Entwickler legt einen neuen Service an und meldet ihn im "Service Catalog"
des Unternehmens an. Statt eine Wiki-Seite manuell zu pflegen, fuellt er ein
Formular aus. Der Catalog ist eine Sammlung von YAML-Dateien im `catalog/`-Ordner.

### Eingabefelder

| Feld | Typ | Pflicht | Validierung |
|---|---|---|---|
| `service_name` | input | ja | nur lowercase, Bindestriche, 3–40 Zeichen |
| `owner` | input | ja | GitHub-Username (`@xy` oder `xy`) |
| `repo_url` | input | ja | beginnt mit `https://github.com/` |
| `environment` | dropdown | ja | dev / staging / prod |
| `description` | textarea | nein | freier Text |

### Was der Workflow tun soll

1. Issue-Body parsen.
2. Eine neue Datei `catalog/<service_name>.yml` anlegen mit folgender Struktur:
```yaml
service: <name>
owner: <owner>
repo: <repo_url>
environment: <env>
description: <desc>
created_at: <ISO-Datum>
created_by: <issue-author>
```
3. Datei committen (direkt auf `main`).
4. Issue kommentieren: "Service registriert: `catalog/<name>.yml`".
5. Issue mit Label `service-registered` markieren und schliessen.

### Schritt fuer Schritt

1. **Issue Form anlegen:**
   - In GitHub auf **Add file** → **Create new file**.
   - Pfad: `.github/ISSUE_TEMPLATE/register-service.yml`.
   - Inhalt: kopiert aus `example-doc-request.yml`, passt Name, Beschreibung,
     Felder und Labels an.
   - Wichtig: Label hier `service-register` (oder aehnlich) – der Workflow filtert darauf.
2. **Workflow anlegen:**
   - Pfad: `.github/workflows/register-service.yml`.
   - Inhalt: kopiert aus `example-doc-handler.yml`.
   - Anpassen:
     - `name:` aendern.
     - Filter auf neues Label umstellen (`if: contains(... 'service-register')`).
     - Schritt "Datei erzeugen" auf `catalog/`-Ordner umlenken.
     - Template entfaellt – ihr schreibt das YAML direkt im Workflow per `heredoc`
       (`cat > catalog/$NAME.yml << EOF ... EOF`).
3. Commit beider Dateien auf `main`.
4. Test: neues Issue mit dem neuen Formular erstellen.
5. Pruefen: Datei in `catalog/` da? Kommentar im Issue? Label gesetzt?

### Stretch-Goals

- Validierung im Workflow: lehnt Service ab, wenn `catalog/<name>.yml` schon existiert.
- Liste aller Services in einer `catalog/README.md` automatisch aktualisieren.

---

## Use Case B: Repo-Einladung

### Was soll passieren

Ein Plattform-Nutzer moechte einer Person Zugriff auf ein bestimmtes Repo geben
(z.B. neues Teammitglied, externer Berater). Statt direkt im Repo zu klicken,
stellt er die Anfrage ueber ein Formular. Das hat zwei Effekte:
- Es ist nachvollziehbar (Audit-Trail im Issue).
- Ein Plattform-Admin kann die Anfrage pruefen, bevor sie ausgefuehrt wird.

### Wichtige Klaerung: Mock vs. Echt

Ein echter API-Call zum Hinzufuegen eines Collaborators braucht einen Personal
Access Token mit `admin:org` oder `repo`-Rechten. Das geht in der Schulung nicht
sauber aus dem Workflow heraus. Wir machen daher die **Mock-Variante**: Der
Workflow erzeugt einen Issue-Kommentar mit einer fertigen Anweisung fuer den
Plattform-Admin (vier Augen ist eh besser).

Wer mit eigenem Repo + PAT arbeiten will: siehe Stretch-Goals am Ende.

### Eingabefelder

| Feld | Typ | Pflicht | Validierung |
|---|---|---|---|
| `github_username` | input | ja | GitHub-Username (kein `@`, nur Buchstaben/Ziffern/Bindestriche) |
| `target_repo` | input | ja | Format `owner/repo` |
| `permission` | dropdown | ja | read / triage / write / maintain |
| `reason` | textarea | ja | warum braucht diese Person Zugriff |
| `expires` | dropdown | ja | 7 Tage / 30 Tage / unbegrenzt |

### Was der Workflow tun soll

1. Issue-Body parsen.
2. Issue kommentieren mit einem "Approval-Block":
```
## Zugriffsanfrage zur Freigabe

- Benutzer: @<username>
- Repo: <target_repo>
- Berechtigung: <permission>
- Befristung: <expires>
- Begruendung: <reason>

**An den Plattform-Admin:** Bitte mit Label `approved` markieren,
um die Anfrage als genehmigt zu kennzeichnen.
```
3. Issue mit Label `access-request-pending` markieren.
4. Issue NICHT schliessen (wartet auf Approval-Label).

### Schritt fuer Schritt

1. Issue Form anlegen unter `.github/ISSUE_TEMPLATE/request-access.yml`.
2. Workflow anlegen unter `.github/workflows/request-access.yml`.
3. Workflow-Aufbau wie beim Beispiel, aber: kein File-Commit, nur Kommentar +
   Label. Nutzt `gh issue comment` und `gh issue edit --add-label`.
4. Test: Issue erstellen, Kommentar pruefen, Label pruefen.

### Stretch-Goals

- **Approval-Workflow**: zweiter Workflow, der auf das Setzen des Labels
  `approved` triggert (`on: issues: types: [labeled]`), kommentiert "Anfrage
  genehmigt" und schliesst das Issue.
- **Echter API-Call** (nur mit eigenem Repo + Personal Access Token):
  Token als Repo-Secret `PLATFORM_TOKEN` hinterlegen, Workflow ruft
  `gh api -X PUT /repos/<owner>/<repo>/collaborators/<user>` auf.
  Warnung: Token-Handling ist sicherheitskritisch; tut das nur in Test-Repos.

### Stolpersteine

- Wenn das `expires`-Feld eine Befristung verlangt, aber der Workflow nichts
  damit tut: in echt muesste ein Scheduler den Zugriff entziehen. Das ist in
  unserer Mock-Variante nicht umgesetzt – als Diskussionspunkt fuer Phase 4
  parkieren.

---

## Use Case C: S3-Bucket per Terraform-Template rendern

### Was soll passieren

Ein Entwickler braucht einen S3-Bucket. Statt selbst Terraform-Code zu schreiben,
fuellt er ein Formular aus. Der Workflow nimmt das vorhandene TF-Template aus
`templates/terraform-s3-snippet.tf`, ersetzt die Platzhalter mit den
Formular-Inputs und erzeugt eine fertige `.tf`-Datei als Pull Request. Ein
Plattform-Admin reviewed den PR.

**Wichtig:** Es findet **kein** `terraform apply` statt. Wir generieren nur Code.
Das ist auch in echten IDPs ein ueblicher Zwischenschritt – Code-Review vor Deployment.

### Eingabefelder

| Feld | Typ | Pflicht | Validierung |
|---|---|---|---|
| `bucket_name` | input | ja | lowercase, 3–63 Zeichen, Bindestriche erlaubt, keine Punkte (S3-Regeln) |
| `region` | dropdown | ja | eu-central-1 / eu-west-1 / us-east-1 |
| `environment` | dropdown | ja | dev / staging / prod |
| `project_tag` | input | ja | Projekt-Kennzeichnung fuer Tagging |
| `versioning` | dropdown | ja | enabled / disabled |

### Was der Workflow tun soll

1. Issue-Body parsen.
2. Inhalt von `templates/terraform-s3-snippet.tf` laden.
3. Platzhalter ersetzen:
   - `__BUCKET_NAME__` → `bucket_name`
   - `__REGION__` → `region`
   - `__ENVIRONMENT__` → `environment`
   - `__TAG_PROJECT__` → `project_tag`
4. Ergebnis schreiben nach `generated-tf/<bucket_name>.tf`.
5. Branch erstellen: `feature/s3-<bucket_name>`.
6. Datei dort committen.
7. Pull Request oeffnen gegen `main` mit aussagekraeftigem Titel und Beschreibung.
8. Issue kommentieren: "PR erzeugt: #<pr-number>".
9. Issue mit Label `tf-pending-review` markieren (offen lassen).

### Schritt fuer Schritt

1. **Issue Form anlegen** unter `.github/ISSUE_TEMPLATE/request-s3-bucket.yml`.
   - Wichtig: Issue Forms koennen NICHT per Regex validieren (nur `required`).
     Den Bucket-Namen daher im Workflow pruefen (Bash `=~` mit
     `^[a-z0-9][a-z0-9-]{1,61}[a-z0-9]$`) und bei Verstoss abbrechen.
2. **Workflow anlegen** unter `.github/workflows/request-s3-bucket.yml`.
   - Aufbau wie Beispiel, aber:
     - Schritt "Template rendern": `sed`-Aufruf, der die vier Platzhalter
       ersetzt. Beispiel:
```bash
sed -e "s|__BUCKET_NAME__|$BUCKET|g" \
    -e "s|__REGION__|$REGION|g" \
    -e "s|__ENVIRONMENT__|$ENV|g" \
    -e "s|__TAG_PROJECT__|$PROJECT|g" \
    templates/terraform-s3-snippet.tf > generated-tf/$BUCKET.tf
```
     - Schritt "PR erstellen": `peter-evans/create-pull-request@v6` oder `gh pr create`.
3. **Ordner `generated-tf/` anlegen** (per `.gitkeep` oder beim ersten Commit
   automatisch).
4. Test mit einem realistischen Bucket-Namen (z.B. `cloudhelden-demo-bucket`).
5. PR pruefen: ist die `.tf`-Datei korrekt gerendert? Stehen alle Werte drin?

### Stolpersteine

- Bucket-Namen-Validierung im Issue Form ist tricky. Wenn der User einen
  ungueltigen Namen eingibt, laeuft der Workflow trotzdem. Daher zweite
  Validierung im Workflow selbst einbauen (Workflow bricht ab und kommentiert
  Fehler).
- `sed` mit Sonderzeichen im Wert kann brechen. Wenn ein User in `project_tag`
  ein `/` oder `&` eingibt: `sed`-Trenner anpassen oder Werte vorher escapen.
- `peter-evans/create-pull-request@v6` braucht einen Branch-Namen, der noch
  nicht existiert. Bei zweitem Lauf mit gleichem Bucket-Namen: Fehler. Loesung:
  Timestamp anhaengen oder im Workflow pruefen.

### Stretch-Goals

- Workflow validiert, dass `bucket_name` nicht schon in `generated-tf/` existiert.
- Workflow laeuft `terraform fmt` und `terraform validate` (ohne Provider-Init)
  ueber die erzeugte Datei und kommentiert das Ergebnis am PR.
- PR-Beschreibung enthaelt automatisch einen Hinweis "Bitte vor Apply pruefen:
  Tags, Naming, Public-Access-Block".

---

## Use Case D: Doku-Vorlage anfordern (Erweiterung)

Use Case D ist das Beispiel im Skelett. Fuer die eigene Uebung **erweitert** ihr
das Beispiel um drei Aspekte – das macht aus dem Beispiel einen produktionsnahen
Workflow.

### Erweiterungen

1. **Validierung des Service-Namens.** Wenn der Name nicht dem Schema entspricht
   (lowercase, 3–40 Zeichen, Bindestriche): Workflow erzeugt KEINE Datei,
   sondern kommentiert das Issue mit einem Fehlerhinweis und schliesst das Issue
   mit Label `rejected`.
2. **Duplikat-Check.** Wenn `generated-docs/<service_name>.md` schon existiert:
   Workflow lehnt ab und verlangt einen anderen Namen.
3. **Owner-Notification.** Workflow kommentiert das Issue zusaetzlich mit
   `@<owner-username>` (aus dem Owner-Feld), damit der genannte Owner per
   GitHub-Notification informiert wird.

### Schritt fuer Schritt

1. `example-doc-handler.yml` als Basis nehmen (NICHT die Datei direkt
   editieren – kopieren als `example-doc-handler-v2.yml` und das Original
   deaktivieren oder loeschen).
2. Vor dem Datei-Erzeugen-Schritt einen "Validate"-Schritt einbauen:
```yaml
- name: Validate inputs
  id: validate
  run: |
    NAME="${{ steps.parse.outputs.service_name }}"
    if [[ ! "$NAME" =~ ^[a-z0-9-]{3,40}$ ]]; then
      echo "INVALID=true" >> $GITHUB_OUTPUT
      echo "REASON=Service-Name entspricht nicht dem Schema" >> $GITHUB_OUTPUT
    elif [[ -f "generated-docs/$NAME.md" ]]; then
      echo "INVALID=true" >> $GITHUB_OUTPUT
      echo "REASON=Service '$NAME' existiert bereits" >> $GITHUB_OUTPUT
    else
      echo "INVALID=false" >> $GITHUB_OUTPUT
    fi
```
3. Folgende Schritte mit `if: steps.validate.outputs.INVALID == 'false'` konditionieren.
4. Einen Reject-Pfad einbauen mit `if: steps.validate.outputs.INVALID == 'true'`:
   - Kommentar mit Fehlertext.
   - Label `rejected`.
   - Issue schliessen.

### Stolpersteine

- Konditionale Steps: `if:` auf Job-Ebene vs. Step-Ebene unterscheiden.
- `outputs` zwischen Steps brauchen die Syntax `echo "KEY=value" >> $GITHUB_OUTPUT`.
- `@<username>` im Kommentar funktioniert nur, wenn der Username gueltig ist –
  GitHub macht daraus dann eine Notification.

### Stretch-Goals

- Reject-Pfad wartet 5 Minuten und loescht dann das Issue komplett (per `gh issue delete`).
- Owner-Feld bekommt eine GitHub-User-Search-Validierung (per API).

---

## Akzeptanzkriterien (fuer alle Use Cases)

Eure Implementierung gilt als erfolgreich, wenn alle Punkte erfuellt sind:

- [ ] Es gibt ein neues Issue Form, das im **New Issue**-Chooser auftaucht.
- [ ] Das Formular hat mindestens 3 sinnvolle Pflichtfelder.
- [ ] Der Workflow laeuft automatisch nach Issue-Erstellung (Actions-Tab zeigt gruen).
- [ ] Der Workflow erzeugt ein **sichtbares Ergebnis** im Repo (Datei oder PR).
- [ ] Das Issue wird mit einem aussagekraeftigen Kommentar versehen.
- [ ] Das Issue erhaelt ein Label, das den Status erkennen laesst.
- [ ] (Use Cases A/D) Das Issue wird automatisch geschlossen.
- [ ] (Use Cases B/C) Das Issue bleibt offen und wartet auf einen Folgeschritt.

---

## Reflexion (fuer Phase 4)

Bereitet zur Demo kurze Antworten auf folgende Fragen vor – das ist der
gedankliche Mehrwert, nicht der Code:

1. **Was war an eurer Umsetzung Self-Service?**
   (Hinweis: der Nutzer hat die Aktion ohne Ticket an Ops bekommen.)

2. **Was war Governance?**
   (Hinweis: PR-Review bei Use Case C, Approval-Label bei B, Validierung bei D,
   alles ist im Issue + Git-Historie dokumentiert.)

3. **Was unterscheidet eure Loesung von einem "echten" IDP?**
   (Erwartete Punkte: kein zentrales Portal, kein Service Catalog mit UI,
   kein Monitoring, keine Berechtigungs-Granularitaet, kein Multi-Repo-Scope.)

4. **Was muesste man ergaenzen, damit das in einer 50-Personen-Firma traegt?**
   (Erwartete Punkte: zentrales Plattform-Repo, Templates als Code, echte
   Backends fuer die Aktionen, Audit-Logging, Self-Service-Portal-UI obenauf.)

---

## Wo ihr Hilfe findet

- `docs/CHEATSHEET-issue-forms.md` – Syntax-Spickzettel fuer Issue Forms
- `docs/CHEATSHEET-actions.md` – Syntax-Spickzettel fuer Workflows
- `docs/TROUBLESHOOTING.md` – die haeufigsten Fehler mit Loesung
- Offizielle GitHub-Doku zu Issue Forms:
  <https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues/syntax-for-issue-forms>
- Beispiel-Workflow im Skelett: `.github/workflows/example-doc-handler.yml` –
  die Kommentare dort sind euer wichtigster Referenzpunkt.
