# Aktuális projektállapot

**Frissítve:** 2026-11-24

## Repository és kiadás
- Repository: https://github.com/trifo75/terraform_applied_studybook
- Alapág: `main`
- A legutóbb ismert jóváhagyott könyvverzió: `1.2.1` (VERSION fájl).
- A borító és a v1.2.1 verziójelölés javítását tartalmazó PR #3 merged állapotú.
- Merge commit: `6c23170b4ffb7bcab963ccab4f806c40051c28e2`.
- A PR #3 a README-ben a felhasználó által feltöltött `assets/terraform_applied_cover.png` képre hivatkozik, eltávolította a helyettesítő SVG-t, és beállította a VERSION 1.2.1 értékét.
- A projektmemória alapját létrehozó PR #4 merged; a `context_memory/` könyvtár a main ág része.
- A v1.2.0 PDF reading proof, nem teljes körűen validált végleges könyv.

## Bináris artefaktumok munkafolyamata
- Elfogadott folyamat: a chatben létrehozott bináris fájl Base64-kódolt `.b64` szövegfájlként kerül külön branch-re és PR-be.
- A felhasználó tölti le és dekódolja a `.b64` fájlt a saját környezetében.
- A PR és a fájlnév világosan jelezze a Base64-formátumot; a feltöltés meglétét és teljességét ellenőrizni kell.
- A folyamatdöntés önmagában nem változtatja meg a könyv verzióját.

## Aktuális munkafolyamat
- A context memory jelen módosítása külön dokumentációs PR-ben történik.
- A legutóbbi ellenőrzéskor nem volt nyitott PR a repositoryban.
- A v1.2.2 reading proof PDF elkészült a chatben, de a GitHub-repóba feltöltése még nem történt meg; a hozzá tartozó Base64-fájl a chatből letölthető. Ezt ne tekintsd GitHubon elérhető artefaktumnak.

## Következő lépések
1. A jelenlegi context-memory PR felülvizsgálata és merge-elése.
2. A kézirat aktuális állapotának felmérése.
3. A fejezetek kidolgozása és technikai ellenőrzése hivatalos HashiCorp dokumentációval.
4. Reading proof frissítése az érdemi kéziratjavítások után.

Ez pillanatfelvétel: minden munkamenet elején ellenőrizd a GitHub main ágat, a nyitott PR-eket és a kiadási fájlokat.
