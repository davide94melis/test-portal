# Review Documentazione BR Monitoring V2

Data review: `2026-05-05`

Documentazione analizzata:
- `brs/monitoring-v2/BR - Agency Desk Monitoraggio V6.docx` → `br-docs-converted/BR_Agency_Desk_Monitoraggio_V6.md`
- `brs/monitoring-v2/202604_Macchina Stati Monitoring_v1.xlsx` → `br-docs-converted/202604_Macchina_Stati_Monitoring_v1.md`
- `brs/monitoring-v2/202604_Tracciati Report GEN AI-Piattaforma_v2.xlsx` → `br-docs-converted/202604_Tracciati_Report_GenAI_Piattaforma_v2.md`
- `brs/monitoring-v2/202604_Deck Mockup_Monitoraggio_v10.pptx` → `br-docs-converted/202604_Deck_Mockup_Monitoraggio_v10.md`

Codebase verificati:
- BE (ba-back-end) → `C:\Users\davmelis\Documents\Github\ba-back-end`
- FE (ba-front-end) → `C:\Users\davmelis\Documents\Github\ba-front-end`
- EM (ba-email-manager) → `C:\Users\davmelis\Documents\Github\ba-email-manager`
- DM (ba-document-manager) → `C:\Users\davmelis\Documents\Github\ba-document-manager`

## Esito sintetico

La documentazione del BR Monitoring V2 e' complessivamente ricca e dettagliata nei flussi principali, ma presenta **incoerenze critiche tra i documenti** (soprattutto nei formati degli ID) e **gap funzionali significativi** nella definizione del pipeline GenAI per il monitoraggio e nelle specifiche tecniche di integrazione (COVNO, dashboard real-time). Il mockup (64 slide) e' il documento piu' completo e aggiornato, ma diverge dal BR su nomenclatura, formati ID e alcune funzionalita'.

Problemi trovati: **21 totali** (7 bloccanti, 8 non bloccanti, 6 disallineamenti col codice)

---

## Parte 1 — Per il team funzionale

Questa sezione elenca i punti che richiedono chiarimento o correzione nella documentazione.

### Problemi bloccanti

Questi impediscono una pianificazione affidabile.

#### 1. Formato ID Pratica incoerente tra documenti e sezioni

- **Categoria**: Incoerenza
- **Bloccante**: Si
- **Dove**: BR sez. 3 (ID "M1", "M2"), BR sez. 8.6 (ID "P1"), Mockup slide 3-5 (ID "Prat001")
- **Problema**: Lo stesso concetto — l'identificativo univoco della pratica di monitoraggio — ha tre formati diversi nei documenti:
  - Il BR nelle sezioni 1-7 usa il formato **"M1", "M2"** (prefisso M + numero)
  - Il BR nella sezione 8.6 (sincronizzazione COVNO) usa il formato **"P1"** (prefisso P + numero)
  - Il Mockup usa il formato **"Prat001"** (prefisso "Prat" + numero a 3 cifre con zero-padding)
  
  Non e' chiaro quale sia il formato definitivo, se la generazione e' automatica o manuale, e quale pattern di numerazione si debba usare.
- **Impatto**: Senza un formato definito, non si puo' progettare il sistema di generazione ID, la ricerca, il display e l'integrazione COVNO. L'ID e' presente ovunque nel sistema — ogni scelta ha impatto su tutte le schermate e i flussi.
- **Domanda per il funzionale**: Qual e' il formato definitivo dell'ID pratica? La generazione e' automatica (auto-incrementale, sequenziale) o segue un pattern specifico? Il formato deve essere coerente con quello delle pratiche nelle altre fasi (SACE, Booking, ecc.)?
- **Risposta:** *(inserire qui la risposta)*

#### 2. Formato ID Evento incoerente e struttura gerarchica ambigua

