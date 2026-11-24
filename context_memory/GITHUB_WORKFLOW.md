# GitHub-munkafolyamat

Repository: https://github.com/trifo75/terraform_applied_studybook

## Általános GitHub-folyamat

- A main ág a jóváhagyott, stabil állapot.
- Érdemi módosítás külön feature/fix/docs branch-en történjen.
- A változásokat Pull Requestben mutasd be.
- A felhasználó ellenőrzi és merge-öli a PR-t; ne merge-ölj helyette, hacsak kifejezetten nem kéri.
- Munkakezdés előtt olvasd le a friss main állapotát, releváns fájlokat és nyitott PR-eket.
- A PR írja le a célt, változásokat, ellenőrzéseket és fennmaradó korlátokat.

## Bináris artefaktumok Base64-munkafolyamata

A projektben a chatben előállított bináris artefaktumokat (például PDF-et) Base64-kódolt, szöveges `.b64` fájlként kell előkészíteni és a repositoryba feltölteni. A felhasználó a GitHubról letöltött `.b64` fájlt saját környezetében alakítja vissza az eredeti bináris formátumra.

1. Készítsd el és ellenőrizd az eredeti bináris artefaktumot.
2. Kódold Base64 formátumba. A kódolás az eredeti fájl bájtjain történjen, ne a fájl szöveges reprezentációján.
3. A kimenetet egyértelműen nevezd el, például `terraform_applied_v1.2.2_reading-proof.pdf.b64`. A kiterjesztés maradjon `.b64`; ne nevezd át `.pdf`-re.
4. A `.b64` fájlt UTF-8 szövegként töltsd fel külön branch-re, majd nyiss róla PR-t. A GitHub szöveges fájlműveleteit használd; ne add át a Base64-karakterláncot Git blob API-nak `encoding: base64` beállítással, mert az az eredeti bináris bájtokat hozná létre, nem a letölthető `.b64` szövegfájlt.
5. A PR leírásában nevezd meg az eredeti fájl nevét és formátumát, a Base64-fájl nevét, az eredeti fájl méretét és – ha rendelkezésre áll – SHA-256 ellenőrzőösszegét. Jelezd, hogy a PR-ban tárolt artefaktum Base64-szöveg, nem közvetlenül megnyitható PDF vagy más bináris fájl.
6. Ellenőrizd a GitHubon a feltöltött fájl meglétét és teljességét. Csak a ténylegesen sikeres feltöltés után állítsd, hogy a fájl elérhető a repóban.
7. A felhasználó merge-eli a PR-t, letölti a `.b64` fájlt, majd saját környezetében dekódolja. A dekódolt fájl méretét és lehetőség szerint SHA-256 értékét az eredetihez kell hasonlítani.

Példák dekódolásra:

- Linux: `base64 --decode input.pdf.b64 > output.pdf`
- macOS: `base64 -D -i input.pdf.b64 -o output.pdf`
- PowerShell: `[IO.File]::WriteAllBytes("output.pdf", [Convert]::FromBase64String((Get-Content -Raw "input.pdf.b64")))`

A Base64-szövegben lévő sortörések dekódoláskor megengedettek. A dekóder platformonként eltérő kapcsolóit a célrendszeren ellenőrizd.

## Korlátok és biztonság

- A Base64 körülbelül egyharmaddal növeli az adatmennyiséget; nagy fájloknál ellenőrizd a GitHub- és connector-eszközök méretkorlátait. Ha a fájl nem fér át, ne állítsd, hogy feltöltötted; jelezd a korlátot és egyeztess alternatívát.
- Ne tárolj titkot, tokent, jelszót, személyes azonosítót vagy érzékeny állományt a repóban.
- Ne tedd közzé a dekódolt bináris állományt más formátumként, és ne adj hozzá olyan letöltési linket, amely nem ellenőrzött.
- A felhasználó feladata a dekódolás; a ChatGPT feladata a Base64-fájl előállítása, a PR elkészítése és a feltöltés ellenőrzése.

A ChatGPT az adott beszélgetésben elérhető GitHub-integráción keresztül olvashatja és módosíthatja a repositoryt a csatlakoztatott jogosultságok szerint. Ez nem helyettesíti a felhasználói felülvizsgálatot és jóváhagyást. Ne kérj és ne tárolj tokent vagy jelszót.
