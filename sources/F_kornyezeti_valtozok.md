# F. Környezeti és TF_* változók

A `TF_VAR_name` környezeti változó a `name` input variable értékét adhatja meg. A Terraform CLI viselkedését több `TF_CLI_ARGS` és `TF_LOG` jellegű változó is befolyásolhatja. A provider saját környezeti változókat is használhat hitelesítéshez és konfigurációhoz.

A pontos lista és prioritás verzió- és providerfüggő. Ne helyezz titkot shell historyba, CI logba vagy nyilvános pipeline változóba. A környezeti változók öröklődhetnek gyermekfolyamatokra, ezért a futtatókörnyezet jogosultságait is ellenőrizd.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Környezeti változók

| Változó | Szerep |
|---|---|
| `TF_VAR_name` | `name` nevű input variable értéke |
| `TF_LOG` | Részletes naplózás szintje |
| `TF_LOG_PATH` | Naplófájl helye |
| `TF_INPUT` | Interaktív bemenet engedélyezése |
| `TF_CLI_ARGS` | Alapértelmezett CLI argumentumok |
| `TF_DATA_DIR` | Terraform munkakönyvtári adatok helye |
| `TF_WORKSPACE` | Kiválasztott workspace környezeti beállítása |

A változók viselkedése és az egyes CLI-verziók támogatása változhat. A `TF_LOG` debug információkat írhat ki, amelyek titkos adatot is tartalmazhatnak. A `TF_CLI_ARGS` kényelmes automatizálásban, de ne használd úgy, hogy a parancs tényleges viselkedése rejtve maradjon a futtató számára.
