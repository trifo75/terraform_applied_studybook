# Aktuális projektállapot

**Frissítve:** 2026-11-24

## Repository és kiadás
- Repository: https://github.com/trifo75/terraform_applied_studybook
- Alapág: `main`
- A legutóbb jóváhagyott könyvverzió a `VERSION` fájl szerint: `1.2.1`.
- A borító és a v1.2.1 verziójelölés javítását tartalmazó PR #3 merged állapotú.
- Merge commit: `6c23170b4ffb7bcab963ccab4f806c40051c28e2`.
- A PR #3 a README-ben a felhasználó által feltöltött `assets/terraform_applied_cover.png` képre hivatkozik, eltávolította a helyettesítő SVG-t, és beállította a VERSION 1.2.1 értékét.
- A projektmemória alapját létrehozó PR #4 merged; a `context_memory/` könyvtár a main ág része.

## PDF reading proof
- A `terraform_applied_v1.2.2_reading-proof.pdf` fájl **már megtalálható a main ág gyökérkönyvtárában**.
- A GitHub main tree alapján ellenőrzött fájlméret: 2,627,501 bájt.
- GitHub-fájl: https://github.com/trifo75/terraform_applied_studybook/blob/main/terraform_applied_v1.2.2_reading-proof.pdf
- Fontos verziókülönbség: a PDF fájlneve és metaadata v1.2.2 reading proofot jelöl, miközben a repository `VERSION` fájlja jelenleg 1.2.1. Ez az állapot önmagában nem jelenti a v1.2.2 könyvkiadás jóváhagyását.
- A reading proof ellenőrző/olvasási példány, nem teljes körűen validált végleges könyv.

## Bináris artefaktumok munkafolyamata
- A jövőben a chatben létrehozott bináris fájlok Base64-kódolt `.b64` szövegfájlként kerülhetnek külön branch-re és PR-be.
- A felhasználó tölti le és dekódolja a `.b64` fájlt a saját környezetében.
- A PR és a fájlnév világosan jelezze a Base64-formátumot; a feltöltés meglétét és teljességét ellenőrizni kell.
- Ez a munkafolyamat a már main ágon lévő PDF-re nem vonatkozik; az közvetlenül PDF-fájlként található meg a repositoryban.
- A folyamatdöntés önmagában nem változtatja meg a könyv verzióját.

## Aktuális munkafolyamat
- A projektmemória alapját létrehozó PR #4 merged.
- A Base64-alapú bináris artefaktum-kezelést dokumentáló PR #5 nyitott, még nem merged. A PR a `context_memory/` fájlok módosításait tartalmazza.
- A jelenlegi PR a PDF tényleges repository-állapotát és a VERSION fájl szerinti verziót rögzíti; nem módosítja a könyv `VERSION` fájlját.
- A felhasználó ellenőrzi és merge-öli a PR-eket.

## Következő lépések
1. A nyitott dokumentációs PR-ek felülvizsgálata és merge-elése a felhasználó döntése alapján.
2. A kézirat aktuális állapotának felmérése.
3. A fejezetek kidolgozása és technikai ellenőrzése hivatalos HashiCorp dokumentációval.
4. Reading proof frissítése az érdemi kéziratjavítások után.

Ez pillanatfelvétel: minden munkamenet elején ellenőrizd a GitHub main ágat, a nyitott PR-eket és a kiadási fájlokat.
