# 1. Bevezetés az Infrastructure as Code világába

## A fejezet célja

Ebben a fejezetben megismered azt a szemléletet, amelyre a Terraform használata épül. Mielőtt konfigurációkat írnánk vagy parancsokat futtatnánk, tisztázni kell, mit nevezünk infrastruktúrának, miért érdemes azt kódként kezelni, és milyen problémát old meg a Terraform.

A fejezet végére képes leszel:
- megkülönböztetni az infrastruktúra kézi, szkriptalapú és deklaratív kezelését;
- elmagyarázni az Infrastructure as Code (IaC) lényegét, előnyeit és korlátait;
- értelmezni a desired state, actual state, drift és reconciliation fogalmakat;
- bemutatni a Terraform Core, a configuration language/HCL és a provider szerepét;
- áttekinteni a Write–Plan–Apply munkafolyamatot;
- felismerni a Terraform használatának fontos határait és kockázatait.

**Előfeltétel:** nincs szükség korábbi Terraform- vagy felhőismeretre.

---

## 1.1. Mit nevezünk infrastruktúrának?

Az infrastruktúra azoknak a technikai erőforrásoknak és szolgáltatásoknak az együttese, amelyekre egy alkalmazás vagy informatikai rendszer működése támaszkodik. Nem kizárólag fizikai szervereket jelent.

Ide tartozhat például:
- számítási kapacitás: fizikai szerver, virtuális gép, konténerplatform vagy serverless futtatókörnyezet;
- tárolás: blokkos tároló, objektumtár, fájlrendszer vagy adatbázis;
- hálózat: virtuális hálózat, alhálózat, útválasztási szabály, tűzfal és terheléselosztó;
- identitás és hozzáférés: felhasználó, szerepkör, jogosultsági szabály és szolgáltatásazonosító;
- platform- és alkalmazásszolgáltatás: DNS-rekord, üzenetsor, monitoring-erőforrás vagy SaaS-konfiguráció.

Az infrastruktúra lehet helyben üzemeltetett (on-premises), nyilvános felhőben, több felhőben vagy ezek kombinációjában.

### 1.1.1. Infrastruktúra mint egymásra épülő erőforrások rendszere

Az erőforrások ritkán függetlenek egymástól. Egy virtuális gép például hálózathoz, alhálózathoz, hálózati interfészhez és jogosultsági szabályokhoz kapcsolódhat. Egy adatbázis használhat titkosítási kulcsot, privát hálózati elérést és mentési szabályzatot.

Ezek a kapcsolatok **függőségek (dependencies)**. A függőségek meghatározhatják, milyen sorrendben lehet erőforrásokat létrehozni vagy törölni. Ha egy virtuális gép egy alhálózatra támaszkodik, a hálózatnak a gép előtt kell rendelkezésre állnia; törléskor gyakran fordított sorrend szükséges.

A Terraform képes a konfigurációban szereplő hivatkozásokból függőségi kapcsolatokat felismerni, és ezek alapján műveleti gráfot építeni. A függőségek részletes kezelésével későbbi fejezet foglalkozik.

### 1.1.2. Miért nehéz az infrastruktúra kézi kezelése?

Kisebb környezetben kényelmes lehet egy grafikus felületen létrehozni néhány erőforrást. A nehézség a környezet növekedésével jelentkezik:

1. **Ismételhetőség:** ugyanazokat a lépéseket több környezetben vagy több alkalommal kell végrehajtani.
2. **Konzisztencia:** könnyű véletlenül eltérő beállítást alkalmazni két hasonló rendszerben.
3. **Nyomon követhetőség:** utólag nehéz megállapítani, ki, mikor és miért módosított egy beállítást.
4. **Dokumentáltság:** a tényleges állapot sokszor csak a grafikus felületen vagy az üzemeltetők emlékezetében létezik.
5. **Helyreállítás:** egy környezet újraépítése időigényes lehet, ha a létrehozás lépései nincsenek rögzítve.
6. **Skálázás:** a kézi műveletek száma gyorsabban nőhet, mint az üzemeltetői kapacitás.

A kézi kezelés ettől még nem minden esetben helytelen. Egyszeri vizsgálat, prototípus vagy vészhelyzeti beavatkozás során lehet indokolt. Tartós, ismétlődő és csapat által kezelt infrastruktúránál azonban fontos, hogy a változtatások ellenőrizhető folyamatban történjenek.

---

## 1.2. Mi az Infrastructure as Code?

Az **Infrastructure as Code (IaC)** magyarul infrastruktúra kódként: olyan megközelítés, amelyben az infrastruktúra kívánt felépítését és beállításait géppel feldolgozható, verziókezelhető leírásban rögzítjük.

