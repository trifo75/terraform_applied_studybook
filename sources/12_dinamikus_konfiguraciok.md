# 12. Dinamikus konfigurációk és sablonok

*dynamic blocks, templatefile, jsonencode, yamlencode*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A dinamikus konfiguráció lehetővé teszi, hogy a bemeneti adatokból ismétlődő vagy összetett konfigurációs részeket állítsunk elő. A `dynamic` blokk egy provider által definiált beágyazott blokkot generál. Ez nem általános resource-generátor: a provider sémájában létező nested block típushoz használható.

A `templatefile` külső sablonfájlt renderel megadott változókkal. A `jsonencode` és `yamlencode` Terraform értéket alakít JSON vagy YAML szöveggé. Strukturált adatok előállításánál ezek biztonságosabbak, mint a kézzel összefűzött stringek.

## Szintaxis és alap példa

```hcl
locals {
  app_config = {
    name    = "catalog"
    port    = 8080
    enabled = true
  }

  app_json = jsonencode(local.app_config)
}

resource "local_file" "app_config" {
  filename = "${path.module}/app-config.json"
  content  = local.app_json
}
```

## Részletes magyarázat

A `jsonencode` kezeli a string escape-elést és a típusok JSON reprezentációját. A `templatefile` sablonban `${var}` jellegű kifejezésekkel helyettesíthetők értékek; a sablonnak átadott map kulcsai határozzák meg, milyen nevekre hivatkozhatunk.

A dinamikus blokkok túlzott használata olvashatatlanná teheti a konfigurációt. Ha egy egyszerű, statikus nested block is elég, az gyakran jobb választás.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Írj JSON fájlt `jsonencode` használatával.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Renderelj egy `.tftpl` sablont `templatefile` segítségével.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Keress provider-dokumentációban olyan nested blockot, amely `dynamic` segítségével ismételhető.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Dinamikus blokkok és sablonok

A `dynamic` blokk akkor hasznos, ha egy resource vagy más blokk ismétlődő nested block-ot tartalmaz, és az ismétlődés bemeneti adatokból áll elő. Nem minden ismétlődő szerkezethez kell dynamic blokk: ha külön resource-példányokról van szó, inkább `for_each` vagy `count` kell.

```hcl
variable "rules" {
  type = list(object({
    name = string
    port = number
  }))
}

# A példa szintaktikai minta; a nested blokk neve provider/resource-függő.
dynamic "rule" {
  for_each = var.rules
  content {
    name = rule.value.name
    port = rule.value.port
  }
}
```

A `rule` blokknevet és a benne elérhető `rule.value` objektumot a dynamic blokk címkéje határozza meg. A `content` részben a generált nested block argumentumai szerepelnek. Mivel a nested block neve provider-specifikus, a fenti minta önmagában nem teljes resource-konfiguráció.

A `templatefile` fájlalapú sablonból állít elő szöveget. A sablonban `${...}` jelöléssel lehet értékeket behelyettesíteni, a függvény második argumentuma pedig mapként adja át a változókat. JSON/YAML előállításakor lehetőség szerint `jsonencode` vagy `yamlencode` segítségével kódold az adatstruktúrát, ne kézzel fűzz össze idézőjeleket és vesszőket. Így elkerülhetők az escape-elési és szintaktikai hibák.
