# 3. Terraform CLI és az első teljes Terraform-projekt

## A fejezet célja

A 2. fejezetben telepítetted a Terraform CLI-t, és kialakítottad a fejlesztői környezetet. Most végigmegyünk egy teljes, helyi példán: elkészítjük a konfigurációt, inicializáljuk a projektet, áttekintjük a végrehajtási tervet, létrehozzuk az erőforrást, ellenőrizzük az eredményt, majd eltávolítjuk a teszterőforrást.

A fejezet végére képes leszel:
- felismerni a legfontosabb Terraform CLI-parancsok szerepét;
- létrehozni és inicializálni egy egyszerű Terraform-projektet;
- értelmezni a végrehajtási terv jelöléseit;
- megkülönböztetni a konfigurációt, a valós objektumot és a state-et;
- lekérdezni a Terraform által nyilvántartott erőforrásokat és output értékeket;
- biztonságosan eltávolítani a tanulóprojekt erőforrását.

**Előfeltétel:** a Terraform CLI telepítése és a terminál alapvető használata a 2. fejezet szerint. A példához nem kell felhőfiók vagy cloud credential.

## 3.1. A Terraform CLI munkafolyamata

A Terraform CLI parancsokkal vezérli a konfiguráció feldolgozását és az infrastruktúra kezelését. A tipikus munkafolyamat:

1. Konfigurációs fájlok létrehozása vagy módosítása.
2. A munkakönyvtár inicializálása: `terraform init`.
3. A konfiguráció formázása: `terraform fmt`.
4. A konfiguráció ellenőrzése: `terraform validate`.
5. A tervezett változtatások áttekintése: `terraform plan`.
6. A terv jóváhagyása és végrehajtása: `terraform apply`.
7. Az eredmény ellenőrzése.

A parancsokat a projekt gyökérkönyvtárából futtasd, vagyis abból a könyvtárból, amelyben a gyökérmodul konfigurációs fájljai találhatók. A Terraform a könyvtárban található konfigurációs fájlokat együtt dolgozza fel. A fájlnevek és a fájlok sorrendje nem határozza meg a végrehajtás sorrendjét.

| Parancs | Feladat |
|---|---|
| `terraform version` | Megjeleníti a használt CLI verzióját. |
| `terraform init` | Előkészíti a munkakönyvtárat, letölti a szükséges providereket és modulokat, inicializálja a backendet. |
| `terraform fmt` | A konfigurációkat szabványos formára rendezi. |
| `terraform validate` | Ellenőrzi a konfiguráció szintaktikai és belső konzisztenciáját. |
| `terraform plan` | Végrehajtási tervet készít. |
| `terraform apply` | Jóváhagyás után végrehajtja a tervet. |
| `terraform show` | Megjeleníti a state-et vagy egy mentett tervet. |
| `terraform output` | Megjeleníti a root module output értékeit. |
| `terraform state list` | Felsorolja a state-ben nyilvántartott erőforrásokat. |
| `terraform destroy` | Jóváhagyás után eltávolítja a kezelt erőforrásokat. |

A további CLI-parancsokat és kapcsolókat a 13. fejezet tárgyalja részletesen.

## 3.2. A munkafolyamat parancsai

### 3.2.1. Inicializálás – terraform init

A `terraform init` előkészíti a munkakönyvtárat a további műveletekhez. Többek között:
- feldolgozza a provider-követelményeket;
- letölti és telepíti a szükséges provider plugineket;
- letölti a konfigurációban hivatkozott modulokat;
- inicializálja a konfigurált backendet;
- létrehozza vagy frissíti a provider-verziókat rögzítő `.terraform.lock.hcl` fájlt.

~~~bash
terraform init
~~~

Az `init` nem hozza létre a konfigurációban deklarált erőforrásokat. A célja a munkakörnyezet előkészítése. Új projektben és frissen klónozott repositoryban is futtasd. Akkor is szükség lehet rá, ha megváltozik a provider-követelmény, a modul forrása vagy a backend konfigurációja. Többször is biztonságosan futtatható.

