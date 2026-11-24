# 19. Security

*secrets, least privilege, state protection, supply chain*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform biztonsága több rétegből áll: a fejlesztői gép, a CI/CD futtatókörnyezet, a provider hitelesítés, a state tárolása, a modul- és providerforrások, valamint az infrastruktúra jogosultságai mind részei a kockázati felületnek.

A least privilege elv szerint a Terraform futtató identitása csak a szükséges műveletekre kapjon jogosultságot. Éles környezetben előnyös a rövid életű credential vagy workload identity, ahol ez elérhető. Hosszú életű access key-ket ne tároljunk repositoryban vagy tfvars fájlban.

A `sensitive = true` a CLI kimenet bizonyos részeit maszkolja, de a state és a tervfájl továbbra is tartalmazhatja az értéket. A remote backend (távoli backend) hozzáférését, titkosítását, verziózását és naplózását külön kell konfigurálni.

## Szintaxis és alap példa

```hcl
variable "api_token" {
  type        = string
  sensitive   = true
  description = "Titokkezelőből vagy biztonságos futtatókörnyezetből átadott token."
}
```

A változó deklarációja nem oldja meg a titok tárolását. A tényleges titokforrás lehet CI secret store vagy dedikált secrets manager; a pontos integráció a környezettől függ.

## Részletes magyarázat

Biztonsági ellenőrzőlista:
- Ne commitoljunk state-et, tervfájlt vagy titkos tfvars fájlt.
- Korlátozzuk a backend olvasási/írási jogosultságait.
- Rögzítsük és vizsgáljuk felül a provider- és modulverziókat.
- A pull request pipeline legalább fmt, validate és plan ellenőrzést végezzen.
- A prod apply legyen jóváhagyáshoz és megfelelő identitáshoz kötve.
- A naplókban ne jelenjenek meg titkok.

A titok `sensitive` jelölése hasznos, de nem biztonsági határ.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Írj biztonsági ellenőrzőlistát egy csapat Terraform repositoryjához.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Magyarázd el a `sensitive` korlátait.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Tervezd meg a CI identitás minimális jogosultságait.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Biztonság: fenyegetési felület és védekezési rétegek

A Terraform biztonságát nem lehet egyetlen `sensitive = true` beállítással megoldani. A védelem a teljes életciklust érinti: forráskód, provider letöltés, hitelesítés, plan artifact, state backend, CI napló, hozzáférési jogosultság és infrastruktúra API.

### Alapelvek

1. **Least privilege:** a Terraform futtató identitás csak a szükséges erőforrásokon és műveleteken rendelkezzen jogosultsággal.
2. **Rövid életű hitelesítés:** ahol lehet, használj OIDC vagy workload identity alapú hitelesítést statikus kulcsok helyett.
3. **State-védelem:** titkosítás nyugalmi állapotban és átvitel közben, szigorú IAM, verziózás és audit.
4. **Secret-kezelés:** ne commitolj jelszót, API-kulcsot vagy privát kulcsot a repository-ba.
5. **Provider supply chain:** rögzíts provider verziókat és ellenőrzőösszegeket a lock fájlban.
6. **Plan review:** vizsgáld meg a törléseket, cseréket, nyilvános hálózati hozzáférést és jogosultságváltozásokat.

### Sensitive és ephemeral

A `sensitive = true` elsősorban a megjelenítést korlátozza; az érték a state-ben vagy plan fájlban ettől még szerepelhet. Az ephemeral változók és értékek bizonyos Terraform-verziókban és meghatározott kontextusokban lehetővé teszik, hogy egyes érzékeny adatok ne kerüljenek a state-be vagy a plan fájlba. Ez nem általános titokeltüntető kapcsoló: csak a dokumentált ephemeral kontextusokban használható, és a provider/resource támogatásának is meg kell felelnie.

### Biztonsági ellenőrzőlista

- `.tfstate`, mentett plan és titkos `.tfvars` fájl ne kerüljön nyilvános repository-ba.
- A `.gitignore` csak kiegészítő védelem; a titkokat eleve ne írd a forrásba.
- CI logban ne jeleníts meg környezeti változókat és tokeneket.
- A Terraform futtató identitás ne legyen fiókszintű admin, ha nincs rá indok.
- A provider és modul forrásait megbízható registryből vagy ellenőrzött repositoryból szerezd be.
