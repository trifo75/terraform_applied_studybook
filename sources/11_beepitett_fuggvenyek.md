# 11. Beépített függvények

*string, collection, numeric, filesystem, encoding functions*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform beépített függvényei kifejezésekben használhatók. Gyakori string függvény a `lower`, `upper`, `trimspace`, `replace`, `format` és `join`. Kollekciókhoz használható többek között a `length`, `concat`, `merge`, `keys`, `values`, `lookup`, `contains`, `distinct`, `sort` és `flatten`. Számoláshoz rendelkezésre áll például a `min`, `max`, `ceil` és `floor`.

A függvények használata előtt fontos megérteni a bemeneti és visszatérési típust. A `lookup` alapértelmezett értéke például megakadályozhatja a hiányzó kulcs miatti hibát. A `try` és `can` bizonyos kifejezéshibák kezelésére szolgál, de nem szabad velük elfedni a konfigurációs problémákat.

## Szintaxis és alap példa

```hcl
locals {
  raw_name = "  Demo-App  "
  name     = lower(trimspace(local.raw_name))
  tags     = merge(
    { ManagedBy = "Terraform" },
    { Environment = "dev" }
  )
  ports = [for p in [80, 443, 8080] : tostring(p)]
}

output "normalized_name" {
  value = local.name
}
```

## Részletes magyarázat

A függvények egymásba ágyazhatók. A `trimspace` eltávolítja a szélső whitespace karaktereket, a `lower` kisbetűsíti az eredményt. A `merge` mapeket egyesít; azonos kulcs esetén a későbbi argumentum értéke felülírja a korábbit. A `tostring` a számokat stringgé alakítja.

A teljes függvénylista verzió és nyelvi környezet szerint a HashiCorp dokumentációban kereshető. Ne találgassunk egy ritkán használt függvény nevét: ellenőrizzük a pontos szignatúrát.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Normalizálj egy változóban kapott nevet.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Egyesíts két tag mapet, és vizsgáld meg az ütköző kulcsot.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Írj `for` expressiont `tostring` függvénnyel.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Függvények gyakorlati csoportosítása

A Terraform függvényei nem tetszőleges programozási nyelvi könyvtárat alkotnak; a konfigurációs kifejezésekben használható, dokumentált műveletek. A leggyakoribb csoportok:

- Szöveg: `lower`, `upper`, `trimspace`, `replace`, `format`, `join`, `split`.
- Gyűjtemény: `length`, `lookup`, `contains`, `keys`, `values`, `merge`, `flatten`, `distinct`, `toset`, `tolist`.
- Szám: `min`, `max`, `ceil`, `floor`, `abs`.
- Fájlrendszer: `file`, `fileexists`, `fileset`, `templatefile`.
- Kódolás: `jsonencode`, `yamlencode`, `base64encode`, `base64decode`.
- Hálózati: `cidrhost`, `cidrnetmask`, `cidrsubnet`.

```hcl
locals {
  raw_names = [" API ", "Worker", "api"]
  names     = distinct([for name in local.raw_names : lower(trimspace(name))])
  tags      = merge(var.default_tags, { component = "backend" })
  config    = jsonencode({
    services = local.names
    tags     = local.tags
  })
}
```

A fenti példa normalizálja a neveket, kiszűri az ismétlődéseket, összefésüli a tag-eket, majd JSON-szöveget állít elő. A `merge` esetén az azonos kulcs későbbi mapben megadott értéke felülírja a korábbit. Ezért a sorrend számít.

A `file` és `templatefile` csak olyan fájlokat olvas, amelyek már a Terraform futtatásakor rendelkezésre állnak; nem arra szolgálnak, hogy az apply közben létrejövő fájlokat dinamikusan visszaolvassák. A függvények részletes referenciáját a C függelékben találod. Ismeretlen függvénynél ne találgass: ellenőrizd a hivatalos Function Reference oldalon.
