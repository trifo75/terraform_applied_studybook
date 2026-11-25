# 2. Terraform telepítése és fejlesztői környezet kialakítása

## A fejezet célja

Ebben a fejezetben előkészítjük azt a környezetet, amelyben a későbbi fejezetek Terraform-konfigurációit létrehozhatod, formázhatod és ellenőrizheted. Megtanulod telepíteni a Terraform CLI-t Windows, macOS és Linux rendszeren, ellenőrizni a telepítést, kiválasztani és rögzíteni a használt verziót, valamint kialakítani egy áttekinthető projektmappát.

A fejezet végére képes leszel:
- kiválasztani az operációs rendszeredhez illeszkedő Terraform-telepítési módszert;
- ellenőrizni, hogy a telepített Terraform CLI elérhető-e a terminálból;
- megkülönböztetni a Terraform Core, a provider és a szerkesztő szerepét;
- értelmezni a `required_version` és a `required_providers` beállítások közötti különbséget;
- létrehozni egy rendezett, verziókezelésre alkalmas Terraform-projektmappát;
- megkülönböztetni a `.terraform/` munkakönyvtárat és a `.terraform.lock.hcl` függőségi lock fájlt;
- elvégezni az első, infrastruktúrát nem létrehozó ellenőrzéseket.

**Előfeltétel:** az 1. fejezetben bemutatott alapfogalmak ismerete. Korábbi Terraform-telepítésre vagy felhőszolgáltatói fiókra nincs szükség.
## 2.1. Milyen elemekből áll a fejlesztői környezet?

A Terraform használatához nem elegendő önmagában egy szövegszerkesztő. A fejlesztői környezet több, egymást kiegészítő elemből áll.

### 2.1.1. Terraform CLI és Terraform Core

A Terraform CLI (Command Line Interface) az a parancssori eszköz, amellyel a Terraform-műveleteket elindítjuk. A telepített `terraform` végrehajtható állományon keresztül adunk ki parancsokat, például:
- `terraform version`: megmutatja a telepített Terraform-verziót;
- `terraform init`: előkészíti a munkakönyvtárat, többek között letölti a szükséges providereket;
- `terraform fmt`: egységes formára rendezi a Terraform-konfigurációkat;
- `terraform validate`: ellenőrzi a konfiguráció szintaktikai és belső konzisztenciáját;
- `terraform plan`: végrehajtási tervet készít.

A Terraform Core a CLI által elindított Terraform-motor. Feldolgozza a konfigurációt, kiértékeli a kifejezéseket, kezeli az állapotot, és együttműködik a providerekkel. A provider külön komponens: az adott infrastruktúra-szolgáltató vagy API felé végzi az erőforrás-műveleteket.

A Terraform CLI telepítése tehát nem jelenti azt, hogy minden provider is telepítve van. A providereket a projekt konfigurációja alapján, jellemzően a `terraform init` tölti le.

A Terraform parancsokat terminálból futtatjuk. Windows alatt jellemző a PowerShell, macOS-en a Zsh, Linuxon a Bash; WSL-ben külön Linux-környezet használható. A shell-ek szintaxisa eltérhet, ezért a példák jelzik a célkörnyezetet.

A Terraform-fájlokat bármely szövegszerkesztővel módosíthatod. A VS Code HashiCorp Terraform bővítménye szintaxiskiemelést és kódkiegészítést ad, de nem helyettesíti a CLI-ellenőrzéseket. A Git verziókezeléshez és csapatmunkához ajánlott.

A helyi példákhoz nem kell felhőfiók. Felhőprovider használatakor külön gondoskodj a hitelesítésről és jogosultságokról; valódi hozzáférési adatot ne írj konfigurációba vagy verziókezelt fájlba.
## 2.2. Terraform-verziók és verziókorlátozás

### 2.2.1. Miért fontos a verzió?

