# 3. Terraform CLI és az első teljes Terraform-projekt

*A munkakönyvtár, init-plan-apply ciklus és erőforrás-életciklus*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform egy munkakönyvtárból dolgozik. A konfiguráció több `.tf` fájlra bontható, de ugyanazon könyvtár gyökérmoduljának részeként együtt értelmeződik. A fájlnevek elsősorban szervezési célúak; a Terraform nem hajtja végre őket lexikografikus sorrendben.

Az alap munkafolyamat: konfiguráció megírása, `init`, `fmt`, `validate`, `plan`, majd ellenőrzés után `apply`. Az `init` inicializálja a backend és provider környezetet. A `plan` a konfiguráció, a state és a provider által lekérdezett aktuális adatok alapján javasol változtatásokat. Az `apply` végrehajtja a tervet, és frissíti a state-et.

A `destroy` a konfigurációhoz tartozó kezelt erőforrások törlésére készít tervet és hajt végre műveletet. Ezt csak tudatosan, megfelelő környezetben használjuk. A `-auto-approve` kihagyja a megerősítést, ezért automatizálásban is csak kontrollált pipeline-ban alkalmazzuk.

## Szintaxis és alap példa

```hcl
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

resource "local_file" "notes" {
  filename = "${path.module}/notes.txt"
  content  = "Első Terraform-projekt"
}
```

A `path.module` az aktuális modul könyvtárára mutató Terraform érték. A resource címének teljes alakja `local_file.notes`. Ez a cím fontos a state-ben, a tervben és a későbbi hivatkozásokban.

## Részletes magyarázat

Futtatási sorrend:

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
terraform state list
terraform destroy
```

A mentett terv (`tfplan`) alkalmazásakor a Terraform az előzetesen jóváhagyott tervet hajtja végre. A tervfájl érzékeny adatokat is tartalmazhat, ezért ne töltsük fel nyilvános helyre. A `terraform state list` a state-ben nyilvántartott címeket sorolja fel.

A tervben a `+` létrehozást, a `-` törlést, a `~` módosítást jelez; a csere gyakran `-/+` jelöléssel jelenik meg. A pontos tervet mindig olvassuk el, különösen a törléseket és cseréket.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Hozz létre egy `local_file` erőforrást.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Készíts mentett tervet, majd alkalmazd.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Sorold fel a state-ben szereplő erőforrásokat, végül töröld a teszterőforrást.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Az első projekt teljes életciklusa

Hozz létre egy új könyvtárat, majd mentsd bele a következő fájlt `main.tf` néven:

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

resource "local_file" "greeting" {
  filename = "${path.module}/greeting.txt"
  content  = "Az első Terraform-erőforrásom."
}

output "file_path" {
  value = local_file.greeting.filename
}
```

A `terraform` blokk megadja a provider-követelményt. A `resource` blokk egy `local_file` típusú erőforrást deklarál `greeting` helyi névvel. A `filename` és `content` a provider resource sémájához tartozó argumentumok. Az output a létrehozott resource egyik attribútumát teszi láthatóvá a root module használója számára.

Futtasd a parancsokat egymás után:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
terraform destroy
```

Az `init` letölti a szükséges providert és előkészíti a munkakönyvtárat. A `fmt` egységesíti a formázást. A `validate` a konfiguráció belső helyességét ellenőrzi. A `plan` megmutatja a tervezett változásokat. Az `apply` jóváhagyás után létrehozza a fájlt. Az `output` kiírja a kimeneti értéket, a `destroy` pedig eltávolítja a Terraform által kezelt fájlt.

### Hogyan olvasd a tervet?

A tervben gyakran találkozol ezekkel a jelölésekkel:

- `+` – létrehozás (create)
- `~` – módosítás (update in-place)
- `-/+` – törlés és újralétrehozás (replace)
- `-` – törlés (destroy)

A pontos megjelenítés változhat, de a művelet jelentése a lényeg. A `-/+` különösen fontos: egy látszólag kis módosítás is cserét okozhat, ha az adott provider API-ja vagy resource-sémája nem engedi az attribútum helyben történő módosítását.

### Biztonságos munkaszokás

Valós környezetben ne futtasd vakon az `apply -auto-approve` parancsot. Előbb vizsgáld meg a tervet, különösen a törléseket, cseréket, hálózati szabályokat, hozzáféréseket és adatvesztést okozó műveleteket. Automatizált folyamatban mentsd a tervet fájlba, és ugyanazt a tervfájlt add át az apply lépésnek:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

A mentett plan fájl érzékeny adatokat is tartalmazhat, ezért kezeld titkos vagy legalábbis korlátozott hozzáférésű build artifactként.
