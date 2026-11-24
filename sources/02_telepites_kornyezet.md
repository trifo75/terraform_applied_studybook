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

---

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

### 2.1.2. Terminál vagy shell

A Terraform parancsokat terminálból futtatjuk. A terminál egy felület, a shell pedig az a parancsértelmező, amely a beírt utasításokat feldolgozza.

Gyakori shell-ek:
- Windows: PowerShell; emellett használható Windows Subsystem for Linux (WSL) is;
- macOS: zsh az alapértelmezett shell sok újabb telepítésen, de bash is használható;
- Linux: gyakori a bash, de több más shell is elérhető.

A parancspéldáknál mindig feltüntetjük, melyik környezetre vonatkoznak. A PowerShell és a bash/zsh szintaxisa nem minden esetben cserélhető fel egymással.

### 2.1.3. Kódszerkesztő

A Terraform-konfigurációk egyszerű szövegfájlok, ezért bármely megfelelő szövegszerkesztővel módosíthatók. A Terraform-specifikus támogatás azonban kényelmesebbé teszi a munkát.

Példaként a Visual Studio Code (VS Code) és a HashiCorp által kiadott Terraform bővítmény használható. A bővítmény többek között szintaxiskiemelést, kódkiegészítést, navigációt, formázási támogatást és diagnosztikai jelzéseket ad.

Fontos különbség: a szerkesztő bővítménye nem maga a Terraform CLI. A bővítmény nem helyettesíti a terminálból futtatott `terraform fmt`, `terraform validate` vagy `terraform plan` parancsokat.

### 2.1.4. Verziókezelés

A Git használata erősen ajánlott. A Terraform-konfigurációk forráskódok: érdemes követni a változásaikat, visszakeresni, ki és mikor módosított egy fájlt, és együtt dolgozni másokkal.

A Git nem a Terraform része, és a Terraform telepítéséhez sem kötelező. A későbbi közös munkához, kódellenőrzéshez és automatizáláshoz azonban alapvető eszköz.

### 2.1.5. Felhőfiók és hitelesítő adatok

A Terraform tanulásának kezdetén nem szükséges felhőfiók vagy cloud credential. A konfigurációk egy része helyi, külső infrastruktúrát nem módosító példákkal is ellenőrizhető.

Amikor később felhőszolgáltatói providert használunk, külön kell gondoskodni a hitelesítésről és a megfelelő jogosultságokról. A hozzáférési adatokat nem szabad a Terraform-konfigurációba, Git-repositoryba vagy nyilvános példába beírni.

---

## 2.2. Terraform-verziók és verziókorlátozás

### 2.2.1. Miért fontos a verzió?

A Terraform folyamatosan fejlődik. Egy újabb kiadás javításokat, új funkciókat vagy viselkedésbeli változásokat tartalmazhat. Emiatt nem szerencsés, ha egy csapat tagjai vagy a CI-rendszer egymástól eltérő, dokumentálatlan Terraform-verzióval dolgoznak.

A hivatalos HashiCorp telepítési oldal ellenőrzésekor elérhető stabil kiadás a **Terraform 1.17.1** volt. Ez a szám a dokumentum készítésekor ellenőrzött referencia, nem örök érvényű követelmény. A telepítés előtt mindig nézd meg a hivatalos oldalon az aktuális kiadást és az esetleges támogatási közleményeket.

Hivatalos forrás: https://developer.hashicorp.com/terraform/install

A verzió kiválasztásakor három szempontot különítsünk el:

1. **Telepített CLI-verzió:** amit a saját gépeden a `terraform version` megjelenít.
2. **Projekt által elfogadott CLI-verzió:** amit a konfiguráció `required_version` beállítása megenged.
3. **Provider-verzió:** a külön letöltött provider plugin verziója, amelyet a `required_providers` és a lock fájl kezel.

Ezek nem ugyanazok. A provider nem a Terraform Core része, és a provider verziószáma nem a Terraform CLI verzióját jelöli.

### 2.2.2. A required_version szerepe

A `required_version` a `terraform` blokkon belül adja meg, hogy a konfiguráció milyen Terraform CLI-verziókkal használható.

