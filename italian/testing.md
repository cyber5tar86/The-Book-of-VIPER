## TDD e VIPER

### Breve Panoramica del TDD

TDD - Test Driven Development. Presuppone che i test siano scritti per primi, poi l'implementazione. Se copriamo le classi reali con protocolli, vengono per primi. Dopo che tutti i componenti sono scritti - possono essere integrati.

Grazie a questo approccio si ottiene una descrizione completa del comportamento atteso della classe nei test. I test possono servire come esempi di utilizzo della classe. Inoltre il TDD ci permette di guardare imparzialmente alla classe dal punto di vista dell'utente di quella classe prima di scriverla.

L'argomento TDD in iOS è stato coperto in maggior dettaglio nell'[articolo](http://habrahabr.ru/company/rambler-co/blog/263087/) di Andrey Rezanov.

## VIPER

Solitamente la separazione nei livelli principali dell'applicazione è sufficiente per testare abbastanza bene il livello di servizio e Core, ma con la presenza di un "grasso" ViewController sorgono problemi nel testing del livello Presentation. Pertanto molti lo lasciano senza copertura di test sufficiente. Capiamo come i moduli VIPER possono aiutarci nella copertura della View con i test.

L'approccio generale per il testing è il seguente: circondiamo l'oggetto della classe testata con mock protocolli delle dipendenze. Chiamiamo metodi dell'interfaccia/manipoliamo le proprietà, controlliamo le chiamate ai metodi dei mock.

### Testing della View

Esiste un certo numero di librerie che aiutano nel testing del livello UI, e di recente abbiamo anche i UI test. È sufficiente questo per una copertura completa della View? A nostro avviso - no.

I UI test possono servire come un buon modo per scrivere test di accettazione. L'intera applicazione di produzione agisce come scatola nera, e attraverso l'interfaccia dell'applicazione cerchiamo di ottenere un risultato o un altro.

Il problema con i UI test è che nel caso di un VC sovraccarico, l'intera View è per noi una scatola nera con molti stati, e per il testing è necessario un numero troppo grande di test.

Come può aiutarci VIPER qui? Dal ViewController viene estratta la logica per la preparazione e presentazione dei dati nel Presenter. Di conseguenza, sulla View ricade la responsabilità solo per la gestione e il passaggio di eventi al Presenter, così come per la visualizzazione dell'UI. È necessario prestare particolare attenzione al fatto che View - non è necessariamente UI, ma una classe attraverso la quale avviene l'interazione del mondo esterno con il modulo. Cosa significa questo per noi? IBOutlet e IBAction **devono** essere resi pubblici. Come risultato otteniamo la possibilità di testare tutti i possibili tocchi, riempimenti di campi e altro senza simulazione di tocchi, ricerca dei pulsanti giusti per testo su di essi e altre cose inaffidabili.

Riassumiamo, evidenziando 2 tipi principali di test:

- Interagiamo con IBOutlet/chiamiamo IBAction -> controlliamo che siano stati chiamati i metodi corrispondenti del mock Presenter
- Chiamiamo metodi del protocollo attraverso cui comunica il Presenter -> controlliamo che cambino IBOutlet/View

Separatamente si può evidenziare il testing dei metodi del ciclo di vita del ViewController/View, sui quali dobbiamo orientarci in un modo o nell'altro, poiché spesso il Presenter non può iniziare la configurazione della View prima di `-viewDidLoad` o `-viewWillAppear`.

### Testing del Router

I metodi del router vengono chiamati quando è necessario effettuare una transizione dal ViewController corrente. Di conseguenza, i test coprono i metodi di transizione/chiusura del controller corrente. Testiamo le chiamate di vari animatori di transizione.

### Testing dell'Interactor

L'Interactor è l'anello di collegamento nel lavoro con vari servizi. È attraverso di esso che avviene il lavoro con lo Storage, in esso vengono creati oggetti PONSO.

La maggior parte dei test riguarda il controllo delle chiamate di alcuni servizi a seconda delle risposte di altri.

### Testing del Presenter

Il Presenter può essere chiamato l'anello di collegamento del modulo, poiché in esso avviene il proxy delle richieste da una parte del modulo all'altra. I test su tale trasferimento costituiscono la parte del leone dei test del Presenter.

È da notare separatamente che il Presenter è il punto di ingresso per il modulo, i dati dal controller precedente vengono passati ad esso. Su questo è necessario scrivere test anche.

Il Presenter è il luogo dove più spesso si trova il numero massimo di ramificazioni logiche. Questo deve essere preso in considerazione anche, e in ingresso devono essere fornite tutte le possibili combinazioni di valori fondamentalmente diverse tra loro.

### Testing dell'Assembly

L'Assembly configura le dipendenze dei componenti del modulo. I suoi test sono responsabili di controllare che il modulo sia composto dalle parti giuste e che tutte le dipendenze siano riempite.

Fortunatamente, con una struttura rigorosa del modulo, questi test possono essere creati automaticamente.