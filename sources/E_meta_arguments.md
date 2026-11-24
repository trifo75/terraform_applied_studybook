# E. Meta-arguments és Lifecycle

A meta-argumentumok a Terraform Core működését befolyásolják, nem csupán egy adott provider resource argumentumai.

- `count`: számozott resource/module példányok.
- `for_each`: kulcs alapján címzett példányok.
- `depends_on`: explicit függőség.
- `provider`: kiválasztott provider konfiguráció.
- `lifecycle`: életciklus-viselkedés.
- `provisioner`: külső művelet végrehajtása; csak indokolt esetben, mert kevésbé deklaratív.

A lifecycle fontos elemei közé tartozik a `create_before_destroy`, `prevent_destroy`, `ignore_changes` és `replace_triggered_by`. Ezek használata befolyásolhatja a tervet; minden esetben ellenőrizd a konkrét hatást.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Meta-arguments és lifecycle

A meta-argumentumok a Terraform Core viselkedését befolyásolják, nem az adott provider API-jának szokásos resource attribútumai.

- `count`: számozott példányok létrehozása.
- `for_each`: map vagy set alapján névvel azonosított példányok.
- `depends_on`: explicit függőség.
- `provider`: provider konfiguráció kiválasztása.
- `lifecycle`: létrehozási/törlési viselkedés finomhangolása.
- `provisioner`: külső parancs vagy kapcsolat művelet; csak indokolt, kivételes esetben.

A `lifecycle` blokk gyakori beállításai:
- `create_before_destroy`: csere esetén előbb próbálja létrehozni az új példányt.
- `prevent_destroy`: megakadályozza a Terraform által tervezett törlést, amíg a szabály érvényben van.
- `ignore_changes`: bizonyos attribútumváltozásokat figyelmen kívül hagy a tervezésnél.
- `replace_triggered_by`: más erőforrás változása cserét válthat ki.

Ezeket ne használd a hibás konfiguráció elfedésére. A `prevent_destroy` például nem véd meg attól, hogy a resource blokkot eltávolítsd a konfigurációból, ha ezzel a lifecycle szabály is eltűnik.
