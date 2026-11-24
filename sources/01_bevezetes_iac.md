# 1. Bevezetés az Infrastructure as Code világába

*Infrastructure as Code (IaC) és deklaratív infrastruktúra*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

Az Infrastructure as Code (IaC) az infrastruktúra erőforrásainak géppel feldolgozható konfigurációként történő leírása. Ahelyett, hogy egy adminisztrátor a felhőkonzolban kézzel hozna létre hálózatot, virtuális gépet vagy adatbázist, a kívánt állapotot kódban rögzítjük. A Terraform ezt a konfigurációt összeveti a nyilvántartott és a tényleges állapottal, majd tervet készít a szükséges változtatásokról.

A deklaratív megközelítésben elsősorban azt írjuk le, hogy milyen állapotot szeretnénk elérni. Nem kell minden lépést imperatív módon felsorolnunk. A Terraform a függőségek alapján állapítja meg a végrehajtás sorrendjét. Ez nem jelenti azt, hogy a végrehajtás kockázatmentes: egy hibás terv erőforrásokat törölhet vagy cserélhet le, ezért az ellenőrzés és a változtatások felülvizsgálata alapvető.

Az IaC fő előnyei: ismételhetőség, verziókezelhetőség, auditálhatóság, automatizálhatóság és a környezetek közötti konzisztencia. A kód review-zható, tesztelhető és pipeline-ba illeszthető. Ugyanakkor a state, a titkok, a jogosultságok és az erőforrások költsége külön figyelmet igényel.

A Terraform provider-alapú rendszer. A Terraform Core értelmezi a konfigurációt és koordinálja a műveleteket; a provider plugin kommunikál az adott szolgáltató API-jával. Így ugyanaz a nyelv használható AWS, Azure, Google Cloud, OCI és számos más rendszer kezelésére. A resource típusok viszont provider-specifikusak.

## Szintaxis és alap példa

```hcl
terraform {
  required_version = ">= 1.6.0"
}

resource "local_file" "hello" {
  filename = "${path.module}/hello.txt"
  content  = "Infrastructure as Code"
}
```

A példa a `local` provider fájlerőforrását használja, tehát nem hoz létre felhőinfrastruktúrát. A `terraform` blokk a Terraform CLI verziójára vonatkozó korlátozást adja meg. A `resource` blokk egy kezelt objektumot deklarál. A `local_file` a resource típusa, a `hello` a konfiguráción belüli helyi címke. A `filename` és `content` argumentumokat a provider értelmezi.

## Részletes magyarázat

A `terraform init` előkészíti a munkakönyvtárat, letölti a szükséges provider plugineket. A `terraform fmt` egységes formára rendezi a HCL-t. A `terraform validate` ellenőrzi a konfiguráció belső helyességét. A `terraform plan` megmutatja a tervezett módosításokat, az `apply` pedig végrehajtja azokat. Tanuláskor először mindig a tervet olvassuk el.

A deklaratív működés nem azonos a „mindig csak hozzáad” viselkedéssel. Ha a konfigurációból eltávolítunk egy kezelt resource blokkot, a Terraform a következő alkalmazáskor törölheti a hozzá tartozó infrastruktúrát. Éles környezetben ezért szükséges a review, a megfelelő jogosultságkezelés és a mentési/rollback stratégia.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Magyarázd el saját szavaiddal a deklaratív és imperatív megközelítés különbségét.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Milyen problémát old meg a provider?

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Milyen veszélye lehet egy resource blokk eltávolításának?

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Részletes értelmezés: miért több az IaC, mint egy automatizált telepítőscript?

Egy hagyományos telepítőscript elsősorban műveletek sorozatát írja le: hozz létre egy hálózatot, majd egy virtuális gépet, végül állíts be rajta néhány értéket. Az Infrastructure as Code (IaC) ezzel szemben a kívánt állapotot írja le. A Terraform ebből a leírásból következteti ki, milyen műveletekre van szükség. Ez a deklaratív megközelítés teszi lehetővé, hogy ugyanazt a konfigurációt többször lefuttassuk: ha a tényleges infrastruktúra már megfelel a konfigurációnak, a terv jellemzően nem javasol változtatást.

A „jellemzően” szó fontos. A Terraform nem varázslatosan garantálja, hogy minden futtatás teljesen azonos eredményt ad. A provider API-ja, a külső rendszer állapota, az adatforrások eredményei, az időközben végrehajtott kézi módosítások és a konfigurációban használt dinamikus értékek mind befolyásolhatják a tervet. Ezért a `terraform plan` nem formaság, hanem a változtatás előtti ellenőrzés egyik legfontosabb pontja.

### A deklaratív működés gondolatmenete

1. Leírod a kívánt erőforrásokat és azok fontos tulajdonságait.
2. A Terraform betölti a konfigurációt, a provider-eket és a korábbi state-et.
3. Lekérdezi a provider segítségével a távoli objektumok aktuális állapotát, amennyire a provider és az adott művelet ezt lehetővé teszi.
4. Összeveti a kívánt és a nyilvántartott/tényleges állapotot.
5. Execution plan formájában bemutatja a tervezett műveleteket.
6. Jóváhagyás után végrehajtja a tervet, majd frissíti a state-et.

### Mikor érdemes Terraformot használni?

Terraform különösen hasznos, ha több környezetet kell konzisztensen felépíteni, infrastruktúra-változásokat verziókezelni és code review-val ellenőrizni, vagy több felhőszolgáltató és külső rendszer erőforrásait közös munkafolyamatban kezelni. Nem minden konfigurációkezelési feladat Terraform-feladat: egy operációs rendszer csomagjainak és szolgáltatásainak részletes konfigurálására gyakran más eszköz alkalmasabb. A Terraform elsősorban infrastruktúra- és szolgáltatás-API-k erőforrásainak életciklusát kezeli.

### Tervezési példa

```hcl
resource "local_file" "hello" {
  filename = "${path.module}/hello.txt"
  content  = "Terraform kezeli ezt a fájlt."
}
```

Ez a példa a `local` provider segítségével egy helyi fájlt kezel. Oktatási célra jó, mert nem igényel felhőfiókot. Éles infrastruktúrában viszont a helyi fájlok életciklusa nem azonos egy felhőerőforráséval: a fájl a Terraformot futtató gépen jön létre, ezért a futtatási környezet és a fájl elérhetősége is számít.

### Ellenőrző kérdések

- Mi a különbség a deklaratív konfiguráció és az imperatív parancssorozat között?
- Miért nem szabad a `plan` eredményét automatikusan végrehajtandó utasításnak tekinteni?
- Milyen feladatot bíznál Terraformra, és melyiket inkább konfigurációkezelő eszközre?