### 3.2.2. Formázás – terraform fmt

A `terraform fmt` egységesíti a Terraform-konfigurációk formázását.

~~~bash
terraform fmt
~~~

A parancs szükség esetén módosítja a konfigurációs fájlokat. Ellenőrzésre használható a `-check` kapcsoló:

~~~bash
terraform fmt -check
~~~

A `-check` nem írja át a fájlokat. Nem megfelelő formázás esetén hibakóddal jelez, ezért automatizált ellenőrzésben is hasznos. Több alkönyvtár rekurzív kezeléséhez a `-recursive` kapcsoló használható.

### 3.2.3. Validáció – terraform validate

A `terraform validate` ellenőrzi, hogy a konfiguráció szintaktikailag helyes és belsőleg konzisztens-e.

~~~bash
terraform validate
~~~

A parancs a blokkok, argumentumok és attribútumhivatkozások helyességét, valamint a típusokkal kapcsolatos konzisztenciát vizsgálja. Inicializált munkakönyvtárra van szüksége, amelyben a hivatkozott providerek és modulok elérhetők.

A `validate` nem bizonyítja, hogy egy felhőszolgáltatás elérhető, a hitelesítő adat érvényes, vagy a felhasználónak van jogosultsága egy művelethez. A konfiguráció általános helyességét ellenőrzi, nem a teljes külső környezetet.

Backend-hozzáférés nélkül a validációhoz így készíthető elő a munkakönyvtár:

~~~bash
terraform init -backend=false
~~~

Ez kihagyja a backend inicializálását, de a szükséges provider- és modultelepítési lépéseket továbbra is elvégzi.

### 3.2.4. Terv készítése – terraform plan

A `terraform plan` összeveti a konfigurációt a korábbi state-tel, szükség szerint lekérdezi a kezelt objektumok aktuális adatait, majd végrehajtási tervet készít.

~~~bash
terraform plan
~~~

A terv megmutatja, milyen változtatásokat javasol a Terraform. Önmagában a `plan` nem hajtja végre a létrehozási, módosítási vagy törlési műveleteket.

A terv áttekintésekor ellenőrizd:
- a megfelelő erőforrások szerepelnek-e benne;
- csak a kívánt létrehozások és módosítások jelennek-e meg;
- nincs-e váratlan törlés vagy újralétrehozás;
- a terv a megfelelő környezetre és bemenetekre vonatkozik-e.

A terv fájlba is menthető:

~~~bash
terraform plan -out=tfplan
terraform show tfplan
~~~

A mentett terv később ugyanígy alkalmazható:

~~~bash
terraform apply tfplan
~~~

A mentett plan fájl tartalmazhatja a teljes konfigurációt, a tervezett változtatásokat és érzékeny értékeket is. Ne töltsd fel nyilvános repositoryba, és csak korlátozott hozzáférésű helyen tárold. A `.gitignore` fájlban a `*.tfplan` kizárása hasznos védelem.

### 3.2.5. Végrehajtás – terraform apply

A `terraform apply` végrehajtja a tervben szereplő műveleteket. Ha nem adsz át neki mentett tervfájlt, először új tervet készít, majd interaktív módban jóváhagyást kér.

~~~bash
terraform apply
~~~

Csak akkor írd be a jóváhagyásnál a `yes` értéket, ha elolvastad a tervet, és a benne szereplő műveleteket valóban végre szeretnéd hajtani. Más válasz vagy a művelet megszakítása elutasítja a végrehajtást.

A `-auto-approve` kapcsoló kihagyja a kézi jóváhagyást:

~~~bash
terraform apply -auto-approve
~~~

Tanulás és kézi munka során ne használd. Automatizálásban is csak olyan folyamatban indokolt, ahol a terv ellenőrzése, a jogosultságok és a végrehajtás jóváhagyása más módon biztosított.

### 3.2.6. Törlés – terraform destroy

A `terraform destroy` a Terraform által kezelt erőforrások eltávolítására készít tervet, majd jóváhagyás után végrehajtja azt.

