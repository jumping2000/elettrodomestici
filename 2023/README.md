# 🌊 Lavatrice smart 💧


| Sommario |
| :---: |
| [Introduzione](#introduzione) |
| [Prerequisiti](#prerequisiti) | 
| [Dispositivi](#dispositivi) | 
| [Come funziona](#come-funziona) | 
| [Funzonalità](#funzionalità) | 
| [User interface](#user-interface) | 
| [Configurazione](#configurazione) |

## Introduzione

L'articolo riprende quanto avevamo già scritto nel 2020 con l'articolo **[Elettrodomestici Smart](https://hassiohelp.eu/2020/04/05/elettrodomestici-smart-con-home-assistant/)** che ha introdotto un package per poter monitorare lo stato degli elettrodomestici non smart, una card Lovelace con effetti grafici CSS ed informazioni di funzionamento. In particolare in questa edizione 2023 useremo alcuni custom ricchi di funzionalità come Button Card ed Apex Card che oramai sono diventati dei veri e propri classici delle interfacce sviluppate per Home Assistant e introdurremo ulteriori elementi sia a livello di dati disponibili che a livello di grafica, ma la perla che rende l'uso del package veramente superiore sotto tutti punti di vista rispetto a pacchetti analoghi, è l'utilizzo di una "macchina a stati finiti" [FSM](https://it.wikipedia.org/wiki/Automa_a_stati_finiti) sviluppata in ESPHome che restituisce lo stato del dispositivo in maniera molto precisa.

I punti di forza di questo pacchetto quindi sono:
1. Adattabile per l'uso su tablet, smartphone, PC
2. Macchina a stati finiti per ricavare lo stato della lavatrice
3. Utilizzo di codice YAML non obsoleto e riduzioni del "codice inutile"
4. Uso estensivo degli "_anchor_" per centralizzare le parti YAML da personalizzare
5. Versioni per chi utilizza grafica "storage o "yaml" 
6. Riduzione del numero di componenti custom necessari
7. Utilizzo dei dati statistici conservati nel DB di HA senza appensatire il DB

<span style="color:red">Come usuale prestate attenzione a collegare il vostro dispositivo ad una presa che sia in grado di supportarne il carico, purtroppo gli incidenti domestici sono sempre dietro l'angolo.</span>

## Prerequisiti 

Per poter utilizzare il packages occorrono alcune card e alcune configurazioni abbastanza comuni, le card e i csutom sono disponibili sul community store [HACS](https://hacs.xyz/):

| Card/Custom | Uso |
| :---: | --- |
| [Button Card](https://github.com/custom-cards/button-card) | Obbligatorio |
| [ApexChart Card](https://github.com/RomRider/apexcharts-card) | Obbligatorio |
| [Browser Mod](https://github.com/thomasloven/hass-browser_mod) | Obbligatorio |
| [Card Mod](https://github.com/thomasloven/lovelace-card-mod) | Obbligatorio |
| [Layout Card](https://github.com/thomasloven/lovelace-layout-card) | Facoltativo |
| [Bar Card](https://github.com/custom-cards/bar-card) | Facoltativo |
| [Multiple Entity Row](https://github.com/benct/lovelace-multiple-entity-row) | Facoltativo |

L'utilizzo delle card _facoltative_ permette una migliore visualizzazione delle informazioni soprattutto con i dispositivi smartphone ma nulla vieta di poterle rimuovere utilizzando dei layout non "auto addatbili" (questo vale per la Layout Card). La rimozione dei componenti custom e dl relativo adattamento del codice YAML è lasciato al lettore "evoluto" in grado di procedere in autonomia.

| Configurazioni obbligatorie |
| :---: |

E' necessario:
* abilitare i packages come descritto qui: [Packages](https://www.home-assistant.io/docs/configuration/packages/).
* configurare Home Assistant per avere i sensori di tempo e di data come spiegato qui: [Time & Date](https://www.home-assistant.io/integrations/time_date/)
* configurre il sensore che restituisce il costo dell'energia (package accessorio ad hoc presente nella configurazione)

## Dispositivi

E' fondamentale avere una presa, o un relay “smart” o altro sistema smart per esempio basato sul principio della pinza amperometrica come Shelly 3M o PZEM che restituisca il consumo in watt/Kw  della lavatrice. Per ottenere il massimo risultato ed utilizzare quindi la FSM occorre un dispositivo in grado di essere configurato con il firmware ESPHome, se non è possibile ci sono due alternative:
1. utilizzare una versione ridotta (_lite_) del package;
2. adattare le automazioni già sviluppate nll'articolo del 2020 al proprio caso.

Nel repository su Github sono presenti due packages già pronti per l'uso, la versione FSM e la versione _lite_, lasciamo agli utenti adattare il codice YAML per realizzare la FSM con le automazioni.

## Come funziona

L’idea alla base dell'articolo è che grazie alle informazioni di potenza restituite dalla presa smart siamo in grado di capire in che stato si trova il nostro elettrodomestico e quindi possiamo costruire una serie di sensori che diano delle informazioni da visualizzare nella UI di Home Assistant o piuttosto per avvertirci quando stendere i panni tramite un servizio di notifica.

Vediamo in dettaglio quali sono gli stati per la lavatrice, le transizioni da uno stato ad un altro sono guidate dalla potenza assorbita dalla lavatrice stessa:

| Stato | Caratteristiche |
| :---: | --- |
| Idle | Stato spento della della lavatrice | 
| Riempimento | Stato iniziale del lavaggio con il carico dell'acqua |
| Lavaggio | Avvio del programma con il riscaldamento dell'acqua e lavaggio dei panni vero e proprio |
| Risciacquo | Fase di risciacquo in cui sono eliminati i residui di detersivo |
| Centrifuga | Fase di rotazione del cestello ed eliminazione dell'acqua |
| Svuotare | Stato finale del programma di lavaggio |

Questo stati sono calcolati nella versione FSM del package e nella versione _lite_ rispettivamente da:
* FSM implementata nel codice ESPHome e restituita dal sensore `sensor.washing_machine_status`
* template sensor `sensor.washing_machine_status` 

Ovviamente la versione FSM sarà molto più precisa, _se opportunamente configurata con i dati di potenza della propria lavatrice_, perchè lo stato della lavatrice dipenderà sia dalla potenza assorbita che dallo stato attuale, mentre la versione con template sensor solo dalla potenza assorbita. Anche chi possiede una lavatrice smart che espone in HA lo stato dell'elettrodomestico potrà utilizzare il package aggiustando il codice in maniera minimale adattandolo ai dati esposti dall'integrazione smart.


## Funzionalità del Package

Il package espone una grande serie di dati, funzionalità, grafici e configurazioni, proviamo a riassumerle:

Dati
* Dati giornalieri, settimanali, mensili ed annuali di energia (kWh) consumata e relativo costo in €
* Numero dei cicli giornalieri, settimanali, mensili ed annuali di lavaggio effettuati 
* Durata dei cicli giornalieri, settimanali, mensili ed annuali di lavaggio effettuati
* Numero e durata totale dei cicli effettuati
* Durata ultimo lavaggio

Grafici
* Grafico XY della potenza consumata
* Grafico a barre dei cicli di funzionamento per programmare la manutenzione
* Grafico a istogramma dell'energia consumata negli ultimi 30 giorni
* Grafico a barre per corrente, tensione, potenza apparente e reattiva

Funzionalità presenti
* Timer con vari programmi lavatrice per tenere sotto controllo il tempo trascorso
* Integrazione con Grocy per tenere traccica di detersivo ed ammorbidente utilizzati
* Actionable Notification configurabili con Telegram e Companion app alla fine del ciclo di lavaggio
* Notifiche per i cambi di stato

Per evitare l'uso inutile di entità per configurare alcune aspetti "statici" o comunque poco variabili, relativi al funzionamento, abbiamo preferito usare gli [anchor](https://github.com/thomasloven/hass-config/wiki/Misc-tricks) impostabili nella sezione "IMPOSTAZIONI DEL PACKAGES".
Al contrario gli aspetti di configurazione che sono più soggetti a variazioni sono configurabili da UI.

Altra funzinalità innovativa del package è l'uso dellle funzionalità di esposizione dei dati statistici delle card standard Lovelace e Apex senza quindi incidere nelle dimensioni del DB, il `recorder` può quindi rimanere configurato con i classici 5-10 giorni di memorizzazione dei dati ma è possibile visualizzare i dati di energia consumata degli ultimi 30 giorni o più se uno desidera.

## User Interface

La card è in sostanzialmente una _custom button-card_ con funzionalità di _container_ che al suo interno contiene due importanti sezioni:
* la parte alta è costituita da una _picture-elements card_
* la parte bassa è costituita da una fila orizzontale di 5 bottoni a loro volta realizzati con _custom button-card_

La picture-elements card ha la funzione di _contenitore_ sia per elementi grafici che informazioni testuali:
* **sulla parte sinistra** è presente l'immagine grafica della lavatrice, i CSS presenti nella configurazione fanno in modo che ad ogni stato corrisponda una animazione
* **nella parte centrale** è collocato il grafico della potenza assorbita dalla lavatrice
* **nella parte destra** è presente il grafico a barre dei cicli di lavaggio che indica quanit cicli sono stati effttuati e dove il 100% rappresenta il numero di cicli dopo il quale è  necesario efettuare una manutenzione.
* **in alto a destra** troviamo le informazioni testuali relative ad energia, potenza, stato, n° cicli.
* **in basso** sotto l'immagine della lavatrice è presente la scelta del programma della lavatrice con il tempo rimasto

La sezione orizzontale di bottoni in basso presenta le seguenti informazioni:
* **Info**: cliccando si apre una finestra con le informazioni di dettaglio relative ai consumi/cicli/costi per giorno, settimana, mese, anno oltre alle durate e cicli totali. Sono inoltre peresenti le informaizoni relative alla mnutenzione da effettuare e la sua configurazione e le impostazioni relartive alle noticihe.
* **Settimana** / **Mese** / **Anno**: informaizoni relative all'energia consumata e i costi associati
* **Lavatrice**: bottone che con il tocco singolo spegne/accende la presa collegata all'elettrodomestico, con il tocco prolungato permette apre la finestra con i dati e i grafici di energia recuperati dalle informazioni statistiche contenute nel DB di HA

Come detto la card è adattabile al dispositivo usato e al suo orientamento, non si tratta ina card pienamente _responsive_ ma si è cercato di rendere la fruizione buona sia su smartphone che su PC o tablet.

## Configurazione
| Impostazioni Package |
| :---: |

Nella sezione "IMPOSTAZIONI DEL PACKAGE" presente nel file _washing_machine_xxx.yaml_ sono da configurare:
1. servizi di notifica
2. i principali dati necessari per il funzionamento come lo switch della presa o il sensore di potenza.

Per i **servizi di notifica** sono da configurare i seguenti paramentri, ovviamente l'utente smaliziato 
* _SCRIPT Centro notifiche: &DEFAULT_SCRIPT_ se utilizzate il famoso cEntro Notifiche di HassioHelp basta lasciare solo questo paramentro
* _SERVIZIO Notify: &DEFAULT_NOTIFY_ dove inserire l'entità _notify_ principale, ad esempio _notify.telegram_
* _SERVIZIO Mobile: &DEFAULT_NOTIFY_MOBILE_ dove inserire l'entità _notify_ del servizio di notifica della [Companion App](https://companion.home-assistant.io/) di HA
* _MEDIA PLAYER Alexa: &DEFAULT_MEDIA_PLAYER_ALEXA_ deve inserire un' entità media player tra quelle configurate, ad esempio _media_player.alexa_sala
* _MEDIA PLAYER Google: &DEFAULT_MEDIA_PLAYER_GOOGLE_ deve inserire un'entità media player di proporio gusto, ad esempio _media_player.google_sala
* _SERIVICE TTS: &DEFAULT_SERVICE_TTS_ dove inserire il servizio TTS, ad esempio _tts.google_translate_say_

Andando nel dettaglio i sequenti sono i paramentri che si possono modificare, gli altri presenti non sono da cambiare:
* _PLUG LAVATRICE SWITCH: &CONF_SWITCH_ENTITY_ dove inserire l'entità _switch_ che comanda la presa smart
* _PLUG LAVATRICE POTENZA (W): &CONF_POWER_ENTITY_ dove inserire l'entità _sensor_ che espone la potenza
* _PLUG LAVATRICE ENERGIA (kWh): &CONF_ENERGY_ENTITY_ dove inserire l'entità _sensor_ che espone l'energia
* _GROCY DETERSIVO ID: &CONF_DETERSIVO_ID_ dove inserire l'ID corrispondente al detersivo
* _GROCY DETERSIVO (mm): &CONF_DETERSIVO_AMOUNT_ dove inserire la quantità di detersivo
* _GROCY AMMORBIDENTE ID: &CONF_AMMORBIDENTE_ID_ dove inserire l'ID corrispondente all'ammorbidente
* _GROCY AMMORBIDENTE (mm): &CONF_AMMORBIDENTE_AMOUNT_ dove inserire la quantità di ammorbidente
* _RITARDO TRA LE VARIE NOTIFICHE IN MINUTI: &DELAY_NOTIFICATION_ indica quanti minuti passano tra una notifica ed un altra

Altri parameteri di configurazione sono presenti nell'interfaccia utente:
* "_Ripetzioni Notifica_" indica il numero di ripetizioni della notifica di fine lavaggio
* "_Cicli di avviso manutenzione_" indica dopo quanti cicli il ssitema notifica di effettuare la manutenzione.

| Lovelace |
| :---: |

Anche nella parte lovelace è stato fatto un uso estensivo degli [anchor](https://github.com/thomasloven/hass-config/wiki/Misc-tricks) in modo ca centralizzare le impostazionida variare, purtroppo questo tipo di modifica è possibile relizzarla solo con la modalità lovelace YAML, per la modalià storage non è possibile inserire _anchor_.

Vediamo i principali asetti di configurazione dell'interfaccia Lovelace:
* _SFONDO_IMMAGINE: &CONF_BACKGROUND_IMAGE_ immagine di sfondo
* _GRANDEZZA_CARATTERI_BASE: &CONF_FONT_SIZE_ grandezza caratteri per gli elementi della picture-elements card
* _ALTEZZA_LINEA: &CONF_LINE_HEIGHT_ altezza dell'elemento di stile CSS _line-height_
* _COLORSCHEME_RGB_XXX: &CONF_COLOR_XXX_ definizione di colori
* _PLUG_LAVATRICE_XXX_: associazione con le entità relative a vari parametri come potenza energia etc

| Template button card |
| :---: |

Per utilizzare i button-card templates occorre procedere alla seguente configurazione nel proprio file ui-lovelace.yaml o in altro se avete configurate più interfacce Lovelace in modalità YAML.


```yaml
button_card_templates:
  !include_dir_merge_named /button_card_templates/pkg_elettrodomestici
```

| Struttura dei file |
| :---: |

La struttura dei file è la seguente, effettuando il download da [Github](https://github.com/jumping2000) del pacchetto occorre rispettare la posizione dei file nel filesystem come da schema sottostante, in alternativa l'utente esperto può riposizionare i files nella maniera che più preferisce. 
L'unica eccezione è la card Lovelace che può essere posizionata nelle viste - [View Lovelace](https://www.home-assistant.io/dashboards/views/) - già presenti.

```bash
.
└── addons/
└── backup/
├── config/
│   │
│   ├── button_card_templates/
│   │   └── pkg_elettrodomestici/
│   │       └── button_card_templates.yaml
│   │
│   ├── packages/
│   │   └── elettrodomestici/
│   │       ├── washing_machine_fsm.yaml
│   │       └── washing_machine_lite.yaml
│   │
│   ├── lovelace/
│   │   └── card_elettrodomestici/
│   │       └── washing_machine_card.yaml
│   │
│   ├── www/
│   │   └── hassiohelp/
│   │       └── pkg_elettrodomestici/
│   │           ├── lavatrice.png
│   │           ├── lavatrice_oblo.png
│   │           ├── lavatrice_panni.png
│   │           ├── sfondo_black.png
│   │           ├── sfondo_white.png
│   │           ├── sfondo_gray.png
│   │           └── sfondo_gray2.png
│   │ 
│   ├── home-assistant.log
│   └── secret.yaml
└── share/
└── ssl/
```

## Ringraziamenti