A Terraform folyamatosan fejlődik. Egy újabb kiadás javításokat, új funkciókat vagy viselkedésbeli változásokat tartalmazhat. Emiatt nem szerencsés, ha egy csapat tagjai vagy a CI-rendszer egymástól eltérő, dokumentálatlan Terraform-verzióval dolgoznak.

A verziók változnak, ezért mindig a telepítés napján elérhető stabil kiadást válaszd a hivatalos oldalon.

Hivatalos forrás: https://developer.hashicorp.com/terraform/install

A verzió kiválasztásakor három szempontot különítsünk el:
1. **Telepített CLI-verzió:** amit a saját gépeden a `terraform version` megjelenít.
2. **Projekt által elfogadott CLI-verzió:** amit a konfiguráció `required_version` beállítása megenged.
3. **Provider-verzió:** a külön letöltött provider plugin verziója, amelyet a `required_providers` és a lock fájl kezel.

Ezek nem ugyanazok. A provider nem a Terraform Core része, és a provider verziószáma nem a Terraform CLI verzióját jelöli.

### 2.2.2. A required_version szerepe

A `required_version` a `terraform` blokkon belül adja meg, hogy a konfiguráció milyen Terraform CLI-verziókkal használható.

```hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
```

A példa elemei:
- `terraform`: a Terraform saját beállításait tartalmazó blokk;
- `required_version`: a Terraform CLI-re vonatkozó verziókorlátozás;
- `>= 1.10.0`: legalább 1.10.0 verzió szükséges;
- `< 2.0.0`: a 2.0.0 vagy újabb főverzió nem elfogadott.

A két feltételt vessző választja el; mindkettőnek teljesülnie kell. Ez a korlátozás nem telepíti és nem frissíti a Terraformot. Azt ellenőrzi, hogy a futtatott CLI megfelel-e a projekt követelményének.

A verziókorlátozás megválasztása tudatos döntés. Ne másold át automatikusan a fenti tartományt minden projektbe: a ténylegesen használt Terraform-funkciókhoz és a csapat által tesztelt verziókhoz kell igazítani.

### 2.2.3. A provider verziója külön beállítás

A provider forrását és verziótartományát a `required_providers` blokkban deklaráljuk. Egy későbbi fejezetben részletesen foglalkozunk a providerekkel; itt a verziók közötti különbség megértése a cél.

```hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
```

A példa magyarázata:
- `required_providers`: a konfigurációhoz szükséges providereket sorolja fel;
- `local`: a konfigurációban használt lokális provider neve;
- `source`: a provider globális forráscíme; itt a HashiCorp által publikált `local` provider;
- `version`: a providerre vonatkozó verziókorlátozás.

A `~> 2.5` pessimistic constraint. A két komponensből álló alak a 2.5-ös vagy annál újabb, de 3.0.0 előtti verziókat engedi. A verziókorlátozások pontos jelentését mindig ellenőrizd, mielőtt egy projektben alkalmazod.

A `required_version` tehát a Terraform CLI-re, a `required_providers.version` pedig egy provider pluginra vonatkozik.
## 2.3. A Terraform CLI telepítése

A Terraform hivatalosan több operációs rendszerre és processzorarchitektúrára elérhető. A legkényelmesebb módszer általában az adott rendszer csomagkezelője. A kézi telepítésnél a hivatalos HashiCorp letöltési oldalról származó archívumot kell használni.

**Biztonsági alapelv:** végrehajtható programot csak megbízható forrásból tölts le. Ne használj ismeretlen weboldalról származó, újracsomagolt Terraform binárist.

### 2.3.1. Windows – hivatalos bináriscsomag