~~~bash
terraform destroy
~~~

A törlés adatvesztést vagy szolgáltatáskimaradást okozhat. Éles környezetben különösen körültekintően használd, és mindig ellenőrizd, mely erőforrásokat érinti. A tanulóprojektben a parancs csak a projekt által létrehozott helyi fájlt távolítja el. Más projektben vagy nem megfelelő munkakönyvtárban futtatva azonban más erőforrásokat is törölhet.

## 3.3. Az első teljes Terraform-konfiguráció

Egy helyi szövegfájlt kezelő projektet készítünk. A HashiCorp által publikált `local` providert használjuk, ezért nem kell felhőfiók vagy külső infrastruktúra.

A projekt fájlszerkezete:

~~~text
first-terraform-project/
├── .gitignore
├── versions.tf
├── main.tf
└── outputs.tf
~~~

A fájlok szerepe:
- `versions.tf`: a Terraform CLI és a provider követelményeit rögzíti;
- `main.tf`: a kezelt erőforrást deklarálja;
- `outputs.tf`: a futás eredményéből fontos értéket jelenít meg;
- `.gitignore`: kizárja a helyi és érzékeny fájlokat a Git követéséből.

A fájlok különválasztása szervezési konvenció. A Terraform ugyanígy feldolgozná a konfigurációt egyetlen `.tf` fájlban is.

### 3.3.1. Projektmappa létrehozása

PowerShellben:

~~~powershell
New-Item -ItemType Directory -Path first-terraform-project
Set-Location first-terraform-project
~~~

Linux vagy macOS terminálban:

~~~bash
mkdir first-terraform-project
cd first-terraform-project
~~~

A további parancsokat ebből a könyvtárból futtasd.

### 3.3.2. Verziókövetelmények – versions.tf

Hozd létre a `versions.tf` fájlt:

~~~hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
~~~

A blokk elemei:
- `terraform`: a Terraform saját beállításait tartalmazza;
- `required_version`: a használható Terraform CLI-verziókat korlátozza;
- `required_providers`: deklarálja a konfigurációhoz szükséges providereket;
- `local`: a provider helyi neve;
- `source`: a provider globális forráscíme;
- `version`: a provider elfogadható verziótartománya.

A `~> 2.5` a 2.5.0 vagy újabb, de 3.0.0 előtti 2.x verziókat engedi. A ténylegesen kiválasztott verziót a Terraform a lock fájlban rögzíti. A `required_version` nem telepíti és nem frissíti a Terraform CLI-t; azt határozza meg, hogy a futtatott CLI megfelel-e a projekt követelményének.

### 3.3.3. Erőforrás deklarálása – main.tf

Hozd létre a `main.tf` fájlt:

~~~hcl
resource "local_file" "greeting" {
  filename = "${path.module}/greeting.txt"
  content  = "Az első Terraform-erőforrásom."
}
~~~

A `resource` blokk egy Terraform által kezelt objektumot deklarál. A részei:

- `resource`: erőforrás deklarálására szolgáló HCL-blokk;
- `local_file`: az erőforrás típusa; a `local` a provider neve, a `file` a provider resource típusa;
- `greeting`: az erőforrás helyi neve;
- `filename`: a létrehozandó fájl elérési útja;
- `content`: a fájlba írandó UTF-8 szöveg;
- `path.module`: Terraform által biztosított útvonalérték, amely az aktuális modul könyvtárára mutat.

A resource teljes címe: `local_file.greeting`. A cím a típusból és a helyi névből áll; ezt használja a Terraform a tervben és a state-ben.

A `local_file` resource a helyi fájlrendszeren hoz létre vagy kezel fájlt. Ha a megadott fájl már létezik, a provider a konfiguráció szerinti tartalomra írhatja át. Ezért a példában kizárólag a tanulóprojekthez fenntartott fájlnevet használj.

Hivatalos dokumentáció: https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file

### 3.3.4. Kimenet deklarálása – outputs.tf

Hozd létre az `outputs.tf` fájlt:

