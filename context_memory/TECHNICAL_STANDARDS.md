# Technikai és forrásellenőrzési szabványok

## Források
A Terraform Core, HCL és CLI elsődleges forrása a hivatalos HashiCorp dokumentáció: https://developer.hashicorp.com/terraform/docs. Provider-specifikus állításhoz az adott provider hivatalos, verziózott dokumentációját használd.

## Verziók
- A példáknál deklaráld a Terraform Core-verziót vagy verziótartományt.
- A provider verzióját külön rögzítsd; a provider nem a Terraform Core része.
- Ne találgass verziószámot: ellenőrizd hivatalos release note és registry adatok alapján.
- Jelezd a funkciók minimális verzióját és a verziók közötti eltéréseket.

## Ellenőrzés
Ahol lehetséges, futtasd: `terraform fmt -check`, `terraform init -backend=false`, `terraform validate`, majd biztonságos tesztkörnyezetben `terraform plan`. Csak azt állítsd, amit ténylegesen ellenőriztél; a parancsok alkalmazhatósága projektfüggő.

## Biztonság
Ne kérj és ne tárolj valódi cloud secretet. State és plan fájlok érzékeny adatot tartalmazhatnak. Költséges vagy módosító példáknál jelezd a kockázatot és a destroy/takarítás lépéseit.
