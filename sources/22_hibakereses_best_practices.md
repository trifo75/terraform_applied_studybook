# 22. Hibakeresés és best practice-ek

*diagnosztika, tervolvasás, állapot és csapatmunka*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform hibakeresés első lépése a teljes hibaüzenet elolvasása. A diagnosztika gyakran megadja a fájlt, sort, resource címet és a provider API válaszát. A hibákat érdemes kategorizálni: HCL szintaxis, típushiba, hiányzó bemenet, provider inicializálás, hitelesítés, jogosultság, API-korlát, state drift vagy függőségi probléma.

A `terraform validate` a konfiguráció statikus hibáit keresi. A `terraform plan` segít megérteni a tervezett változtatást. A `terraform console` kifejezések kipróbálására használható. A `TF_LOG` részletes naplózást kapcsolhat be, de a log érzékeny adatokat is tartalmazhat, ezért csak kontrolláltan és rövid ideig használjuk.

A drift azt jelenti, hogy a valós infrastruktúra eltér a state-ben és konfigurációban elvárttól. A plan ezt sok esetben jelzi. Ne javítsuk a driftet kézi state-szerkesztéssel; előbb értsük meg, melyik állapot a kívánt és mi okozta az eltérést.

## Szintaxis és alap példa

```bash
terraform fmt -recursive
terraform validate
terraform plan
terraform console
```

A console-ban például kipróbálható:

```hcl
> lower("Terraform")
"terraform"
> merge({env = "dev"}, {team = "platform"})
{
  "env" = "dev"
  "team" = "platform"
}
```

## Részletes magyarázat

Best practice összefoglaló:
- Tartsd kicsiben és egyértelmű felelősségűen a modulokat.
- Adj típusokat, leírásokat és validációt a bemenetekhez.
- Rögzíts providerverziót és commitold a lock fájlt.
- Használj remote state-et, lockingot és korlátozott hozzáférést csapatmunkában.
- Mindig olvasd el a plant; külön figyelj törlésre és cserére.
- Ne használj `-target`-et normál telepítési stratégiaként.
- Teszteld a modulok logikáját és a pipeline-t.
- Dokumentáld a futtatási előfeltételeket és a rollback eljárást.

A jó Terraform kód nem attól jó, hogy rövid, hanem attól, hogy a szándéka, függőségei és kockázatai érthetők.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Diagnosztizálj egy szándékosan hibás HCL konfigurációt.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Magyarázd el a drift fogalmát és kezelésének lépéseit.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Készíts csapatszintű Terraform best-practice ellenőrzőlistát.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Hibakeresési módszertan

A hibakeresésnél először azonosítsd, melyik rétegben keletkezik a hiba:

1. **HCL szintaxis:** hiányzó zárójel, hibás idézőjel, rossz blokkstruktúra.
2. **Típus és kifejezés:** nem megfelelő típus, hiányzó attribútum, érvénytelen index.
3. **Inicializálás:** hiányzó provider, modul vagy backend inicializálás.
4. **Provider/API:** hitelesítési hiba, jogosultság, kvóta, régió vagy API-korlátozás.
5. **State:** drift, lock, rossz workspace vagy hibás erőforrás-cím.
6. **Életciklus:** nem módosítható attribútum, függőségi probléma vagy sikertelen törlés.

Hasznos parancsok:

```bash
terraform fmt -check -recursive
terraform validate
terraform providers
terraform state list
terraform state show 'local_file.example'
terraform show
terraform console
```

A `TF_LOG` változóval részletes naplózás kapcsolható be, például `TF_LOG=DEBUG`. A napló érzékeny értékeket is tartalmazhat, ezért csak ellenőrzött környezetben használd, és megosztás előtt tisztítsd meg.

### Best practice-ek

- Egy root module egy jól meghatározott telepítési egységet kezeljen.
- Provider- és modulverziókat rögzíts.
- Készíts tervet és vizsgáld át apply előtt.
- State-et ne módosíts kézzel; a `terraform state` parancsokat is körültekintően használd.
- Ne használd a `-target` kapcsolót normál telepítési stratégiaként; elsősorban kivételes helyreállítási helyzetekre való.
- A modulok legyenek kicsik, világos bemenetekkel és kimenetekkel.
- Automatizálásban ne engedélyezz korlátlan párhuzamos apply futásokat ugyanazon state-en.
