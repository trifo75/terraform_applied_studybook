# H. Gyakori hibák és hibaüzenetek

| Tünet | Lehetséges ok | Első ellenőrzés |
|---|---|---|
| Initialization required | Hiányzó vagy megváltozott init | `terraform init` |
| Provider not installed | Provider hiányzik | `required_providers`, lock fájl |
| Unsupported argument | Hibás resource argumentum | Provider verziódokumentáció |
| Invalid index | Nem létező lista/map elem | `count`, `for_each`, feltételek |
| Reference to undeclared | Hibás név vagy scope | Resource/variable/local cím |
| Authentication failed | Hibás vagy hiányzó credential | Provider hitelesítési útmutató |
| Permission denied | Kevés jogosultság | IAM/RBAC policy |
| State lock error | Másik futás lockolja a state-et | Aktív művelet és backend |
| Drift | Kézi vagy külső módosítás | `terraform plan` |

Ne a hibaüzenet utolsó sorát olvasd csak el: a teljes diagnosztika gyakran tartalmazza a pontos fájlt, sort és erőforráscímet.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Gyors hibakeresési táblázat

| Tünet | Lehetséges ok | Első lépés |
|---|---|---|
| Initialization required | Még nem futott init, vagy változott a backend/provider/module | `terraform init` |
| Unsupported argument | Hibás vagy más provider verzióhoz tartozó argumentum | Ellenőrizd a provider dokumentációt és verziót |
| Invalid index | Üres lista vagy hiányzó elem | Ellenőrizd a `count`, `for_each` és indexelést |
| Missing required argument | Kötelező resource/variable argumentum hiányzik | Olvasd el a blokk sémáját |
| Error acquiring the state lock | Másik futás tartja a lockot | Ellenőrizd, fut-e még másik Terraform |
| Provider configuration not present | Hiányzó vagy rossz provider alias | Ellenőrizd a provider blokkot és a modul-hozzárendelést |
| Inconsistent result after apply | Provider/API nem a várt értéket adta vissza | Vizsgáld a provider verziót és API választ |

A hibaüzenet teljes szövegét olvasd végig, mert a Terraform gyakran megadja a fájlt, sort és érintett resource címet. A hibát ne `-target` vagy state kézi módosításával próbáld elsőként elfedni.
