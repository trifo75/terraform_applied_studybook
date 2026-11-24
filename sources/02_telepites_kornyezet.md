# 2. Terraform telepítése és fejlesztői környezet kialakítása

*CLI telepítés, verziókezelés, editor és projektmappa*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform CLI telepíthető a HashiCorp hivatalos csomagjaiból, operációs rendszerhez illeszkedő csomagkezelővel vagy manuálisan letöltött binárisból. Csapatmunkában ne a „latest” verzióra hagyatkozzunk: a használt verziót dokumentáljuk, és a projektben `required_version` korlátozással jelezzük. A verziófrissítés legyen tudatos, tesztelt változtatás.

A telepítés után a `terraform version` mutatja a CLI verzióját. A `terraform -help` és az egyes parancsok `-help` opciója a helyi CLI súgója. A fejlesztői környezethez HCL-szintaxist ismerő szerkesztő, Git és terminál szükséges. A szerkesztő kiegészítései kényelmesebbé teszik a munkát, de nem helyettesítik a Terraform saját validációját.

A projektet külön könyvtárban tartsuk. A `.terraform/` könyvtár letöltött modulokat, providereket és helyi metaadatokat tartalmaz; rendszerint nem kerül verziókezelésbe. A `.terraform.lock.hcl` ezzel szemben fontos: rögzíti a provider verziókat és ellenőrző összegeket, ezért általában commitoljuk.

## Szintaxis és alap példa

```hcl
terraform {
  required_version = ">= 1.6.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
```

A `required_version` a Terraform CLI-re vonatkozik, míg a `required_providers` a provider forrását és verziótartományát deklarálja. A `~> 2.5` úgynevezett pessimistic constraint (óvatos verziókorlátozás): a 2.5-ös verziótól enged frissítést a 3.0.0 előtti verziókra; a pontos szemantika miatt mindig ellenőrizzük a verziókényszerítés dokumentációját. Az `init` létrehozza vagy frissíti a lock fájlt.

## Részletes magyarázat

Ajánlott kezdő munkamenet:

```bash
terraform version
mkdir terraform-lab
cd terraform-lab
# Hozd létre a main.tf fájlt.
terraform fmt -recursive
terraform init
terraform validate
terraform plan
```

Ne másoljunk be hozzáférési kulcsokat a konfigurációba vagy a Git repositoryba. A felhőszolgáltató hitelesítését a szolgáltató által támogatott credential chain (hitelesítési forrásokat meghatározott sorrendben vizsgáló mechanizmus) vagy workload identity (a munkaterheléshez rendelt identitás) használatával, illetve környezeti változókon vagy CLI-profilon keresztül oldjuk meg.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Ellenőrizd a telepített Terraform verziót.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Hozz létre projektmappát és `main.tf` fájlt.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Magyarázd el a `.terraform/` és `.terraform.lock.hcl` szerepének különbségét.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Telepítés, verziók és reprodukálhatóság

A Terraform CLI önálló bináris program. A provider-ek külön pluginok, amelyeket a `terraform init` tölt le a konfigurációban deklarált követelmények alapján. A két verziót ezért külön kell kezelni: a Terraform Core verziója és az egyes provider-ek verziói nem ugyanazt jelentik.

A projektben a `required_version` korlátozza, milyen Terraform CLI-verzióval dolgozhatunk. A `required_providers` blokk a provider-forrásokat és verziótartományokat rögzíti. A `terraform.lock.hcl` pedig a ténylegesen kiválasztott provider-verziókat és ellenőrzőösszegeket tartalmazza. A lock fájlt általában verziókezelőbe kell tenni, mert így a csapattagok és a CI-rendszer azonos provider-változatokat telepíthetnek.

```hcl
terraform {
  required_version = ">= 1.6.0, < 2.0.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
```

A fenti korlátozás csak példa: egy projektben a verziótartományt a ténylegesen tesztelt Terraform- és provider-verziókhoz kell igazítani. A `~>` operátor pessimistic constraint, azaz óvatos verziókorlátozás. A `~> 2.5` a 2.5-ös verziótól a 3.0.0 előtti kiadásokat engedi; a `~> 2.5.0` szűkebb tartományt, a 2.5.x kiadásokat engedi.

### Fejlesztői környezet

Érdemes olyan szerkesztőt választani, amely támogatja a HCL szintaxiskiemelését, formázását és lehetőség szerint a Terraform Language Server funkcióit. A szerkesztő kényelmi eszköz; a végső ellenőrzést továbbra is a Terraform CLI végzi.

Javasolt projektindítási ellenőrzés:

```bash
terraform version
terraform fmt -check -recursive
terraform init
terraform validate
```

A `validate` csak inicializált munkakönyvtárban végez teljes konfiguráció-ellenőrzést, és nem helyettesíti a `plan` futtatását. A validáció nem bizonyítja, hogy a felhőben létrehozható az erőforrás: például jogosultság, kvóta, régió vagy API-oldali korlátozás csak a tervezés vagy végrehajtás során derülhet ki.

### Gyakori telepítési hibák

- **PATH probléma:** a shell nem találja a `terraform` binárist. Ellenőrizd a telepítési helyet és a PATH változót.
- **Provider letöltési hiba:** proxy, DNS, tűzfal vagy registry-elérési probléma is okozhatja.
- **Verzióütközés:** a konfiguráció által megkövetelt Terraform-verzió nem egyezik a telepítettel.
- **Lock fájl eltérése:** több platformon dolgozó csapatnál szükség lehet a további platformok checksumjainak rögzítésére a `terraform providers lock` paranccsal.

Ne töröld automatikusan a `.terraform` könyvtárat vagy a lock fájlt hibakereséskor. Előbb értsd meg, miért keletkezett az eltérés: a lock fájl törlése nem ártalmatlan „gyorsjavítás”, mert új provider-verzió kiválasztásához vezethet.