Ahelyett, hogy minden alkalommal kézzel kattintanánk végig a létrehozási lépéseket, leírjuk például, hogy:
- milyen típusú erőforrásokra van szükség;
- milyen tulajdonságokkal rendelkezzenek;
- hogyan kapcsolódjanak egymáshoz;
- milyen értékek legyenek környezetenként eltérők.

Az IaC nem egyszerűen annyit jelent, hogy „van egy szkriptünk”. A lényeg az infrastruktúra állapotának kifejezett, újra felhasználható és ellenőrizhető leírása, valamint az ehhez kapcsolódó munkafolyamat.

### 1.2.1. Az IaC fő előnyei

**Ismételhetőség:** ugyanaz a konfiguráció megfelelő feltételek mellett újra felhasználható. Ez csökkenti a kézi lépésekből eredő eltéréseket.

**Verziókezelhetőség:** a konfiguráció Githez hasonló verziókezelő rendszerben tárolható. Láthatóvá válik, mi változott, mikor és milyen módosítás részeként.

**Felülvizsgálhatóság:** a módosítások végrehajtása előtt másik csapattag is átnézheti a kódot és a tervezett változtatásokat.

**Dokumentáció:** a konfiguráció egyben leírja az infrastruktúra szándékolt felépítését. Ez nem helyettesít minden üzemeltetési dokumentációt, de javíthatja az átláthatóságot.

**Automatizálhatóság:** a validálás, tervezés és alkalmazás automatizált folyamatba illeszthető.

**Skálázhatóság:** a jól megtervezett konfigurációk és modulok segítségével nagyobb infrastruktúrák is egységesebb elvek szerint kezelhetők.

### 1.2.2. Az IaC nem varázsszer

A kódként tárolt infrastruktúra önmagában nem garantálja a helyes működést. Egy hibás konfiguráció kódként is hibás marad, sőt automatizáltan akár sok erőforrásra is kiterjeszthető.

Az IaC bevezetése új felelősségeket is jelent:
- a repository hozzáféréseit védeni kell;
- a titkokat nem szabad egyszerű szövegként a konfigurációba írni;
- a módosításokat felül kell vizsgálni;
- a state és a futtatási jogosultságok védelme szükséges;
- a költség- és biztonsági hatásokat végrehajtás előtt mérlegelni kell.

A verziókezelés a konfiguráció változásait rögzíti, de nem feltétlenül tartalmaz minden érzékeny vagy futás közben keletkező adatot. A Terraform state például külön védelmet igényel; erről a későbbi state fejezet részletesen szól.

---

## 1.3. Imperatív és deklaratív megközelítés

Az infrastruktúra automatizálásának két fontos gondolkodásmódja az imperatív és a deklaratív megközelítés. A gyakorlatban léteznek kevert rendszerek is, ezért nem minden eszköz sorolható kizárólag az egyik kategóriába.

### 1.3.1. Imperatív megközelítés

Az **imperatív (imperative)** megközelítés azt írja le, hogy milyen lépéseket, milyen sorrendben kell végrehajtani egy cél eléréséhez.

Egy elvi folyamat:
1. hozz létre egy hálózatot;
2. hozz létre benne egy alhálózatot;
3. készíts egy virtuális gépet;
4. csatlakoztasd a gépet az alhálózathoz;
5. állítsd be a hozzáférési szabályokat.

A végrehajtó félnek vagy szkriptnek kell gondoskodnia a lépések sorrendjéről, az ismételt futtatás kezeléséről, a hibákról és arról, hogy egy részlegesen sikeres futás után hogyan folytassa a munkát.

Az imperatív megközelítés előnye, hogy pontosan szabályozható a műveletek sorrendje és az egyedi eljárás. Hátránya, hogy a kívánt végeredményhez vezető lépéseket és az aktuális állapothoz való alkalmazkodást gyakran nekünk kell megírnunk.

### 1.3.2. Deklaratív megközelítés

A **deklaratív (declarative)** megközelítésben azt írjuk le, hogy milyen állapotot szeretnénk elérni, nem pedig minden egyes végrehajtási lépést.

Például a kívánt állapot:
- létezzen egy meghatározott hálózat;
- legyen benne két alhálózat;
- működjön egy virtuális gép az egyik alhálózatban;
- a géphez csak meghatározott hálózati forgalom legyen engedélyezett.

A rendszer összeveti ezt a leírást az általa ismert aktuális állapottal, és meghatározza a szükséges műveleteket. A konkrét létrehozási, módosítási vagy törlési lépéseket nem nekünk kell minden alkalommal felsorolnunk.

### 1.3.3. A két megközelítés összehasonlítása

