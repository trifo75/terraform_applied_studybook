# C. Beépített függvények

Gyakran használt függvények csoportosítva:

- String: `lower`, `upper`, `title`, `trimspace`, `replace`, `format`, `join`, `split`, `substr`
- Collection: `length`, `concat`, `flatten`, `distinct`, `sort`, `keys`, `values`, `lookup`, `merge`, `contains`, `zipmap`
- Numeric: `min`, `max`, `ceil`, `floor`, `abs`
- Conversion: `tostring`, `tonumber`, `tobool`, `tolist`, `tomap`, `toset`
- Encoding: `jsonencode`, `jsondecode`, `yamlencode`, `yamldecode`, `base64encode`
- Filesystem: `file`, `fileexists`, `fileset`, `templatefile`
- Error handling: `try`, `can`

A lista tanulási segédlet, nem teljes referencia. A pontos argumentumokat és visszatérési típusokat a hivatalos függvényreferenciában ellenőrizd.

## Hivatalos dokumentáció

- [Terraform dokumentáció](https://developer.hashicorp.com/terraform)
- [Terraform CLI dokumentáció](https://developer.hashicorp.com/terraform/cli)

## Kiegészítő tanulási anyag
## Függvényhasználati minták

```hcl
locals {
  normalized = lower(trimspace(var.name))
  unique_tags = distinct(var.tags)
  tag_map = merge(var.default_tags, { Name = local.normalized })
  cidr_subnet = cidrsubnet(var.network_cidr, 8, 2)
  json_config = jsonencode({
    name = local.normalized
    tags = local.tag_map
  })
}
```

A függvényeknél különösen figyelj a bemeneti típusokra és az üres értékekre. A `lookup(map, key, default)` lehetővé teszi hiányzó map-kulcs kezelését; a `try` és `can` a hibás kifejezések kontrollált kezelésére szolgál. A `coalesce` az első nem null/üres értéket választja, de a típusoknak kompatibilisnek kell lenniük. A teljes, naprakész listát a Terraform hivatalos Function Reference tartalmazza.