A Windows telepítési oldalon válaszd ki a rendszerednek megfelelő architektúrát. A legtöbb mai, Intel vagy AMD processzoros Windows számítógéphez az AMD64 csomag megfelelő. ARM-alapú Windows eszközhöz ARM64 kiadást válassz.
1. Nyisd meg a https://developer.hashicorp.com/terraform/install oldalt.
2. A Windows szekcióban válaszd ki a megfelelő architektúrát.
3. Töltsd le a ZIP archívumot a hivatalos HashiCorp letöltési oldalról.
4. Hozz létre egy állandó könyvtárat, például `C:\\Tools\\Terraform`.
5. Csomagold ki a ZIP fájlban található `terraform.exe` állományt ebbe a könyvtárba.
6. Add hozzá ezt a könyvtárat a felhasználói vagy rendszer-szintű `Path` környezeti változóhoz.
7. Nyiss egy új PowerShell ablakot, majd ellenőrizd a telepítést.

A `Path` környezeti változóban felsorolt könyvtárakban a shell külön elérési út megadása nélkül is megkeresi a futtatható programokat. A `Path` értékéhez a könyvtárat add hozzá, ne magát a `terraform.exe` fájlt.

PowerShellben az elérési út ellenőrizhető:

```powershell
$env:Path -split ';'
```

A parancs a pontosvesszők mentén külön sorokra bontja a `Path` értékét. Ha a Terraform könyvtára nem szerepel a listában, a környezeti változót javítani kell, majd új terminált kell nyitni.

Ellenőrzés:

```powershell
terraform version
terraform -help
```

Ha a PowerShell azt jelzi, hogy a `terraform` parancs nem található, ellenőrizd, hogy a bináris a megadott könyvtárban van-e, a könyvtár szerepel-e a `Path` változóban, új terminálablakot nyitottál-e, és nem hibás kiterjesztésű fájlt csomagoltál-e ki.

A WSL külön Linux-környezetet biztosít Windows alatt. Ha ezt választod, a Terraformot a WSL disztribúciójához telepítsd, és a WSL termináljából futtasd. Ne keverd össze a Windowsos `terraform.exe`-t a Linux binárissal.
### 2.3.3. macOS – Homebrew

macOS alatt a Homebrew csomagkezelővel telepíthető a Terraform.

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

Az első parancs hozzáadja a HashiCorp csomagtárolóját (tap) a Homebrew konfigurációjához. A második telepíti a Terraform csomagot ebből a tárolóból.

Ellenőrzés:

```bash
terraform version
terraform -help
```

Ha a `brew` parancs sem ismert, először a Homebrew telepítését és a shell konfigurációját kell rendbe tenni. A Homebrew telepítésének aktuális útmutatója: https://brew.sh/

### 2.3.4. Linux – Ubuntu és Debian

Ubuntu és Debian rendszeren a HashiCorp hivatalos APT csomagtárolója használható. Az alábbi parancsok a HashiCorp által dokumentált telepítési folyamatot követik.

```bash
wget -O - https://apt.releases.hashicorp.com/gpg \
  | sudo gpg --dearmor \
  -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform
```

A parancsok szerepe:
1. A `wget` letölti a HashiCorp csomagtárolójának nyilvános aláírókulcsát.
2. A `gpg --dearmor` a kulcsot az APT által használható formátumba alakítja.
3. A `dpkg --print-architecture` lekérdezi a rendszer csomagarchitektúráját.
4. Az `/etc/os-release` vagy az `lsb_release -cs` alapján a rendszer kiadásának kódneve kerül a repository beállításába.
5. A `tee` elmenti a csomagtároló címét a megfelelő APT konfigurációs fájlba.
6. Az `apt update` frissíti a csomaglistákat.
7. Az `apt install terraform` telepíti a Terraform csomagot.

A `sudo` rendszergazdai jogosultsággal futtatja az adott parancsot. Csak olyan gépen add meg ezeket a jogosultságokat, amelyet te kezelsz, és ahol érted a végrehajtott műveleteket.

A fenti repository-beállítás a HashiCorp dokumentációjában szereplő Ubuntu/Debian útmutatót követi. Régebbi vagy származtatott disztribúciókon a kiadás kódnevének feloldása eltérhet; ilyenkor a hivatalos telepítési oldalon ellenőrizd a saját disztribúciódra vonatkozó utasítást.

