# 5. Adattípusok és adatszerkezetek

*string, number, bool, list, set, map, object, tuple*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform alaptípusai a `string`, `number` és `bool`. A komplex típusok több értéket fognak össze. A `list(T)` rendezett, azonos típusú elemek sorozata; a `set(T)` egyedi elemek rendezetlen halmaza; a `map(T)` string kulcsokhoz rendel azonos típusú értékeket. Az `object({...})` névvel ellátott attribútumok típusát írja le, a `tuple([...])` pedig pozíciónként meghatározott típusokat tartalmaz.

A típusdeklarációk egyértelművé teszik, hogy a modul milyen típusú bemeneti értékeket vár. Ez a modul használati felületének (module interface; a szakirodalomban gyakran module contract) fontos része: a bemeneti változók meghatározzák, milyen adatokat adhat át a modul használója, az output értékek pedig azt, milyen eredményeket tesz elérhetővé a modul. A pontos típusmegadás segít időben felismerni a hibás adatokat. Az `any` nem korlátozás nélküli, konkrét adattípus; típuskorlátozásként (type constraint) azt jelzi, hogy a Terraform a ténylegesen átadott érték alapján következtessen a megfelelő típusra. Újrafelhasználható moduloknál ezért általában célszerű a lehető legpontosabban meghatározni a bemeneti értékek típusát.

A `null` hiányzó vagy nem megadott értéket jelölhet. Egy argumentum `null` értéke gyakran úgy viselkedik, mintha az argumentum nem lenne megadva, de az adott resource/provider szemantikája számít.

## Szintaxis és alap példa

```hcl
variable "app" {
  type = object({
    name     = string
    replicas = number
    labels   = map(string)
    zones    = set(string)
    ports    = list(number)
  })

  default = {
    name     = "catalog"
    replicas = 2
    labels   = { team = "platform", env = "dev" }
    zones    = ["eu-central-1a", "eu-central-1b"]
    ports    = [8080, 8443]
  }
}

output "app_name" {
  value = var.app.name
}
```

## Részletes magyarázat

A változó értékének meg kell felelnie az objektum sémájának. A `labels` kulcsai stringek, értékei stringek. A `zones` setként egyediséget garantál, így a sorrendjére nem szabad építeni. A `ports` listaként megtartja a sorrendet.

A típuskonverziók bizonyos esetekben automatikusan megtörténhetnek, de a modul bemeneteinél érdemes egyértelmű típusokat használni. Gyakori hiba, hogy valaki listát és mapet kever, vagy olyan attribútumra hivatkozik, amely nem része az object típusnak.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Deklarálj string, number és bool típusú változókat.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Készíts `map(string)` és `list(number)` változót.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Definiálj egy legalább négy attribútumos object típust és hivatkozz az egyik mezőjére.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Adattípusok részletesen

A Terraform a primitív típusok mellett gyűjtemény- és strukturális típusokat használ. A `list(T)`, `set(T)` és `map(T)` collection típusok: az elemek típusa közös. Az `object({...})` és `tuple([...])` strukturális típusok: az attribútumok vagy pozíciók típusát külön-külön is meg lehet adni.

```hcl
variable "zones" {
  type    = list(string)
  default = ["eu-west-1a", "eu-west-1b"]
}

variable "allowed_ports" {
  type    = set(number)
  default = [443, 80, 443]
}

variable "tags" {
  type = map(string)
  default = {
    application = "catalog"
    owner       = "platform"
  }
}

variable "service" {
  type = object({
    name     = string
    replicas = number
    enabled  = bool
  })
}
```

A `list` sorrendje jelentős, index alapján elérhető (`var.zones[0]`). A `set` egyedi elemeket tárol, ezért nincs stabil indexelési sorrendje; akkor használd, ha a sorrend nem része az adatok jelentésének. A `map` kulcs–érték párok gyűjteménye, ahol a kulcsok szövegek, az értékek azonos típusúak. Az `object` névvel ellátott mezőket tartalmaz, a `tuple` pedig rögzített pozíciójú, akár eltérő típusú elemekből állhat.

### Típuskonverzió

A Terraform bizonyos kontextusokban automatikus konverziót végezhet, például egy számmá alakítható szöveget `number` típusú változóhoz rendelhet. A túlzott implicit konverzió azonban nehezen olvasható konfigurációhoz vezethet. A bemeneteknél adj meg pontos `type` korlátozást, és ha valóban konverziót szeretnél, használd tudatosan a `tostring`, `tonumber` vagy `tobool` függvényt.

### Az `any` jelentése

Az `any` nem konkrét, minden értéket egyformán kezelő adattípus. A típuskorlátozásban azt jelzi, hogy a Terraform a tényleges érték alapján következtessen a megfelelő típusra. Ez moduloknál hasznos lehet, ha a modul az értéket általánosan továbbadja, de ha a modul belsőleg mezőket olvas vagy műveletet végez rajta, célszerű pontos `object`, `map` vagy más konkrét típust deklarálni.

### `null` és opcionális attribútumok

A `null` egy érték hiányát jelzi. Erőforrás-argumentumként sok esetben azt eredményezi, hogy Terraform úgy kezeli, mintha az argumentum nem lenne megadva; a pontos viselkedés a provider sémájától is függ. Az object típusban az `optional(...)` segítségével opcionális attribútum deklarálható, akár alapértékkel:

```hcl
variable "settings" {
  type = object({
    size        = string
    description = optional(string, "Managed by Terraform")
  })
}
```

Ilyenkor a `description` elhagyható a bemeneti objektumból, és a megadott alapértéket kapja. Ne keverd össze az opcionális attribútumot a `nullable = true` beállítással: az előbbi egy objektummező jelenlétére, az utóbbi a teljes változó `null` értékének megengedésére vonatkozik.

### Tipikus típushibák

- Listát vársz, de map-et adsz át: ellenőrizd a zárójelezést és a típusdeklarációt.
- `set` értéket indexelsz: alakítsd listává csak akkor, ha a sorrend valóban szükséges.
- Object attribútum hiányzik: deklaráld opcionálisnak, vagy add át kötelező mezőként.
- Map értékei eltérő típusúak: egységesítsd az értékeket, vagy használj `object` típust.
- A `tuple` és `list` összekeveredik: listában az elemek azonos típusúak, tuple-ben a pozíciók típusa eltérhet.

### Gyakorlófeladat megoldása

Készíts változót egy szolgáltatás konfigurációjához, amelynek kötelező mezője a név és a példányszám, opcionális mezője pedig a leírás:

```hcl
variable "service" {
  type = object({
    name        = string
    replicas    = number
    description = optional(string, "No description")
  })

  validation {
    condition     = var.service.replicas >= 1 && var.service.replicas <= 10
    error_message = "A replicas értékének 1 és 10 közé kell esnie."
  }
}
```

A típusdeklaráció megadja a bemenet szerkezetét, az `optional` alapértéket biztosít, a validation pedig üzleti szabályt ellenőriz. A három elem különböző feladatot lát el: a típus a formai szerkezetet, az alapérték a hiányzó mező kezelését, a validáció pedig az elfogadható értéktartományt szabályozza.
