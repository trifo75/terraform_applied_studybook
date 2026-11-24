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

## Bináris artefaktumok Base64-formátuma
- A bináris fájlokat a projekt GitHub-folyamatában Base64-kódolt `.b64` szövegfájlként tároljuk, ha a közvetlen bináris feltöltés nem támogatott vagy a megállapodott munkafolyamat ezt írja elő.
- A Base64-kódolás az eredeti fájl bájtjait reprezentálja. A kimenet szöveges UTF-8 fájl, amelynek neve az eredeti fájl nevét és kiterjesztését, majd a `.b64` utótagot tartalmazza.
- A `.b64` nem az eredeti fájlformátum: például a `.pdf.b64` dekódolás után válik PDF-fé.
- A GitHubra feltöltött `.b64` fájlt UTF-8 szövegként kell kezelni. A Git blob API `encoding: base64` módja nem helyettesíti ezt: az a Base64-szöveget dekódolja és a bináris bájtokat tárolja.
- Rögzítsd az eredeti bináris fájl méretét és – ha rendelkezésre áll – SHA-256 ellenőrzőösszegét. Dekódolás után ezekkel ellenőrizhető a sértetlenség.
- A Base64-szöveg sortöréseket tartalmazhat; dekódoláskor olyan eszközt használj, amely a whitespace karaktereket elfogadja.
- Nagy fájloknál számolj a Base64 kb. 4/3-os méretnövekedésével és az eszközök/GitHub méretkorlátaival.

## Biztonság
Ne kérj és ne tárolj valódi cloud secretet. State és plan fájlok érzékeny adatot tartalmazhatnak. Költséges vagy módosító példáknál jelezd a kockázatot és a destroy/takarítás lépéseit. Bináris vagy Base64-artefaktumba se kerüljön secret, token, jelszó vagy más érzékeny adat.
