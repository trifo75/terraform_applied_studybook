# Terraform – Teljes oktatókönyv és nyelvi referencia

Ez a csomag a 22 fejezetet és 10 függeléket külön Markdown-fájlokban tartalmazza. A kéziratot a korábban jóváhagyott könyvstruktúra és szerkesztési elvek alapján önállóan újradolgozott tanulóanyagként állítottam össze.

## Szerkesztési és verziózási megjegyzések

- A magyarázatok magyarul készültek, az angol Terraform-terminológia megtartásával.
- A provider-specifikus példákat külön kell kezelni; a `local` providerrel bemutatott minták felhőfüggetlen tanulópéldák.
- A Terraform dokumentáció jelenleg 1.17.x ágat is feltüntet; a fejezetekben szereplő minimumverziók nem jelentik azt, hogy minden új funkció minden korábbi verzióban elérhető.
- A `terraform test` 1.6.0-tól érhető el, provider helyettesítése tesztelési célú mock implementációval (provider mocking) 1.7.0-tól.
- A remote backend (távoli backend) beállításait és a felhőszolgáltató resource példákat a tényleges célkörnyezethez kell igazítani.
- A fejezetek tanulási kéziratot alkotnak; éles használat előtt a provider dokumentációját és a tervet mindig ellenőrizni kell.

## Fejezetek és függelékek

A fájlnevek sorszámozottak. A ZIP-ben minden dokumentum külön `.md` fájl.


## Terminológiai szerkesztési elv

A Terraform dokumentációjában és a HCL-ben használt hivatalos angol neveket (például `resource`, `provider`, `backend`, `state`, `workspace`, `output`) a könyv megtartja, mert ezek a kódban, a parancsokban és a hivatalos dokumentációban is így szerepelnek. Első előfordulásukkor magyar magyarázatot kapnak. Ha egy angol szakmai kifejezésnek nincs természetes, egyértelmű magyar megfelelője, az eredeti angol alakot használjuk, szükség esetén rövid magyar magyarázattal és zárójelben az angol terminussal. Kerüljük a szó szerinti, magyarul félrevezető fordításokat.


## Reading proof – v1.2.0

Ez a változat a PDF-es olvasási és jegyzetelési munkapéldány alapja. A tartalmi kiegészítések célja, hogy a fejezetek önállóan is követhetők legyenek. A PDF olvasási proof: a felhasználói átolvasás során jelzett tartalmi, nyelvi és tördelési észrevételek alapján további verziók készülnek.

A Terraform és a provider-ek gyorsan változnak. A példák tanulási célúak; felhőszolgáltató-specifikus erőforrás alkalmazása előtt mindig ellenőrizd a használt provider aktuális dokumentációját.
