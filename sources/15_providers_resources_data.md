# 15. Providers, Resources és Data Sources

*provider konfiguráció, resource életciklus, adatlekérdezés*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A provider plugin egy külső API-hoz vagy szolgáltatáshoz kapcsolja a Terraformot. A `required_providers` deklarálja a provider forrását és verziókorlátját. A `provider` blokk a példány konfigurációját adja meg, például régiót vagy tenantot. Hitelesítő adatot lehetőleg ne írjunk a konfigurációba.

A `resource` blokk létrehozandó vagy kezelt objektumot ír le. A `data` blokk már létező, külső objektum adatait kérdezi le; önmagában nem hozza létre az objektumot. A provider dokumentációja határozza meg a resource argumentumokat, attribútumokat, adatforrásokat és viselkedést.

## Szintaxis és alap példa

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}

resource "local_file" "example" {
  filename = "${path.module}/example.txt"
  content  = "provider resource"
}
```

## Részletes magyarázat

A `local` provider egyszerű, felhőfüggetlen tanulópélda. AWS, Azure, GCP és OCI esetén külön provider konfiguráció és provider-specifikus resource típus szükséges. A felhőpéldák nem cserélhetők fel pusztán a provider nevének átírásával: az API-k, jogosultságok, régiók és erőforrásmodellek eltérnek.

A provider verzióját rögzítsük, a lock fájlt tartsuk verziókezelésben. Nagyobb provider-frissítés előtt olvassuk el a changelogot, és teszteljük a tervet.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Magyarázd el a provider, resource és adatforrás (data source) különbségét.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Keress egy kiválasztott felhőszolgáltató hivatalos resource dokumentációjában kötelező argumentumokat.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Rögzíts provider verziót és inicializáld a projektet.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Provider, resource és data source – a három szerep

A provider a Terraform Core és egy külső API közötti integrációt biztosítja. A provider konfigurációja hitelesítési és régióbeállításokat, illetve egyéb API-paramétereket tartalmazhat. A resource egy Terraform által kezelt objektum életciklusát írja le. A data source ezzel szemben meglévő vagy külső adatot olvas be, de önmagában nem hoz létre és nem kezel életciklust.

```hcl
terraform {
  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "~> 3.6"
    }
  }
}

resource "random_pet" "name" {
  length = 2
}

data "http" "example" {
  url = "https://example.com"
}

output "generated_name" {
  value = random_pet.name.id
}
```

A minta két különböző provider funkciót szemléltet. A `random_pet` resource állapotot hoz létre a Terraform számára; a `http` data source külső HTTP-választ olvas. A `data` blokkban a lekérdezés eredményét `data.<típus>.<név>.<attribútum>` alakban lehet hivatkozni. A data source olvasása is függhet jogosultságtól, hálózattól és API-elérhetőségtől.

A provider verzióját és forrását rögzítsd. Ne keverd össze a provider aliasokat a resource nevekkel: alias segítségével több különböző provider-konfigurációt adhatsz meg, például két régióhoz vagy két fiókhoz. A hitelesítő adatokat lehetőleg környezeti vagy workload identity mechanizmussal add át, ne hardcode-old a konfigurációban.