~~~hcl
output "greeting_file_path" {
  description = "A létrehozott szövegfájl elérési útja."
  value       = local_file.greeting.filename
}
~~~

Az `output` blokk a root module egy értékét teszi elérhetővé a futás végén, illetve a `terraform output` paranccsal.

- `greeting_file_path`: az output neve;
- `description`: rövid leírás;
- `value`: a megjelenítendő kifejezés;
- `local_file.greeting.filename`: a `greeting` nevű erőforrás `filename` attribútumára hivatkozik.

A hivatkozás első része a resource típusa, a második a helyi neve, a harmadik az attribútuma. A hivatkozás egyben jelzi a Terraformnak, hogy az output a resource adataitól függ. Az output nem hoz létre új fájlt, csak megjeleníti a fájl útvonalát.

## 3.4. A projekt végrehajtása

Ha elkészült a három konfigurációs fájl, a projekt gyökérkönyvtárában futtasd a következő parancsokat.

### 3.4.1. Inicializálás

~~~bash
terraform init
~~~

A Terraform felismeri a `hashicorp/local` provider-követelményt, letölti a megfelelő provider verziót, és létrehozza vagy frissíti a `.terraform.lock.hcl` fájlt. A lock fájlt tartsd verziókezelésben.

### 3.4.2. Formázás és validáció

~~~bash
terraform fmt
terraform validate
~~~

A `fmt` egységesíti a fájlok formázását, a `validate` ellenőrzi a konfiguráció belső helyességét. Ellenőrzésként a formázás módosítás nélkül is vizsgálható:

~~~bash
terraform fmt -check
~~~

### 3.4.3. Terv áttekintése

~~~bash
terraform plan
~~~

Első futtatáskor a tervnek egy `local_file.greeting` erőforrás létrehozását kell jeleznie. A létrehozás jele a `+`. Ha a projektet korábban már alkalmaztad, és az erőforrás megfelel a konfigurációnak, a terv azt is jelezheti, hogy nincs szükség változtatásra.

Ellenőrizd az erőforrás címét, a fájl útvonalát és a tartalmat is. Ha váratlan törlést vagy cserét látsz, ne folytasd az apply művelettel.

### 3.4.4. Alkalmazás

~~~bash
terraform apply
~~~

Olvasd el a megjelenített tervet, majd a jóváhagyási kérdésre csak akkor válaszolj `yes` értékkel, ha a művelet megfelel a szándékodnak.

Sikeres végrehajtás után a projektmappában megjelenik a `greeting.txt` fájl. A tartalma:

~~~text
Az első Terraform-erőforrásom.
~~~

### 3.4.5. Eredmény ellenőrzése

PowerShell:

~~~powershell
Get-Content .\greeting.txt
~~~

Linux/macOS:

~~~bash
cat greeting.txt
~~~

A Terraform által nyilvántartott output megjelenítése:

~~~bash
terraform output
terraform output greeting_file_path
~~~

A state-ben szereplő erőforrások listázása:

~~~bash
terraform state list
~~~

A kimenetben a következő címet kell látnod:

~~~text
local_file.greeting
~~~

Az erőforrás részletesebb adatainak megjelenítése:

~~~bash
terraform state show local_file.greeting
~~~

A state parancsok a Terraform nyilvántartását mutatják, nem helyettesítik a fájl tényleges meglétének ellenőrzését. A state érzékeny értékeket is tartalmazhat; ne oszd meg nyilvánosan.

## 3.5. Mit jelent a state ebben a folyamatban?

A Terraform state az a nyilvántartás, amely összekapcsolja a konfigurációban deklarált erőforrásokat a ténylegesen kezelt objektumokkal, és eltárolja az erőforrásokhoz tartozó adatokat.

A tanulóprojekt alapértelmezés szerint helyi backendet használ, ezért a state a projekt könyvtárában, a `terraform.tfstate` fájlban található.

Három dolgot különböztess meg:

1. **Konfiguráció:** a `.tf` fájlokban leírt kívánt állapot.
2. **Valós objektum:** a fájlrendszeren ténylegesen létező `greeting.txt` fájl.
3. **State:** a Terraform nyilvántartása arról, hogy a `local_file.greeting` című erőforrást melyik objektumhoz kapcsolja, és milyen adatokat ismer róla.

A state nem maga az infrastruktúra, és nem is a konfiguráció másolata. A Terraform a következő plan során a konfigurációt, a korábbi state-et és a provider által elérhető aktuális adatokat együtt használja a tervezett műveletek meghatározásához.

A state érzékeny adatokat is tartalmazhat. Ne töltsd fel nyilvános repositoryba, ne küldd el ellenőrizetlen csatornán, és ne szerkeszd kézzel. A state biztonságos kezelését és a távoli backendet a 18. fejezet tárgyalja részletesen.

## 3.6. A terv műveleti jelölései

A `terraform plan` kimenetében a műveleteket rövid jelölések mutatják.

| Jelölés | Jelentés | Magyarázat |
|---|---|---|
| `+` | Létrehozás | Az erőforrás még nem szerepel a state-ben, a Terraform létrehozza. |
| `~` | Helyben módosítás | A provider az erőforrás cseréje nélkül módosíthatja. |
| `-` | Törlés | A Terraform eltávolítja a kezelt erőforrást. |
| `-/+` | Csere | A Terraform törli a meglévő objektumot, majd újat hoz létre. |
| `+/-` | Csere | Előbb az új objektum jön létre, majd a régi törlődik. |

Nem minden resource módosítható helyben. Hogy egy argumentum változása módosítást vagy cserét eredményez-e, azt az adott provider resource dokumentációja határozza meg.

A terv összesítése például így nézhet ki: `Plan: 1 to add, 0 to change, 0 to destroy.` Ez egy létrehozást, nulla módosítást és nulla törlést jelent. A számok a projekt aktuális állapotától függnek, ezért egy mintakimenet nem minden futtatásra érvényes.

## 3.7. Mi történik, ha megváltozik a konfiguráció?

Ha átírod a `content` értékét a `main.tf` fájlban:

~~~hcl
resource "local_file" "greeting" {
  filename = "${path.module}/greeting.txt"
  content  = "A Terraform módosította a fájl tartalmát."
}
~~~

Ezután futtasd:

~~~bash
terraform plan
~~~

A terv megmutatja, milyen változás várható. A módosítás végrehajtásához ismét jóvá kell hagynod az apply műveletet:

~~~bash
terraform apply
~~~

A konfiguráció módosítása önmagában nem változtatja meg a fájlt. A változás az apply során történik meg.

Ha a fájlt Terraformon kívül törlöd, a következő plan a provider által észlelt eltérés alapján újralétrehozást javasolhat. Ez jól mutatja, hogy a Terraform a deklarált állapotot a későbbi futtatások során is összeveti a kezelt objektumok aktuális állapotával.

## 3.8. A tanulóprojekt eltávolítása

Ha befejezted a példát, távolítsd el a Terraform által létrehozott fájlt:

~~~bash
terraform destroy
~~~

Ellenőrizd a törlési tervet. Ebben a projektben a `local_file.greeting` erőforrás törlésének kell szerepelnie. Csak akkor hagyd jóvá, ha valóban a tanulóprojekt fájlját szeretnéd eltávolítani.

Sikeres törlés után:
- a `greeting.txt` fájl eltűnik;
- a `terraform state list` nem sorolja fel a korábbi resource-t;
- a konfigurációs fájlok és a lock fájl megmaradnak;
- a state már nem tartalmazza a törölt erőforrást.

A projektmappát ezután kézzel is törölheted, ha nincs rá szükséged. A `terraform destroy` nem törli a konfigurációs fájlokat, a `.terraform/` könyvtárat vagy a teljes projektmappát.

## 3.9. Gyakori hibák és megoldásuk