- **Categoria**: Incoerenza
- **Bloccante**: Si
- **Dove**: BR sez. 3 (formato "M3C2E4"), Mockup slide 8 (formato "E001"), Mockup slide 12-15 (formato "Mon001")
- **Problema**: L'ID dell'evento di monitoraggio ha almeno tre formati nei documenti:
  - Il BR definisce una struttura gerarchica **"M3C2E4"** (Pratica 3, Covenant 2, Evento 4) che incorpora l'ID del covenant e della pratica padre
  - Il Mockup in alcune slide usa **"E001"** (prefisso E + numero a 3 cifre)
  - Il Mockup in altre slide usa **"Mon001"** (prefisso Mon + numero a 3 cifre)
  
  La struttura gerarchica "M3C2E4" del BR e' funzionalmente ricca (permette di navigare dalla gerarchia padre) ma diversa da qualsiasi formato nel mockup. Se l'ID e' gerarchico, come si gestiscono i covenant con piu' eventi? Se e' flat, come si risale alla pratica padre?
- **Impatto**: La struttura dell'ID determina il modello dati (relazioni tra entita'), la navigazione nell'interfaccia, e la logica di generazione. Formati diversi implicano architetture diverse.
- **Domanda per il funzionale**: Qual e' il formato definitivo dell'ID evento? Deve essere gerarchico (incorporando pratica e covenant) o flat? Se flat, come si collega all'evento il covenant e la pratica di appartenenza?
- **Risposta:** *(inserire qui la risposta)*

#### 3. Sovrapposizione tra stati "Da attenzionare" e "In Scadenza" nel donut chart

- **Categoria**: Ambiguita'
- **Bloccante**: Si
- **Dove**: BR sez. 8.1 (donut chart con 4 segmenti: OK, In Scadenza, Da attenzionare, Scaduto), BR sez. 5 (macchina stati con 6 stati: OK, In Scadenza, Da attenzionare, Scaduto, Fuori Soglia, Conclusa), Macchina Stati XLSX (definizioni trigger)
- **Problema**: Il donut chart della dashboard mostra 4 segmenti, ma la macchina stati definisce 6 stati. I 2 stati mancanti nel donut sono:
  - **Fuori Soglia**: non e' nel donut. Viene aggregato in "Da attenzionare"? O e' un sotto-stato?
  - **Conclusa**: non e' nel donut. Le pratiche concluse escono dal conteggio?
  
  Inoltre, "Da attenzionare" e "In Scadenza" hanno trigger che possono sovrapporsi temporalmente: un covenant puo' avere valore entro il 10% della soglia (→ Da attenzionare) E scadenza entro 30 giorni (→ In Scadenza) allo stesso tempo. Quale stato prevale?
- **Impatto**: Senza regole di priorita' tra stati, il donut mostra dati inaffidabili. Senza sapere se "Fuori Soglia" e "Conclusa" sono inclusi o esclusi, i totali non tornano.
- **Domanda per il funzionale**: (a) Come si mappano i 6 stati della macchina stati ai 4 segmenti del donut? (b) Se un covenant e' contemporaneamente "In Scadenza" e "Da attenzionare", quale stato prevale? (c) Le pratiche "Conclusa" escono dal donut? (d) "Fuori Soglia" dove va nel donut?
- **Risposta:** *(inserire qui la risposta)*

#### 4. Meccanismo di aggiornamento real-time della dashboard non specificato

- **Categoria**: Gap funzionale
- **Bloccante**: Si
- **Dove**: BR sez. 8.1 ("I valori del grafico sono aggiornati in tempo reale"), Mockup slide 2-3 (dashboard con donut chart)
- **Problema**: Il BR afferma che il donut chart deve aggiornarsi "in tempo reale" ma non specifica:
  - Il meccanismo tecnico (WebSocket, Server-Sent Events, polling con intervallo, push notification)
  - La granularita' dell'aggiornamento (ogni cambio di stato? batch periodico?)
  - Se "tempo reale" significa davvero real-time (sub-secondo) o near-real-time (minuti)
  - Se l'aggiornamento riguarda solo il donut o anche le tre tabelle sottostanti
  
  Questa e' una scelta architetturale significativa che impatta performance, costi infrastrutturali e complessita' di implementazione.
