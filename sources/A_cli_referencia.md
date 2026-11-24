# A. Terraform CLI parancsreferencia

| Parancs | Feladat |
|---|---|
| `terraform init` | Munkakönyvtár és backend inicializálása |
| `terraform fmt` | HCL formázása |
| `terraform validate` | Konfiguráció statikus ellenőrzése |
| `terraform plan` | Változtatási terv készítése |
| `terraform apply` | Terv végrehajtása |
| `terraform destroy` | Kezelt erőforrások törlése |
| `terraform output` | Output értékek megjelenítése |
| `terraform state list` | State erőforráscímeinek listázása |
| `terraform state show` | Egy state objektum részletei |
| `terraform console` | Kifejezések interaktív kiértékelése |
| `terraform test` | Terraform tesztfájlok futtatása |
| `terraform show` | Terv vagy state megjelenítése |
| `terraform providers` | Provider függőségek megjelenítése |

A kapcsolók és részletes szintaxis verziófüggő lehet. Mindig ellenőrizd a `terraform <parancs> -help` kimenetét.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Parancsok biztonságos használata

| Parancs | Mire való? | Fontos megjegyzés |
|---|---|---|
| `terraform init` | Backend, provider és modul inicializálása | Biztonságosan újrafuttatható; backendváltásnál migrációs döntés kell. |
| `terraform fmt` | HCL formázása | `-check` ellenőrző módban fut. |
| `terraform validate` | Konfiguráció ellenőrzése | Nem ellenőrzi a felhőbeli jogosultságot és kvótát. |
| `terraform plan` | Változtatási terv | Önmagában nem módosít infrastruktúrát. |
| `terraform apply` | Terv végrehajtása | Ellenőrizd a tervet és a célkörnyezetet. |
| `terraform destroy` | Kezelt objektumok törlése | Adatvesztést okozhat. |
| `terraform output` | Root outputok megjelenítése | Sensitive outputhoz külön körültekintés kell. |
| `terraform state` | State vizsgálata/módosítása | Haladó, körültekintő használat. |
| `terraform workspace` | CLI workspace kezelés | Nem teljes biztonsági izoláció. |
| `terraform test` | HCL tesztek futtatása | Apply teszt valós erőforrást hozhat létre. |

A parancsok pontos kapcsolóit mindig ellenőrizd a telepített verzió `-help` kimenetével.
