# Terminológiai irányelvek

## Alapszabály
Ne erőltessünk szó szerinti, természetellenes magyar fordítást. Ha nincs jó magyar megfelelő, tartsuk meg az angol szakkifejezést, és első előforduláskor magyarázzuk el. Ha van természetes magyar megfelelő, használjuk, az angol terminust zárójelben megadva.

| Fogalom | Javasolt alak |
|---|---|
| Infrastructure as Code | infrastruktúra kódként (Infrastructure as Code, IaC) |
| resource | erőforrás (resource) |
| data source | adatforrás (data source) |
| input variable | bemeneti változó (input variable) |
| output | kimenet (output) |
| provider | provider; szükség esetén szolgáltatóként magyarázd |
| state | állapot (state) |
| backend | backend; az állapot tárolását és elérését biztosító mechanizmus |
| module | modul (module) |
| module interface | modul használati felülete (module interface); angolul gyakran module contract |
| dependency | függőség (dependency) |
| lifecycle | életciklus (lifecycle) |
| workspace | workspace; ne mosd össze a külön infrastruktúra-környezettel |
| plan | végrehajtási terv (plan) |
| apply | konfiguráció alkalmazása (apply) |

## Fontos típusmagyarázat
Az `any` nem konkrét univerzális típus. Típuskorlátozásban olyan jelzés, amelynél a Terraform a tényleges értékből következteti ki a szükséges típust.

Az angol azonosítókat, HCL-kulcsszavakat, függvényneveket és CLI-parancsokat ne fordítsd le. Új terminus esetén frissítsd ezt a fájlt és a szószedet-függeléket.
