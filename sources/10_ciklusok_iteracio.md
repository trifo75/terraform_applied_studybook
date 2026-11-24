# 10. Ciklusok és iteráció

*for expressions, count, for_each, dynamic blocks*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform több iterációs eszközt biztosít. A `for` expression listából vagy mapből új listát/mapet képez. A `count` számozott példányokat hoz létre, a `for_each` pedig map vagy set elemeihez rendelt, stabil kulcsú példányokat. A `dynamic` blokk ismétlődő beágyazott blokkok előállítására szolgál, ha a provider sémája ilyen blokkot vár.

A `for_each` sok esetben biztonságosabb, ha az elemeknek stabil azonosítójuk van. `count` esetén egy lista közepéről törölt elem miatt az indexek eltolódhatnak, ami nem kívánt cseréket eredményezhet.

## Szintaxis és alap példa

```hcl
variable "files" {
  type = map(string)
  default = {
    app    = "application configuration"
    proxy  = "reverse proxy configuration"
    worker = "background worker configuration"
  }
}

resource "local_file" "generated" {
  for_each = var.files
  filename = "${path.module}/${each.key}.txt"
  content  = each.value
}

output "file_names" {
  value = [for key, value in var.files : "${key}.txt"]
}
```

## Részletes magyarázat

A `for_each` minden map kulcshoz külön resource példányt hoz létre. A `each.key` a kulcs, az `each.value` a hozzá tartozó érték. A példánycímek például `local_file.generated["app"]` formájúak. A `for` expression egy új listát állít elő.

Set esetén a `each.key` és `each.value` az adott elem. A kulcsok legyenek determinisztikusak és ne tartalmazzanak titkos vagy később változó adatot.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Alakíts át egy listát nagybetűs listává `for` expressionnel.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Hozz létre három resource példányt `for_each` segítségével.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Hasonlítsd össze a `count` és `for_each` címzését.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## `for`, `count`, `for_each` és `dynamic` – mikor melyiket?

A `for` kifejezés értéktranszformációra szolgál: meglévő listából, mapből vagy objektumból új gyűjteményt állít elő. A `count` és a `for_each` ezzel szemben resource- vagy module-példányokat hoz létre. A `dynamic` blokk ismétlődő, beágyazott konfigurációs blokkok előállítására használható, nem önálló resource-példányok létrehozására.

```hcl
variable "services" {
  type = map(object({
    port    = number
    enabled = bool
  }))
  default = {
    api    = { port = 8080, enabled = true }
    worker = { port = 9090, enabled = false }
  }
}

resource "local_file" "service" {
  for_each = {
    for name, cfg in var.services : name => cfg
    if cfg.enabled
  }

  filename = "${path.module}/${each.key}.txt"
  content  = "port=${each.value.port}"
}
```

A `for_each` map kulcsai stabil azonosítók. Ha a `worker` elem kikapcsolt, a `api` példány címe továbbra is `local_file.service["api"]` marad. `count` esetén az indexek eltolódhatnak, ha a lista közepéről törölsz elemet; ezért névvel azonosított erőforrásoknál a `for_each` gyakran biztonságosabb.

`for_each` kulcsa nem lehet plan során ismeretlen érték, mert a Terraformnak előre tudnia kell, hány példányt és milyen címeken kezeljen. Ez az egyik leggyakoribb iterációs hiba.
