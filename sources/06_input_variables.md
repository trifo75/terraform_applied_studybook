# 6. Input Variables

*bemenetek, default, validation, sensitive, nullable*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

Az input variable a modul külső konfigurációs felülete. A `variable` blokkban megadható típus, leírás, alapértelmezett érték, validáció, érzékenységi jelölés és nullázhatóság. Ha nincs `default`, az értéket a Terraform futtatáskor kéri vagy valamelyik bemeneti forrásból olvassa.

Értékforrások többek között: `-var`, `-var-file`, automatikusan betöltött `terraform.tfvars`, `terraform.tfvars.json`, illetve `*.auto.tfvars` fájlok. A parancssori és automatikus betöltési prioritás részleteit a CLI dokumentációban ellenőrizni kell. A titkokat ne commitoljuk tfvars fájlban.

A `sensitive = true` elrejti az értéket bizonyos CLI-kimenetekben, de nem titkosítja automatikusan a state-et vagy a tervfájlt. A bizalmas adatok védelme backend-, hozzáférés- és naplózási kérdés is.

## Szintaxis és alap példa

```hcl
variable "environment" {
  type        = string
  description = "dev, test vagy prod"
  validation {
    condition     = contains(["dev", "test", "prod"], var.environment)
    error_message = "Az environment csak dev, test vagy prod lehet."
  }
}

variable "admin_password" {
  type      = string
  sensitive = true
  nullable  = false
}
```

A validációs feltétel hibás bemenetnél saját, olvasható hibaüzenetet ad. A `sensitive` jelölés nem helyettesíti a titokkezelő rendszert.

## Részletes magyarázat

A változó értékét megadhatod például:

```bash
terraform plan -var='environment=dev'
terraform plan -var-file='dev.tfvars'
```

A `dev.tfvars` tartalma:

```hcl
environment = "dev"
```

A változó értékeinek környezetenkénti elkülönítése hasznos, de az eltérő környezetekhez tartozó backend- és jogosultságbeállításokat is külön kell kezelni.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Adj validációt egy környezetváltozóhoz.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Készíts külön tfvars fájlt tesztkörnyezethez.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Magyarázd el, miért nem titkosítja a `sensitive` a state-et.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Input variables: bemeneti felület kialakítása

A root module input variable értékeit több forrásból kaphatja. Tipikus forrás a `default`, a `terraform.tfvars`, az automatikusan betöltött `*.auto.tfvars`, a `-var` vagy `-var-file` kapcsoló, illetve a `TF_VAR_name` környezeti változó. Több értékforrás esetén a Terraform meghatározott precedenciasorrendet alkalmaz; ezért a reprodukálható futtatáshoz célszerű a használt bemeneteket dokumentálni és a titkos adatokat elkülöníteni.

```hcl
variable "instance_count" {
  description = "A létrehozandó példányok száma."
  type        = number
  default     = 2

  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 5
    error_message = "Az instance_count értéke 1 és 5 közé essen."
  }
}

variable "admin_password" {
  description = "Adminisztrátori jelszó."
  type        = string
  sensitive   = true
}
```

A `sensitive = true` elrejti az értéket a Terraform egyes felhasználói felületein, de nem titkosítja automatikusan a state-et vagy a mentett plan fájlt. Ezért a jelszó forrását, a state backend hozzáférését és a CI naplóit is védeni kell.

A `nullable = false` megakadályozza, hogy a változó teljes értéke `null` legyen. A `default` hiánya esetén a Terraform bekéri az értéket interaktív módban, vagy hibát ad, ha a futtatás nem interaktív és az érték nem érkezik meg.

### Javasolt változótervezési elvek

- Minden változóhoz adj `description` mezőt.
- Használj a lehető legpontosabb típust.
- Validációval ellenőrizd a tartományokat és megengedett értékeket.
- A titkokat ne írd a verziókezelt `*.tfvars` fájlba.
- Ne hozz létre változót pusztán azért, mert egy érték egyszer szerepel: a változó akkor hasznos, ha valódi konfigurációs döntést tesz állíthatóvá.