| Szempont | Imperatív | Deklaratív |
|---|---|---|
| Mit írunk le? | A végrehajtandó lépéseket | A kívánt állapotot |
| Ki határozza meg a műveleteket? | Jellemzően a szerző | A végrehajtó rendszer |
| Ismételt futtatás | A szkriptnek kell kezelnie | Az állapot-összehasonlítás segít |
| Változáskezelés | Eljárások és feltételek | A kívánt és aktuális állapot különbsége |
| Tipikus kihívás | Részleges futás, sorrend, ismételhetőség | Állapotmodell, függőségek, provider-viselkedés |

A Terraform konfigurációs nyelve deklaratív: a konfiguráció a kezelendő erőforrásokat és azok kívánt tulajdonságait írja le. A Terraform Core ebből és a state-ből, valamint a providerek által szolgáltatott információkból állítja elő a végrehajtási tervet.

A deklaratív leírás nem jelenti azt, hogy a rendszer minden helyzetben automatikusan képes kitalálni a helyes üzleti döntést. A konfiguráció szerzőjének kell meghatároznia a kívánt állapotot, a biztonsági korlátokat és a változtatások elfogadhatóságát.

---

## 1.4. Desired state, actual state, drift és reconciliation

A deklaratív infrastruktúrakezelés megértéséhez négy fogalmat érdemes pontosan elkülöníteni.

### 1.4.1. Desired state – kívánt állapot

A **desired state** az az állapot, amelyet a konfiguráció alapján el szeretnénk érni.

Példa: egy szolgáltatáshoz két azonos szerepű alkalmazáspéldányt szeretnénk, egy adott hálózati környezetben, meghatározott címkékkel és hozzáférési szabályokkal.

A desired state nem feltétlenül egyetlen fájlban található. Több konfigurációs fájl, változó, modul és kifejezés együtt határozhatja meg.

### 1.4.2. Actual state – tényleges állapot

Az **actual state** az infrastruktúra valós, külső rendszerekben fennálló állapota. Például az, hogy a felhőszolgáltatónál jelenleg hány virtuális gép létezik, milyen méretűek, és milyen hálózathoz kapcsolódnak.

A Terraform nem közvetlenül „lát mindent” minden rendszerben. A provider API-kon keresztül kérdez le és módosít erőforrásokat, a state pedig nyilvántartja a konfigurációs erőforráspéldányok és a külső objektumok közötti kapcsolatokat, valamint metaadatokat.

### 1.4.3. Drift – eltérés a nyilvántartott és a tényleges állapot között

**Driftnek** nevezzük, amikor az infrastruktúra tényleges állapota eltér attól, amit a Terraform konfigurációja és nyilvántartott állapota alapján várnánk.

Driftet okozhat például:
- valaki kézzel módosít egy erőforrást a szolgáltató felületén;
- egy másik automatizmus módosítja ugyanazt az objektumot;
- egy külső rendszer automatikusan átállít egy tulajdonságot;
- egy erőforrás a Terraformtól függetlenül eltűnik vagy megváltozik.

A drift nem feltétlenül hiba: lehet szándékos, ideiglenes vagy külső folyamat következménye is. Fel kell ismerni és el kell dönteni, hogy a konfigurációt, a külső rendszert vagy a kezelési folyamatot kell-e módosítani.

### 1.4.4. Reconciliation – egyeztetés a kívánt állapottal

A **reconciliation** az a folyamat, amely során a rendszer összeveti a kívánt és az aktuális állapotot, majd a különbség megszüntetésére törekszik.

A Terraform esetében a konfiguráció és a state alapján, a provider által elérhető külső információk figyelembevételével készül terv. A terv megmutatja, milyen műveletekkel lehet a kezelt infrastruktúrát a konfigurációhoz igazítani.

A Terraform nem folyamatosan futó, minden változást azonnal visszaállító vezérlő. A hagyományos CLI-munkafolyamatban a felhasználó vagy automatizmus indítja el a műveletet. Ha valaki kézzel módosít egy erőforrást, a Terraform nem feltétlenül állítja vissza azonnal a konfiguráció szerinti állapotot.

### 1.4.5. Idempotencia – ugyanaz a kívánt állapot, ismételt végrehajtás

Az **idempotencia (idempotency)** azt jelenti, hogy egy művelet azonos feltételek mellett történő ismételt végrehajtása nem okoz újabb és újabb, nem kívánt változást; a rendszer ugyanahhoz a stabil eredményhez jut.

Infrastruktúra-automatizálásban ez azért fontos, mert a konfigurációt nemcsak egyszer kell alkalmazni. Újra futtathatjuk ellenőrzés, módosítás vagy helyreállítás céljából.

A Terraform deklaratív működése és state-kezelése támogatja az idempotens munkafolyamatot, de **nem szabad azt feltételezni, hogy minden Terraform-konfiguráció és minden provider-művelet feltétel nélkül idempotens**. Külső API-k, provider-implementációk, nem determinisztikus értékek, időközben változó külső állapotok vagy hibásan megtervezett erőforrások miatt ismételt futás is eredményezhet változást.

