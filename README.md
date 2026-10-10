# dati-trasporti

Orari dei treni del Sud Italia in JSON, pronti da usare in un'app: un file per operatore, le stazioni
con i codici di ogni fonte e gli scioperi annunciati. Li scrive una CI ogni settimana (gli scioperi
ogni giorno) e si leggono da GitHub Pages:

    https://lordmarte-a669.github.io/dati-trasporti/<file>

Pages risponde con gzip, `ETag` (304 se non è cambiato niente) e `access-control-allow-origin: *`.

> **In costruzione.** Oggi c'è solo `interruttori.json`; i file degli orari arrivano con la prima
> esecuzione della CI.

## File

| File | Contenuto |
|---|---|
| `dati.json` | Indice, scritto per ultimo: `schema`, `versione`, validità di ogni operatore (`validoDal`/`validoAl`), SHA-256 dei file. Leggere prima questo. |
| `treni-TI.json` | Trenitalia: le corse con almeno una fermata in Puglia, Basilicata, Calabria o Molise |
| `treni-FNB.json` | Ferrotramviaria (Bari Nord) |
| `treni-FAL.json` | Ferrovie Appulo Lucane |
| `treni-FDG.json` | Ferrovie del Gargano |
| `stazioni.json` | Anagrafe unica: per ogni stazione i codici di ViaggiaTreno, lefrecce, RFI, eticket e del Punto di Accesso Nazionale, con coordinate da OpenStreetMap |
| `scioperi.json` | Scioperi del settore ferroviario e del trasporto locale che riguardano questi operatori, dal Ministero dei Trasporti |
| `interruttori.json` | Interruttori di emergenza dell'app [Sciamn](https://lordmarte-a669.github.io/sciamn/): **non fanno parte dei dati**, si possono ignorare |

Lo schema dei file è versionato (`schema` in `dati.json`): un cambio incompatibile alza il numero.

## Fonti e licenze

- Orari dal **Punto di Accesso Nazionale** italiano (Reg. UE 2017/1926); per Trenitalia attraverso il
  GTFS di [deryclem/trenitalia-gtfs](https://github.com/deryclem/trenitalia-gtfs) (CC-BY-4.0).
- Coordinate delle stazioni © **OpenStreetMap** contributors, [ODbL](https://opendatacommons.org/licenses/odbl/):
  `stazioni.json` è distribuito sotto ODbL.
- Scioperi dal feed pubblico del Ministero delle Infrastrutture e dei Trasporti.

I dettagli sono in `LICENZA.md` (arriva con i primi dati). Orari indicativi: per viaggiare fanno fede
le fonti ufficiali.