| Jelenség | Lehetséges ok és megoldás |
|---|---|
| A `terraform init` nem tudja letölteni a providert | Ellenőrizd a hálózati kapcsolatot, a provider forráscímét, a registry elérhetőségét és a proxybeállításokat. |
| A `terraform validate` inicializálást kér | Futtasd a `terraform init` parancsot a projekt gyökérkönyvtárában. |
| A `terraform plan` nem várt törlést jelez | Ne alkalmazd automatikusan. Ellenőrizd a konfigurációt, a munkakönyvtárat, a workspace-t, a state-et és a terv részleteit. |
| Az apply felülír egy helyi fájlt | A `local_file` resource a konfiguráció szerinti tartalmat írja a megadott útvonalra. Használj a tanulóprojekthez fenntartott fájlnevet. |
| A `terraform output` nem mutat értéket | Ellenőrizd, hogy van-e output blokk, és sikeresen lefutott-e az apply. |
| A `terraform state list` üres | Lehet, hogy még nem futott sikeres apply, másik munkakönyvtárban vagy workspace-ben vagy, vagy az erőforrást már törölted. |
| A módosítás után a terv cserét jelez | Egyes resource argumentumok módosítása csak törléssel és újralétrehozással lehetséges. Olvasd el a tervet és a provider dokumentációját. |
| A destroy több erőforrást törölne, mint vártad | Állj meg, és ellenőrizd a projektet, a workspace-t, a backendet és a teljes tervet. Ne hagyd jóvá, amíg nem érted a törlés okát. |

## 3.10. Gyakorlófeladatok és teljes megoldások

A feladatokhoz szükséges fogalmakat és parancsokat a korábbi alfejezetek már bemutatták. A megoldások minden feladat után szerepelnek; egyik sem igényel előzetesen nem ismertetett Terraform-fogalmat.

### 1. feladat – CLI-parancsok szerepe

**Cél:** megkülönböztetni a projekt előkészítését, ellenőrzését, tervezését és végrehajtását végző parancsokat.

**Előfeltétel:** a 3.1–3.2. alfejezetek.

**Feladat:** párosítsd a parancsokat a feladatukkal.

A) `terraform init`  
B) `terraform fmt`  
C) `terraform validate`  
D) `terraform plan`  
E) `terraform apply`  
F) `terraform destroy`

1. A konfiguráció formázása.
2. A munkakönyvtár és a függőségek előkészítése.
3. A konfiguráció belső helyességének ellenőrzése.
4. A tervezett változtatások megjelenítése végrehajtás nélkül.
5. A terv jóváhagyás utáni végrehajtása.
6. A kezelt erőforrások törlése jóváhagyás után.

**Megoldás és indoklás:**
- A → 2. Az `init` előkészíti a munkakönyvtárat és letölti a providereket.
- B → 1. A `fmt` egységesíti a konfiguráció formázását.
- C → 3. A `validate` a konfiguráció belső helyességét ellenőrzi.
- D → 4. A `plan` megmutatja a tervezett műveleteket, de nem hajtja végre őket.
- E → 5. Az `apply` jóváhagyás után végrehajtja a tervet.
- F → 6. A `destroy` eltávolítja a kezelt erőforrásokat.

### 2. feladat – Saját első resource

**Cél:** egy helyi fájlt kezelő Terraform-erőforrás deklarálása és tervének ellenőrzése.

**Előfeltétel:** a 3.3. alfejezet, telepített Terraform CLI és működő hálózati kapcsolat a provider letöltéséhez.

**Feladat:**
1. Hozz létre egy `my-first-resource` nevű projektmappát.
2. Hozd létre benne a `versions.tf` és `main.tf` fájlokat.
3. A `versions.tf` fájlban deklaráld a HashiCorp Local providert a fejezetben látott módon.
4. A `main.tf` fájlban hozz létre egy `local_file` resource-t `message` néven.
5. A resource a `message.txt` fájlba írja ezt: `Terraform kezeli ezt a fájlt.`
6. Futtasd az inicializálást, a formázást, a validációt és a terv készítését.
7. Ellenőrizd, hogy a terv egy erőforrás létrehozását jelzi-e.

**Megoldás és indoklás:**

A `versions.tf` tartalma:

~~~hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
~~~

