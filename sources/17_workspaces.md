# 17. Workspaces és környezetkezelés

*workspace, state elkülönítés és környezeti stratégia*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform CLI workspace ugyanazon konfigurációhoz több külön state-et biztosít. Hasznos lehet például rövid életű fejlesztői példányokhoz. A workspace azonban nem teljes környezetizoláció: ugyanaz a konfiguráció, backend és jogosultsági környezet maradhat érvényben, ezért nem minden esetben megfelelő dev/test/prod elválasztásra.

A környezetek elkülönítésére gyakori megoldás külön root module, külön backend/state és külön jogosultságok használata. A workspace választásakor gondoljuk végig, mennyire különbözik az egyes környezetek infrastruktúrája, hozzáférése és életciklusa.

## Szintaxis és alap példa

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select dev
terraform workspace show
```

```hcl
locals {
  environment = terraform.workspace
}
```

## Részletes magyarázat

A `terraform.workspace` az aktuális workspace nevét adja. Ezt használhatjuk névképzésre vagy feltételes beállításra, de ne hagyatkozzunk kizárólag erre a prod biztonságos elkülönítésére.

A workspace váltása előtt mindig ellenőrizzük, melyik state-et fogjuk módosítani. A rossz workspace-ben futtatott `apply` valós infrastruktúrát érinthet.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Hozz létre `dev` és `test` workspace-t.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Jelenítsd meg a `terraform.workspace` értékét outputként.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Írd le, miért nem elég önmagában a workspace az éles környezet védelméhez.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Workspaces és környezetstratégia

A CLI workspace-ek ugyanazon konfigurációhoz külön state-et biztosítanak. Ez hasznos lehet azonos felépítésű, rövid életű példányokhoz, például fejlesztői tesztkörnyezetekhez. A workspace azonban nem jelent teljes környezeti izolációt: a konfiguráció, a backend és a hozzáférési modell közös lehet, ezért a production és a development környezetek szétválasztására önmagában nem mindig megfelelő.

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select dev
terraform workspace show
```

A `terraform.workspace` kifejezéssel a konfiguráció lekérdezheti az aktuális workspace nevét. Ezt használhatod címkékhez vagy egyszerű paraméterezéshez, de ne építs kritikus biztonsági döntést kizárólag a workspace névre.

Nagyobb rendszereknél gyakori megoldás a külön root module vagy külön könyvtár környezetenként, elkülönített state backenddel és jogosultságokkal. A választás a környezetek közötti eltérés mértékétől, a csapatműködéstől, a hozzáférési követelményektől és a release folyamattól függ.