- **Impatto**: WebSocket richiede infrastruttura dedicata; polling puo' bastare ma va dimensionato. Senza sapere i requisiti reali, si rischia di sovra- o sotto-ingegnerizzare la soluzione. La piattaforma attuale non ha precedenti di comunicazione real-time.
- **Domanda per il funzionale**: Cosa si intende per "aggiornato in tempo reale"? E' accettabile un refresh automatico ogni N minuti (es. 5 min)? Oppure l'utente deve vedere le variazioni istantaneamente senza ricaricare? L'aggiornamento riguarda solo il donut o l'intera dashboard?
- **Risposta:** *(inserire qui la risposta)*

#### 5. Pipeline GenAI per monitoraggio non definito nel documento Tracciati

- **Categoria**: Gap funzionale
- **Bloccante**: Si
- **Dove**: BR sez. 6 (flusso GenAI per certificati di conformita'), Tracciati GenAI XLSX (contiene solo pipeline per SACE, Pre-Closing, Booking, Post-Closing), Mockup slide 18-22 (schermata assegnazione GenAI/manuale)
- **Problema**: Il BR descrive un flusso di estrazione GenAI per i certificati di conformita' nella fase di monitoraggio: il sistema carica il documento, lo invia alla GenAI che estrae i valori dei covenant, e l'operatore valida o corregge. Tuttavia, il documento "Tracciati Report GenAI-Piattaforma" non contiene **nessun pipeline per la fase di monitoraggio**. Sono definiti solo i pipeline per:
  - SACE (estrazione dati finanziari)
  - Pre-Closing (verifica documenti)
  - Booking (registrazione operazione)
  - Post-Closing (covenant numerici, descrittivi, obblighi informativi)
  
  Il pipeline Post-Closing per i covenant ha una struttura simile a quella che servirebbe per il monitoraggio (estrae valori covenant da documenti), ma non e' chiaro se il monitoraggio riusi lo stesso pipeline o ne necessiti uno dedicato.
- **Impatto**: Senza la definizione del pipeline GenAI (input document type, campi da estrarre, formato output, mapping con le entita'), non e' possibile pianificare l'integrazione GenAI per il monitoraggio. E' una funzionalita' core del flusso.
- **Domanda per il funzionale**: (a) Il monitoraggio riutilizza il pipeline GenAI del Post-Closing (stessi campi: COVENANT_NUM_*, COVENANT_DESC_*, OBBLIGO_INF_*)? (b) Se no, serve un nuovo tracciato dedicato — quali campi deve estrarre e da quale tipo di documento? (c) Il documento "Tracciati" verra' aggiornato con la sezione monitoraggio?
- **Risposta:** *(inserire qui la risposta)*

#### 6. Formato file COVNO e regole di validazione non specificati

- **Categoria**: Gap funzionale
- **Bloccante**: Si
- **Dove**: BR sez. 8.6 ("Sincronizzazione COVNO"), Mockup slide 25-27 (upload COVNO)
- **Problema**: Il BR descrive la sincronizzazione con COVNO (database ISP per covenant e obblighi) tramite upload di file, ma non specifica:
  - Il formato del file (CSV, XLSX, XML, JSON?)
  - La struttura/schema del file (quali colonne/campi, in che ordine)
  - Le regole di validazione (campi obbligatori, formati date, valori ammessi)
  - Il comportamento in caso di errore (cosa succede se un record non matcha? parziale o tutto-o-niente?)
  - Il campo di matching (il Tracking ID e' lo stesso del Booking? come si mappa pratica-covenant?)
  - La frequenza di sincronizzazione (on-demand, schedulata, event-driven?)
  
  Il BR usa anche il termine "tracking id" per il matching, ma non chiarisce se e' lo stesso `bookingTrackingId` gia' presente nel sistema o un campo diverso.
- **Impatto**: Senza queste specifiche, non si puo' implementare il parser, la validazione, la gestione errori e il matching. E' un'integrazione critica perche' COVNO e' la fonte dati dei covenant da monitorare.
- **Domanda per il funzionale**: (a) Qual e' il formato del file COVNO (CSV, XLSX, altro)? (b) Qual e' lo schema (colonne, tipi, campi obbligatori)? (c) Come si mappa un record COVNO a una pratica nel sistema (via tracking ID del booking)? (d) In caso di errore su un record, l'intero upload fallisce o si prosegue con i record validi? (e) Puo' ISP fornire un file di esempio?
- **Risposta:** *(inserire qui la risposta)*

#### 7. Tipo documento e output GenAI per certificati di conformita' non definiti

- **Categoria**: Gap funzionale
- **Bloccante**: Si
- **Dove**: BR sez. 6 (flusso di estrazione valori covenant), Mockup slide 18-22 (upload certificato, assegnazione GenAI/manuale)
- **Problema**: Il BR descrive che l'operatore carica un "certificato di conformita'" e il sistema puo' estrarre i valori del covenant tramite GenAI. Tuttavia:
  - Non e' definito il **tipo di documento** accettato (PDF, DOCX, scansione immagine?)
  - Non e' definita la **struttura attesa** del certificato (e' un documento standard? varia per banca?)
  - Non e' definito il **formato di output** dell'estrazione GenAI (quali campi restituisce? con che formato?)
  - Non e' definito il **livello di confidenza** sotto il quale la GenAI deve segnalare incertezza
  - Non e' chiaro se il documento e' uno per covenant o uno per pratica (un certificato contiene i valori di tutti i covenant? o uno per ciascuno?)
  
  Il mockup mostra una schermata di assegnazione GenAI/manuale, ma non il risultato dell'estrazione ne' come l'operatore lo valida.
- **Impatto**: Il flusso GenAI e' il cuore della automazione del monitoraggio. Senza sapere cosa entra e cosa esce, non si puo' progettare ne' il prompt, ne' il parser del risultato, ne' l'interfaccia di validazione.
- **Domanda per il funzionale**: (a) Che tipo di documento e' il "certificato di conformita'"? (b) Ha un formato standard o varia per cliente/banca? (c) Un certificato contiene i valori di un solo covenant o di tutti? (d) Quali campi deve estrarre la GenAI dal certificato? (e) Come si presenta il risultato dell'estrazione all'operatore per la validazione?
- **Risposta:** *(inserire qui la risposta)*

### Problemi non bloccanti

#### 1. Colonna "# Documenti in Lavorazione" nel mockup assente dal BR

- **Categoria**: Incoerenza
- **Bloccante**: No
- **Dove**: Mockup slide 3-5 (tabella pratiche recenti), BR sez. 8 (tabelle dashboard)
- **Problema**: Il mockup mostra una colonna "# Documenti in Lavorazione" nelle tabelle della dashboard che non e' descritta nel BR. Non e' chiaro se e' un requisito nuovo non ancora documentato o un elemento di design superfluo.
- **Domanda per il funzionale**: La colonna "# Documenti in Lavorazione" va inclusa? Se si', cosa conta esattamente (certificati caricati ma non ancora validati? documenti in coda GenAI?)?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 2. Macchina stati: versioni "(old)" vs versioni correnti

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: Macchina Stati XLSX (fogli con suffisso "(old)" accanto ai fogli correnti)
- **Problema**: Il documento della macchina stati contiene sia le versioni attuali sia le versioni precedenti con suffisso "(old)" per i livelli Evento, Covenant e Pratica. Non c'e' indicazione esplicita su quali siano le versioni definitive. Le versioni differiscono nei trigger, negli stati intermedi e nelle transizioni.
- **Domanda per il funzionale**: Le versioni senza suffisso "(old)" sono da considerarsi le versioni definitive? Le versioni "(old)" possono essere ignorate?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 3. Trigger "Da attenzionare": soglia 10% e regola 30%+40% non formalizzate

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: Macchina Stati XLSX (transizione verso "Da attenzionare"), BR sez. 5
- **Problema**: Il trigger per lo stato "Da attenzionare" e' definito come: "valore covenant entro il 10% della soglia" OPPURE "deviazione del 30% dal valore atteso con delta del 40% rispetto alla rilevazione precedente". Le percentuali non sono formalizzate:
  - "Entro il 10% della soglia" e' valore > soglia * 0.9? O |valore - soglia| < soglia * 0.1?
  - Il 30% di deviazione e' dal valore target o dal valore della soglia?
  - Il 40% di delta e' calcolato sul valore assoluto o relativo?
  - Queste soglie sono configurabili per covenant o fisse?
- **Domanda per il funzionale**: Potete fornire le formule esatte per il calcolo dei trigger "Da attenzionare" con un esempio numerico?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 4. Flusso "Salta monitoraggio" e "Ignora soglia": condizioni e permessi non definiti

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR sez. 7 (eccezioni), Mockup slide 30-32 (popup eccezione)
- **Problema**: Il BR descrive due flussi di eccezione ("Salta monitoraggio" per scaduti e "Ignora soglia" per fuori soglia) ma non specifica:
  - Chi puo' attivare queste eccezioni (ruolo/permesso richiesto)
  - Se richiedono approvazione (workflow di approvazione? doppia conferma?)
  - Se sono reversibili (si puo' annullare un "salta monitoraggio"?)
  - Se c'e' un limite (si puo' saltare ogni scadenza all'infinito?)
  - Se vengono loggate per audit trail
- **Domanda per il funzionale**: (a) Quali ruoli possono attivare le eccezioni? (b) Servono approvazioni? (c) Sono reversibili? (d) C'e' un limite al numero di eccezioni per pratica/covenant?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 5. Spread tables: fonte dati e modalita' di creazione non definite

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR sez. 7.3 (spread tables), Mockup slide 35-38 (tabelle spread covenant-based e temporal-based)
- **Problema**: Il BR indica che Deloitte crea tabelle di spread (margini) per covenant (covenant-based) o per periodo (temporal-based), ma non specifica:
  - La fonte dati per i valori di spread (inserimento manuale? import da file? calcolo automatico?)
  - Chi li crea e con quale interfaccia (solo Deloitte? anche banca?)
  - La relazione con il covenant (uno spread per covenant? per pratica?)
  - La formula di calcolo se automatico
- **Domanda per il funzionale**: (a) I valori di spread vengono inseriti manualmente da Deloitte o importati? (b) Ogni covenant ha la sua tabella di spread o e' a livello pratica? (c) C'e' una formula di calcolo o sono valori arbitrari?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 6. Mailing list e notifiche: template e trigger non definiti

- **Categoria**: Gap funzionale
- **Bloccante**: No
- **Dove**: BR sez. 7.5 (mailing list), Mockup slide 40-42 (gestione mailing list)
- **Problema**: Il BR menziona una mailing list per notifiche legate agli eventi di monitoraggio ma non specifica:
  - Quali eventi attivano una notifica (cambio stato? scadenza imminente? eccezione?)
  - Il contenuto/template delle email per ciascun evento
  - I destinatari per tipo di evento (tutti nella mailing list? solo per ruolo?)
  - Se le notifiche sono configurabili dall'utente (on/off per tipo)
- **Domanda per il funzionale**: (a) Quali eventi generano notifica email? (b) I template delle email sono gli stessi di altre fasi o servono template dedicati per il monitoraggio? (c) I destinatari sono fissi o configurabili per evento?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 7. Scadenziere: logica di calcolo date e regole di ricorrenza

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR sez. 7.4 (scadenziere), Mockup slide 33-34 (vista scadenziere)
- **Problema**: Il BR descrive uno scadenziere per i covenant con periodicita' (annuale, semestrale, trimestrale, mensile) ma non specifica:
  - Come si calcolano le date di scadenza ricorrenti (da data prima rilevazione? da data inizio covenant?)
  - Cosa succede quando una scadenza cade in un giorno festivo/weekend
  - Se la scadenza e' la data entro cui il documento deve essere caricato o la data del valore di riferimento
  - Se il preavviso (30 giorni per "In Scadenza") e' configurabile
- **Domanda per il funzionale**: (a) La data di scadenza e' calcolata da quale data base? (b) Come si gestiscono festivi/weekend? (c) I 30 giorni di preavviso sono fissi o configurabili?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

#### 8. Terminologia "Pratica" vs "Dossier" vs "Fascicolo"

- **Categoria**: Ambiguita'
- **Bloccante**: No
- **Dove**: BR sez. 1-8 (uso misto di "pratica" e "dossier"), Mockup (usa "pratica" e "fascicolo")
- **Problema**: Il BR alterna tra "pratica di monitoraggio", "dossier" e "fascicolo" senza definire se sono sinonimi o concetti distinti. Nel codice esistente, l'entita' si chiama `Practice` (con discriminator "C"). La coerenza terminologica e' importante per evitare confusione nell'interfaccia utente e nel modello dati.
- **Domanda per il funzionale**: "Pratica", "dossier" e "fascicolo" sono sinonimi nel contesto del monitoraggio? Quale termine deve apparire nell'interfaccia utente?
- **Risposta:** *(inserire qui la risposta)*
- **Nota**: se non arriva chiarimento, il team tecnico procedera' con l'assunzione indicata nella Parte 2.

---

## Parte 2 — Per il team tecnico

Questa sezione contiene le assunzioni che il team tecnico adottera' in assenza di chiarimenti dal funzionale. Ogni assunzione e' legata a un problema della Parte 1.

### Assunzioni proposte

| # | Problema rif. | Assunzione proposta | Rischio se errata | Costo di correzione |
|---|---|---|---|---|
| A-001 | NB-1 (Documenti in Lavorazione) | La colonna mostra il conteggio dei certificati di conformita' caricati ma non ancora validati (stato "In attesa di conferma" + "Verifica dati in corso"). Da includere come requisito. | La colonna potrebbe contare qualcosa di diverso o non essere necessaria | Basso — cambio query/conteggio |
| A-002 | NB-2 (Versioni old) | Le versioni senza suffisso "(old)" sono le definitive. Le versioni "(old)" vengono ignorate. | Se le versioni "(old)" contengono logica ancora valida, potremmo perdere requisiti | Medio — reingegnerizzare la macchina stati |
| A-003 | NB-3 (Trigger Da attenzionare) | Formula: "entro il 10%" = valore covenant che supera il 90% della soglia senza raggiungerla (soglia * 0.9 <= valore < soglia). Regola 30%+40%: deviazione >= 30% dal valore target E variazione >= 40% rispetto alla rilevazione precedente. Soglie fisse, non configurabili per covenant. | Se le soglie sono configurabili serve un'interfaccia di configurazione. Se le formule sono diverse, la macchina stati si comporta in modo errato | Medio — cambio logica di calcolo + eventuale UI configurazione |
| A-004 | NB-4 (Eccezioni) | Le eccezioni richiedono ruolo DTT (Deloitte), non necessitano approvazione, sono reversibili (si puo' riattivare un monitoraggio saltato), non hanno limite, e vengono loggate in un audit trail dedicato. | Se servono approvazioni multi-step, serve un workflow aggiuntivo | Medio — aggiunta workflow approvazione |
| A-005 | NB-5 (Spread tables) | I valori di spread sono inseriti manualmente da utenti DTT tramite form nell'interfaccia. Ogni pratica ha la sua configurazione di spread (covenant-based o temporal-based, non entrambe). | Se i valori vengono importati da file, serve un parser. Se una pratica puo' avere entrambi i tipi, serve logica aggiuntiva | Medio — import + logica duale |
| A-006 | NB-6 (Mailing list) | Template email dedicati per il monitoraggio, analoghi a quelli delle altre fasi. Eventi notificati: cambio stato pratica, scadenza imminente (30gg), eccezione attivata. Destinatari: tutti gli utenti nella mailing list della pratica, non configurabili per tipo evento. | Se i template sono piu' complessi o le regole di routing diverse, serve rework | Basso — cambio template e regole routing |
| A-007 | NB-7 (Scadenziere) | Le date di scadenza ricorrenti si calcolano dalla data di prima rilevazione, avanzando per periodicita' (trimestrale = +3 mesi). Festivi/weekend: la scadenza non si sposta (si usa la data calendario). Il preavviso di 30 giorni e' fisso. La "scadenza" e' la data entro cui il documento di monitoraggio deve essere caricato. | Se la base di calcolo e' diversa o il preavviso e' configurabile, serve rework della logica | Basso — cambio calcolo date |
| A-008 | NB-8 (Terminologia) | "Pratica", "dossier" e "fascicolo" sono sinonimi. Nell'interfaccia si usa "Pratica" per coerenza con il resto della piattaforma e con l'entita' `Practice` nel codice. | Se sono concetti distinti servono entita' e UI separate | Alto — cambio modello dati |

### Disallineamenti col codice

| # | Concetto BR | Nel codice | File/Classe | Nota |
|---|---|---|---|---|
| D-001 | 6 stati monitoraggio: OK, In Scadenza, Da attenzionare, Scaduto, Fuori Soglia, Conclusa | `PsmStateCodeEnum`: nessuno stato con prefisso MONITORING_*. Esistono solo stati per SACE, PRE_CLOSING, BOOKING, POST_CLOSING | BE: `PsmStateCodeEnum.java` | Servono nuovi valori enum per tutti e 6 gli stati + gli stati intermedi di processing documento (In attesa di conferma, Verifica dati in corso, Completato) |
| D-002 | Fase Monitoring nel PSM | BE: `PsmPhaseCodeEnum` ha valore `MONITORING`. FE: `psm-phase.enum.ts` ha `Monitoring` e `MonitoringOutcome`. Ma `psm-phase-code.enum.ts` nel FE NON ha MONITORING | BE: `PsmPhaseCodeEnum.java`, FE: `psm-phase-code.enum.ts`, `psm-phase.enum.ts` | Il BE e' parzialmente allineato (ha la fase), il FE ha una incoerenza interna tra i due enum file |
| D-003 | Email di notifica per eventi monitoraggio | `EmailTemplateEnum`: nessun template per monitoraggio. `NOTIFY_MONITORING_WEBINARS` esiste ma e' per webinar formativi, non per il flusso di monitoraggio | BE: `EmailTemplateEnum.java` | Servono nuovi template: scadenza imminente, cambio stato, eccezione attivata, assegnazione documento |
| D-004 | GenAI estrazione valori covenant da certificato conformita' | `GenAiFormKeyEnum` ha chiavi COVENANT_NUM_*, COVENANT_DESC_*, OBBLIGO_INF_* ma sono per il Post-Closing (commento: "BR v28 sez. 3.1.26"). Non esistono chiavi dedicate al monitoraggio | BE: `GenAiFormKeyEnum.java` | Se il monitoraggio riusa il pipeline Post-Closing, le chiavi esistenti potrebbero bastare. Altrimenti servono nuove chiavi MONITORING_* |
| D-005 | Periodicita' covenant: annuale, semestrale, trimestrale, mensile, altro | `MonitoringPeriodicityEnum`: ANNUALE, SEMESTRALE, TRIMESTRALE, MENSILE, ALTRO — **allineato** | BE: `MonitoringPeriodicityEnum.java` | Unico enum gia' allineato. Verificare che i valori corrispondano 1:1 con quelli del BR |
| D-006 | Entita' monitoraggio: pratica, covenant, evento, scadenziere, spread | Nessuna entita' dedicata al monitoraggio nel codice. `Practice.java` esiste con discriminator "C" per le pratiche generali. `monitoring.component.ts` nel FE e' un componente minimale con solo eventi documento | BE: `Practice.java`, FE: `monitoring.component.ts` | Servono nuove entita': MonitoringPractice (o estensione Practice con nuovo discriminator), MonitoringCovenant, MonitoringEvent, MonitoringSchedule, MonitoringSpread + controller, service, repository per ciascuna |

---

## Riepilogo per br-analyzer

Assunzioni confermate: *(nessuna ancora — in attesa di revisione utente)*

Bloccanti aperti: B-1 (formato ID pratica), B-2 (formato ID evento), B-3 (mapping stati donut), B-4 (real-time dashboard), B-5 (pipeline GenAI monitoraggio), B-6 (formato COVNO), B-7 (tipo documento certificato conformita')

Assunzioni proposte in attesa di validazione: A-001 ... A-008