A `main.tf` tartalma:

~~~hcl
resource "local_file" "message" {
  filename = "${path.module}/message.txt"
  content  = "Terraform kezeli ezt a fájlt."
}
~~~

Futtasd:

~~~bash
terraform init
terraform fmt
terraform validate
terraform plan
~~~

A tervben a `local_file.message` erőforrás létrehozását kell látnod. A `+` jel létrehozást jelent. A feladat a terv ellenőrzéséig tart; az erőforrás létrehozása előtt győződj meg róla, hogy a fájl útvonala a saját tesztmappádra mutat.

### 3. feladat – Terv mentése és megjelenítése

**Cél:** mentett tervfájl készítése és annak ellenőrzése.

**Előfeltétel:** a 3.2.4. és 3.3. alfejezetek, valamint a 2. feladat konfigurációja.

**Feladat:**
1. Készítsd el a végrehajtási tervet `tfplan` néven.
2. Jelenítsd meg a mentett tervet.
3. Ne hajtsd végre a tervet.
4. Magyarázd el, miért kell óvatosan kezelni a mentett tervfájlt.

**Megoldás és indoklás:**

~~~bash
terraform plan -out=tfplan
terraform show tfplan
~~~

Az első parancs fájlba menti a tervet, a második olvasható formában megjeleníti. A feladat nem tartalmaz `terraform apply tfplan` parancsot, ezért a terv nem kerül végrehajtásra.

A mentett terv érzékeny adatokat is tartalmazhat, ezért nem szabad nyilvános repositoryba feltölteni vagy ellenőrizetlenül megosztani.

### 4. feladat – State vizsgálata

**Cél:** megkeresni a Terraform által nyilvántartott erőforrásokat.

**Előfeltétel:** a 3.4. és 3.5. alfejezetek, valamint egy sikeresen alkalmazott projekt.

**Feladat:**
1. Futtasd a `terraform state list` parancsot.
2. Jelenítsd meg a `local_file.greeting` erőforrás részleteit.
3. Kérd le a `greeting_file_path` output értékét.
4. Hasonlítsd össze az output útvonalát a fájl tényleges helyével.

**Megoldás és indoklás:**

~~~bash
terraform state list
terraform state show local_file.greeting
terraform output greeting_file_path
~~~

A state listában a `local_file.greeting` címet kell látnod. A `state show` az erőforrás nyilvántartott attribútumait mutatja. Az output parancs az `outputs.tf` fájlban deklarált útvonalat jeleníti meg.

Ha az erőforrás nem szerepel a listában, ellenőrizd, hogy a megfelelő projektmappában vagy-e, sikeres volt-e az apply, és ugyanazt a workspace-t használod-e.

### 5. feladat – Konfiguráció módosítása

**Cél:** megfigyelni, hogyan készít a Terraform új tervet a konfiguráció megváltozásakor.

**Előfeltétel:** a 3.7. alfejezet és a sikeresen alkalmazott első projekt.

**Feladat:**
1. Módosítsd a `local_file.greeting` resource `content` értékét erre: `A fájl tartalmát frissítettem.`
2. Futtasd a `terraform plan` parancsot.
3. Olvasd el, milyen változtatást tervez a Terraform.
4. Ha a terv a megfelelő fájl módosítását mutatja, futtasd az `apply` parancsot.
5. Ellenőrizd a fájl új tartalmát.

**Megoldás és indoklás:**

A módosított resource:

~~~hcl
resource "local_file" "greeting" {
  filename = "${path.module}/greeting.txt"
  content  = "A fájl tartalmát frissítettem."
}
~~~

Parancsok:

~~~bash
terraform plan
terraform apply
~~~

A tervnek a `local_file.greeting` tartalmának változását kell jeleznie. A pontos műveleti jelölés a provider erőforrás-viselkedésétől függ. Az apply után a fájlnak az új szöveget kell tartalmaznia.

### 6. feladat – Biztonságos törlés

**Cél:** a tanulóprojekt által létrehozott erőforrás eltávolítása és a state ellenőrzése.

