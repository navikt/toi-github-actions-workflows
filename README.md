# Reusable workflows for Github Actions for team Toi sine JVM backend-applikasjoner

Hva er det? Se https://docs.github.com/en/actions/sharing-automations/reusing-workflows

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
git pull origin main
```

### 3-A: Non-breaking change
Flytt eksisterende tag til nyeste commit:
```bash
git tag -fa v14 -m "v14 minor change" 
git push origin refs/tags/v14 --force
```
Du er ferdig. Endringen vil bli tatt i bruk alle steder som allerede referer til denne versjonen, uten at du trenger å endre noe på bruksstedet.

### 3-B: Breaking change
1. Opprett ny tag med det nye versjonsnummeret:

```bash
git tag -a v15 -m "v15 breaking change"
git push origin refs/tags/v15
```

2. Oppdater workflows i appene til å bruke det nye versjonsnumeret.


# Henvendelser

## For Nav-ansatte
* Dette Git-repositoriet eies av [team Toi](https://teamkatalog.nav.no/team/76f378c5-eb35-42db-9f4d-0e8197be0131).
* Slack: [#arbeidsgiver-toi-dev](https://nav-it.slack.com/archives/C02HTU8DBSR)

## For folk utenfor Nav
* Teknologiavdelingen i [Arbeids- og velferdsdirektoratet](https://www.nav.no/no/NAV+og+samfunn/Kontakt+NAV/Relatert+informasjon/arbeids-og-velferdsdirektoratet-kontorinformasjon)