A helyes elv: azonos konfiguráció és változatlan külső feltételek mellett a második tervnek ideális esetben nem kell újabb módosítást javasolnia. Ezt a tényleges környezetben mindig ellenőrizni kell.

---

## 1.5. Mi a Terraform, és miből áll?

A **Terraform** infrastruktúra kódként kezelésére szolgáló eszköz. Ember által olvasható konfigurációs fájlokban írhatjuk le vele a felhő- és helyben üzemeltetett infrastruktúra, valamint különböző külső szolgáltatások kezelendő erőforrásait.

A Terraform egységes munkafolyamatot ad, de nem maga valósítja meg minden platform összes API-műveletét. A működés több, egymástól elkülönülő részből áll.

### 1.5.1. Terraform Core

A **Terraform Core** a Terraform központi motorja. Fő feladatai közé tartozik:
- a konfiguráció beolvasása és feldolgozása;
- a változók, kifejezések és hivatkozások kiértékelésének koordinálása;
- a függőségek feltérképezése;
- a state kezelése;
- a provider pluginekkel való kommunikáció;
- a végrehajtási terv elkészítése és a műveletek koordinálása.

A Terraform Core nem tartalmazza minden felhőszolgáltató összes erőforrásának saját implementációját. Ezt a feladatot a providerek látják el.

### 1.5.2. Terraform Configuration Language és HCL

A Terraform konfigurációját a **Terraform Configuration Language** nyelven írjuk. Ennek natív szintaxisa a **HCL (HashiCorp Configuration Language)** nyelven alapul.

A HCL a konfiguráció leírásának nyelvi/szintaktikai alapja. Segítségével blokkokat, argumentumokat, kifejezéseket és hivatkozásokat írhatunk.

Például:

```hcl
terraform {
  required_version = ">= 1.5.0"
}
```

Ez egy egyszerű, provider nélküli konfigurációs részlet. A `terraform` blokk a Terraform saját beállításait tartalmazza, a `required_version` argumentum pedig megadja, milyen Terraform Core-verziót fogad el a konfiguráció. A fenti verziókorlát példaértékű, nem minden projektre kötelező ajánlás.

A HCL nem általános célú programozási nyelv, és a Terraform sem pusztán HCL-parser. A Terraform saját konfigurációs nyelve a HCL szintaxisára és Terraform-specifikus jelentésére épül.

### 1.5.3. Provider – kapcsolat a külső rendszerekkel

A **provider** egy plugin, amely lehetővé teszi, hogy a Terraform egy adott platformmal vagy szolgáltatással kommunikáljon. A provider tipikusan az adott rendszer API-jait használja.

Provider példák:
- AWS provider;
- AzureRM provider;
- Google provider;
- Oracle Cloud Infrastructure (OCI) provider;
- Kubernetes provider;
- GitHub provider.

A provider határozza meg többek között, milyen resource- és data source-típusok érhetők el, milyen argumentumokat fogadnak el, és hogyan történnek a külső rendszerben a lekérdezési és módosítási műveletek.

**Lényeges különbség:** a Terraform Core és a provider nem ugyanaz. A Core a Terraform általános működését koordinálja; a provider az adott külső rendszerhez kapcsolódó műveleteket valósítja meg.

A provider nem azonos a felhőszolgáltatóval. Az AWS például szolgáltató/platform, az AWS provider pedig az a plugin, amelyen keresztül a Terraform az AWS API-ival kommunikál.

### 1.5.4. Resource és data source – két eltérő szerep

A **resource** olyan objektumot képvisel, amelynek életciklusát a Terraform kezeli. A Terraform létrehozhatja, módosíthatja vagy törölheti – a konkrét lehetőségek a provider és az erőforrástípus működésétől függenek.

A **data source** ezzel szemben külső adat lekérdezésére szolgál. Tipikusan már létező objektum vagy külső információ beolvasására használjuk, nem annak életciklusát kezeljük vele.

Példa:
- egy új virtuális hálózat létrehozása: resource;
- egy már létező hálózat adatainak lekérdezése: data source.

### 1.5.5. State – a Terraform nyilvántartása

A **state** a Terraform által kezelt infrastruktúrára vonatkozó állapotadatokat tartalmazza. Különösen fontos szerepe, hogy összekapcsolja a konfigurációban szereplő resource-példányokat a külső rendszerben létező objektumokkal.

A state nem egyszerű napló és nem maga a konfiguráció. A konfiguráció a kívánt állapot leírása; a state a Terraform nyilvántartása a kezelt objektumokról és a szükséges metaadatokról.

A state érzékeny adatokat is tartalmazhat, ezért hozzáférését és tárolását védeni kell. A state és a backend részletes tárgyalása külön fejezetben következik.

---

## 1.6. Hogyan dolgozik együtt a Terraform Core, a HCL és a provider?