**Előfeltétel:** a 3.8. alfejezet és egy sikeresen alkalmazott tanulóprojekt.

**Feladat:**
1. Futtasd a `terraform destroy` parancsot.
2. Ellenőrizd a törlési tervet, és csak a tanulóprojekt erőforrásának törlését hagyd jóvá.
3. Futtasd újra a `terraform state list` parancsot.
4. Ellenőrizd, hogy a létrehozott fájl eltűnt-e.

**Megoldás és indoklás:**

~~~bash
terraform destroy
terraform state list
~~~

A törlési tervben a tanulóprojekt erőforrásának kell szerepelnie. Jóváhagyás után a Terraform eltávolítja a fájlt, és kiveszi az erőforrást a state-ből. A state listában a korábbi resource cím már nem jelenhet meg. A konfigurációs fájlok megmaradnak, a `destroy` nem törli a projekt teljes könyvtárát.

## 3.11. Összefoglalás

Ebben a fejezetben végigvettük a Terraform CLI alapvető munkafolyamatát, és létrehoztunk egy helyi erőforrást kezelő projektet.

A legfontosabb tudnivalók:
- Az `init` előkészíti a munkakönyvtárat és telepíti a szükséges provider plugineket.
- A `fmt` egységesíti a konfiguráció formázását.
- A `validate` a konfiguráció szintaktikai és belső konzisztenciáját ellenőrzi.
- A `plan` végrehajtás nélkül megmutatja a tervezett változtatásokat.
- Az `apply` jóváhagyás után végrehajtja a tervet.
- A `destroy` jóváhagyás után eltávolítja a kezelt erőforrásokat.
- A resource címe a típusból és a helyi névből áll, például `local_file.greeting`.
- A state összekapcsolja a konfigurációban deklarált erőforrást a Terraform által kezelt objektummal.
- A mentett plan és a state érzékeny adatokat tartalmazhat, ezért védeni kell őket.
- A törlés és az apply előtt mindig olvasd el a teljes tervet.

### Tanulási ellenőrzőpont

A következő fejezet előtt ellenőrizd, hogy önállóan meg tudod-e válaszolni:

1. Miért kell a `terraform init` parancsot futtatni az első plan előtt?
2. Miben különbözik a `terraform validate` és a `terraform plan`?
3. Végrehajt-e változtatást a `terraform plan`?
4. Mit jelent a `+`, a `~` és a `-/+` jelölés?
5. Mi a különbség a konfiguráció, a valós objektum és a state között?
6. Miért kell védeni a state és a mentett tervfájl tartalmát?
7. Mit távolít el a `terraform destroy`, és mit nem?

Ha ezekre saját szavaiddal tudsz válaszolni, és önállóan végig tudod vinni a tanulóprojekt létrehozását, ellenőrzését és eltávolítását, készen állsz a HCL nyelvi alapjainak részletes feldolgozására.

## Hivatalos források és további olvasnivaló

A Terraform CLI és a provider viselkedése verziónként változhat. A parancsok használatakor a telepített CLI-hez, a providerhez pedig a kiválasztott provider-verzióhoz tartozó dokumentációt ellenőrizd.

1. **Terraform CLI parancsok áttekintése:** https://developer.hashicorp.com/terraform/cli/commands
2. **terraform init:** https://developer.hashicorp.com/terraform/cli/commands/init
3. **terraform validate:** https://developer.hashicorp.com/terraform/cli/commands/validate
4. **terraform plan:** https://developer.hashicorp.com/terraform/cli/commands/plan
5. **terraform apply:** https://developer.hashicorp.com/terraform/cli/commands/apply
6. **terraform show:** https://developer.hashicorp.com/terraform/cli/commands/show
7. **Terraform resource blokkok:** https://developer.hashicorp.com/terraform/language/resources
8. **Provider-követelmények:** https://developer.hashicorp.com/terraform/language/providers/requirements
9. **HashiCorp Local provider – local_file resource:** https://registry.terraform.io/providers/hashicorp/local/latest/docs/resources/file
