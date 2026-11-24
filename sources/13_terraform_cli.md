# 13. Terraform CLI részletesen

*init, plan, apply, destroy, console, state, import*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform CLI parancsai a konfiguráció előkészítésétől az állapotkezelésig lefedik a napi munkát. Az `init` inicializálja a könyvtárat; az `fmt` formáz; a `validate` ellenőriz; a `plan` tervet készít; az `apply` végrehajt; a `destroy` törlési tervet alkalmaz. A `show` tervet vagy state-et jelenít meg, az `output` output értékeket kérdez le, a `console` interaktív kifejezéskiértékelő.

A `state` alparancsokkal a Terraform által nyilvántartott erőforrások vizsgálhatók és bizonyos műveletek biztonságosan elvégezhetők. A state közvetlen JSON-szerkesztése nem ajánlott. Az `import` meglévő infrastruktúrát kapcsol a konfigurációhoz és state-hez; az importálás után a konfigurációt is meg kell írni, hogy a kívánt állapot egyértelmű legyen.

## Szintaxis és alap példa

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan -out=change.tfplan
terraform show change.tfplan
terraform apply change.tfplan
terraform output
terraform state list
```

## Részletes magyarázat

Éles pipeline-ban gyakori minta, hogy a plan külön lépésben készül, artefaktumként megőrzik, review után pedig ugyanazt a tervet alkalmazzák. A tervfájl érzékeny adatot tartalmazhat. A `-target` diagnosztikai vagy kivételes műveletekhez való, nem helyettesíti a helyes konfigurációs modell kialakítását.

A `-replace` egy adott resource példány cseréjét kéri. A `-refresh-only` a state frissítésére szolgáló tervmód. Minden ilyen kapcsoló hatását a használt CLI verzió dokumentációjában ellenőrizzük.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Készíts tervet mentett fájlba és jelenítsd meg.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Vizsgáld meg a state listát.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Magyarázd el, miért veszélyes a tervfájl nyilvános megosztása.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Haladó CLI-munkafolyamatok

A fő parancsok mellett fontos a `terraform show`, `terraform output`, `terraform state`, `terraform providers`, `terraform console`, `terraform graph` és `terraform import` is. Ezek különböző feladatokat oldanak meg; nem helyettesítik egymást.

- `terraform show`: state vagy mentett plan olvasható megjelenítése.
- `terraform output`: root module output értékeinek lekérdezése.
- `terraform state list/show`: a state-ben nyilvántartott címek és részletek vizsgálata.
- `terraform providers`: a konfiguráció provider-követelményeinek megtekintése.
- `terraform console`: HCL kifejezések interaktív kipróbálása.
- `terraform graph`: függőségi gráf előállítása.
- `terraform import`: meglévő infrastruktúra-objektum hozzárendelése Terraform resource címhez.

Az import nem feltétlenül generál teljes konfigurációt. A resource blokkot a megfelelő provider típusával és argumentumaival neked kell kialakítanod, majd tervvel ellenőrizni, hogy a konfiguráció és a már létező objektum összhangban van-e. A modern importálási munkafolyamatok konfigurációvezérelt import blokkot is használhatnak; a választott módszert a használt Terraform-verzió dokumentációja szerint válaszd.

### Mentett plan

```bash
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
```

A mentett terv az adott konfiguráció, bemenetek és tervezési pillanat eredményét rögzíti. Ha a környezet közben megváltozik, a terv elavulhat, és az apply visszautasíthatja. A plan fájlt érzékeny adatokat tartalmazó artifactként kell védeni.