A következő egyszerűsített folyamat segít megérteni a szerepeket:

1. A felhasználó Terraform konfigurációt ír HCL-alapú nyelven.
2. A Terraform Core beolvassa és értelmezi a konfigurációt.
3. A Core megállapítja, mely providerekre van szükség, és kommunikál velük.
4. A provider a saját platformjának API-ján keresztül információt kér le, illetve végrehajtja a neki delegált műveleteket.
5. A Core a konfigurációt, a state-et és a külső állapotról elérhető információkat felhasználva tervet készít.
6. Jóváhagyás után a Core a terv szerint koordinálja a műveleteket, a providerek pedig végrehajtják a platformfüggő lépéseket.
7. A Terraform frissíti a state-et az eredmények alapján.

Ez a leírás szándékosan egyszerűsített. A tényleges működésben a provider plugin inicializálása, a verziófeloldás, a dependency graph, az API-hívások, a részleges hibák és a state-frissítés összetettebb folyamatot alkotnak.

**Fogalmi ábra:**

```text
Terraform konfiguráció (HCL-alapú)
              |
              v
        Terraform Core
        /     |      \\
       /      |       \\
  konfiguráció  state  függőségek
       \\      |       /
        \\     v      /
        Végrehajtási terv
              |
              v
       Provider plugin
              |
              v
      Külső rendszer API-ja
              |
              v
      Kezelt infrastruktúra
```

Az ábra nem minden belső részlet végrehajtási sorrendjét mutatja, hanem a fő felelősségi határokat.

---

## 1.7. A Terraform alapmunkafolyamata: Write, Plan, Apply

A Terraform alapvető munkafolyamata három fő lépésből áll.

### 1.7.1. Write – konfiguráció írása

Ebben a szakaszban leírjuk, milyen erőforrásokat és tulajdonságokat szeretnénk kezelni. A konfigurációt általában szöveges `.tf` fájlokban tároljuk, és verziókezelőben követjük.

Fontos kérdések:
- Mi a kívánt infrastruktúra?
- Mely erőforrások tartoznak a megoldáshoz?
- Milyen függőségek vannak közöttük?
- Mely értékek környezetfüggők?
- Milyen jogosultságokkal fogja a Terraform kezelni őket?

### 1.7.2. Plan – végrehajtási terv készítése

A `terraform plan` parancs a konfiguráció és a Terraform által ismert állapot alapján megtervezi a szükséges változtatásokat. A terv javasolhat létrehozást, módosítást vagy törlést.

A terv célja, hogy a változtatásokat még végrehajtás előtt át lehessen tekinteni. Különösen fontos ellenőrizni:
- mely erőforrások jönnek létre;
- melyek módosulnak;
- melyek törlődnek vagy cserélődnek le;
- vannak-e nem várt vagy kockázatos változtatások;
- a terv a megfelelő környezetre és jogosultságokra vonatkozik-e.

A plan nem helyettesíti a teljes biztonsági felülvizsgálatot. Nem garantálja, hogy a külső rendszerben a későbbi apply pillanatában minden feltétel változatlan marad, és nem minden külső hatás jelezhető előre tökéletesen.

### 1.7.3. Apply – a terv alkalmazása

A `terraform apply` végrehajtja a szükséges műveleteket. Interaktív használatkor a Terraform a végrehajtás előtt jóváhagyást kérhet; automatizált munkafolyamatban a jóváhagyás módját a pipeline és a használt kapcsolók határozhatják meg.

Az apply tényleges infrastruktúra-módosítást okozhat. Erőforrást hozhat létre, módosíthat vagy törölhet, és ezzel költséget, szolgáltatáskiesést vagy adatvesztési kockázatot is előidézhet.

Ezért a tervet mindig a megfelelő környezetben, megfelelő jogosultságokkal és tudatos jóváhagyással kell alkalmazni.

### 1.7.4. A munkafolyamat ciklikus

```text
Write -> Plan -> Review -> Apply
  ^                         |
  |_________________________|
```

Az infrastruktúra igényei változnak, ezért a konfigurációt módosítjuk, új tervet készítünk, felülvizsgáljuk, majd alkalmazzuk. Csapatban a Review lépés jellemzően kódellenőrzést és a terv eredményének megvitatását is jelenti.

---

## 1.8. Mire alkalmas a Terraform, és mire nem?

### 1.8.1. Tipikus felhasználási területek

A Terraform különösen hasznos, ha:
- több környezetben hasonló infrastruktúrát kell létrehozni;
- infrastruktúra-változtatásokat verziókezelten és felülvizsgálhatóan szeretnénk végezni;
- több platform vagy szolgáltatás erőforrásait egységes munkafolyamatban kell kezelni;
- a kézi létrehozás helyett reprodukálható folyamatra van szükség;
- modulokkal szeretnénk újra felhasználható infrastruktúra-komponenseket kialakítani.

