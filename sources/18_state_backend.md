# 18. State és Backend

*state mapping, remote backend (távoli backend), locking, migration*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform state a konfigurációban deklarált resource-példányokat (kezelt infrastruktúra-objektumokat) a valós infrastruktúra objektumaihoz rendeli. Emellett metaadatokat és attribútumértékeket tárol, amelyek szükségesek a következő terv elkészítéséhez. A state nem egyszerű gyorsítótár: a Terraform működésének központi eleme.

A helyi backend fájlban tárolja a state-et, ami tanulásra megfelelő, csapatmunkára viszont kockázatos. A remote backend (távoli backend) központi tárolást és több esetben state lockingot (a state párhuzamos módosításának zárolását) biztosít. A lock megakadályozza, hogy két párhuzamos művelet egyszerre módosítsa ugyanazt a state-et. A konkrét locking támogatás backendfüggő.

A state érzékeny adatokat is tartalmazhat. Hozzáférését korlátozni, titkosítását és mentését biztosítani kell. Ne commitoljuk a state-et Gitbe, és ne szerkesszük kézzel JSON-ként.

## Szintaxis és alap példa

```hcl
terraform {
  backend "s3" {
    bucket       = "company-terraform-state"
    key          = "network/dev/terraform.tfstate"
    region       = "eu-central-1"
    use_lockfile = true
  }
}
```

Ez AWS S3 backend példa; a bucketet előzetesen létre kell hozni, és megfelelő IAM jogosultság szükséges. A `use_lockfile` elérhetőségét és a backend pontos beállításait a használt Terraform verzió hivatalos dokumentációjában ellenőrizni kell.

## Részletes magyarázat

Backend konfiguráció módosítása után `terraform init` szükséges. Átálláskor a CLI felajánlhatja a state migrálását. Mielőtt migrálunk, készítsünk ellenőrzött biztonsági mentést, és egyeztessünk a csapattal, hogy senki ne fusson párhuzamos műveletet.

A `terraform state list`, `terraform state show` és más state parancsok a state vizsgálatát segítik. A `state mv`, `state rm` és import műveletek hatással vannak a Terraform nyilvántartására; csak pontos erőforráscímekkel és mentés után használjuk őket.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Magyarázd el, milyen kapcsolatot tárol a state.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Sorold fel a remote backend (távoli backend) három előnyét.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Tervezz biztonságos migrációt local backendről remote backendre.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## State és backend mélyebben

A state a Terraform által kezelt objektumok és a konfigurációs címek közötti leképezést (mapping) tárolja. Emellett olyan attribútumokat is tartalmazhat, amelyekre a következő terv elkészítéséhez szükség van. A state nem pusztán cache, és nem szabad kézzel szerkeszteni. A state-ben érzékeny értékek is lehetnek, ezért hozzáférését és mentését védeni kell.

A backend határozza meg, hol és hogyan tárolódik a state. A local backend fájlt használ; a remote backend központi, több felhasználó által elérhető tárolót és gyakran locking mechanizmust biztosít. Csapatmunkánál a távoli state, a zárolás és a hozzáférés-szabályozás alapvető.

### Locking és párhuzamos futtatás

A state locking megakadályozza, hogy több Terraform-folyamat egyszerre írja ugyanazt a state-et, ha az adott backend támogatja a zárolást. Ne használd rutinszerűen a `-lock=false` kapcsolót: ezzel megszüntetheted a párhuzamos módosítás elleni védelmet. Ha beragadt lockot látsz, először győződj meg róla, hogy a másik futás valóban befejeződött. A `force-unlock` csak a megfelelő lock ID-val és indokolt esetben használható.

### S3 backend megjegyzés

A jelenlegi Terraform S3 backend dokumentáció `use_lockfile = true` beállítással natív S3 lock fájl alapú zárolást támogat. A korábbi DynamoDB-alapú lockolás elavultnak jelölt megoldás; új kialakításnál a hivatalos, használt Terraform-verzióhoz tartozó backend dokumentációt kell követni. Az S3 bucket verziózása, titkosítása, szigorú IAM-jogosultságai és a state objektumhoz való hozzáférés korlátozása külön biztonsági kontroll.

### Backend migráció

Backend-változtatáskor ne töröld a régi state-et. A `terraform init -migrate-state` megpróbálja átmásolni a meglévő state-et az új backendbe; a `-reconfigure` ezzel szemben a korábbi backend-konfigurációt figyelmen kívül hagyva inicializál újra. A két kapcsoló nem felcserélhető. Migráció előtt készíts biztonságos mentést és ellenőrizd, hogy nincs aktív Terraform-futás.
