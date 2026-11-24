# 9. Feltételes logika

*conditional expression, boolean operators, dynamic creation*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform feltételes kifejezésének alakja `feltétel ? igaz_érték : hamis_érték`. A feltételnek logikai értékre kell kiértékelődnie, a két ág pedig kompatibilis típusú értéket adjon. A `&&`, `||` és `!` logikai operátorok összetettebb feltételeket képeznek.

Feltételes kifejezést használhatunk érték kiválasztására, de resource létrehozásának vezérlésére gyakran a `count` vagy `for_each` alkalmas. A feltételeket érdemes egyszerűen tartani; bonyolult logikát locals értékekbe szervezhetünk.

## Szintaxis és alap példa

```hcl
variable "enable_monitoring" {
  type    = bool
  default = false
}

locals {
  monitoring_tier = var.enable_monitoring ? "enhanced" : "basic"
}

resource "local_file" "monitoring" {
  count    = var.enable_monitoring ? 1 : 0
  filename = "${path.module}/monitoring.txt"
  content  = "Monitoring: ${local.monitoring_tier}"
}
```

## Részletes magyarázat

Ha a változó hamis, a `count` nulla, így a resource-nak nincs példánya. Ha igaz, egy példány jön létre. A `count`-os resource-ra indexszel hivatkozunk: `local_file.monitoring[0]`, ezért a nulla vagy egy példányos erőforrás hivatkozásainál figyelni kell az indexelésre.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Írj feltételes kifejezést dev/prod méret kiválasztására.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Hozz létre opcionális resource-t boolean változó alapján.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Gondold végig, hogyan változik a resource címe, ha `count`-ot használsz.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Feltételes kifejezések és biztonságos opcionális erőforrások

A Terraformban a feltételes kifejezés alakja `feltétel ? igaz_érték : hamis_érték`. Az eredmény két ágának kompatibilis típusúnak kell lennie, mert a Terraformnak egyetlen kifejezéstípust kell meghatároznia.

```hcl
variable "create_file" {
  type    = bool
  default = true
}

resource "local_file" "optional" {
  count    = var.create_file ? 1 : 0
  filename = "${path.module}/optional.txt"
  content  = "Only when enabled"
}

output "optional_path" {
  value = var.create_file ? local_file.optional[0].filename : null
}
```

A `count` használatakor az erőforrás példányai indexelt gyűjteményt alkotnak. Ha `count = 0`, a `[0]` index nem létezik, ezért az outputban a hivatkozást feltételes kifejezéssel védjük. A feltétel kiértékelése nem helyettesíti a típusok és az ismeretlen értékek megértését: plan során egyes értékek még nem ismertek, ezért a Terraform nem minden esetben tudja azonnal eldönteni a feltétel eredményét.

A `try` függvény egy vagy több kifejezést próbál ki, és az első hibamentes eredményt adja vissza. Hasznos lehet bizonytalan szerkezetű külső adatok normalizálásakor, de ne rejts el vele valódi konfigurációs hibákat. A `can` azt vizsgálja, hogy egy kifejezés hiba nélkül kiértékelhető-e; gyakran használják változóvalidációban.
