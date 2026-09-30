# Reusable workflows for Github Actions for team Toi sine JVM backend-applikasjoner

Hva er det? Se https://docs.github.com/en/actions/sharing-automations/reusing-workflows

## Automatisk oppdatering av Docker run-time base-image

### Hvorfor
Det oppdages stadig nye sikkerhetssissues i Docker "base image"-ene vi bruker. Image-ene patches fortløpende av leverandøren. De patchede versjonene publiseres med samme tag. Hver gang vi bygger henter vi ned nyeste versjon av image-et med den tag-en vi referer til. Det betyr at hver gang vi bygger og deployer til prod så får vi sannsnynligvis lukket noen sikkerhetsissues. Det vil være bra for sikkerheten å gjøre dette ofte, f.eks. daglig,  men vi ønsker å slippe å gjøre det manuelt. Derfor bruker vi en scheduled workflow, som regelmessig sjekker om det foreligger en ny versjon, og i så fall bygger og deployer til prod.

### Hvordan
#### Terminologi
Har forsøkt å bruke Docker sin terminologi i navngivingen av parametre:
* `baseimage-tagged-ref` er hele `registry/path:tag`
* `baseimage-digest` er `sha256:...`
* `BASE_IMAGE_DIGEST_PINNED_REF` er `registry/path:tag@sha256:...`

#### Angivelse av base-image flyttes ut av Dockerfile
Hvilket Docker run-time base-image som skal brukes oppgis i appens workflow, i `baseimage-tagged-ref`:
```yaml
jobs:
  build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@v16
    with:
      java-version: '25'
      baseimage-tagged-ref: europe-north1-docker.pkg.dev/cgr-nav/pull-through/nav.no/jre:openjdk-25
    permissions:
      contents: read
      id-token: write
```
Appens Dockerfile må bruke build-argumentet i siste (typisk eneste) `FROM`:
```dockerfile
ARG BASE_IMAGE_DIGEST_PINNED_REF
FROM ${BASE_IMAGE_DIGEST_PINNED_REF}
```

#### Opprett en scheduled workflow
I appen, legg til en scheduled workflow, for eksempel `.github/workflows/oppdater-docker-baseimage.yaml`:
 ```yaml
 name: Oppdater Docker run-time base-image
 on:
   schedule:
     - cron: '40 5 * * 1'
   workflow_dispatch:

 jobs:
   call-oppdater-docker-baseimage:
     uses: navikt/toi-github-actions-workflows/.github/workflows/oppdater-docker-baseimage.yaml@v16
     with:
       deploy-workflow-filnavn: deploy.yml
     permissions:
       id-token: write
       actions: write
 ```
I eksemplet ovenfor er `deploy.yaml` filnavnet på appens deploy-workflow, altså den som kaller `toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml`. Appens deploy-workflow må ha `workflow_dispatch` under `on:` for at din nye scheduled workflow skal kunne starte deploy-workflowen.

#### Flere apper i ett repo

Når flere apper deler repo trenger vi app-unike navn på artefaktene som inneholder digest. Løses ved at hver app sender 
navnet sitt som parameter til både `lagre-docker-baseimage-digest.yaml` og `oppdater-docker-baseimage.yaml`. 
Artefaktet heter da `baseimage-digest-<app-navn>`. 



# Versjonering
Tidligere brukte ikke appene våre versjoner da de refererte til egenskrevne workflows. Vi bare referete til nyeste commit på main-branchen, ved å skrive `@main` i
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@main
```
Ulempen med dette er at en workflow som har fungert hittil i en app slutter å fungere uten at du har endret noe i appen sitt repo. Det kan skje når en endring med en feil eller en breaking change blir pushet til dette repoet (repoet som inneholder de felles worflows som appene sine worflows refererer til).

Denne risikoen unngår vi ved at konsumenten låser seg til et versjonsnummer ved å skrive `@v1`, `@v2`, `@v3`, osv. slik:
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@v1
```

Tanken er at versjonsnummer hos konsumenten (bruksstedet i hver enkelt app) bare trengs å økes ved breaking change. Det gir mindre vedlikeholdsarbeid for app-utviklerne.

I en periode lagde vi Github releases, men nå bruker vi bare Git tags (etter juni 2026). Det er det eneste vi trenger, og det blir mindre å forholde seg til.

## Release-prosedyre

### Versjonsformat

* Git tag-navnet startet med liten "v" etterfulgt av et heltall.
* Bruk kun major versjonsnummer i tagnavnet: `v14`, `v15`, osv., ikke `v14.1` eller `v14.1.2`.
* Øk versjonsnummeret bare ved breaking change. F.eks. ny obligatorisk input, fjernet output, omdøpte workflow.


### 0: Hvordan teste en endring i en workflow før den blir released?
Opprett en feature branch og bruk navnet på feature-branchen din i konsumenten der du normalt skriver versjonsnummer, slik:
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@min-feature-branch
```
* Husk at felles-workflows i dette repoet refererer til hverandre med versjonsnummer, så det kan hende du må legg inn branch-navnet ditt flere steder.
* Når du er klar til å merge til main, Husk å bytte ut feature-branch navnet med riktig versjonsnummer.

### 1: Commit endringene til main

**Spesielt for breaking changes:** Felles-workflowene i dette repoet referer til hverandre med versjonsnummer, så oppgradering til ny versjon må også gjøres i dem, ikke bare i appene. Endre filene til å bruke nytt versjonsummer - f.eks. `v15` istedenfor `v14` eller feature-branch navnet - selv om det ennå ikke finnes en Git tag med det nye versjonsnummeret. Tag-en skal du lage i eget trinn nedenfor. Det er ok at HEAD på main ikke er kjørbar inntil tag-en kommer på plass, fordi det påvirker ikke konsumenter som er tag-låst, og det bør ikke finnes noen konsumenter som refererer til `@main`.

### 2: Sørg for at du er på main og har siste versjon lokalt
```bash
git checkout main
git pull
```

### 3-A: Non-breaking change
Flytt eksisterende tag til nyeste commit:
```bash
git tag -fa v16 -m "v16 non-breaking change" 
git push origin refs/tags/v16 --force
```
Du er ferdig. Endringen vil bli tatt i bruk alle steder som allerede referer til denne versjonen, uten at du trenger å endre noe på bruksstedet.

### 3-B: Breaking change
1. Opprett ny tag med det nye versjonsnummeret:

```bash
git tag -a v17 -m "v17 breaking change"
git push origin refs/tags/v17
```

2. Oppdater workflows i appene til å bruke det nye versjonsnumeret.


# Henvendelser

## For Nav-ansatte
* Dette Git-repositoriet eies av [team Toi](https://teamkatalog.nav.no/team/76f378c5-eb35-42db-9f4d-0e8197be0131).
* Slack: [#arbeidsgiver-toi-dev](https://nav-it.slack.com/archives/C02HTU8DBSR)

## For folk utenfor Nav
* Teknologiavdelingen i [Arbeids- og velferdsdirektoratet](https://www.nav.no/no/NAV+og+samfunn/Kontakt+NAV/Relatert+informasjon/arbeids-og-velferdsdirektoratet-kontorinformasjon)
