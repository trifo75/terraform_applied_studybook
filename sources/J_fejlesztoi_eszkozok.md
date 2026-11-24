# J. Fejlesztői eszközök és ellenőrző parancsok

Alapellenőrzési parancsok:

```bash
terraform version
terraform fmt -recursive
terraform fmt -check -recursive
terraform init
terraform validate
terraform plan
terraform show
terraform providers
terraform output
terraform state list
terraform test
```

A kiegészítő eszközök között lehet HCL-kompatibilis editor, Git, CI rendszer, Terraform lintelő, konfigurációs biztonsági scanner és policy-as-code eszköz. A választott eszközök verzióját és konfigurációját dokumentáld. Az eszközök jelzéseit vizsgáld felül: a statikus scanner nem feltétlenül ismeri a futó környezet teljes kontextusát.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Fejlesztői minőségkapu

```bash
terraform version
terraform fmt -check -recursive
terraform init -backend=false
terraform validate
terraform providers
terraform test
```

A parancsok különböző szinteket ellenőriznek. A `fmt` formáz, a `validate` a konfigurációt vizsgálja, a `providers` a provider-függőségeket listázza, a `test` pedig tesztfájlokat futtat. A `-backend=false` elkerüli a backend inicializálását, de nem minden projektben helyettesíti a teljes initet.

Kiegészítő eszközök:
- **TFLint:** Terraform és provider-specifikus lint szabályok.
- **Checkov:** konfigurációs biztonsági és compliance ellenőrzés.
- **Trivy:** IaC és konténeres környezetek biztonsági vizsgálata.
- **terraform-docs:** modul dokumentációjának generálása.
- **pre-commit:** ellenőrzések automatikus futtatása commit előtt.

Az eszközök kimenete nem automatikus igazság: vizsgáld meg a szabályt, a kontextust és az esetleges false positive találatokat.
