# Ruby Release Prozess

Beschreibung des Wegs von Commits zu einem stabilen Release.

Umfasst Erstellung eines RC, der je nach Projekt in-house oder auch von externen
getestet werden kann (`staging`-Server)
sowie Erstellung von Hotfixes.

Nicht vorgesehen sind "Release-Channels" o.ä., wie z.B. Backport von Fixes für ältere Versionen.
Wir supporten nur die jeweils neueste stabile Version.

Einige Zwischenschritte wie z.B. "Fix Showstoppers" sind natürlich nur relevant, wennn es auch welche gibt, dies wird hier nicht weiter erwähnt.

Bei "kleineren" Releases (wenige Commits/Entwickler in der Iteration),
können die Pull-Request auch weggelassen werden (sind nur für dezentrale Kollaboration und Nachvollziehbarkeit).

Die Arbeitsschritte für RC und Release unnterscheiden sich in wenigen Details.

```diff
+ Frage: Auch für releases immer PR machen?
```

## Rollen

- 👨‍💻 Developer
- 👩‍🍳 Release-Koch (jeweils ein Developer, rotierend)
- 🕵️‍ QA/Tester
- 👸 Produktowner
- 🌀 CI (automatisierte Schritte)

## Branches

Liste der Branches (auf `origin` aka GitHub), die spezielle Bedeutung haben.
Alle anderen sind Arbeits-Branches für Entwickler (im Idealfall mit Initialen-Präfix).

- **`master`**
    - aktuellster Entwicklungsstand (möglicherweise instabil/ungetestet)
    - 🌀 Auto-Deploy to `test`-Servers
    - enthält *auch* das aktuellste release (da dieses von `stable` "zurückgemerged" wird)

- **`next`**
    - Arbeits-Branch, temporär während ein Release/-Candidate erstellt wird
    - benötigt für CI (es kann nicht direkt auf protected Branches gepusht werden, da vorher automatische Tests laufen)
    - auch gut für Nachvollziehbarkeit und als Referenz für Skripte


- **`release`**
    - neuester Release/-Candidate (möglicherweise noch Work-In-Progress)
    - 🌀 Auto-Deploy to `staging`-Servers

- **`stable`**
    - neuester **stabiler** Release/-Candidate
    - für alle externen Instanzen **die** offizielle Referenz bzgl. Updates, Support, etc.

- `zhdk/deploy`
    - ***nur relevant für Ops (nicht Dev)***
    - im Normalfall gleicher Stand wie `stable` (aber wir haben Flexibilität)


## Prozess

Grob:

1. DEMO: `master` abnehmen
1. RC: `master` beschreiben & semver & push `release`
1. QA: `release` abnehmen
1. RELEASE: semver & tag & push & merge back to master

### 👨‍💻 👩‍🍳 🕵️‍ 👸 Demo

Demo zeigt den aktuellen Stand (normalerweise origin/master)
Falls etwas grundsätzlich kaputt ist → Prozess verschieben, zurück auf Null.

Showstopper/Blocker/Nacharbeiten im Pull-Request vermerken.

Erste Release-Notes schreiben (kann später noch angepasst werden),
z.B. auf Grundlage der Commit-Messages.

Wennn nötig den Text im PR schreiben/editieren, damit kollaboriert werden kann.

```diff
+ Frage: Am besten push auf `next` *vor* der Demo, damit alles klar ist?
```

```sh
# Referenz auf commit der demonstriert wurde, z.B. `6b5ea1ce279be17f4cf3cfdd35921665fe702228`
NEXT_RELEASE=$(git rev-parse origin/master)

# push auf `next`, um diese Referenz zu "speichern"
git push origin +${NEXT_RELEASE}:refs/heads/next

# Änderungsliste aus dem git holen:
./dev/release-notes

# PR machen (von next auf release!)
open "https://github.com/Madek/madek/compare/release...next?expand=1"
```

### 👨‍💻 Fix Showstoppers

Alle Fixes für das Release-In-Progress werden auf `next` delivered (statt `master`).

Beachten: Es *könnte* auch parallel am über-nächsten release gearbeitet werden
(also delivery/merge auf `master`) – 
das ist eine personelle/organisatorische Entscheidung und stört den Prozess nicht.

### 👩‍🍳 Release-Candidate

Neues Release in der Liste anlegen, als RC markieren.
Commit & Push auf `release`.
Neuen RC für QA ankündigen ("`v1.0.0-RC.1` is now on `staging`")

- Liste: `./config/releases.yml`
- RC markieren: `version_pre: 'RC.1'`

```sh
git checkout -B next && git reset --hard origin/next

# release anlegen
open config/releases.yml
git add ./config/releases.yml

# make release commit and push to next
git commit -m 'release: v1.0.0-RC.1'
git push origin --tags
git push origin HEAD:next

# PR wird nun gemerged:
git push origin HEAD:release
```

### 🕵️‍ QA: Freigabe RC

RC wird auf `staging`-Servern der Qualitätskontrolle unterzogen.
Wenn es externe Partner gibt, können diese ebenfalls testen und feedback geben.
Freigabe des RC erfolgt jeweils durch Produktowner oder deren Delegation.
Anderfalls werden jeweils Tickets für Bugs erfasst.

### 👨‍💻 Fix RC Showstoppers

Weiterhin: Alle Fixes für das Release-In-Progress werden auf `next` delivered.

### 👩‍🍳 Release

Neues Release in der Liste editieren, "Candidate"-marker weg.
Commit & **Tag** & Push auf `stable`.
Neuen stabilen Release kommunizieren, evtl. Demos/Inventories anpassen.

*Ticket-System anpassen: jeweils Iteration abschliessen/Milestones benennen/etc.*

- Liste: `./config/releases.yml`
- RC marker weg: `version_pre: null`

```sh
git checkout -B next && git reset --hard origin/next

# release editieren
open config/releases.yml
git add ./config/releases.yml

# make release commit and push
git commit -m 'release: v1.0.0'
git tag --sign -f 'v1.0.0' -m 'v1.0.0' HEAD
git push origin --tags
git push origin HEAD:next
git push origin HEAD:release
git push origin HEAD:zhdk/deploy

# nun auf `master` zurück-mergen (eventuell nicht-linear)
RELEASE=$(git rev-parse HEAD)
git checkout master
git reset --hard origin/master
git merge -m "Merge branch 'release'" origin/release
# deal with CI, then push to master
git push origin HEAD:master
```

### 👸 Release ankündigen

[TODO]

## Probleme

1. merge `release` auf `master` *kann* nicht-linear sein. 
  - Meta-Check sollte diesen Spezialfall erkennen und erlauben
  - aktuell nötig: [config anpassen (`GIT_LINEAR_HISTORY_CHECK_START_SHA`)](https://github.com/Madek/madek/commit/e064b603daeaf4079e6d4b4cdfe12a03044d838a)

1. commit message body nicht im changes skript enthalten