### 1.8.2. Nem minden konfigurációs feladat Terraform-feladat

A Terraform elsődleges célja az infrastruktúra és a külső szolgáltatások erőforrásainak deklaratív kezelése. Nem általános célú operációsrendszer-konfigurációs eszköz, és nem helyettesít automatikusan minden deployment-, konfigurációmenedzsment- vagy alkalmazásüzemeltetési rendszert.

| Feladat | Jellemzően megfelelő eszközkategória |
|---|---|
| Virtuális hálózat és virtuális gép létrehozása | Terraform / IaC |
| Operációs rendszer csomagjainak telepítése | konfigurációmenedzsment vagy image build |
| Alkalmazás binárisának buildelése | build rendszer / CI |
| Alkalmazás folyamatos kiadása | CI/CD és deployment eszköz |
| Infrastruktúra metrikáinak megfigyelése | monitoring/observability rendszer |
| Incidens kezelése | üzemeltetési és incidenskezelési folyamat |

Ezek a kategóriák együtt is használhatók. Például a Terraform létrehozhatja a Kubernetes klasztert, egy másik eszköz pedig telepítheti az alkalmazást a klaszterre. A pontos határ a rendszer architektúrájától függ.

### 1.8.3. Fontos korlátok és kockázatok

**Külső API-függőség:** a Terraform a provider API-in keresztül dolgozik. Ha az API nem érhető el, korlátozott, vagy megváltozik, a művelet sikertelen lehet.

**Provider-viselkedés:** az egyes providerek eltérően kezelhetik az erőforrások frissítését, cseréjét, lekérdezését és törlését. Mindig az adott provider és resource dokumentációja az irányadó.

**Állapotkezelés:** a state elvesztése, sérülése, jogosulatlan módosítása vagy nem megfelelő megosztása komoly működési és biztonsági problémát okozhat.

**Párhuzamos módosítás:** ha több szereplő vagy automatizmus ugyanazt az infrastruktúrát kezeli, összehangolásra és megfelelő state-kezelésre van szükség.

**Költség:** egy erőforrás létrehozása vagy hosszabb idejű fenntartása díjköteles lehet. A Terraform nem költségoptimalizáló rendszer, és nem garantálja, hogy a konfiguráció költségmentes.

**Biztonság:** a Terraformmal létrehozott infrastruktúra lehet hibásan védett. A biztonságos architektúrához jogosultságkezelés, hálózati korlátozás, titokkezelés, naplózás és ellenőrzés is szükséges.

**Nem minden változtatás visszafordítható:** egy törlés vagy erőforráscsere adatvesztést, szolgáltatáskiesést vagy helyreállítási munkát okozhat. A plan gondos ellenőrzése és a mentési/rollback stratégia alapvető.

---

## 1.9. Gyakori félreértések

### „A Terraform egy szkript, amely végrehajtja a sorokat.”

Nem. A Terraform konfigurációs nyelve deklaratív. A konfiguráció leírja a kívánt erőforrásokat és tulajdonságokat; a Core a függőségek és állapotinformációk alapján készít tervet.

### „A provider maga a Terraform.”

Nem. A Terraform Core és a provider külön komponens. A Core általános vezérlést és tervezést végez, a provider pedig az adott külső rendszer API-jával kommunikál.

### „A state a konfiguráció másolata.”

Nem. A konfiguráció a kívánt állapot leírása. A state a Terraform által kezelt erőforráspéldányok és a külső objektumok közötti kapcsolatokat, valamint egyéb működéshez szükséges adatokat tárolja.

### „A Terraform folyamatosan figyeli a felhőt és azonnal visszaállít minden eltérést.”

A szokásos Terraform CLI-munkafolyamat nem folyamatos reconciliation loop. A felhasználónak vagy automatizmusnak műveletet kell indítania. Más rendszerek – például folyamatos vezérlőhurkot alkalmazó platformok – más működési modellt követhetnek.

### „Ha a plan nem mutat hibát, akkor a változtatás biztosan veszélytelen.”

Nem. A plan a Terraform által ismert és előre jelezhető változtatásokat mutatja. A külső környezet, jogosultságok, szolgáltatói korlátozások és alkalmazásszintű hatások külön vizsgálatot igényelhetnek.

### „A deklaratív működés miatt nem kell érteni a végrehajtás következményeit.”

Éppen ellenkezőleg. A Terraform a műveletek technikai részét koordinálja, de a kívánt állapot és a változtatások üzleti, biztonsági és költségbeli következményei a konfiguráció készítőjének felelősségi körében maradnak.

---

## 1.10. Gyakorlófeladatok és teljes megoldások

A feladatok fogalmiak; egyikhez sem szükséges Terraform telepítés, provider-konfiguráció vagy felhőfiók. A megoldásokat csak az önálló próbálkozás után olvasd el.