RHEL-alapú rendszereken és Fedora alatt a HashiCorp külön RPM repository-t biztosít. A pontos, disztribúcióhoz illő utasítást a hivatalos telepítési oldalon válaszd ki.
Manuális telepítésnél töltsd le a hivatalos oldalról a megfelelő verziójú, rendszerű és architektúrájú ZIP archívumot. Linux AMD64 példa:

```bash
unzip terraform_<VERSION>_linux_amd64.zip
sudo install terraform /usr/local/bin/terraform
```

A `<VERSION>` helyére a letöltött verzió kerül; ARM64 rendszerhez másik archívum szükséges. A célkönyvtár legyen a `PATH` változóban.

Ellenőrizd a HashiCorp checksum-lista digitális aláírását és az archívum SHA-256 értékét. A checksum egyezése önmagában nem igazolja a lista eredetét. Útmutató: https://developer.hashicorp.com/terraform/tutorials/cli/verify-archive
## 2.4. A telepítés ellenőrzése

Nyiss új terminált:

```bash
terraform version
terraform -help
```

Az első parancs a CLI verzióját, a második a súgót jeleníti meg. A `terraform plan -help` egy adott parancs opcióit mutatja meg. Több telepítés esetén a `Get-Command terraform -All` (PowerShell), illetve a `command -v terraform` (Linux/macOS) megmutatja a használt bináris helyét.
## 2.5. A szerkesztő és a Terraform-bővítmény beállítása

A példákhoz a Visual Studio Code használatát javasoljuk, de más szerkesztővel is dolgozhatsz. A lényeg, hogy a Terraform- és HCL-fájlokat egyszerű szövegként kezelje, és ne alakítsa át önkényesen a tartalmukat.

VS Code-ban telepítsd a hivatalos **HashiCorp Terraform** bővítményt: https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform

A bővítmény szintaxiskiemelést, IntelliSense-kiegészítést és diagnosztikát adhat. Provider-specifikus kiegészítésekhez szükség lehet a projekt inicializálására. A szerkesztő nem helyettesíti a CLI-ellenőrzéseket: a `fmt` formáz, a `validate` ellenőriz, a `plan` a tervezett változásokat mutatja be.
## 2.6. Projektmappa és fájlszerkezet

A Terraform egy munkakönyvtárban található konfigurációs fájlokat együtt dolgozza fel. A projektet ezért érdemes külön mappában tartani.

Például:

```text
terraform-learning/
├── README.md
├── .gitignore
├── versions.tf
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars.example
```

A fájlnevek szerepe:
- `README.md`: a projekt célját, használatát és előfeltételeit dokumentálja;
- `.gitignore`: megadja a Git számára, mely fájlokat és könyvtárakat ne kövesse;
- `versions.tf`: a Terraform- és provider-verziókövetelményeket tartalmazhatja;
- `main.tf`: a fő erőforrás-konfigurációk helye lehet;
- `variables.tf`: bemeneti változók deklarációi kerülhetnek ide;
- `outputs.tf`: a konfiguráció kimeneteit tartalmazhatja;
- `terraform.tfvars.example`: példaértékeket mutat, de nem tartalmaz valódi titkokat.

A Terraform nem követeli meg ezt a fájlelosztást. A `.tf` kiterjesztésű konfigurációs fájlokat a munkakönyvtárban együtt kezeli. A fájlok szétválasztása szervezési konvenció, amely nagyobb projektekben javítja az áttekinthetőséget.

### 2.6.1. A .terraform/ könyvtár

A `terraform init` létrehozza a helyi `.terraform/` könyvtárat, és ide töltheti le többek között a providereket és a modulokat. A könyvtár a munkakönyvtárhoz kapcsolódó inicializálási adatokat is tartalmazhat.