\`\`\`hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
\`\`\`

A példa elemei:
- `terraform`: a Terraform saját beállításait tartalmazó blokk;
- `required_version`: a Terraform CLI-re vonatkozó verziókorlátozás;
- `>= 1.10.0`: legalább 1.10.0 verzió szükséges;
- `< 2.0.0`: a 2.0.0 vagy újabb főverzió nem elfogadott.

A két feltételt vessző választja el; mindkettőnek teljesülnie kell. Ez a korlátozás nem telepíti és nem frissíti a Terraformot. Azt ellenőrzi, hogy a futtatott CLI megfelel-e a projekt követelményének.

A verziókorlátozás megválasztása tudatos döntés. Ne másold át automatikusan a fenti tartományt minden projektbe: a ténylegesen használt Terraform-funkciókhoz és a csapat által tesztelt verziókhoz kell igazítani.

### 2.2.3. A provider verziója külön beállítás

A provider forrását és verziótartományát a `required_providers` blokkban deklaráljuk. Egy későbbi fejezetben részletesen foglalkozunk a providerekkel; itt a verziók közötti különbség megértése a cél.

\`\`\`hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
\`\`\`

A példa magyarázata:
- `required_providers`: a konfigurációhoz szükséges providereket sorolja fel;
- `local`: a konfigurációban használt lokális provider neve;
- `source`: a provider globális forráscíme; itt a HashiCorp által publikált `local` provider;
- `version`: a providerre vonatkozó verziókorlátozás.

A `~> 2.5` pessimistic constraint. A két komponensből álló alak a 2.5-ös vagy annál újabb, de 3.0.0 előtti verziókat engedi. A verziókorlátozások pontos jelentését mindig ellenőrizd, mielőtt egy projektben alkalmazod.

A `required_version` tehát a Terraform CLI-re, a `required_providers.version` pedig egy provider pluginra vonatkozik.

---

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

\`\`\`powershell
$env:Path -split ';'
\`\`\`

A parancs a pontosvesszők mentén külön sorokra bontja a `Path` értékét. Ha a Terraform könyvtára nem szerepel a listában, a környezeti változót javítani kell, majd új terminált kell nyitni.

Ellenőrzés:

\`\`\`powershell
terraform version
terraform -help
\`\`\`

Ha a PowerShell azt jelzi, hogy a `terraform` parancs nem található, ellenőrizd, hogy a bináris a megadott könyvtárban van-e, a könyvtár szerepel-e a `Path` változóban, új terminálablakot nyitottál-e, és nem hibás kiterjesztésű fájlt csomagoltál-e ki.

### 2.3.2. Windows – WSL használata

Ha Windows alatt Linux-alapú fejlesztői környezetet szeretnél, használhatod a Windows Subsystem for Linux (WSL) szolgáltatást. Ebben az esetben a Terraformot a WSL-en belüli Linux disztribúcióhoz telepíted, és a Linux terminálban futtatod.

A WSL külön Linux-környezetet ad, saját fájlrendszerrel, csomagkezelővel és shell-lel. Ha ezt az utat választod, a későbbi parancsoknál következetesen a WSL terminálját használd. Ne keverd össze a Windowsra telepített `terraform.exe`-t a WSL-ben telepített Linux binárissal.

### 2.3.3. macOS – Homebrew

macOS alatt a Homebrew csomagkezelővel telepíthető a Terraform.

\`\`\`bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
\`\`\`

Az első parancs hozzáadja a HashiCorp csomagtárolóját (tap) a Homebrew konfigurációjához. A második telepíti a Terraform csomagot ebből a tárolóból.

Ellenőrzés:

\`\`\`bash
terraform version
terraform -help
\`\`\`

Ha a `brew` parancs sem ismert, először a Homebrew telepítését és a shell konfigurációját kell rendbe tenni. A Homebrew telepítésének aktuális útmutatója: https://brew.sh/

### 2.3.4. Linux – Ubuntu és Debian

Ubuntu és Debian rendszeren a HashiCorp hivatalos APT csomagtárolója használható. Az alábbi parancsok a HashiCorp által dokumentált telepítési folyamatot követik.

\`\`\`bash
wget -O - https://apt.releases.hashicorp.com/gpg \\
  | sudo gpg --dearmor \\
  -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" \\
  | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform
\`\`\`

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

### 2.3.5. Linux – RHEL-alapú rendszerek és Fedora

RHEL-alapú rendszereken a HashiCorp RPM repository használható. A pontos parancsok disztribúciótól és verziótól függhetnek.

RHEL példa:

\`\`\`bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform
\`\`\`

Fedora példa:

\`\`\`bash
wget -O- https://rpm.releases.hashicorp.com/fedora/hashicorp.repo \\
  | sudo tee /etc/yum.repos.d/hashicorp.repo

sudo dnf -y install terraform
\`\`\`

A `yum` és a `dnf` RPM-alapú rendszerek csomagkezelő eszközei. A `yum-config-manager` repository-beállítást ad hozzá, a `dnf install` pedig telepíti a csomagot.

Ne használd automatikusan a RHEL-re megadott repository-t Fedora rendszeren vagy fordítva. Mindig a saját disztribúciódhoz és verziójához tartozó hivatalos utasítást válaszd.

### 2.3.6. Linux – manuális bináristelepítés

Ha a disztribúciódhoz nem érhető el megfelelő csomag, vagy a csomagkezelő helyett egy konkrét Terraform-verziót szeretnél telepíteni, használhatod a hivatalos ZIP archívumot.

A folyamat általános lépései:
1. A HashiCorp telepítési oldalán válaszd ki a kívánt verziót és a megfelelő operációs rendszer/processzor kombinációt.
2. Töltsd le az archívumot.
3. Ellenőrizd a letöltés hitelességét és sértetlenségét.
4. Csomagold ki a `terraform` binárist.
5. Helyezd egy olyan könyvtárba, amely szerepel a `PATH` környezeti változóban, például `/usr/local/bin`.
6. Ellenőrizd a verziót.

Példa a kicsomagolásra, ha az archívum már a jelenlegi könyvtárban van:

\`\`\`bash
unzip terraform_<VERSION>_linux_amd64.zip
sudo install terraform /usr/local/bin/terraform
\`\`\`

A `<VERSION>` helyére a ténylegesen letöltött verziót kell írni; a szögletes zárójel itt helykitöltő, nem szó szerint beírandó érték. Az `install` parancs a binárist a célhelyre másolja, és a célfájl jogosultságait is beállítja.

A példa AMD64 architektúrájú Linux rendszerhez készült. ARM64 vagy más architektúra esetén a megfelelő archívumot kell letölteni. A `/usr/local/bin` használata gyakori, de a célkönyvtárnak léteznie kell, és a `PATH` változóban elérhetőnek kell lennie.

### 2.3.7. A bináris ellenőrzése

A telepítés előtt különösen manuális letöltés esetén érdemes ellenőrizni, hogy az archívum valóban a HashiCorp által közzétett állomány-e, és a letöltés nem sérült-e meg.

A HashiCorp a kiadások mellett SHA-256 ellenőrzőösszeg-fájlt és annak digitális aláírását is közzéteszi. A biztonságos ellenőrzés logikája:
1. Töltsd le az adott Terraform-verzió archívumát.
2. Töltsd le ugyanahhoz a verzióhoz a `SHA256SUMS` és `SHA256SUMS.sig` fájlokat.
3. A HashiCorp hivatalos biztonsági oldalán ellenőrizd a nyilvános PGP-kulcs ujjlenyomatát.
4. Importáld a nyilvános kulcsot a GPG kulcstárába.
5. Ellenőrizd a checksum fájl aláírását.
6. Ellenőrizd a letöltött archívum SHA-256 értékét a checksum fájl alapján.

A checksum egyezése önmagában csak azt mutatja, hogy a letöltött fájl megegyezik az ellenőrzőösszeg-fájlban megadottal. A checksum fájl digitális aláírásának ellenőrzése segít megbizonyosodni arról, hogy az ellenőrzőösszeg-listát valóban a HashiCorp írta alá.

A részletes, aktuális folyamatot a HashiCorp hivatalos útmutatója ismerteti: https://developer.hashicorp.com/terraform/tutorials/cli/verify-archive

A csomagkezelő által kezelt telepítéseknél a repository aláírásának és a csomagkezelő bizalmi láncának szerepe is fontos. A biztonsági ellenőrzést nem szabad egy tetszőleges, harmadik féltől származó checksum weboldalra alapozni.

---

## 2.4. A telepítés ellenőrzése és a CLI súgója

A telepítés után nyiss új terminálablakot, és futtasd:

\`\`\`bash
terraform version
\`\`\`

A kimenet megmutatja a Terraform CLI telepített verzióját. A pontos szöveg a kiadástól függ, ezért ne várj minden gépen karakterre pontosan azonos kimenetet.

A következő parancs az általános súgót jeleníti meg:

\`\`\`bash
terraform -help
\`\`\`

Egy konkrét parancs részletes súgója például:

\`\`\`bash
terraform plan -help
\`\`\`

A `-help` kapcsolóval a parancs argumentumait és opcióit ismerheted meg. Ez különösen hasznos, mert a CLI súgója a ténylegesen telepített verzióhoz tartozik.

### 2.4.1. Melyik Terraform fut le valójában?

Előfordulhat, hogy egy gépen több Terraform-bináris is megtalálható. Ilyenkor nem biztos, hogy az a verzió fut le, amelyet legutóbb telepítettél.

Windows PowerShell:

\`\`\`powershell
Get-Command terraform -All
\`\`\`

Linux vagy macOS:

\`\`\`bash
command -v terraform
which -a terraform
\`\`\`

Ezek segítenek megállapítani, hogy a shell honnan tölti be a `terraform` parancsot. Ha több példány is szerepel az útvonalon, ellenőrizd a sorrendet, és távolítsd el vagy rendezd a felesleges bejegyzéseket.

### 2.4.2. Tab completion

A Terraform CLI támogatja a parancsok és bizonyos argumentumok shell-szintű kiegészítését Bash és Zsh alatt.

\`\`\`bash
terraform -install-autocomplete
\`\`\`

A parancs telepíti az automatikus kiegészítéshez szükséges shell-beállítást. A módosítás érvényesítéséhez indíts új terminált, vagy töltsd újra a shell konfigurációs fájlját.

A kiegészítés nem szükséges a Terraform működéséhez; kényelmi funkció.

---

## 2.5. A szerkesztő és a Terraform-bővítmény beállítása

A példákhoz a Visual Studio Code használatát javasoljuk, de más szerkesztővel is dolgozhatsz. A lényeg, hogy a Terraform- és HCL-fájlokat egyszerű szövegként kezelje, és ne alakítsa át önkényesen a tartalmukat.

### 2.5.1. A HashiCorp Terraform bővítmény

VS Code-ban nyisd meg az Extensions nézetet, keress rá a **HashiCorp Terraform** bővítményre, majd telepítsd a hivatalos kiadótól származó bővítményt.

Hivatalos bővítményoldal: https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform

A bővítmény többek között az alábbi képességeket kínálhatja:
- HCL és Terraform szintaxiskiemelés;
- IntelliSense és kódkiegészítés;
- definícióra ugrás és kódnavigáció;
- formázási támogatás;
- diagnosztikai jelzések;
- Terraform-parancsok indítása a szerkesztőből.

A bővítmény Terraform Language Servert is használ. A provider-specifikus kiegészítésekhez a projekt inicializálása és a provider sémájának letöltése szükséges lehet. Ezért egy új projekt megnyitásakor a szerkesztő nem feltétlenül ismer minden provider-erőforrást azonnal.

### 2.5.2. Formázás és validáció: ki miért felel?

Három különböző ellenőrzési szintet érdemes megkülönböztetni:

1. **Szerkesztői támogatás:** gépelés közben segít észrevenni a hibákat, de a jelzései nem minden esetben véglegesek.
2. **Terraform CLI ellenőrzések:** a `terraform fmt` és `terraform validate` a Terraform saját eszközei.
3. **Végrehajtási terv:** a `terraform plan` a konfiguráció, a state és a provider információi alapján megmutatja a tervezett infrastruktúra-változásokat.

Ezek nem helyettesítik egymást. A zöld szerkesztői jelzés nem bizonyítja, hogy a konfiguráció sikeresen inicializálható vagy végrehajtható.

---

## 2.6. Projektmappa és fájlszerkezet

A Terraform egy munkakönyvtárban található konfigurációs fájlokat együtt dolgozza fel. A projektet ezért érdemes külön mappában tartani.

Például:

\`\`\`text
terraform-learning/
├── README.md
├── .gitignore
├── versions.tf
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars.example
\`\`\`

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

A `.terraform.lock.hcl` a provider-függőségek kiválasztott verzióit és a csomagok ellenőrzőösszegeit rögzíti. Ez segít abban, hogy a csapat tagjai és a CI-környezet ugyanazokat a provider-verziókat használják.

A fájlt általában verziókezelésbe kell tenni. Nem azonos a `.terraform/` könyvtárral, és nem szabad összekeverni a kettőt.

A lock fájl alapvetően a providerek kiválasztását rögzíti; nem helyettesíti a konfigurációban megadott verziókorlátozásokat. A `required_providers` azt határozza meg, milyen verziók elfogadhatók, a lock fájl pedig a kiválasztott verziót és annak ellenőrzőösszegeit tartja nyilván.

### 2.6.3. Példa .gitignore fájl

\`\`\`gitignore
# Terraform által létrehozott helyi munkakönyvtár
.terraform/

# Lokális változófájlok – csak a ténylegesen titkos vagy gépspecifikus fájlokat zárd ki
*.tfvars
*.tfvars.json

# Állományok, amelyek érzékeny adatot tartalmazhatnak
*.tfstate
*.tfstate.*
*.tfplan

# Összeomlási napló
crash.log
crash.*.log

# A lock fájlt szándékosan NEM zárjuk ki:
# .terraform.lock.hcl
\`\`\`

A `.gitignore` a Git viselkedését szabályozza, nem a Terraformét. A minták a Git által követésbe még fel nem vett fájlokra hatnak; egy korábban már követett fájlt a `.gitignore` önmagában nem vesz ki a repositoryból.

A `*.tfvars` kizárása gyakori, mert ilyen fájlokban lehetnek környezetspecifikus értékek vagy titkok. Ha a projektnek vannak olyan nem érzékeny, megosztandó tfvars fájljai, amelyeknek verziókezelésben kell lenniük, a kizárási szabályt tudatosan kell kialakítani. A példa nem univerzális szabály minden repositoryhoz.

A `.terraform.lock.hcl` fájlt ezzel szemben általában commitoljuk. A provider-verziók és ellenőrzőösszegek reprodukálhatósága miatt fontos.

### 2.6.4. Fájlok elnevezése

A Terraform a munkakönyvtárban található `.tf` és `.tf.json` konfigurációs fájlokat dolgozza fel. A `main.tf`, `variables.tf` és `outputs.tf` elnevezés konvenció, nem kötelező nyelvi szabály.

A `versions.tf` fájl is csak szervezési elnevezés. A benne található `terraform` blokk ugyanúgy működik, mintha a `main.tf` fájlban szerepelne.

---

## 2.7. Első projekt létrehozása és biztonságos ellenőrzése

Most létrehozunk egy minimális Terraform-projektet. A példa célja a telepítés, a projektmappa, a verziókorlátozás és az ellenőrző parancsok kipróbálása. Nem hozunk létre felhő-erőforrást, és nem szükséges hozzá cloud credential.

### 2.7.1. Projektmappa létrehozása

Hozz létre egy `terraform-learning` nevű könyvtárat, majd lépj be:

Windows PowerShell:

\`\`\`powershell
New-Item -ItemType Directory -Path terraform-learning
Set-Location terraform-learning
\`\`\`

macOS vagy Linux:

\`\`\`bash
mkdir terraform-learning
cd terraform-learning
\`\`\`

A `mkdir` könyvtárat hoz létre, a `cd` pedig belép az adott könyvtárba. PowerShellben a `Set-Location` ugyanezt a szerepet tölti be.

### 2.7.2. versions.tf létrehozása

Hozz létre egy `versions.tf` nevű fájlt a következő tartalommal:

\`\`\`hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
\`\`\`

A blokk csak a Terraform CLI-re vonatkozó verziótartományt rögzíti. A fejezetben bemutatott tartomány oktatási példa; a projektben mindig az alkalmazott funkciókhoz és a csapat által tesztelt verziókhoz igazítsd.

### 2.7.3. Formázás

Futtasd:

\`\`\`bash
terraform fmt
\`\`\`

A parancs a Terraform-konfigurációs fájlokat a Terraform által elvárt formára rendezi. A formázás nem módosít infrastruktúrát.

Ha csak ellenőrizni szeretnéd, hogy szükséges-e formázási módosítás, használd:

\`\`\`bash
terraform fmt -check
\`\`\`

A `-check` kapcsolóval a parancs nem írja át a fájlokat; hibajelzést ad, ha formázási eltérést talál.

### 2.7.4. Inicializálás backend nélkül

A minimális projektben nincs provider és nincs külön backend-beállítás. A validációs folyamat előkészítéséhez futtasd:

\`\`\`bash
terraform init -backend=false
\`\`\`

A `terraform init` előkészíti a munkakönyvtárat. A `-backend=false` kapcsoló azt kéri, hogy a Terraform ne inicializálja a konfiguráció backendjét. Ez a példa nem állít be remote state-et.

A parancs létrehozhatja a `.terraform/` könyvtárat és a működéshez szükséges helyi metaadatokat.

### 2.7.5. Validáció

Ezután:

\`\`\`bash
terraform validate
\`\`\`

A `validate` ellenőrzi, hogy a konfiguráció szintaktikailag helyes-e és belsőleg konzisztens-e. A parancs futtatása előtt a munkakönyvtárat inicializálni kell.

Ebben a minimális példában nincs provider és nincs resource blokk, ezért a validáció egyszerű. A későbbi, providerre épülő konfigurációk esetén az inicializálás provider-letöltést is igényelhet.

### 2.7.6. Miért nem futtatunk még plan vagy apply parancsot?

A `terraform plan` a konfigurációból és az állapotból végrehajtási tervet készít. Az `apply` a terv alapján módosíthatja az infrastruktúrát.

Ebben a fejezetben a cél a fejlesztői környezet kialakítása, nem az infrastruktúra létrehozása. Ezért nincs szükség `terraform apply` futtatására. A következő fejezetben készítjük el az első teljes Terraform-projektet, és ott részletesen áttekintjük a CLI-munkafolyamatot.

---

## 2.8. Gyakori hibák és megoldásuk

### 2.8.1. A terraform parancs nem található

**Jelenség:** a shell olyan üzenetet jelenít meg, amely szerint a `terraform` parancs nem ismert vagy nem található.

**Lehetséges okok:**
- a Terraform nincs telepítve;
- a bináris könyvtára nincs a `PATH` változóban;
- a telepítés után nem nyitottál új terminálablakot;
- több Terraform-példány közül nem a megfelelő található meg először.

**Megoldás:** ellenőrizd a bináris helyét és a `PATH` értékét, majd vizsgáld meg a `Get-Command terraform -All` (PowerShell), illetve a `command -v terraform` (Linux/macOS) kimenetét.

### 2.8.2. A CLI verziója nem felel meg a projekt követelményének

**Jelenség:** a Terraform jelzi, hogy a futtatott verzió nem felel meg a konfiguráció `required_version` korlátozásának.

**Ok:** a telepített Terraform CLI a megadott verziótartományon kívül esik.

**Megoldás:** telepíts olyan verziót, amely megfelel a projekt követelményének, vagy a projekt felelős karbantartóival egyeztetve módosítsd a korlátozást. Ne lazítsd a verziófeltételt pusztán azért, hogy a hibaüzenet eltűnjön.

### 2.8.3. Nem megfelelő processzorarchitektúrájú binárist töltöttél le

**Jelenség:** a program nem indul el, vagy az operációs rendszer nem tudja futtatni.

**Ok:** például ARM64 rendszerre AMD64 binárist töltöttél le.

**Megoldás:** ellenőrizd az operációs rendszer és a processzor architektúráját, majd töltsd le a megfelelő csomagot a HashiCorp hivatalos oldaláról.

### 2.8.4. A szerkesztő nem ismeri fel a Terraform-fájlt

**Jelenség:** nincs HCL-szintaxiskiemelés vagy kódkiegészítés.

**Lehetséges okok:**
- a Terraform-bővítmény nincs telepítve vagy engedélyezve;
- nem a projekt gyökérkönyvtárát nyitottad meg;
- a fájl kiterjesztése nem `.tf`;
- az új projekt még nincs inicializálva, ezért a provider-séma nem érhető el.

**Megoldás:** ellenőrizd a bővítményt, nyisd meg a projektmappát, és szükség esetén futtasd a `terraform init` parancsot. A szerkesztő jelzéseit mindig ellenőrizd a CLI eszközeivel is.

### 2.8.5. A .terraform könyvtár bekerült a Git repositoryba

**Jelenség:** a Git változásként mutatja a `.terraform/` tartalmát.

**Ok:** hiányzik a megfelelő `.gitignore` szabály, vagy a könyvtárat már korábban hozzáadták a Git követéséhez.

**Megoldás:** vedd fel a `.terraform/` mintát a `.gitignore` fájlba. Ha a könyvtárat a Git már követi, a kizárási szabály önmagában nem elég; a követésből külön el kell távolítani úgy, hogy a helyi fájlok megmaradjanak. Ezt körültekintően, a Git működésének ismeretében végezd el.

### 2.8.6. A .terraform.lock.hcl fájlt kizártad

**Jelenség:** a lock fájl nem jelenik meg a Git változásai között, vagy nem kerül be a commitba.

**Ok:** túl általános kizárási szabályt használsz, amely a lock fájlt is elrejti.

**Megoldás:** ellenőrizd a `.gitignore` mintáit, és gondoskodj róla, hogy a `.terraform.lock.hcl` verziókezelésbe kerüljön. A lock fájl a reprodukálható provider-verzióválasztás fontos része.

### 2.8.7. A validate inicializálás nélkül fut

**Jelenség:** a `terraform validate` jelzi, hogy a konfiguráció nincs inicializálva, vagy hiányzik a provider telepítése.

**Ok:** a munkakönyvtárban még nem futott le a szükséges `terraform init`.

**Megoldás:** futtasd a `terraform init` parancsot a projekt gyökerében, majd ismételd meg a validációt. A provideres példákhoz internetkapcsolat és a provider csomag letöltéséhez megfelelő hozzáférés is szükséges lehet.

---

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

\`\`\`bash
terraform version
terraform -help
\`\`\`

PowerShellben ugyanezeket a parancsokat használhatod. A verziószám a telepített CLI-t azonosítja. A súgó megjelenése azt mutatja, hogy a shell megtalálta és el tudta indítani a Terraformot.

Ha a parancs nem található, Windows alatt a `Get-Command terraform -All`, Linux/macOS alatt a `command -v terraform` segít a hiba okának felderítésében. A megoldás a telepítés vagy a `PATH` beállításának javítása.

### 2. feladat – A Terraform CLI és a provider verziójának megkülönböztetése

**Cél:** felismerni, hogy a konfiguráció melyik beállítása melyik komponensre vonatkozik.

**Előfeltétel:** a 2.2. alfejezet ismerete.

**Feladat:** értelmezd az alábbi konfigurációt, és válaszolj a kérdésekre.

\`\`\`hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
\`\`\`

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

\`\`\`hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
\`\`\`

5. Hozz létre egy `.gitignore` fájlt, amely legalább a `.terraform/` könyvtárat és a state fájlokat kizárja.

**Megoldás és indoklás:**

Windows PowerShell:

\`\`\`powershell
New-Item -ItemType Directory -Path my-first-terraform-project
Set-Location my-first-terraform-project
New-Item -ItemType File -Path versions.tf
New-Item -ItemType File -Path .gitignore
\`\`\`

Linux vagy macOS:

\`\`\`bash
mkdir my-first-terraform-project
cd my-first-terraform-project
touch versions.tf .gitignore
\`\`\`

A `versions.tf` tartalma:

\`\`\`hcl
terraform {
  required_version = ">= 1.10.0, < 2.0.0"
}
\`\`\`

A `.gitignore` minimális tartalma:

\`\`\`gitignore
.terraform/
*.tfstate
*.tfstate.*
\`\`\`

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

\`\`\`bash
terraform fmt -check
terraform init -backend=false
terraform validate
\`\`\`

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

---

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
- A szerkesztői támogatás hasznos, de nem helyettesíti a Terraform CLI ellenőrzéseit.
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

---

## Hivatalos források és további olvasnivaló

A telepítési utasítások és a CLI viselkedése idővel változhatnak. Minden telepítés előtt ellenőrizd az aktuális hivatalos dokumentációt.

1. **Terraform telepítése:** https://developer.hashicorp.com/terraform/install
2. **Terraform CLI dokumentáció:** https://developer.hashicorp.com/terraform/cli
3. **Terraform CLI konfigurációs fájl:** https://developer.hashicorp.com/terraform/cli/config/config-file
4. **Terraform CLI környezeti változók:** https://developer.hashicorp.com/terraform/cli/config/environment-variables
5. **Terraform inicializálás:** https://developer.hashicorp.com/terraform/tutorials/cli/init
6. **Terraform bináris archívumok ellenőrzése:** https://developer.hashicorp.com/terraform/tutorials/cli/verify-archive
7. **HashiCorp Terraform bővítmény VS Code-hoz:** https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform
