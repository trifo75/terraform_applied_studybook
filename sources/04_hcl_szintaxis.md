# 4. HCL szintaxis és nyelvi elemek

*blokkok, argumentumok, kifejezések, kommentek*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A HashiCorp Configuration Language (HCL) blokkokból és argumentumokból épül fel. A blokk egy konfigurációs egység, például `resource`, `variable`, `output`, `provider` vagy `module`. A blokk címkéi azonosítják a blokk típusát vagy nevét. Az argumentum egy névhez rendelt érték.

A HCL kifejezések literálokat, változóhivatkozásokat, függvényhívásokat, operátorokat, feltételes kifejezéseket és komplex értékeket tartalmazhatnak. A stringek idézőjelek között állnak; interpolációhoz `${...}` használható, bár önálló kifejezésként gyakran közvetlenül a kifejezés a tisztább forma. A `#` és `//` egysoros, a `/* ... */` többsoros kommentet jelöl.

A `.tf` fájlok a konfigurációt, a `.tfvars` fájlok jellemzően változóértékeket, a `.tftest.hcl` fájlok teszteket tartalmaznak. A fájlkiterjesztés önmagában nem változtatja meg a HCL nyelv alapvető kifejezésmodelljét, de meghatározza, hogy Terraform milyen célra olvassa be a fájlt.

## Szintaxis és alap példa

```hcl
variable "environment" {
  type        = string
  description = "A telepítés környezete."
  default     = "dev"
}

locals {
  name_prefix = "demo-${var.environment}"
}

output "resource_prefix" {
  value = local.name_prefix
}
```

## Részletes magyarázat

A `variable` blokk deklarálja a bemenetet. A `locals` blokk számított, modulon belüli értéket nevez el. Az `output` a modul eredményét teszi láthatóvá a hívó modul vagy a CLI számára. A `var.` és `local.` előtagok névteret jelölnek; nem tetszőleges szövegek.

A `terraform fmt` a HCL kanonikus formázását végzi. A kézzel igazított szóközök helyett hagyatkozzunk erre az eszközre. A `validate` a szintaktikai és statikus konfigurációs hibák egy részét jelzi, de nem bizonyítja, hogy az API-hívás vagy a létrehozás sikerülni fog.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Írj `variable`, `locals` és `output` blokkokat.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Alakítsd át a stringet interpolációval környezetfüggővé.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Futtasd a `terraform fmt` és `terraform validate` parancsokat.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## HCL kifejezések és blokkok részletesen

A Terraform konfiguráció alapvető építőelemei a blokkok és az argumentumok. A blokk egy adott konfigurációs egységet jelöl, az argumentum pedig egy értéket rendel egy névhez. A blokk címkéi a blokk típusától függően azonosítják a deklarált objektumot.

```hcl
variable "environment" {
  type        = string
  description = "A telepítési környezet neve."
  default     = "dev"
}

locals {
  name_prefix = "demo-${var.environment}"
}

output "resource_prefix" {
  value = local.name_prefix
}
```

A `variable`, `locals` és `output` blokk különböző célt szolgál. A változó kívülről érkező bemenetet fogad, a local a konfiguráción belül újrahasznosítható kifejezést nevez el, az output pedig értéket tesz elérhetővé a modul határán.

A HCL kifejezések értéket állítanak elő. Ilyen a literál (`"hello"`, `42`, `true`), a hivatkozás (`var.environment`), a függvényhívás (`lower("DEV")`), az objektumkifejezés (`{ name = "web" }`) és a feltételes kifejezés (`var.enabled ? 1 : 0`). Az interpolációs szintaxis `${...}` szövegbe ágyazott kifejezést jelöl; ha a teljes érték egy kifejezés, nem szükséges idézőjelek közé tenni.

### Kommentek és formázás

A HCL támogatja a `#`, `//` és `/* ... */` kommenteket. A csapatban válasszatok egységes stílust; a `terraform fmt` a formázást automatizálja, de a kommentek tartalmi minőségét nem javítja. A komment azt magyarázza, hogy egy nem magától értetődő döntés miért született, ne a kódot fordítsa le magyar mondatokra.

### Kifejezés vagy szöveg?

Gyakori hiba, hogy egy kifejezést idézőjelek közé tesznek, és így szó szerinti szöveggé válik:

```hcl
# Helyes
count = var.create_network ? 1 : 0

# Hibás szándék esetén: ez csak szöveg
count = "var.create_network ? 1 : 0"
```

A második példa nem logikai kifejezésként működik. A Terraform típuskonverziója bizonyos helyzetekben megpróbálhat értéket konvertálni, de erre nem szabad programozási technikaként támaszkodni.
