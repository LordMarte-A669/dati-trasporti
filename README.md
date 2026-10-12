# dati-trasporti

Orari dei treni del Sud Italia in JSON, pronti da usare in un'app: un file per operatore, le stazioni
con i codici di ogni fonte e gli scioperi annunciati. Li scrive una CI ogni settimana (gli scioperi
ogni giorno) e si leggono da GitHub Pages:

    https://lordmarte-a669.github.io/dati-trasporti/<file>

Pages risponde con gzip, `ETag` (304 se non è cambiato niente) e `access-control-allow-origin: *`.

> **Stato.** Gli orari (`dati.json`, `treni-*.json`) sono online e aggiornati ogni lunedì dalla CI.
> `stazioni.json` e `scioperi.json` arrivano più avanti.

## File

| File | Contenuto |
|---|---|
| `dati.json` | Indice, scritto per ultimo: `schema`, `versione`, e per ogni operatore validità (`validoDal`/`validoAl`), `sha256` del file, numero di `corse`, `doppie`, `versioneFonte` e `asset` della fonte. Leggere prima questo. |
| `treni-TI.json` | Trenitalia: le corse con almeno una fermata in Puglia, Basilicata, Calabria o Molise |
| `treni-FNB.json` | Ferrotramviaria (Bari Nord) |
| `treni-FAL.json` | Ferrovie Appulo Lucane |
| `treni-FDG.json` | Ferrovie del Gargano |
| `stazioni.json` | *(in arrivo)* Anagrafe unica: per ogni stazione i codici di ViaggiaTreno, lefrecce, RFI, eticket e del Punto di Accesso Nazionale, con coordinate da OpenStreetMap |
| `scioperi.json` | *(in arrivo)* Scioperi del settore ferroviario e del trasporto locale che riguardano questi operatori, dal Ministero dei Trasporti |
| `interruttori.json` | Interruttori di emergenza dell'app [Sciamn](https://lordmarte-a669.github.io/sciamn/): **non fanno parte dei dati**, si possono ignorare |

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
  "fermate":   [["rfi:S11119", "BARI CENTRALE", 41.1177, 16.8697, "830011119"]],
  "calendari": ["0111110011111000"],
  "corse":     [["23539", null, "REG", 2, [[1, 837], [2, 845, 1], [0, 885, 0, 1]], 0]]
}
```

- `fermate[i] = [stop_id, nome, lat, lon, nap]`. `lat`/`lon` a 5 decimali o `null`. `nap` = codice della
  fermata nella fonte (il Punto di Accesso Nazionale).
- `calendari[j]` = stringa di bit con origine comune `validoDal`: il bit `k` vale `1` se e solo se la
  corsa parte il giorno `validoDal + k`. Gli `0` in coda si tolgono; stringhe uguali si scrivono una
  volta sola e si condividono.
- `corse[n] = [numero, nome, categoria, route_type, passaggi, j]`:
  - `numero` è il numero del treno; per i bus (e per ogni nome che non è un numero) è `"x"` + il nome
    senza spazi né simboli (così non collide coi numeri dei treni).
  - `nome` è sempre presente: `null` per i treni, il nome originale della corsa quando il `numero` è un
    `x…`. In coda è riservato `numeroArrivo`, oggi non scritto.
  - `categoria` ∈ `REG RV IC FR BUS`; `route_type` 2 treno, 3 bus; `j` è l'indice in `calendari`.
  - `passaggi = [[i, arrivo, sosta?, flag?]…]`: `i` indice in `fermate`; `arrivo` in minuti dal
    riferimento GTFS del giorno di partenza (oltre 1440 dopo la mezzanotte); `sosta` = partenza − arrivo
    (assente = 0); `flag` = `2·(niente salita) + (niente discesa)` (assente = 0).
- **Estensione compatibile:** chi legge ignora gli elementi in più in coda a ogni tupla e le chiavi in
  più dell'oggetto. Un campo nuovo si aggiunge solo in coda; cambiare posizione o significato di un
  campo è un cambio incompatibile e alza `schema`.

### `stop_id` provvisori

Per FNB, FAL e FDG le fermate escono oggi come `fnb:nap-…`, `fal:nap-…`, `fdg:nap-…` (il codice dopo
`nap-` è l'identificativo della fermata nel Punto di Accesso Nazionale). È una forma **provvisoria**: con
`stazioni.json` diventeranno `<rete>:<codice eticket>`. Il quinto campo di ogni fermata (`nap`) serve
proprio a tradurli quando arriverà l'anagrafe. `dati.json` dichiara questa provvisorietà nel campo
`provvisorio`.

## Fonti e licenze

- Tutti gli orari vengono dal **Punto di Accesso Nazionale** italiano (NeTEx, Reg. UE 2017/1926):
  Trenitalia, Ferrotramviaria, Ferrovie Appulo Lucane e Ferrovie del Gargano.
- I confini delle regioni usati per scegliere le corse del Sud vengono da
  [openpolis/geojson-italy](https://github.com/openpolis/geojson-italy) (ISTAT, CC-BY-4.0).
- Coordinate delle stazioni © **OpenStreetMap** contributors, [ODbL](https://opendatacommons.org/licenses/odbl/):
  `stazioni.json` (in arrivo) sarà distribuito sotto ODbL.
- Scioperi (in arrivo) dal feed pubblico del Ministero delle Infrastrutture e dei Trasporti.

I dettagli sono in `LICENZA.md`. Orari indicativi: per viaggiare fanno fede le fonti ufficiali.
