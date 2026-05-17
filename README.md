# przyklad_blok4_drugie_cd — Reusable CD Workflows

## Opis

Centralne repozytorium z **reusable workflows** do deploymentu Terraform.  
Inne repozytoria w organizacji wywołują te workflow zamiast duplikować kod pipeline'a.

### Architektura

```
                ┌──────────────────────────────────────┐
                │  przyklad_blok4_drugie_cd             │
                │  (centralne repo z workflow)           │
                │                                       │
                │  .github/workflows/                   │
                │    ├── reusable-plan.yml    ◄────────── wywoływany przez CI repo
                │    └── reusable-apply.yml   ◄────────── wywoływany przez CI repo
                └──────────────────────────────────────┘
                          ▲                ▲
                          │                │
    ┌─────────────────────┘                └──────────────────┐
    │                                                         │
┌───┴──────────────────────┐          ┌───┴──────────────────────┐
│  przyklad_blok4_drugie_ci │          │  inne-repo-z-terraform   │
│  (repo z kodem TF)        │          │  (kolejne repo infra)    │
│                            │          │                          │
│  CI: fmt + validate + lint │          │  Ten sam wzorzec:        │
│  CD: woła reusable-plan    │          │  uses: org/cd-repo/...   │
│      woła reusable-apply   │          │                          │
└────────────────────────────┘          └──────────────────────────┘
```

### Dlaczego rozdzielać CI od CD?

| Aspekt | CI w repo z kodem | CD w centralnym repo |
|--------|-------------------|----------------------|
| Odpowiedzialność | Zespół deweloperski | Zespół platformowy |
| Zmiana | Razem z kodem infra | Niezależnie od projektów |
| Standaryzacja | Każdy robi po swojemu | Jedna wersja planu/apply |
| Audyt | Per-repo | Centralne logowanie |
| Bezpieczeństwo | Każdy repo zarządza | Sekrety w jednym miejscu |

### Wywołanie z innego repo

```yaml
jobs:
  deploy:
    uses: TWOJA-ORG/przyklad_blok4_drugie_cd/.github/workflows/reusable-plan.yml@main
    with:
      environment: dev
      working_directory: ./infra
      tfvars_file: environments/dev.tfvars
      backend_key: "myproject/dev.terraform.tfstate"
    secrets: inherit
```

### Wymagania

- Oba repozytoria w tej samej organizacji GitHub (lub CD repo publiczne)
- `secrets: inherit` — wymaga tej samej organizacji
- W CD repo: `Settings → Actions → Access → Accessible from repositories in the organization`
