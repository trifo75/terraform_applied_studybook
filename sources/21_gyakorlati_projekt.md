# 21. Összetett gyakorlati projekt

*moduláris, több környezetet kezelő tanulóprojekt*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

Ebben a projektben egy felhőfüggetlen konfigurációs vázat készítünk, amelyet később AWS, Azure, GCP vagy OCI providerrel lehet kibővíteni. A cél nem egy konkrét felhőszolgáltatás automatikus létrehozása, hanem a Terraform projektfelépítés, modulhatárok, bemenetek, outputok, state és tesztelés gyakorlása.

A projekt root modulja környezeti változókat és közös címkéket kezel. A `modules/naming` modul egységes neveket állít elő. A `modules/config_file` modul egy konfigurációs fájlt generál a bemenetekből. A `dev.tfvars` és `prod.tfvars` eltérő paramétereket adnak át. Éles környezetben külön backend és külön jogosultság szükséges; a workspace önmagában nem biztonsági izoláció.

## Szintaxis és alap példa

Javasolt könyvtárstruktúra:

```text
terraform-lab/
├── main.tf
├── variables.tf
├── locals.tf
├── outputs.tf
├── versions.tf
├── dev.tfvars
├── prod.tfvars
├── modules/
│   ├── naming/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── config_file/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── tests/
    └── naming.tftest.hcl
```

## Részletes magyarázat

A megvalósítás lépései:
1. Deklaráld a Terraform és provider verziókövetelményeket.
2. Készíts `environment`, `project_name` és `application_port` input változókat validációval.
3. A locals blokkban állítsd elő a névprefixet és a közös tageket.
4. Készíts `naming` gyermekmodult (child module), amely outputként visszaadja a normalizált nevet.
5. Készíts konfigurációfájl-generáló modult.
6. Hozz létre dev és prod tfvars fájlokat.
7. Futtasd a fmt, init, validate és plan lépéseket.
8. Írj plan-alapú tesztet a névkonvencióra.
9. Dokumentáld, hogyan kapcsolnád a modult a választott felhőszolgáltatóhez.

A feladat megoldása során ne másold be a titkokat a fájlokba. A felhőszolgáltató integráció külön bővítési lépés, amelyhez a választott provider hivatalos dokumentációját kell követni.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Alap: készítsd el a változókat és locals értékeket.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Összetett: valósítsd meg a két gyermekmodult (child module) és a tfvars fájlokat.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Haladó: adj hozzá tesztet, CI ellenőrzést és egy dokumentált remote backend-tervet.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Gyakorlati projekt: környezetfüggetlen fájlalapú szolgáltatásleíró

A fejezet célja egy olyan moduláris minta végigépítése, amely nem igényel felhőfiókot, mégis bemutatja a változókat, local értékeket, module-okat, outputokat, iterációt és validációt. A példa tanulási célú; a `local` provider nem helyettesíti a felhőinfrastruktúra kezelését.

Javasolt könyvtárstruktúra:

```text
terraform-lab/
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    └── service-file/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

Root `main.tf`:

```hcl
module "service_files" {
  source = "./modules/service-file"

  services = var.services
  output_dir = "${path.root}/generated"
}
```

Root `variables.tf`:

```hcl
variable "services" {
  type = map(object({
    port    = number
    enabled = bool
  }))

  validation {
    condition = alltrue([
      for service in values(var.services) :
      service.port >= 1 && service.port <= 65535
    ])
    error_message = "Minden portnak 1 és 65535 közé kell esnie."
  }
}
```

A child module `variables.tf` ugyanilyen pontos objektumtípust deklaráljon, az `output_dir` pedig `string` legyen. A child module `main.tf` a `for_each` segítségével csak az engedélyezett szolgáltatásokhoz készítsen fájlt. A feladat során figyeld meg, hogy a modulok külön könyvtárban vannak, de ugyanannak a root module futásnak és state-nek a részei.

### Tesztelési forgatókönyv

1. Adj meg két szolgáltatást, amelyek közül az egyik `enabled = true`, a másik `false`.
2. Futtasd a `terraform fmt -recursive`, `terraform init`, `terraform validate` és `terraform plan` parancsokat.
3. Ellenőrizd, hogy a terv csak az engedélyezett szolgáltatás fájlját hozza létre.
4. Állíts be 70000-es portot, és ellenőrizd, hogy a validáció megállítja a hibás bemenetet.
5. Futtasd az apply-t, ellenőrizd a fájlt, majd destroy-jal takarítsd el.

A projekt kiterjeszthető provider-specifikus változatra: például AWS S3 bucket, Azure Storage Account vagy OCI Object Storage bucket létrehozásával. Ilyen bővítésnél a provider erőforrás-sémája és a hitelesítési mód már szolgáltatónként eltér, ezért külön fejezetként, hivatalos provider dokumentációval kell kezelni.
