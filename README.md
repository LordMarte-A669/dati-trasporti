# dati-trasporti

Orari dei treni del Sud Italia in JSON, pronti da usare in un'app: un file per operatore, le stazioni
con i codici di ogni fonte, le sedi universitarie di Bari coi tempi a piedi fino alle stazioni e gli
scioperi annunciati. Li scrive una CI e si leggono da GitHub Pages:

    https://lordmarte-a669.github.io/dati-trasporti/<file>

Pages risponde con gzip, `ETag` (304 se non è cambiato niente) e `access-control-allow-origin: *`.

> **Stato.** Gli orari (`dati.json`, `treni-*.json`) sono aggiornati ogni lunedì dalla CI;
> `stazioni.json` e `sedi.json` si rigenerano quando cambiano le fonti.
> **`scioperi.json` è fermo al 22/10/2026**: il feed del Ministero rifiuta le richieste che non
> arrivano dall'Italia, e la CI gira all'estero. Finché non torna aggiornato, chi lo usa controlli
> `generato` e non lo tratti come «nessuno sciopero».

## File

| File | Contenuto |
|---|---|
| `dati.json` | Indice, scritto per ultimo: `schema`, `versione` (istante ISO); per ogni operatore `file`, `validoDal`/`validoAl` (primo e ultimo giorno di una corsa qualunque: **non** la fine del servizio pieno, che può arrivare mesi prima), `sha256` del file, numero di `corse`, `doppie` (quanti numeri hanno almeno un giorno con due corse), `tipico` (corse attive nel giorno più pieno dei 7 giorni dalla generazione), `versioneFonte` e `asset` della fonte; in `anagrafi` `file`, `sha256` e `generato` di `stazioni.json` e `sedi.json`. Leggere prima questo. |
| `treni-TI.json` | Trenitalia: le corse con almeno una fermata in Puglia, Basilicata, Calabria o Molise |
| `treni-FNB.json` | Ferrotramviaria (Bari Nord) |
| `treni-FAL.json` | Ferrovie Appulo Lucane |
| `treni-FDG.json` | Ferrovie del Gargano |
| `stazioni.json` | Anagrafe unica delle stazioni (515, sulle reti RFI, FAL, FSE, FNB e FDG): per ognuna i codici di ViaggiaTreno, lefrecce, RFI, eticket e del Punto di Accesso Nazionale, con le coordinate da OpenStreetMap. Formato sotto |
| `sedi.json` | Edifici dell'Università di Bari e minuti a piedi fino alle stazioni vicine. Formato sotto |
| `scioperi.json` | Scioperi del settore ferroviario e del trasporto locale che riguardano questi operatori, dal Ministero dei Trasporti. **Fermo, vedi Stato** |
| `interruttori.json`, `banchetto/` | File dell'app [Sciamn](https://lordmarte-a669.github.io/sciamn/): **non fanno parte dei dati**, si possono ignorare |

Lo schema dei file è versionato (`schema` in `dati.json`): un cambio incompatibile alza il numero.

## Formato dei file degli orari

Ogni `treni-<OP>.json` scrive la corsa **programmata** una volta sola, con un calendario a bit condiviso
tra le corse che circolano negli stessi giorni (molto più compatto di una corsa per ogni data). Chi
legge valida il file una volta e poi ne espande le corse del giorno che gli serve.

```json
{
  "schema": 1,
  "agency_id": "TI",
  "versioneFonte": "2026-10-05T15:07:02Z",
  "generato": "2026-10-12T14:35:03Z",
  "validoDal": "2026-10-03",
  "validoAl": "2027-06-12",
  "conteggi":  { "corse": 1, "fermate": 1, "doppie": 0 },
  "fermate":   [["rfi:S11119", "BARI CENTRALE", 41.1177, 16.8697, "830011119"]],
  "calendari": ["0111110011111"],
  "corse":     [["23539", null, "REG", 2, [[1, 837], [2, 845, 1], [0, 885, 0, 1]], 0]]
}
```

- `conteggi`: numeri della lavorazione (corse lette e tenute, fermate, anomalie contate), informativi:
  le chiavi possono crescere.
- `fermate[i] = [stop_id, nome, lat, lon, nap]`. `lat`/`lon` a 5 decimali o `null`. `nap` = codice della
  fermata nella fonte (il Punto di Accesso Nazionale).
- `calendari[j]` = stringa di bit con origine comune `validoDal`: il bit `k` vale `1` se e solo se la
  corsa parte il giorno `validoDal + k`. Gli `0` in coda si tolgono; stringhe uguali si scrivono una
  volta sola e si condividono.
