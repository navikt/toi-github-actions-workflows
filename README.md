# Reusable workflows for Github Actions

Hva er det? Se https://docs.github.com/en/actions/sharing-automations/reusing-workflows

# Versjonering
Prøver i januer 2025 å innføre versjonering på de reusable Github Workflows som ligger her. Det betyr at i appene våre sine workflows, hvor vi hittil har skrevet `@main` i
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@main
```
bør vi nå begynne å skive `@v1`, `@v2`, `@v3`, osv. slik:
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@v1
```

Tanken er at versjonsnummer bare skal økes ved breaking changes.

Det vil fremdeles fungere å skrive `@main`, men da risikerer du at en workflow som har fungert hittil i din app slutter å fungere uten at du har endret noe i appen sitt Github-repo.

Versjonering kan gjøres på flere måter. Jeg prøver først med en pragmatisk løsning, så ser vi om det funker eller om vi må gjøre noe smartere. For enkelhets skyld beholder jeg ett repo for alle reusable workflows og bruker tags og releases for å sette versjonsnummer. Det har to ikke-ideelle konskevenser:
* Et versjonsnummer gjelder for hele dette repoet, dvs. for alle workflows i dette repoet. Det betyr at når vi øker versjonsnummeret pga. en breaking change i ett workflow-skript får alle workflow-skript nytt versjonsnummer, også de ikke har blitt endret.
* Når jeg gjør en non-breaking change på f.eks. v2 (som betyr at versjonsnummeret ikke skal bumpes til v3) må jeg slette releasen "v2" og tag-en "v2". Deretter må jeg opprette tag "v2" på nytt - fra head i main-branchen - og opprette ny release basert på tag "v2". Kan gjøres i Github web-GUI-et.

# Henvendelser

## For Nav-ansatte
* Dette Git-repositoriet eies av [team Toi](https://teamkatalog.nav.no/team/76f378c5-eb35-42db-9f4d-0e8197be0131).
* Slack: [#arbeidsgiver-toi-dev](https://nav-it.slack.com/archives/C02HTU8DBSR)

## For folk utenfor Nav
* IT-avdelingen i [Arbeids- og velferdsdirektoratet](https://www.nav.no/no/NAV+og+samfunn/Kontakt+NAV/Relatert+informasjon/arbeids-og-velferdsdirektoratet-kontorinformasjon)
