# Reusable workflows for Github Actions for team Toi sine JVM backend-applikasjoner

Hva er det? Se https://docs.github.com/en/actions/sharing-automations/reusing-workflows

# Versjonering
Tidligere brukte ikke appene våre versjoner da de refererte til egenskrevne workflows. Vi bare referete til nyeste commit på main-branchen, ved å skrive `@main` i
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@main
```
Ulempen med dette er at en workflow  som har fungert hittil i din app slutter å fungere uten at du har endret noe i din app sitt repo. Det kan skje fordi en endring med en feil eller en braking change ble pushet til dette repoet (repoet som inneholder de felles worflows som din app sin worflow refererer til).

Dette unngår vi ved at konsumenten låser seg til et versjonsnummer ved å skrive `@v1`, `@v2`, `@v3`, osv. slik:
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@v1
```

Tanken er at versjonsnummer økes hos konsumenten/på bruksstedet i hver enkelt app bare ved breaking change. Det gir mindre vedlikeholdsarbeid for utviklere.

## Hvordan teste en endring i en workflow før den blir released?
Opprett en feature branch og bruk navnet på feature-branchen din i konsumenten der du normalt skriver versjonsnummer, slik:
osv. slik:
```
jobs:
    call-build-and-deploy:
    uses: navikt/toi-github-actions-workflows/.github/workflows/build-and-deploy.yaml@min-feature-branch
```

## Release Procedure

### Version format

Versions use two parts: **major.minor** (e.g. `v14.1`).

| Part | When to increment | Example |
|---|---|---|
| **Major** | Breaking change (new required input, removed output, renamed workflow) | `v14.x` → `v15.0` |
| **Minor** | Non-breaking change (new optional input, bug fix, dependency update) | `v14.0` → `v14.1` |

However, the consumer (the app) does not use the major.minor version name. Two tags exist per release:

| Tag | Moves? | Purpose |
|---|---|---|
| `v14` | ✅ Yes | What consumers pin to — always points to the latest compatible release |
| `v14.1` | ❌ No | Immutable — used for auditability and rollback |

---
When introducing breaking changes:

1. Create a `v15.0` (immutable) tag and create the `v15` major tag, both pointing to the same commit.
2. **Do not move `v14`** — consumers pinned to `@v14` remain on the previous behaviour until they explicitly update to `@v15`.


### Steps on every change

All changes should be merged to `main` via a pull request before releasing. It would be nice if if the pull request description documents why the change was made.

#### 1. Create an immutable version tag  on the merge commit
This tag name has a minor part. Increment minor for non-breaking, major for breaking.
```bash
git tag v14.1
git push origin v14.1
```

#### 2. Move the major tag (non-breaking change) or create a new major tag (breaking change) to point to the same commit
This tag name does not have a minor part.
```bash
git tag -fa v14 -m "v14 -> v14.1"
git push origin refs/tags/v14 --force
```

#### 3. Verify the tag is on a commit reachable from `main`

```bash
git branch --contains v14 | grep main
```

If `main` is not in the output, the tag is on the wrong commit — do not proceed.

---


# Henvendelser

## For Nav-ansatte
* Dette Git-repositoriet eies av [team Toi](https://teamkatalog.nav.no/team/76f378c5-eb35-42db-9f4d-0e8197be0131).
* Slack: [#arbeidsgiver-toi-dev](https://nav-it.slack.com/archives/C02HTU8DBSR)

## For folk utenfor Nav
* Teknologiavdelingen i [Arbeids- og velferdsdirektoratet](https://www.nav.no/no/NAV+og+samfunn/Kontakt+NAV/Relatert+informasjon/arbeids-og-velferdsdirektoratet-kontorinformasjon)
