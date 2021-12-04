# dbCreation

## Definizione del database

La base di dati si basa su un insieme di serie storiche di mercati sportivi, con informazioni riguardanti le info del'evento, gli update dello stato del mercato, i runner del mercato (giocatori/team) e la serie storica (BASIC risoluzione 1min, ADVANCED ris 1sec + volume, traded and best to bet) della quota per ogni runner.

Questi sono esempi dei dati raw gia rielaborati, saranno la base di partenza per definire a popolare il DB.

some JSON file sample
- [TENNIS ADVANCED](sample/1.187557995_ADVANCED.json)
- [TENNIS BASIC](sample/1.187557995_BASIC.json)
--------------
- [SOCCER ADVANCED](sample/1.187557995_ADVANCED.json)
- [SOCCER BASIC](sample/1.187557995_ADVANCED.json)
--------------
- [HORSE ADVANCED](sample/1.187557995_ADVANCED.json)
- [HORSE BASIC](sample/1.187557995_ADVANCED.json)
--------------
- [OTHER ADVANCED](sample/1.187557995_ADVANCED.json)
- [OTHER BASIC](sample/1.187557995_ADVANCED.json)
--------------

## Descrizione



### BASIC v ADVANCED

I dati BASIC sono dati gratuiti e più leggeri rispetto ai pesanti ADVANCED, ma molto meno completi.
Le marketInfo sono le stesse (ADVANCED ha in più le info total_volume, prematch_volume, inplay_volume,)

La mia idea è quella di scaricare tutti i dati BASIC dal 2017, mentre per gli ADVANCED comprare solo le porzioni di dati che mi interessano nel tempo (per ora ho 4 mesi di TENNIS e 2 mesi di SOCCER e HORSE).

Dovrei quindi avere la possibilità di ricercare i mercati in maniera sincrona e, se presente, passare sempre e solo il mercato ADVANCED (con volumi e con info di tempo più precise), mentre se manca il mercato ADVANCED passare sempre quello BASIC.

Il DB quindi dovrebbe avere uno schema del genere (per mercati che sono presenti sia BASIC e ADVANCED)

marketInfo
marketInfoAdvanced

runnersBasic
runnersAdvanced

updatesBasic
updatesAdvanced

oddsBasic
oddsAdvanced

La particolarità è che i dati sono caricati in maniera asincrona quindi il DB deve poter caricare in ordine sparso mercati BASIC e ADVANCED, inoltre i dati sono rilasciati ogni giorno e devo avere la possibilità di aggiungere nuovi mercati che completino il db (magari carico tutto BASIC sept-oct-nov e poi solo sep per gli ADVANCED).

Ho pensato di creare queste tabelle

| NAME                   | INFO                                                           | ADVANCED INFO                          |
|------------------------|----------------------------------------------------------------|----------------------------------------|
| market_info            | contiene tutte le info di mercato                              | advanced ha open date precisa e volumi |
| market_updates         | update dello stato del mercato                                 | advanced orari precisi                 |
| market_selections      | metdata delle selezioni                                        | advanced dati più precisi              |
| market_odds            | serie storica della quota per ogni selezioni in un mercato     | advanced res 1sec                      |
|                        |                                                                |                                        |
| runners                | tutti i runners presenti in alemeno un mercato come selezione  |                                        |
| players                | raggruppamento degli pesudomi runners per ogni players univoco |                                        |
|                        |                                                                |                                        |
| market_status          | BASIC / ADVANCED                                               |                                        |
| market_type            | possibili tipi di mercato                                      |                                        |
| sport                  | possibili sport                                                |                                        |
| runner_status          | possibili stati dei runnese                                    |                                        |
|                        |                                                                |                                        |
|                        |                                                                |                                        |
| additional_tennis_info |                                                                |                                        |
| tennis_tournament      | informazioni sul torneo                                        |                                        |
| tennis_rank            | informazione su rank dei giocatori                             |                                        |
| tennis_final_result    | risultato finale                                               |                                        |
| tennis_bookie_odds     | quore prematch dei bookmaker                                   |                                        |
|                        |                                                                |                                        |
|                        |                                                                |                                        |
| additional_soccer_info |                                                                |                                        |
| soccer_league_info     | informazione su lega                                           |                                        |
| soccer_final_result    | risulato finale                                                |                                        |
| soccer_stats           | statistiche della partita                                      |                                        |
| soccer_bookie_odds     | quore prematch dei bookmaker                                   |                                        |

# Schema

My version of db schema
- [DB schema draw.io](https://drive.google.com/file/d/13C4LKcG6bWKcgPyfIEgj7DrkQagPlrrP/view?usp=sharing)
![DB schema](images/DB.drawio.svg)


# Tecnologia da usare

Le informazioni di odds sono molto pesanti e nel precendete progetto usavo un DB NoSQL con oggetto tutto il mercato delle quote. la soluzione però non è efficente in termini di ricerca per range temporali, o per generare grafici con diversi timeframe

Dopo varie ricerche in rete per trovare una soluzione, opterei per un database PostgressSQL vanilla che sfrutti per le quote una struttura simile questa

```sql
    create table "Tickers_Data"
    (
    runner_id       integer                 not null
    constraint runner_id
        references "market_selections",
    market_id       integer                 not null
        constraint market_id
        references "market_info_",
    timestamp       timestamptz              not null,
    ltp             double precision         not null,
    tv              double precision         not null,
    tr              double precision[][],
    batl            double precision[][],
    batb            double precision[][],
    constraint "Tickers_Data_pkey"
    primary key (runner_id,, market_id timestamp)
    );
```

può essere la soluzione migliore per fare ricerca in time range ma anche per prendere tutto il blocco di quota insieme?

- [medium](https://medium.com/@neslinesli93/how-to-efficiently-store-and-query-time-series-data-90313ff0ec20)