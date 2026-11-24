# 20. Testing, Code Quality és Linting

*fmt, validate, test, static analysis, policy checks*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform kódminőségét több szinten ellenőrizhetjük. A `terraform fmt -check` a formázást, a `terraform validate` a konfiguráció statikus helyességét ellenőrzi. A `terraform plan` megmutatja a providerrel együtt értelmezett változtatásokat. A `terraform test` HCL tesztfájlokban megadott futásokat és assertionöket hajt végre.

A Terraform tesztelési keretrendszere 1.6.0-tól érhető el. A tesztek alapértelmezés szerint apply műveletet is végezhetnek, tehát valódi infrastruktúrát hozhatnak létre és költséget okozhatnak. A `command = plan` használatával olyan teszt is írható, amely csak tervet készít. A provider helyettesítése tesztelési célú mock implementációval (provider mocking) 1.7.0-tól támogatott.

Kiegészítő eszközök lehetnek linters (statikus kódellenőrző eszközök), security scanner eszközök (biztonsági elemzőeszközök), policy-as-code eszközök és a CI pipeline-ban (folyamatos integrációs folyamatban) futó ellenőrzések. Ezek nem helyettesítik a Terraform CLI-t, hanem más hibakategóriákat keresnek.

## Szintaxis és alap példa

```hcl
# tests/naming.tftest.hcl
run "plan_has_expected_name" {
  command = plan

  assert {
    condition     = local.name_prefix == "catalog-dev"
    error_message = "A névprefix nem a várt érték."
  }
}
```

## Részletes magyarázat

A teszt fájl kiterjesztése `.tftest.hcl`. A `run` blokk egy tesztfutást határoz meg, az `assert` blokk pedig feltételt ellenőriz. A példában a `local.name_prefix` a tesztelt konfigurációból származik. A teszt csak akkor fordítható le, ha ez a local létezik a konfigurációban.

folyamatos integrációs (CI) környezetben tipikus sorrend: checkout, Terraform telepítés, `init`, `fmt -check`, `validate`, tesztek, plan. A plan felülvizsgálata legyen külön kontrollpont. Provider hitelesítés nélkül a providerfüggő ellenőrzések nem feltétlenül futnak le.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Készíts fmt és validate lépést CI pipeline-hoz (folyamatos integrációs folyamathoz).

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Írj egy plan-alapú `.tftest.hcl` tesztet.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Sorold fel, milyen kockázatot hordozhat az apply-alapú teszt.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Tesztelés és kódminőség

A `terraform fmt` formáz, a `terraform validate` a konfiguráció belső konzisztenciáját ellenőrzi, a `terraform plan` pedig a környezethez viszonyított tervezett változásokat mutatja meg. Ezek eltérő ellenőrzési szintek, egyik sem helyettesíti a másikat.

A `terraform test` a Terraform beépített tesztelési keretrendszere. A tesztfájlok tipikusan `tests` könyvtárban találhatók, kiterjesztésük `.tftest.hcl`. A teszt `run` blokkokkal futtat plan vagy apply műveleteket, majd `assert` kifejezésekkel ellenőrizhet eredményeket. A tesztelés alapfunkciója Terraform v1.6.0-tól érhető el. A mock provider és mock data source képességek verziófüggők; a használt CLI dokumentációját ellenőrizni kell.

```hcl
# tests/basic.tftest.hcl
run "valid_name" {
  command = plan

  variables {
    name = "demo"
  }

  assert {
    condition     = output.name == "demo"
    error_message = "Az output.name értékének demo-nak kell lennie."
  }
}
```

A fenti csak akkor működik, ha a tesztelt konfiguráció deklarálja a `name` változót és a `name` outputot. A tesztfájl a konfiguráció szerződését ellenőrzi; a valós infrastruktúra API-viselkedését nem feltétlenül. A default apply teszt valódi infrastruktúrát is létrehozhat, ezért csak elkülönített tesztfiókban és költségkorlátokkal futtasd.

### Javasolt CI minőségkapu

```bash
terraform fmt -check -recursive
terraform init -backend=false
terraform validate
terraform test
```

A `-backend=false` hasznos lehet statikus ellenőrzéshez, amikor nem akarsz távoli state backendhez csatlakozni. A teljes teszteléshez és planhez ettől eltérő, hitelesített környezet szükséges lehet. Kiegészítő eszközök, például TFLint, Checkov vagy Trivy Terraform-ellenőrzései hasznosak, de ezek szabályait és false positive találatait is értelmezni kell.