Ezt a könyvtárat általában nem tesszük verziókezelésbe. Másik gépen vagy friss klónban a `terraform init` újra elő tudja készíteni a szükséges helyi tartalmat.

### 2.6.2. A .terraform.lock.hcl fájl

A `.terraform.lock.hcl` a provider-függőségek kiválasztott verzióit és a csomagok ellenőrzőösszegeit rögzíti. A fájlt a Terraform CLI hozza létre és frissíti, amikor a `terraform init` során kiválasztja vagy módosítja a szükséges provider-verziókat. A felhasználónak nem kell kézzel megírnia.

A lock fájlt általában verziókezelésbe tesszük, hogy a csapat tagjai és a CI-környezet ugyanazokat a provider-verziókat használják. Nem azonos a `.terraform/` könyvtárral.

A lock fájl alapvetően a providerek kiválasztását rögzíti; nem helyettesíti a konfigurációban megadott verziókorlátozásokat. A `required_providers` azt határozza meg, milyen verziók elfogadhatók, a lock fájl pedig a kiválasztott verziót és annak ellenőrzőösszegeit tartja nyilván.

### 2.6.3. Példa .gitignore fájl

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
```

A `.gitignore` megadja a Gitnek, mely helyi fájlokat ne vegye fel a verziókezelésbe. A példában a Terraform munkakönyvtárát, az állapotfájlokat és a változófájlokat zárjuk ki. A `*.tfvars` szabályt a projekt igényeihez igazítsd: ha egy nem érzékeny változófájlt meg kell osztani, azt ne zárd ki.

A `.terraform.lock.hcl` fájlt ne zárd ki; ezt általában verziókezelésben tartjuk. A `.gitignore` csak a még nem követett fájlokra hat, a már verziókezelt fájlokat önmagában nem távolítja el a repositoryból.
## 2.7. Első projekt és biztonságos ellenőrzések

A példa a CLI és az ellenőrző parancsok kipróbálására szolgál. Nem hoz létre felhő-erőforrást, cloud credential nem szükséges.

### 2.7.1. Projektmappa

PowerShell:

```powershell
New-Item -ItemType Directory -Path terraform-learning
Set-Location terraform-learning
```

Linux/macOS:

```bash
mkdir terraform-learning
cd terraform-learning
```

### 2.7.2. versions.tf

```hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
```

A blokk a CLI elfogadható verziótartományát rögzíti; saját projektben a használt funkciókhoz és tesztelt verzióhoz igazítsd.

### 2.7.3. Ellenőrzések

A projekt gyökérkönyvtárában futtasd:

```bash
terraform fmt -check
terraform init -backend=false
terraform validate
```

A `fmt -check` ellenőrzi, de nem írja át a formázást. Az `init -backend=false` előkészíti a munkakönyvtárat backend inicializálása nélkül. A `validate` a konfigurációt ellenőrzi; előtte inicializálni kell a munkakönyvtárat.

A példa nem tartalmaz providert vagy erőforrást. A `plan` és `apply` használatát a következő fejezet tárgyalja.
## 2.8. Gyakori hibák és megoldásuk

| Jelenség | Ok és megoldás |
|---|---|
| A `terraform` parancs nem található | A bináris nincs a `PATH` változóban, vagy új terminált kell nyitni. Ellenőrizd a `Get-Command terraform -All` vagy `command -v terraform` kimenetét. |
| A CLI verziója nem felel meg a projektnek | A verzió kívül esik a `required_version` tartományán. Telepíts megfelelő verziót, vagy egyeztetve módosítsd a követelményt. |
| A bináris nem indul el | Nem megfelelő processzorarchitektúrájú csomagot töltöttél le. Válaszd az AMD64 vagy ARM64 kiadást. |
| Nem működik a kódkiegészítés | Ellenőrizd a bővítményt, a projekt gyökérkönyvtárát, és szükség esetén futtasd az `init` parancsot. |
| A `.terraform/` bekerül a Git változásai közé | Add hozzá a `.gitignore` fájlhoz. Ha a Git már követi, a kizárási szabály önmagában nem elég. |
| A lock fájl nem kerül commitba | Ellenőrizd, hogy nem zárja-e ki egy általános `.gitignore` minta. |
| A `validate` inicializálást jelez | Futtasd a `terraform init` parancsot, majd ismételd meg a validációt. |
## 2.9. Gyakorlófeladatok és teljes megoldások

A feladatok a fejezetben bemutatott ismereteket használják. A megoldások a feladatok után következnek; egyik feladat sem igényel a korábban ismertetett anyagon kívüli Terraform-fogalmat.

### 1. feladat – A telepített Terraform ellenőrzése

**Cél:** ellenőrizni, hogy a Terraform CLI elérhető-e a saját rendszereden.

**Előfeltétel:** a Terraform telepítése a 2.3. alfejezet alapján.

**Feladat:**
1. Nyiss egy új terminálablakot.
2. Futtasd a `terraform version` parancsot.
3. Futtasd a `terraform -help` parancsot.
4. Jegyezd fel a megjelenített verziószámot és azt, hogy a parancs sikeresen elindult-e.
5. Ha a parancs nem található, ellenőrizd a végrehajtható állomány elérési útját.

**Megoldás és indoklás:**

```bash
terraform version
terraform -help
```

PowerShellben ugyanezeket a parancsokat használhatod. A verziószám a telepített CLI-t azonosítja. A súgó megjelenése azt mutatja, hogy a shell megtalálta és el tudta indítani a Terraformot.

Ha a parancs nem található, Windows alatt a `Get-Command terraform -All`, Linux/macOS alatt a `command -v terraform` segít a hiba okának felderítésében. A megoldás a telepítés vagy a `PATH` beállításának javítása.

### 2. feladat – A Terraform CLI és a provider verziójának megkülönböztetése

**Cél:** felismerni, hogy a konfiguráció melyik beállítása melyik komponensre vonatkozik.

**Előfeltétel:** a 2.2. alfejezet ismerete.

**Feladat:** értelmezd az alábbi konfigurációt, és válaszolj a kérdésekre.

```hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
```
1. Melyik beállítás korlátozza a Terraform CLI verzióját?
2. Melyik beállítás adja meg a provider forrását?
3. Melyik verziókorlátozás vonatkozik a providerre?
4. A provider verziója azonos-e a Terraform CLI verziójával?

**Megoldás és indoklás:**
1. A `required_version` korlátozza a Terraform CLI verzióját.
2. A `source` adja meg a provider forráscímét: `hashicorp/local`.
3. A `required_providers.local.version` korlátozza a provider verzióját: `~> 2.5`.
4. Nem. A Terraform CLI és a provider külön komponens, ezért külön verziószámmal és külön verziókezeléssel rendelkeznek.

### 3. feladat – Projektmappa létrehozása

**Cél:** létrehozni egy rendezett Terraform-projektmappát.

**Előfeltétel:** a terminál alapvető használata és a 2.6. alfejezet.

**Feladat:**
1. Hozz létre egy `my-first-terraform-project` nevű mappát.
2. Lépj be a mappába.
3. Hozz létre benne egy `versions.tf` fájlt.
4. Írd bele az alábbi verziókorlátozást:

```hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
```
5. Hozz létre egy `.gitignore` fájlt, amely legalább a `.terraform/` könyvtárat és a state fájlokat kizárja.

**Megoldás és indoklás:**

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Path my-first-terraform-project
Set-Location my-first-terraform-project
New-Item -ItemType File -Path versions.tf
New-Item -ItemType File -Path .gitignore
```

