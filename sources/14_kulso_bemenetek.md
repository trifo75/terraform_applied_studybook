# 14. Külső bemenetek és konfigurációs adatok

*tfvars, environment variables, files, external data*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform bemenetei származhatnak változófájlokból, környezeti változókból, parancssori argumentumokból vagy adatforrásokból. A `TF_VAR_name` környezeti változó a `name` nevű input variable értékét adhatja meg. A környezeti változók kényelmesek automatizálásban, de a CI logok és a futtatókörnyezet hozzáférési szabályai miatt gondosan kezelendők.

A `file`, `fileexists`, `fileset` és `templatefile` függvények helyi fájlokkal dolgoznak. A `data` blokkok provider által kezelt külső adatokat kérdeznek le. Az `external` adatforrás (data source) külső programmal kommunikál, de csak körültekintően, hordozhatósági és biztonsági szempontok figyelembevételével használjuk.

## Szintaxis és alap példa

```bash
export TF_VAR_environment="dev"
terraform plan
```

```hcl
variable "environment" {
  type = string
}

locals {
  config_text = file("${path.module}/config.txt")
}
```

## Részletes magyarázat

A `TF_VAR_environment` a `environment` nevű változóhoz kapcsolódik. A `file` a Terraform futtató gépén elérhető fájlt olvassa be; nem tölti le automatikusan távoli objektumtárolóból. Fájlbeolvasáskor a fájl létezését és kódolását is ellenőrizni kell.

A külső bemenetek legyenek reprodukálhatók: dokumentáljuk a szükséges fájlokat, környezeti változókat és titokforrásokat. Ne függjünk rejtett, csak egy fejlesztő gépén létező állapotoktól.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Adj meg input variable értéket `TF_VAR_` változóval.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Olvass be egy projektfájlt `file` függvénnyel.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Dokumentáld egy mintaprojekt összes szükséges külső bemenetét.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Külső adatok és bemenetforrások

A Terraform konfiguráció sokszor nem tartalmazhat minden értéket közvetlenül. A környezetfüggő beállításokhoz használhatsz `*.tfvars` fájlt, CI környezeti változókat vagy külső titokkezelőt. A nem titkos, környezetfüggő értékeket célszerű külön fájlban tartani, például `dev.tfvars` és `prod.tfvars` néven.

```bash
terraform plan -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"
```

A `terraform.tfvars` és `*.auto.tfvars` fájlokat a Terraform automatikusan betölti. Más nevű fájlt explicit `-var-file` kapcsolóval kell megadni. A `.tfvars` fájlban HCL értékeket írsz, nem teljes `variable` blokkokat.

A `TF_VAR_<név>` környezeti változó egy input variable értékét adhatja meg. Ez automatizálásban hasznos, de a CI platform titokkezelését és naplózási szabályait is ellenőrizni kell. A `TF_LOG` diagnosztikai naplózást kapcsolhat be; éles környezetben óvatosan használd, mert a részletes napló érzékeny adatokat is tartalmazhat.

Az `external` data source külső programot futtat, ezért hordozhatósági és biztonsági kockázata van. Csak akkor használd, ha nincs jobb provider vagy natív Terraform-megoldás. A programnak a várt JSON bemeneti/kimeneti szerződést kell követnie, és a futtatási környezetben elérhetőnek kell lennie.
