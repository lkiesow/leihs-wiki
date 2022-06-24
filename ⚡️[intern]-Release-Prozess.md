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

Die Arbeitsschritte für RC und Release unterscheiden sich in wenigen Details.

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
./dev/git-release-notes $NEXT_RELEASE

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
SEMVER='1.0.0'
git commit -m "release: $SEMVER"
git tag --sign -f "$SEMVER" -m "$SEMVER" HEAD
git push origin --tags
BRANCHES="next release stable zhdk/deploy"; for RB in $BRANCHES; do git push origin HEAD:${RB}; done

# nun auf `master` zurück-mergen (eventuell nicht-linear)
RELEASE=$(git rev-parse HEAD)
git checkout master
git reset --hard origin/master
git merge -m "Merge branch 'release'" $RELEASE
# deal with CI, then push to master
git push origin HEAD:master
```

### 👸 Release ankündigen

[TODO]

## Probleme

1. Note: "Good To Merge" muss nicht Grün sein für den Release, falls der Release nicht vom Master HEAD gemacht wird, sondern von einem früheren Commit. Dies wird erst ein Thema, wenn man auf den Master zurück merged, siehe nächste Punkte.

1. commit message body nicht im changes skript enthalten


### Shell Walkthrough

> notes by @eins78

* dont forget to check for or create a milestone for this release, and add the release PR to it!
* some notes apple to [Madek](https://github.com/Madek) but most steps are the same so the process is only described here for convenience 

```bash
DEV_INITIALS=mfa
RELEASE_MAJOR_MINOR=99.9
RELASE_PATCH=9
RELEASE_PRE='-RC.9' # or '' for stable release
VERSION_PREFIX='' # for madek its 'v' 

RELEASE_MAIN="${RELEASE_MAJOR_MINOR}.${RELASE_PATCH}"
RELEASE="${RELEASE_MAIN}${RELEASE_PRE}"
RELEASE_NAME="${VERSION_PREFIX}${RELEASE}"
echo $RELEASE_MAIN
echo $RELEASE
echo $RELEASE_NAME

git fetch -q

# if new RC from master
    git checkout origin/master 
# or if target release is patch
    git checkout "origin/v/$RELEASE_MAJOR_MINOR-stable"
# or if RC>1
    git checkout "origin/v/$RELEASE_MAJOR_MINOR-staging"

git checkout -B ${DEV_INITIALS}/v/$RELEASE_MAJOR_MINOR-staging
git submodule update --recursive --force --init

git merge origin/master # if linear or feature release
git pick … # if patch release
# make sure to get all changes from remote (like release notes edited in github web):
git merge --squash "origin/v/${RELEASE_MAJOR_MINOR}-staging"
git merge --squash "origin/${DEV_INITIALS}/v/${RELEASE_MAJOR_MINOR}-staging"

git submodule update --recursive --force --init

code "config/releases/${RELEASE_MAIN}.md"
( ./dev/release-notes || ./dev/git-release-notes ) | pbcopy
# edit
git add "config/releases/${RELEASE_MAIN}.md"
git commit -m "release: ${VERSION_PREFIX}${RELEASE}"
RELEASE_REF="$(git log -n1 --format="%H" HEAD)"
git push -f origin "${RELEASE_REF}:refs/heads/${DEV_INITIALS}/v/$RELEASE_MAJOR_MINOR-staging"

# wait for CI…

# push to release staging branches
git push -f origin "${RELEASE_REF}:refs/heads/v/${RELEASE_MAJOR_MINOR}-staging"
git push -f origin "${RELEASE_REF}:zhdk/staging"

# make a PR
open "https://github.com/leihs/leihs/compare/stable...v/${RELEASE_MAJOR_MINOR}-staging?expand=1"
open "https://github.com/Madek/Madek/compare/stable...v/${RELEASE_MAJOR_MINOR}-staging?expand=1"

###
# MADEK: build assets – upload `madek.tar.gz` to github release and save as draft
cd deploy
bin/archive-build
cd ..
open deploy
open "https://github.com/Madek/Madek/releases/new?tag=${RELEASE_NAME}&prerelease=$(test -z $RELEASE_PRE || echo 1)&title=Madek%20${RELEASE_NAME}"
open "./deploy/tmp/release-builds/${RELEASE_NAME}/"
cat "config/releases/${RELEASE_MAIN}.md" | pbcopy 

# LEIHS: build assets – upload `build-artefacts.tar.gz` to github release, paste notes and save as draft
cd deploy
bin/build-release-archive
cd ..
open "https://github.com/leihs/leihs/releases/new?tag=${RELEASE_NAME}&prerelease=$(test -z $RELEASE_PRE || echo 1)&title=Leihs%20${RELEASE_NAME}"
open "./deploy/tmp/release-builds/${RELEASE_NAME}/"
cat "config/releases/${RELEASE_MAIN}.md" | pbcopy 
###

# tag (leihs: all releases, madek: only stable)
git tag --sign -f "${RELEASE_NAME}" -m "${RELEASE_NAME}" ${RELEASE_REF}
git push --tags --force
./dev/git-tag-submodules "${RELEASE_NAME}"
# now publish the github release (tag already exists)
open "https://github.com/leihs/leihs/releases/"
open "https://github.com/Madek/Madek/releases/"

# only for stable release
git push -f origin "${RELEASE_REF}:refs/heads/v/${RELEASE_MAJOR_MINOR}-stable"
open "https://github.com/Madek/madek/settings/branch_protection_rules/1950244"
git push -f origin "${RELEASE_REF}:refs/heads/stable"
git push -f origin "${RELEASE_REF}:zhdk/deploy"
# cleanup
git push origin ":refs/heads/v/${RELEASE_MAJOR_MINOR}-staging"
git push origin ":refs/heads/${DEV_INITIALS}/v/${RELEASE_MAJOR_MINOR}-staging"

# always merge back to master so we have the correct release history there
git checkout origin/master
git checkout -B ${DEV_INITIALS}/merge-$RELEASE_MAJOR_MINOR
git checkout ${RELEASE_REF} -- "config/releases/${RELEASE_MAIN}.md"
# new release file! increment patch, set pre to "dev" 
(( next_patch_version = $RELASE_PATCH + 1 ))
NEXT_VERSION="${RELEASE_MAJOR_MINOR}.${next_patch_version}"
cp "config/releases/${RELEASE_MAIN}.md" "config/releases/${NEXT_VERSION}.md"
code "config/releases/${NEXT_VERSION}.md"
# edit yaml (increase version, set pre to 'dev') and replace text with '…'. For RCs add a new section.
git add config/releases/
git commit -m 'chore: sync release info'
git push origin -f "HEAD:refs/heads/${DEV_INITIALS}/merge-$RELEASE_MAJOR_MINOR"
# wait for CI…
git push origin HEAD:master

# announcement
echo ":bellhop_bell: Leihs ${RELEASE_NAME} was released :rocket: https://github.com/leihs/leihs/releases/tag/${RELEASE_NAME}"
echo ":bellhop_bell: Madek ${RELEASE_NAME} was released :rocket: https://github.com/Madek/Madek/releases/tag/${RELEASE_NAME}"

# update demo inventories:
cd leihs-instance; ./scripts/release "$RELEASE_NAME"; cd -
cd demo.leihs.zhdk.ch; ./scripts/release "$RELEASE_NAME"; cd -

cd madek-instance; ./scripts/update_madek_latest stable; …

```