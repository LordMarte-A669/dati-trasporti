# Licenza e fonti

I file vengono da fonti diverse, e ognuno ha la licenza che le sue fonti permettono.

| File | Licenza | Da citare |
|---|---|---|
| `stazioni.json`, `sedi.json` | [ODbL 1.0](ODbL-1.0.txt) | © OpenStreetMap contributors · dati-trasporti |
| `treni-*.json`, `dati.json` | Condizioni della fonte, **in attesa di conferma** (sotto) | Punto di Accesso Nazionale · l'operatore |
| `scioperi.json` | Riuso con citazione della fonte | Ministero delle Infrastrutture e dei Trasporti |
| `interruttori.json`, `banchetto/` | Nessuna: non sono dati aperti, sono file dell'app Sciamn | — |

## `stazioni.json` e `sedi.json`: ODbL

Questi due database sono resi disponibili con la Open Database License 1.0
(<https://opendatacommons.org/licenses/odbl/1-0/>, testo in [`ODbL-1.0.txt`](ODbL-1.0.txt)). Eventuali
diritti sui contenuti dei singoli record sono concessi con la Database Contents License
(<https://opendatacommons.org/licenses/dbcl/1-0/>).

L'ODbL è obbligata, non scelta: le coordinate vengono da OpenStreetMap, e chi ne ricava un database deve
distribuirlo con la stessa licenza. Vale anche per chi ridistribuisce una versione modificata di questi
file.

- **Coordinate delle stazioni e centroidi degli edifici**: © OpenStreetMap contributors (Overpass).
- **Minuti e metri a piedi**: router pedonale `routing.openstreetmap.de` (FOSSGIS), su dati OpenStreetMap.
- **Nomi delle stazioni e codici `nap`**: dagli orari del Punto di Accesso Nazionale (vedi sotto).
- **Codici `vt`, `lefrecce` e `rfi`**: dai servizi pubblici di ViaggiaTreno, del sito di vendita
  Trenitalia e di RFI, che non dichiarano una licenza.

## Orari: in attesa di conferma

Gli orari vengono dal Punto di Accesso Nazionale (NAP) alla mobilità multimodale, gestito dal CCISS, che
li raccoglie ai sensi del Regolamento delegato (UE) 2017/1926:

- dataset «OAP Trenitalia - NeTEx Livello 1», titolare **Trenitalia** (`treni-TI.json`);
- dataset «RAP Puglia - NeTEx Livello 1», titolare **Regione Puglia** (`treni-FNB.json`,
  `treni-FAL.json`, `treni-FDG.json`).

Le schede dei due dataset sul NAP non indicano una licenza. Il 10/11/2026 è stata chiesta conferma ai due
titolari. La Regione Puglia pubblica i programmi di esercizio degli stessi operatori su dati.puglia.it con
licenza CC-BY 4.0. Fino alla risposta gli orari **restano dei rispettivi titolari**, e questo repository
non concede su di loro diritti ulteriori.

La lavorazione è fatta da Sciamn: ritaglio dell'area con i confini delle regioni ISTAT, rielaborati da
openpolis/geojson-italy (CC-BY 4.0, attribuzione: ISTAT, openpolis), e compressione nel formato descritto
nel README.

## Scioperi

RSS pubblica del Ministero delle Infrastrutture e dei Trasporti (`https://scioperi.mit.gov.it`), riuso con
citazione della fonte. Il testo degli avvisi resta del Ministero.

## Nessuna garanzia

Gli orari sono **indicativi**: per viaggiare fanno fede le fonti ufficiali degli operatori. Nessuna garanzia
di completezza o di aggiornamento.
