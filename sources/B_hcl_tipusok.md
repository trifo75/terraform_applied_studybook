# B. HCL típusok és szintaxis

| Típus | Leírás | Példa |
|---|---|---|
| `string` | Szöveg | `"hello"` |
| `number` | Szám | `42`, `3.14` |
| `bool` | Logikai | `true` |
| `list(T)` | Rendezett sorozat | `["a", "b"]` |
| `set(T)` | Egyedi elemek halmaza | `toset(["a", "b"])` |
| `map(T)` | Kulcs-érték map | `{env = "dev"}` |
| `object({...})` | Névvel rendelkező mezők | `object({name=string})` |
| `tuple([...])` | Pozíció szerinti típusok | `tuple([string, number])` |

A `null` hiányzó értéket jelölhet. A komplex típusoknál a pontos séma segíti a modulok használatát és validálását.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Típusok rövid referencia és értelmezés

| Típus | Jelentés | Példa |
|---|---|---|
| `string` | Szöveg | `"hello"` |
| `number` | Egész vagy lebegőpontos szám | `42`, `3.14` |
| `bool` | Logikai érték | `true` |
| `list(T)` | Rendezett, azonos típusú elemek | `list(string)` |
| `set(T)` | Egyedi elemek, sorrend nélkül | `set(number)` |
| `map(T)` | Szövegkulcsos, azonos típusú értékek | `map(string)` |
| `object({...})` | Névvel ellátott attribútumok | `object({name=string})` |
| `tuple([...])` | Pozícióhoz kötött, akár eltérő típusok | `tuple([string, number])` |
| `any` | Típuskorlátozásban kikövetkeztetendő típus | `any` |
| `null` | Érték hiánya | `null` |

A collection típusok elemeinek közös típusa van; a structural típusok mezői vagy pozíciói eltérő típusúak lehetnek. Az `any` nem általános típus, amely minden műveletet megenged.
