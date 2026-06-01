# Mini-IDP Skelett – Schulungsrepo

Dieses Repository ist ein **Template-Repo** fuer eine Anfaenger-Schulung zum Thema "Internal Developer Platform (IDP) mit GitHub-Bordmitteln". Es demonstriert, wie Self-Service-Workflows mit GitHub Issue Forms und GitHub Actions umgesetzt werden koennen – ganz ohne externe Tools oder Infrastruktur.

Teilnehmer nutzen dieses Skelett als Ausgangspunkt und implementieren darauf basierend eigene Self-Service-Use-Cases.

**Dauer:** ~90 Min | **Format:** Einzel- oder Paerchenarbeit, jeder in eigenem Repo | **Vorkenntnisse:** GitHub-Account, Browser. Kein lokales Git noetig.

---

## Lernziele

Nach der Uebung koennt ihr:

1. Drei Komponenten eines IDP konkret benennen (Portal, Self-Service, Automatisierung).
2. Den Unterschied zwischen *Plattform* und *Portal* erklaeren.
3. Einen einfachen Self-Service-Workflow mit GitHub-Bordmitteln umsetzen.
4. Den Zusammenhang zu IHK-Themen (Automatisierung, Governance, Dokumentation) herstellen.

---

## Was ist hier ein IDP?

Eine **Internal Developer Platform** (IDP) ist eine Sammlung von Werkzeugen und Workflows, die Entwicklungsteams Self-Service-Zugang zu Infrastruktur und Prozessen gibt – ohne dass jedes Mal ein Plattform-Team manuell eingreifen muss. In dieser Schulung setzen wir eine vereinfachte Variante um: GitHub Issues dienen als **Portal** (Eingabeformular fuer Anfragen), GitHub Actions als **Plattform-Logik** (automatisierte Verarbeitung). Das ist kein vollwertiges Backstage oder Humanitec, aber es vermittelt die Kernidee: Standardisierte Schnittstellen ersetzen Tickets und manuelle Arbeit.

**Plattform vs. Portal:** Die Plattform ist der Motor (Workflows, APIs, Infrastruktur-Code). Das Portal ist das Dashboard (UI, Formulare, Katalog). In unserer Uebung ist das Issue-Formular das Portal und der GitHub Actions Workflow die Plattform.

---

## Phase 1 – Recherche (20 Min)

Lest **eine** der beiden Quellen (oder kombiniert):

- Google Cloud: <https://cloud.google.com/discover/what-is-an-internal-developer-platform?hl=de>
- Red Hat: <https://www.redhat.com/de/topics/platform-engineering/what-is-an-internal-developer-platform>

**Sucht beim Lesen gezielt nach:**

- **Was sind "Golden Paths"?**
- **Welche Self-Service-Aktionen tauchen als Beispiele auf?**

**Ergaenzend (optional, 5 Min):** Eine Live-Demo oeffnen und 3 Self-Service-Aktionen identifizieren:
- <https://demo.port.io/self-serve>
- <https://demo.backstage.io>

**Output:** Auf Papier oder im Editor stichpunktartig: 3 Use Cases, die ihr in echt gesehen habt. Mit Notiz: Welche Eingaben braucht der User? Was passiert dahinter?

---

## Phase 2 – Use Case auswaehlen (5 Min)

Waehlt **einen** Use Case zur Umsetzung. Vier Optionen:

| Option | Beschreibung | Komplexitaet |
|--------|-------------|--------------|
| A | Neuen Service registrieren | niedrig |
| B | Repo-Einladung | mittel |
| C | S3-Bucket per Terraform-Template rendern | mittel-hoch |
| D | Doku-Vorlage anfordern (Erweiterung des Beispiels) | niedrig |

Details zu jedem Use Case findet ihr in [EXERCISES.md](EXERCISES.md).

---

## Phase 3 – Aufsetzen & Implementieren (50 Min)

### Schritt 1: Skelett-Repo nutzen (5 Min)

1. **"Use this template"** oben rechts klicken und ein eigenes Repository erstellen.
2. Namen vergeben (z.B. `mini-idp-<euerName>`).
3. Public oder Private – egal fuer die Uebung.
4. Eigenes Repo oeffnen.

### Schritt 2: Workflow-Permissions aktivieren (2 Min)

1. **Settings** → **Actions** → **General**.
2. Scrollt zu "Workflow permissions".
3. **"Read and write permissions"** auswaehlen.
4. Checkbox **"Allow GitHub Actions to create and approve pull requests"** aktivieren.
5. **Save** klicken.

Ohne diesen Schritt kann kein Workflow Dateien committen oder Issues kommentieren.

### Schritt 3: Beispiel testen (5 Min)

Bevor ihr eigenes baut: erlebt das Beispiel.

1. **Issues** → **New Issue** → "Doku-Vorlage anfordern" waehlen.
2. Felder ausfuellen (Service-Name z.B. `mein-test-service`).
3. **Submit new issue**.
4. **Actions**-Tab oeffnen → Workflow laeuft.
5. Wenn fertig: in `generated-docs/` neue Datei pruefen, Kommentar im Issue pruefen.

Wenn das nicht funktioniert: `docs/TROUBLESHOOTING.md` konsultieren.

### Schritt 4: Eigenen Use Case umsetzen (30 Min)

Folgt [EXERCISES.md](EXERCISES.md) fuer euren gewaehlten Use Case (A/B/C/D).

Kurzanleitung:
1. Issue Form anlegen: kopiert `example-doc-request.yml`, benennt um, passt Felder an.
2. Workflow anlegen: kopiert `example-doc-handler.yml`, benennt um, passt Logik an.
3. Testen wie in Schritt 3.

### Schritt 5: Akzeptanzkriterien pruefen (5 Min)

Euer Mini-IDP funktioniert, wenn:
- Ein Issue kann ueber das Formular erstellt werden.
- Der Workflow laeuft automatisch nach Issue-Erstellung.
- Es entsteht ein sichtbares Ergebnis (Datei, PR, oder Kommentar) im Repo.
- Das Issue wird automatisch geschlossen oder mit einem Label markiert.

---

## Stolpersteine, die typisch auftreten

| Problem | Loesung |
|---|---|
| Workflow laeuft nicht, kein Trigger | `on:` im Workflow falsch. Muss `on: issues: types: [opened]` sein. |
| Workflow laeuft, kann aber nicht schreiben | Settings → Actions → General → Read and write permissions aktivieren. |
| YAML-Fehler im Issue Form | Indentation! YAML ist whitespace-sensitive. Cheatsheet konsultieren. |
| Formular taucht nicht im Issue-Chooser auf | Datei muss in `.github/ISSUE_TEMPLATE/` liegen, Endung `.yml`, valide YAML. |
| Workflow laeuft mehrmals | Bei mehreren Workflows mit gleichem Trigger: Filter via `if:` oder Labels. |
