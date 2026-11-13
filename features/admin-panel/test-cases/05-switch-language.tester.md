# 05 — Switch language — tester cases

Use case: [05-switch-language.md](../05-switch-language.md). Screen text: [ui-ux.md §6](../ui-ux.md#6-german-samples-for-the-tester-cases). Fixtures: [README.md](./README.md#fixtures).

Signed in as ADM-1 on REST-A's Overview.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ADM-05-T01 | AC1 | Open the language dropdown; pick `Deutsch`. | Options `English`, `Español`, `Deutsch`, `বাংলা`. After picking: the same Overview page reloads with `Dashboard-Übersicht`, `Gesamtumsatz`, period `Heute`, sidebar `Bestellverlauf`; the dropdown shows `Deutsch`. |
| ADM-05-T02 | AC2 (ADM-D3) | Still in Deutsch, go back to the restaurant list, then open REST-B's Overview and its Item History. | The restaurant list is in English (untranslated). REST-B's Overview is German. Item History: German sidebar, English table. |
| ADM-05-T03 | AC3 | Tap `Logout`; sign in again; open REST-A's Overview. | English (`Dashboard Overview`). |
