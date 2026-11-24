# G. Fájltípusok és projektstruktúrák

| Fájltípus | Szerep |
|---|---|
| `*.tf` | Terraform konfiguráció |
| `*.tf.json` | JSON formátumú Terraform konfiguráció |
| `*.tfvars` | Változóértékek |
| `*.tfvars.json` | JSON változóértékek |
| `*.tftest.hcl` | Terraform tesztfájl |
| `*.tftest.json` | JSON tesztfájl |
| `*.tfplan` | Mentett tervfájl, érzékeny adatot tartalmazhat |
| `.terraform.lock.hcl` | Provider verzió- és checksum lock |
| `.terraform/` | Lokális inicializált munkakönyvtár metaadatai |

A `.terraform/` tipikusan gitignore-ba kerül; a lock fájlt általában verziókezeljük.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Fájltípusok és ajánlott projektstruktúra

| Fájltípus | Szerep |
|---|---|
| `*.tf` | Terraform konfiguráció |
| `*.tf.json` | JSON formátumú konfiguráció |
| `*.tfvars` | Input variable értékek |
| `*.auto.tfvars` | Automatikusan betöltött változófájl |
| `terraform.tfstate` | Local backend state |
| `terraform.tfstate.backup` | Local state korábbi mentése |
| `terraform.lock.hcl` | Provider verziók és checksumok |
| `*.tftest.hcl` | Terraform tesztfájl |
| `*.tfplan` / `tfplan` | Mentett tervfájl |

A `.terraform/` könyvtár inicializálási adatokat és letöltött modulokat/provider pluginokat tartalmazhat; általában nem kerül verziókezelésbe. A lock fájl viszont igen. A `.gitignore` fájlban legalább a helyi state-et, a titkos tfvars fájlokat és a mentett plan fájlokat kezeld körültekintően.
