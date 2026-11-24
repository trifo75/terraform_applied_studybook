# D. Operátorok és feltételes kifejezések

- Aritmetikai: `+`, `-`, `*`, `/`, `%`
- Összehasonlító: `==`, `!=`, `<`, `>`, `<=`, `>=`
- Logikai: `&&`, `||`, `!`
- Feltételes: `condition ? true_value : false_value`
- Indexelés: `list[0]`, `map["key"]`
- Attribútum: `object.attribute`

Összetett feltételt bonts több, jelentéses locals értékre, ha így könnyebben olvasható. A típuskompatibilitást a feltételes kifejezés két ágánál is tartsd szem előtt.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Operátorok és kiértékelés

| Csoport | Operátorok | Példa |
|---|---|---|
| Aritmetikai | `+ - * / %` | `var.count * 2` |
| Összehasonlító | `== != < > <= >=` | `var.size >= 2` |
| Logikai | `&& || !` | `var.enabled && var.region == "eu"` |
| Feltételes | `? :` | `var.enabled ? "yes" : "no"` |
| Tagelérés | `.` | `var.settings.name` |
| Indexelés | `[]` | `var.zones[0]` |
| Zárójelezés | `()` | `(var.a + var.b) * 2` |

A feltételes kifejezés két ágának kompatibilis típust kell eredményeznie. A logikai kifejezésekben ne támaszkodj olyan rövidzáras kiértékelésre, amely egy hibás indexelést vagy hiányzó attribútumot „elrejtene”; használj `try`, `can` vagy kifejezett feltételt.