- `corse[n] = [numero, nome, categoria, route_type, passaggi, j]`:
  - `numero` è il numero del treno quando la fonte lo dà come sole cifre, o come sigla + spazio + cifre
    (`ET 91137` → `91137`). Per i bus e per ogni altro nome è `"x"` + il nome senza spazi né simboli
    (`SA153` → `xSA153`), così non collide coi numeri dei treni. Non è una chiave: pochi numeri hanno
    due corse lo stesso giorno (un treno che si divide in due sezioni, un bus pubblicato due volte);
    `doppie` in `dati.json` dice quanti.
  - `nome` è sempre presente: `null` quando `numero` è il numero del treno, il nome originale della
    corsa quando è un `x…`. In coda è riservato `numeroArrivo`, oggi non scritto.
  - `categoria` ∈ `REG RV IC FR BUS`; `route_type` 2 treno, 3 bus; `j` è l'indice in `calendari`.
  - `passaggi = [[i, arrivo, sosta?, flag?]…]`: `i` indice in `fermate`; `arrivo` in minuti dal
    riferimento GTFS del giorno di partenza (oltre 1440 dopo la mezzanotte); `sosta` = partenza − arrivo
    (assente = 0); `flag` = `2·(niente salita) + (niente discesa)` (assente = 0).
- **Estensione compatibile:** chi legge ignora gli elementi in più in coda a ogni tupla e le chiavi in
  più dell'oggetto. Un campo nuovo si aggiunge solo in coda; cambiare posizione o significato di un
  campo è un cambio incompatibile e alza `schema`.

### `stop_id`

Ogni `stop_id` è `<rete>:<codice>`, e il codice dipende da cosa la fonte mette a disposizione:

| Rete | `stop_id` | Esempio |
|---|---|---|
| RFI (Trenitalia) | codice di ViaggiaTreno | `rfi:S11119` |
| FSE | codice di ViaggiaTreno | `fse:S11653` |
| FNB, stazioni FAL | codice eticket | `fnb:S01110`, `fal:S02110` |
| FDG, fermate dei bus FAL | `nap-` + identificativo nel Punto di Accesso Nazionale | `fdg:nap-APRICENA_Rete_FDG` |

FDG e i bus non hanno un codice eticket, quindi restano `nap-`. Il quinto campo di ogni fermata (`nap`)
è sempre il codice della fonte, e permette di passare dagli orari all'anagrafe e viceversa.

## Formato di `stazioni.json`

```json
{
  "schema": 1,
  "generato": "2026-10-26",
  "stazioni": [
    { "stop_id": "rfi:S00219", "nome": "TORINO PORTA NUOVA", "lat": 45.06097, "lon": 7.67747, "rete": "rfi",
      "x_codici": { "vt": "S00219", "lefrecce": "830000219", "rfi": 2876, "eticket": null, "nap": "830000219" } }
  ]
}
```

- `stop_id` è lo stesso dei file degli orari; `rete` ∈ `rfi fse fal fnb fdg`.
- `x_codici`: il codice della stazione in ogni fonte, `null` dove quella fonte non la conosce. `vt` è
  ViaggiaTreno, `lefrecce` il sito di vendita Trenitalia, `rfi` l'identificativo numerico di RFI,
  `eticket` il sistema di bigliettazione di FNB e FAL, `nap` il Punto di Accesso Nazionale.
- `lat`/`lon` vengono da OpenStreetMap; sono `null` per le poche stazioni che lì non si trovano.
- Ci sono tutte le fermate dei file degli orari, anche fuori dal Sud quando una corsa ci arriva. Una
  stazione che sparisce dagli orari resta nell'anagrafe.

## Formato di `sedi.json`

- `edifici[] = { codice, ateneo, nome, lat, lon }`: le sedi universitarie, con le coordinate del
  centroide dell'edificio su OpenStreetMap.
- `piedi[] = { da, a, min, m, fonte }`: da un edificio (`codice`) a una stazione (`stop_id`), i minuti e
  i metri a piedi; `fonte` dice quale router li ha calcolati e quando.

## Fonti e licenze

- Tutti gli orari vengono dal **Punto di Accesso Nazionale** italiano (NeTEx, Reg. UE 2017/1926):
  Trenitalia, Ferrotramviaria, Ferrovie Appulo Lucane e Ferrovie del Gargano.
- I confini delle regioni usati per scegliere le corse del Sud vengono da
  [openpolis/geojson-italy](https://github.com/openpolis/geojson-italy) (ISTAT, CC-BY-4.0).
- Coordinate delle stazioni e delle sedi © **OpenStreetMap** contributors,
  [ODbL](https://opendatacommons.org/licenses/odbl/): `stazioni.json` e `sedi.json` sono distribuiti
  sotto ODbL. I minuti a piedi sono calcolati dal router pedonale di FOSSGIS su dati OpenStreetMap.
- I codici ViaggiaTreno, lefrecce e RFI di `stazioni.json` vengono dai servizi pubblici dei rispettivi
  siti, che non dichiarano una licenza.
- Scioperi dal feed pubblico del Ministero delle Infrastrutture e dei Trasporti.

I dettagli sono in `LICENZA.md`. Orari indicativi: per viaggiare fanno fede le fonti ufficiali.
