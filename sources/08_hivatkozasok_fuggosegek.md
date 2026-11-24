# 8. Hivatkozások és függőségek

*resource references, implicit és explicit dependency*

## Tanulási célok

A fejezet végére értsd a fogalmak szerepét, ismerd fel a kapcsolódó HCL mintákat, és tudd azokat kis, ellenőrizhető Terraform-projektben alkalmazni.

## Elméleti háttér

A Terraform gráfot épít a konfigurációból. Ha egy resource argumentumában egy másik resource attribútumára hivatkozunk, a Terraform implicit függőséget állapít meg. Például egy hálózati alhálózat hivatkozhat a VPC azonosítójára. A függőségi gráf határozza meg, mely műveletek végezhetők párhuzamosan és melyeknek kell sorrendben történniük.

Az implicit függőség előnyösebb, mert a valódi adatkapcsolatot fejezi ki. Az `depends_on` explicit függőséget ad meg olyan esetekre, amikor a konfigurációban nincs közvetlen attribútumhivatkozás, de a műveleti sorrend mégis fontos. Túlhasználata feleslegesen sorosíthatja a tervet és elrejtheti a valódi adatkapcsolatot.

## Szintaxis és alap példa

```hcl
resource "local_file" "config" {
  filename = "${path.module}/config.txt"
  content  = "generated"
}

resource "local_file" "marker" {
  filename = "${path.module}/marker.txt"
  content  = local_file.config.filename
}
```

## Részletes magyarázat

A második resource a `local_file.config.filename` értékét használja, ezért implicit függőség jön létre. A Terraform előbb létrehozza vagy frissíti a konfigurációs fájlt, majd a marker fájlt.

A resource cím, például `local_file.config`, konfigurációs hivatkozás. A `count` és `for_each` használatakor az egyes példányok címe indexet vagy kulcsot is tartalmazhat.

## Gyakori hibák és ellenőrzési szempontok

- Ne keverd össze a Terraform Core nyelvi elemeit a provider-specifikus argumentumokkal.
- Ellenőrizd a változók és kifejezések típusát.
- Olvasd el a `terraform plan` teljes kimenetét, különösen a törléseket és cseréket.
- Verziófüggő viselkedésnél ellenőrizd a használt CLI és provider dokumentációját.
- Titkokat ne írj konfigurációba, state-et vagy tervfájlt ne tegyél nyilvános repositoryba.

## Gyakorlófeladatok

### 1. feladat

Rajzold fel két egymásra hivatkozó resource függőségi gráfját.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 2. feladat

Keress olyan példát, ahol `depends_on` valóban indokolt.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.
### 3. feladat

Magyarázd el az implicit függőség előnyét.

**Ellenőrzés:** először futtasd a `terraform fmt` és a `terraform validate` parancsot. Ezután készíts `terraform plan` tervet, és vesd össze a tervben látható műveleteket a feladatban megadott elvárt eredménnyel. A feladat teljes megoldását és a parancsok kimenetének értelmezését a fejezet végén találod.

## Összefoglalás

A fejezet központi fogalmait úgy tekintsd át, hogy egy új projektben önállóan fel tudd ismerni, mikor és miért kell alkalmazni őket. A Terraform konfiguráció akkor karbantartható, ha a szándék, a bemenetek, a függőségek és a változtatások következményei egyértelműek.

## Hivatalos dokumentáció

- [Terraform dokumentáció – Configuration Language](https://developer.hashicorp.com/terraform/language)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Függőségi gráf és műveleti sorrend

A Terraform a konfigurációból függőségi gráfot (dependency graph) épít. Ha egy resource argumentuma egy másik resource attribútumára hivatkozik, a Terraform ezt implicit függőségként felismeri.

```hcl
resource "local_file" "config" {
  filename = "${path.module}/config.txt"
  content  = "configuration"
}

resource "local_file" "checksum" {
  filename = "${path.module}/checksum.txt"
  content  = filesha256(local_file.config.filename)
}
```

A `checksum` resource a `config` resource attribútumára hivatkozik, ezért a Terraform előbb a `config` létrehozását tervezi. Ez jobb, mint kézzel sorrendbe rendezni a blokkokat: a fájlok sorrendje nem határozza meg a végrehajtás sorrendjét.

Az `depends_on` explicit függőséget ad meg, amikor a kapcsolat nem jelenik meg közvetlen attribútumhivatkozásként. Használd takarékosan. A túl sok explicit függőség csökkenti a párhuzamosíthatóságot és túl szigorú sorrendet kényszeríthet a Terraformra. A függőségnek valódi működési okon kell alapulnia, például egy API-szolgáltatás csak egy másik erőforrás létrejötte után érhető el.

A `terraform graph` parancs a függőségi gráf vizualizálásához ad Graphviz formátumú kimenetet. Nagy projektnél a teljes gráf összetett lehet; hibakereséskor érdemes a releváns modulra vagy erőforrásokra összpontosítani.