Linux vagy macOS:

```bash
mkdir my-first-terraform-project
cd my-first-terraform-project
touch versions.tf .gitignore
```

A `versions.tf` tartalma:

```hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
```

A `.gitignore` minimális tartalma:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
```

A `versions.tf` a CLI követelményét rögzíti. A `.gitignore` megakadályozza, hogy a Git a helyi inicializálási könyvtárat és az állapotfájlokat automatikusan új fájlként kezelje. A tényleges projektben a 2.6.3. alfejezetben szereplő további kizárásokat is érdemes megfontolni.

### 4. feladat – Lock fájl vagy munkakönyvtár?

**Cél:** megkülönböztetni a Terraform helyi munkakönyvtárát a verziókezelésbe szánt lock fájltól.

**Előfeltétel:** a 2.6.1. és 2.6.2. alfejezet.

**Feladat:** döntsd el az alábbi állításokról, hogy igazak vagy hamisak.
1. A `.terraform/` könyvtárat általában verziókezelésbe tesszük.
2. A `.terraform.lock.hcl` a provider-verziók kiválasztását rögzíti.
3. A `.terraform.lock.hcl` fájlt általában commitoljuk.
4. A lock fájl és a `.terraform/` könyvtár ugyanazt a célt szolgálja.
5. A `terraform init` új munkakörnyezetben újra letöltheti a szükséges provider csomagokat.

**Megoldás és indoklás:**
1. **Hamis.** A `.terraform/` helyi inicializálási adatokat és letöltött komponenseket tartalmaz; általában kizárjuk a verziókezelésből.
2. **Igaz.** A lock fájl rögzíti a kiválasztott provider-verziókat és ellenőrzőösszegeket.
3. **Igaz.** A lock fájlt általában verziókezelésbe tesszük, hogy a csapat és a CI ugyanazokat a provider-verziókat használja.
4. **Hamis.** A munkakönyvtár helyi, letöltött adatokat tárol; a lock fájl a provider-függőségek kiválasztását rögzíti.
5. **Igaz.** A `terraform init` előkészíti a munkakönyvtárat, és szükség esetén letölti a providereket.

### 5. feladat – A minimális projekt ellenőrzése

**Cél:** végrehajtani a biztonságos, infrastruktúrát nem módosító alapellenőrzéseket.

**Előfeltétel:** a 2.7. alfejezet, működő Terraform CLI és a 3. feladatban létrehozott projektmappa.

**Feladat:**
1. Ellenőrizd, hogy a projekt gyökérkönyvtárában vagy-e.
2. Futtasd a `terraform fmt -check` parancsot.
3. Futtasd a `terraform init -backend=false` parancsot.
4. Futtasd a `terraform validate` parancsot.
5. Magyarázd el, hogy ezek közül melyik módosít infrastruktúrát.

**Megoldás és indoklás:**

```bash
terraform fmt -check
terraform init -backend=false
terraform validate
```

A `fmt -check` a formázást ellenőrzi, de nem írja át a fájlokat. Az `init -backend=false` előkészíti a munkakönyvtárat backend inicializálása nélkül. A `validate` a konfiguráció szintaktikai és belső konzisztenciáját ellenőrzi.

Ezek a parancsok a bemutatott minimális projektben nem hoznak létre felhő-erőforrást. A `terraform apply` ezzel szemben már infrastruktúra-változást hajthat végre, ezért nem része ennek a feladatnak.

A feladat megoldásakor a tényleges parancskimenetet is ellenőrizd. Ha valamelyik parancs hibát jelez, ne tekintsd sikeresnek az ellenőrzést pusztán azért, mert a parancs elindult.

### 6. feladat – Szerkesztői és CLI-ellenőrzés összehasonlítása

**Cél:** megérteni, miért nem elegendő kizárólag a szerkesztő hibajelzéseire hagyatkozni.

**Előfeltétel:** a 2.5.2. alfejezet.

**Feladat:** párosítsd az eszközt a hozzá tartozó szereppel.

A) VS Code Terraform bővítmény  
B) `terraform fmt`  
C) `terraform validate`  
D) `terraform plan`
1. A konfiguráció formázását végzi.
2. Gépelés közben kiegészítéseket és diagnosztikai jelzéseket adhat.
3. A konfiguráció szintaktikai és belső konzisztenciáját ellenőrzi.
4. Megmutatja a tervezett infrastruktúra-változásokat.

**Megoldás és indoklás:**
- A → 2. A bővítmény szerkesztés közben segíti a munkát.
- B → 1. A `fmt` a konfiguráció formázását végzi.
- C → 3. A `validate` a konfiguráció helyességét ellenőrzi.
- D → 4. A `plan` a tervezett változásokat mutatja meg.

A négy eszköz különböző feladatot lát el. Egyik sem helyettesíti automatikusan a másikat.
## 2.10. Összefoglalás

Ebben a fejezetben kialakítottuk a Terraform használatához szükséges alapvető fejlesztői környezetet.

A legfontosabb tudnivalók:
- A Terraform CLI-t a hivatalos HashiCorp telepítési oldalról, vagy az ott dokumentált csomagkezelővel telepítsd.
- A telepítés után a `terraform version` paranccsal ellenőrizd a CLI verzióját.
- A `required_version` a Terraform CLI verzióját korlátozza.
- A `required_providers` blokk a provider forrását és verziótartományát deklarálja.
- A Terraform CLI, a provider és a szerkesztőbővítmény külön komponensek.
- A `.terraform/` helyi munkakönyvtárat általában kizárjuk a Gitből.
- A `.terraform.lock.hcl` fájlt általában verziókezelésbe tesszük.
- A `terraform fmt`, `terraform init` és `terraform validate` a fejlesztési folyamat fontos ellenőrző eszközei.
- A szerkesztő támogatása hasznos, de nem helyettesíti a Terraform CLI ellenőrzéseit.
- Felhőfiókra és cloud credentialre a Terraform telepítéséhez, illetve az ebben a fejezetben szereplő minimális példához nincs szükség.

### Tanulási ellenőrzőpont

A következő fejezet előtt ellenőrizd, hogy önállóan meg tudod-e válaszolni az alábbi kérdéseket:
1. Mi a különbség a Terraform CLI és a provider között?
2. Melyik beállítás korlátozza a Terraform CLI verzióját?
3. Miért érdemes a `.terraform.lock.hcl` fájlt verziókezelésben tartani?
4. Miért nem tesszük általában verziókezelésbe a `.terraform/` könyvtárat?
5. Miben különbözik a `terraform fmt`, a `terraform validate` és a `terraform plan` szerepe?
6. Miért nem bizonyítja a szerkesztő hibajelzésének hiánya, hogy a Terraform-konfiguráció végrehajtható?

Ha ezekre a kérdésekre saját szavaiddal tudsz válaszolni, és a minimális projektet hiba nélkül ellenőrizted, készen állsz a következő fejezetre.
## Hivatalos források és további olvasnivaló

A telepítési utasítások és a CLI viselkedése idővel változhatnak. Minden telepítés előtt ellenőrizd az aktuális hivatalos dokumentációt.
1. **Terraform telepítése:** https://developer.hashicorp.com/terraform/install
2. **Terraform CLI dokumentáció:** https://developer.hashicorp.com/terraform/cli
3. **Terraform CLI konfigurációs fájl:** https://developer.hashicorp.com/terraform/cli/config/config-file
4. **Terraform CLI környezeti változók:** https://developer.hashicorp.com/terraform/cli/config/environment-variables
5. **Terraform inicializálás:** https://developer.hashicorp.com/terraform/tutorials/cli/init
6. **Terraform bináris archívumok ellenőrzése:** https://developer.hashicorp.com/terraform/tutorials/cli/verify-archive
7. **HashiCorp Terraform bővítmény VS Code-hoz:** https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform
