# 7. Locals és Outputs

*számított értékek és modul kimenetek*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A `locals` blokkban a modulon belül újrahasznosítható kifejezéseket nevezünk el. A locals nem bemenet: értéke a konfiguráció más elemeiből számítható. Használatakor `local.<név>` formát írunk. A locals csökkentheti az ismétlést és központi helyre teheti a névképzést, címkéket vagy összetett feltételeket.

Az `output` blokk a modul eredményeit teszi hozzáférhetővé. Root modulban a `terraform output` paranccsal kérdezhetők le; child module esetén a hívó modul a modul outputján keresztül fér hozzá. Outputban is használható `sensitive = true`, ha az érték bizalmas. Ettől az adat még state-ben szerepelhet.

## Szintaxis és alap példa

```hcl
variable "project" {
  type    = string
  default = "learning"
}

variable "environment" {
  type    = string
  default = "dev"
}

locals {
  name_prefix = "${var.project}-${var.environment}"
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

output "name_prefix" {
  value = local.name_prefix
}
```

## Részletes magyarázat

A `local.name_prefix` ugyanazt a számított nevet használhatja több resource-ban. A `common_tags` mapként átadható a provider által támogatott tag argumentumnak. Az output nem hoz létre erőforrást; csak értéket tesz elérhetővé.

Kerüljük a túl sok, egymásra épülő locals használatát, ha ettől nehezen követhetővé válik a konfiguráció. Az elnevezés legyen jelentéses, ne pusztán technikai rövidítés.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Képezz névprefixet két változóból.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Hozz létre közös tag mapet.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Készíts outputot egy resource attribútumából.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Locals és outputs – számítás és eredményközlés

A `locals` blokk nem külső bemenet, hanem a konfiguráció belső olvashatóságát javító névvel ellátott kifejezések gyűjteménye. Ugyanazt a kifejezést több helyen is felhasználhatod anélkül, hogy megismételnéd. A local értéke lehet egyszerű literál, függvényhívás, objektum vagy más erőforrás attribútumára hivatkozó kifejezés.

```hcl
locals {
  common_tags = {
    project     = var.project_name
    environment = var.environment
    managed_by  = "terraform"
  }

  name_prefix = "${var.project_name}-${var.environment}"
}

resource "local_file" "metadata" {
  filename = "${path.module}/${local.name_prefix}.json"
  content  = jsonencode(local.common_tags)
}

output "metadata_file" {
  description = "A létrehozott metaadatfájl elérési útja."
  value       = local_file.metadata.filename
}
```

Az output nem ugyanaz, mint a local. A local a modulon belüli újrafelhasználásra szolgál, az output pedig a modul határán keresztül tesz közzé egy értéket. Egy child module outputját a hívó modul a `module.<név>.<output>` alakban érheti el.

A `sensitive = true` outputnál is csak megjelenítési védelmet ad; a titkos érték továbbra is jelen lehet a state-ben. Outputokba ezért ne tegyél szükségtelen titkokat, és gondoskodj a state megfelelő védelméről.