### 1. feladat – Az infrastruktúra felismerése

**Cél:** különbséget tenni infrastruktúra-erőforrás és alkalmazásfejlesztési tevékenység között.

**Feladat:** sorold be az alábbi elemeket:
1. virtuális hálózat létrehozása;
2. alkalmazás forráskódjának lefordítása;
3. objektumtár létrehozása;
4. DNS-rekord kezelése;
5. alkalmazás tesztjeinek futtatása;
6. adatbázis-szolgáltatás példányának létrehozása.

Jelöld meg, melyek tipikus infrastruktúra-kezelési feladatok.

**Teljes megoldás:**

Tipikus infrastruktúra-kezelési feladat az 1., 3., 4. és 6. pont. Ezek hálózati, tárolási, névfeloldási vagy adatbázis-szolgáltatási erőforrásokat hoznak létre vagy kezelnek.

A 2. és 5. pont alkalmazásfejlesztési/build- és tesztelési tevékenység. Ezek kapcsolódhatnak az infrastruktúra életciklusához, de önmagukban nem tipikus Terraform-erőforráskezelési feladatok.

### 2. feladat – Imperatív vagy deklaratív?

**Cél:** felismerni, hogy egy leírás lépéseket vagy célállapotot rögzít.

**Feladat:** osztályozd a következő állításokat!

A. „Hozz létre egy hálózatot, majd abban egy alhálózatot, végül csatlakoztass hozzá egy gépet.”

B. „Legyen egy hálózat, benne egy alhálózat és egy ahhoz csatlakozó gép.”

C. „Ha a gép még nem létezik, hozd létre; ha létezik, ellenőrizd a méretét, és szükség esetén módosítsd.”

**Teljes megoldás:**

- **A: imperatív.** A leírás végrehajtandó lépéseket és sorrendet ad meg.
- **B: deklaratív.** A kívánt végállapotot rögzíti, a létrehozási lépések meghatározását a végrehajtó rendszerre bízza.
- **C: imperatív jellegű eljárás.** Feltételes lépéseket és műveleti logikát ír elő. Egy deklaratív rendszer belsőleg végezhet hasonló ellenőrzéseket, de a feladat leírása itt az eljárást részletezi.

### 3. feladat – Desired state, actual state és drift

**Cél:** elkülöníteni a konfigurációban rögzített célt a külső rendszer tényleges állapotától.

**Feladat:** egy konfiguráció két alkalmazáspéldányt ír le. A szolgáltatói felületen valaki kézzel létrehoz egy harmadikat.

Válaszolj:
1. Mi a desired state?
2. Mi az actual state?
3. Történt-e drift?
4. Visszaállítja-e a Terraform automatikusan az eltérést abban a pillanatban, amikor a kézi létrehozás megtörténik?

**Teljes megoldás:**

1. A desired state két alkalmazáspéldány, mert ezt írja le a konfiguráció.
2. Az actual state három példány, mert jelenleg ennyi létezik a külső rendszerben.
3. Igen, a kívánt és a tényleges állapot eltér, ezért drift alakult ki.
4. Nem. A hagyományos Terraform CLI nem folyamatosan futó vezérlő. A felhasználónak vagy automatizmusnak új műveletet kell indítania, hogy a Terraform újra felmérje az állapotot és tervet készítsen. A terv eredményét ellenőrizni kell, mielőtt alkalmazzuk.

### 4. feladat – A Terraform komponenseinek szétválasztása

**Cél:** megérteni, melyik komponens milyen felelősséget lát el.

**Feladat:** rendeld a fogalmakhoz a megfelelő leírást!

1. Terraform Core
2. HCL / Terraform Configuration Language
3. Provider
4. State

Leírások:

A. A konfiguráció szintaxisa és kifejezőeszközei.
B. A kezelt erőforráspéldányok és külső objektumok kapcsolatainak nyilvántartása.
C. A külső platform API-jával kommunikáló plugin.
D. A konfiguráció feldolgozását, függőségeket, tervezést és műveletkoordinációt végző központi motor.

**Teljes megoldás:**

1–D, 2–A, 3–C, 4–B.

Különösen fontos, hogy a provider nem a Terraform Core része, és a state sem a kívánt infrastruktúra leírása. A kívánt állapotot a konfiguráció fejezi ki; a state a Terraform nyilvántartását adja.

### 5. feladat – Melyik eszközkategória illik a feladathoz?

**Cél:** felismerni a Terraform szerepét egy teljes üzemeltetési folyamatban.

**Feladat:** válaszd ki, hogy a Terraform vagy inkább más eszközkategória illik-e az elsődleges feladathoz!

1. Virtuális hálózat és alhálózat létrehozása.
2. Operációs rendszerre csomagok telepítése.
3. Alkalmazás lefordítása.
4. DNS-rekord létrehozása.
5. Alkalmazásmetrikák megjelenítése dashboardon.

