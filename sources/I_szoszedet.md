# I. Magyar–angol Terraform szószedet

| Angol kifejezés | Magyar magyarázat |
|---|---|
| Infrastructure as Code | Infrastruktúra kódként |
| provider | Szolgáltató API-jához kapcsoló plugin |
| resource | Kezelt infrastruktúra-objektum |
| adatforrás (data source) | Külső objektum adatainak lekérdezése |
| module | Újrafelhasználható konfigurációs egység |
| state | Terraform állapotnyilvántartás |
| backend | State tárolási és elérési mód |
| plan | Változtatási terv |
| apply | Terv végrehajtása |
| drift | Eltérés a nyilvántartott és valós állapot között |
| dependency graph | Függőségi gráf |
| input variable | Bemeneti változó |
| local value | Modulon belüli számított érték |
| output value | Kimeneti érték |
| workspace | Külön Terraform state-hez tartozó munkaterület |
| locking | Párhuzamos state-módosítás zárolása |
| lifecycle | Erőforrás-életciklus szabályozása |
| provisioner | Külső parancs vagy művelet végrehajtója |
| sensitive | Érzékenyként jelölt érték |
| validation | Bemeneti vagy konfigurációs ellenőrzés |

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)


## Fordítási és terminológiai megjegyzések

| Angol kifejezés | A könyvben használt magyar magyarázat |
|---|---|
| module interface / module contract | A modul használati felülete: a bemeneti változók és kimeneti értékek együttese. A `contract` szót önmagában nem fordítjuk „szerződésnek”. |
| type constraint | Típuskorlátozás; a változóhoz vagy attribútumhoz megengedett értékek típusát írja le. |
| credential chain | Hitelesítési mechanizmusok sorrendje, amelyben a kliens megpróbálja megtalálni a hitelesítő adatokat. |
| provider mocking | A provider tesztelési célú helyettesítése mock implementációval. |
| linting | Statikus kódellenőrzés; a Terraform-konfigurációban és a kapcsolódó fájlokban gyakori hibákat, illetve szabályszegéseket keres. |
| drift | Eltérés a Terraform state-ben nyilvántartott és a tényleges infrastruktúra-állapot között. |
| dependency graph | Függőségi gráf; a Terraform által a műveletek sorrendjének meghatározására használt kapcsolati háló. |

## Kiegészítő tanulási anyag
## Kiegészítő szószedet

| Angol kifejezés | Magyar magyarázat |
|---|---|
| desired state | kívánt állapot, amelyet a konfiguráció leír |
| actual state | tényleges állapot a külső rendszerben |
| drift | eltérés a Terraform által nyilvántartott/kívánt és a tényleges állapot között |
| execution plan | végrehajtási terv |
| dependency graph | függőségi gráf |
| module interface / module contract | a modul bemeneti és kimeneti felülete |
| type constraint | típuskorlátozás |
| state locking | state zárolása |
| ephemeral value | átmeneti érték, amely a támogatott kontextusokban nem kerül state-be/plan fájlba |
| workload identity | munkaterheléshez rendelt identitással történő hitelesítés |
| provider alias | egy provider több konfigurációjának megkülönböztető neve |
| data source | adatforrás, amely külső vagy már létező adatot olvas be |
| resource | kezelt infrastruktúra-objektum deklarációja |
