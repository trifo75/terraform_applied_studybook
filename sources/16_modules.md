# 16. Modules részletesen

*root és child module, input-output contract, registry és source*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A module együtt kezelt Terraform konfigurációs fájlok gyűjteménye. Minden Terraform konfiguráció root module-ként fut; a `module` blokk child module-t hív. A modulok újrafelhasználható építőelemeket hoznak létre, például hálózatot, alkalmazási környezetet vagy jogosultsági csomagot.

Egy jó modul kiszámítható bemeneti és kimeneti szerződéssel rendelkezik. Input variable-ökkel paraméterezhető, outputokkal teszi elérhetővé az eredményeket. A modul legyen egy jól körülhatárolt felelősségű egység, ne minden lehetséges infrastruktúra-probléma általános keretrendszere.

A `source` lehet helyi útvonal, Git repository vagy Terraform Registry cím. Registry modulnál verziót adjunk meg, hogy a build reprodukálható maradjon.

## Szintaxis és alap példa

```hcl
module "labels" {
  source = "./modules/labels"

  project     = "catalog"
  environment = "dev"
}
```

A `modules/labels/variables.tf`:

```hcl
variable "project" { type = string }
variable "environment" { type = string }
```

A `modules/labels/outputs.tf`:

```hcl
output "name_prefix" {
  value = "${var.project}-${var.environment}"
}
```

## Részletes magyarázat

A child module outputja a hívó oldalon `module.labels.name_prefix` formában érhető el. A helyi modulútvonal a root module-hoz képest értelmezett. Modulforrás módosításakor futtassuk újra az `init` parancsot.

Modulfejlesztéskor dokumentáljuk a bemeneteket, kimeneteket, verziókövetelményeket, provider követelményeket, példákat és ismert korlátokat. A modul ne rejtsen el váratlan erőforrás-létrehozást.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Készíts `modules/naming` gyermekmodult (child module) két inputtal és egy outputtal.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Hívd meg root modulból.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Írd le, milyen változtatás igényel új `terraform init` futtatást.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Modulok: újrafelhasználható konfigurációs egységek

A module egy önálló Terraform-konfigurációs könyvtár. A root module az a könyvtár, ahonnan a Terraform CLI parancsot futtatod. A child module-t a root vagy egy másik modul `module` blokkja hívja meg.

```hcl
module "file_example" {
  source      = "./modules/file"
  filename    = "${path.root}/example.txt"
  content     = "Created by a child module"
}
```

A `source` megadhat helyi könyvtárat, registry modult vagy támogatott távoli forrást. A modul bemeneti változói határozzák meg, milyen adatokat vár a használójától; az outputok pedig azt, milyen eredményeket tesz elérhetővé. Ez a modul használati felülete (module interface; gyakran module contract néven hivatkoznak rá).

### Modultervezési elvek

- Egy modul egy jól körülhatárolható feladatért feleljen.
- Ne tegyél minden lehetséges beállítást opcionális változóvá; a túl nagy felület nehezen érthető.
- A bemeneti típusokat pontosan deklaráld, és adj leírást minden változóhoz.
- A modul ne feltételezzen rejtett hitelesítési vagy környezeti állapotot.
- Az outputok csak olyan adatot tegyenek közzé, amelyet a hívó modulnak valóban használnia kell.
- Registry modulnál rögzíts verziót; Git forrásnál lehetőség szerint rögzített refet használj.

A modul nem külön Terraform-folyamat: ugyanannak a root module futásnak a része, közös state-tel. A modul határa szervezési és újrafelhasználási határ, nem automatikus biztonsági vagy state-izolációs határ.