**Teljes megoldás:**

1. Terraform / IaC.
2. Konfigurációmenedzsment vagy image build eszköz.
3. Build rendszer.
4. Terraform / IaC, ha a választott provider támogatja a DNS-szolgáltatást.
5. Monitoring/observability rendszer.

A feladatok egy nagyobb folyamatban összekapcsolódhatnak. A Terraform létrehozhatja az alkalmazás futtatásához szükséges infrastruktúrát, miközben más eszköz építi, telepíti és monitorozza az alkalmazást.

### 6. feladat – A munkafolyamat helyes sorrendje

**Cél:** megérteni a Write–Plan–Apply folyamatot és a felülvizsgálat szerepét.

**Feladat:** rendezd helyes sorrendbe a lépéseket!

- A. A változtatások alkalmazása.
- B. A konfiguráció megírása vagy módosítása.
- C. A terv áttekintése és jóváhagyása.
- D. A végrehajtási terv elkészítése.

**Teljes megoldás:**

Helyes sorrend: **B → D → C → A**.

Először elkészítjük a konfigurációt, utána a Terraform tervet készít. A tervet felülvizsgáljuk, és csak megfelelő eredmény esetén alkalmazzuk. Az Apply után a következő módosítási igény új ciklust indít.

---

## 1.11. Összefoglalás

Ebben a fejezetben az infrastruktúra kódként történő kezelésének alapgondolatait ismerted meg.

A legfontosabb megállapítások:
- Az infrastruktúra a rendszerek működéséhez szükséges számítási, hálózati, tárolási, identitás- és szolgáltatási erőforrásokat foglalja magában.
- Az IaC az infrastruktúra kívánt felépítését és beállításait verziókezelhető, géppel feldolgozható leírásban rögzíti.
- Az imperatív megközelítés a végrehajtandó lépéseket, a deklaratív megközelítés a kívánt állapotot írja le.
- A desired state a célállapot, az actual state a tényleges külső állapot; a kettő közötti eltérés driftet jelenthet.
- A reconciliation az eltérések felismerésének és a kívánt állapot felé történő rendezésének folyamata.
- Az idempotencia ismételhető, stabil eredményre törekvő működést jelent, de a konkrét viselkedés a konfigurációtól és a providertől is függ.
- A Terraform Core, a Terraform Configuration Language/HCL, a provider és a state eltérő szerepet töltenek be.
- A Terraform alapmunkafolyamata: Write → Plan → Review → Apply.
- A Terraform infrastruktúra-kezelő eszköz; nem helyettesít minden build-, deployment-, konfigurációmenedzsment- vagy monitoringrendszert.
- A plan és az automatizálás csökkenti a kockázatot, de nem helyettesíti a szakmai felülvizsgálatot, a biztonsági kontrollokat és a költségek mérlegelését.

### Tanulási ellenőrzőpont

A következő fejezet előtt próbáld meg saját szavaiddal megválaszolni az alábbi kérdéseket:

1. Miért előnyös az infrastruktúrát verziókezelhető konfigurációként leírni?
2. Mi a legfontosabb különbség az imperatív és a deklaratív megközelítés között?
3. Mi a különbség a desired state és az actual state között?
4. Mi okozhat driftet?
5. Miért nem azonos a Terraform Core és a provider?
6. Mire szolgál a state?
7. Miért kell a plan eredményét apply előtt felülvizsgálni?

Ha ezekre pontosan tudsz válaszolni, készen állsz a Terraform telepítésének és a fejlesztői környezet kialakításának megismerésére.

---

## Hivatalos dokumentáció és további olvasmányok

A fejezet fogalmi és technikai állításainak elsődleges forrásai a HashiCorp hivatalos dokumentációjának alábbi oldalai. A dokumentáció folyamatosan frissül; későbbi Terraform-verziók esetén mindig az aktuális változatot ellenőrizd.

1. [What is Terraform?](https://developer.hashicorp.com/terraform/intro) – a Terraform célja, működése, provider-ek és alapmunkafolyamat.
2. [Core Terraform Workflow Overview](https://developer.hashicorp.com/terraform/intro/core-workflow) – Write, Plan és Apply részletes áttekintése.
3. [Terraform Configuration Language](https://developer.hashicorp.com/terraform/language) – a Terraform konfigurációs nyelvének áttekintése.
4. [Configuration Syntax](https://developer.hashicorp.com/terraform/language/syntax/configuration) – blokkok, argumentumok, kifejezések és HCL-alapú szintaxis.
5. [State](https://developer.hashicorp.com/terraform/language/state) – a state szerepe és működése.
6. [What is Infrastructure as Code with Terraform?](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code) – az IaC alapelvei és Terraformmal való kapcsolata.
