# Döntési napló

## 2026-11-24 – Base64-alapú bináris artefaktum-kezelés
A chatben előállított bináris fájlokat – például PDF reading proofot – Base64-kódolt `.b64` szövegfájlként készítjük elő és töltjük fel a repositoryba, külön branch-en és PR-ben. A felhasználó tölti le a `.b64` fájlt, és saját környezetében gondoskodik a dekódolásáról. A PR-ban egyértelműen jelezni kell, hogy a tárolt állomány Base64-szöveg, nem közvetlenül megnyitható bináris fájl. A feltöltés sikerét GitHubon ellenőrizni kell; sikertelen vagy nem ellenőrzött feltöltést tilos kész tényként közölni. Ez a folyamatdöntés önmagában nem emeli a könyv SemVer-verzióját.

## 2026-11-24 – Repository-alapú projektmemória
A projekt tartós kontextusa a repository `context_memory/` könyvtárában, Markdown-fájlokban legyen. Nem teljes chat-átiratot tárolunk, hanem összefoglalt, jóváhagyott és hasznos projektismeretet. Minden új munkamenetben ellenőrizni kell a fájlokat.

## 2026-11-24 – Terminológia
Természetellenes magyar fordítás helyett maradjon az angol szakkifejezés rövid magyar magyarázattal. Példa: „modul használati felülete” (module interface); az angol szakmai használatban előfordul a module contract is.

## 2026-11-24 – Könyvszerkezet és verziózás
A jóváhagyott kiinduló szerkezet 7 rész, 22 fejezet, 10 függelék. A könyv SemVer szerint verziózandó; a projektfolyamat-dokumentáció önmagában nem feltétlenül változtatja a könyv verzióját.
