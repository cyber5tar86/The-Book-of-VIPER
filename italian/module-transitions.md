## Transizioni tra Moduli - Introduzione

L'approccio alla creazione di moduli tramite factory, descritto negli articoli su VIPER canonico, è piuttosto scomodo. Abbiamo deciso di provare i `UIStoryboardSegue` nativi per le transizioni tra moduli. Questo approccio apriva prospettive allettanti - dopotutto, per la transizione a un altro modulo sarebbe stato necessario solo specificare il SegueID e passare dati per lavorare al modulo. Inoltre, per la configurazione dei moduli utilizziamo Typhoon, quindi tutti i ViewController modulari dopo l'inizializzazione tramite Segue hanno già connessioni con altri componenti del modulo.

## Transizioni tra Moduli - tramite ViewController

Questa è la variante più semplice. Il router del modulo chiamante ha un riferimento al suo ViewController, quando si transizione a un altro modulo viene chiamato il metodo `-prepareForSegue:` del ViewController, dove i dati per il modulo successivo vengono passati nel `sender`. All'interno del `-prepareForSegue:` del ViewController chiamante questi dati vengono passati al modulo successivo.

Questo approccio funziona, ma ci sono alcuni svantaggi:
- La logica per configurare il modulo successivo è collocata all'interno della View, non nel Router,
- Non c'è universalità e riutilizzo, questo metodo deve essere implementato in ogni modulo,
- I dati per il lavoro del modulo successivo finiscono nella View, non nel Presenter,
- Ogni modulo sa della struttura di un altro modulo,
- Ogni router sa che lavora con la classe `UIViewController`, e lo schema funziona solo per questa variante.

## Transizioni tra Moduli - tramite ViewController con Blocco di Configurazione

Per risolvere i primi due problemi, sono stati utilizzati method-swizzling e blocchi. Nel `-prepareForSegue:` viene inviato un blocco nel `sender`, nel quale viene eseguita la configurazione del modulo tramite `destinationViewController`. Nel metodo alternativo `-prepareForSegue:` il blocco viene chiamato con destinationViewController dal segue come parametro.

Questo funziona, la logica per configurare il modulo successivo è interamente all'interno del Router, ogni modulo non richiede più di aggiungere il metodo `-prepareForSegue:` al ViewController, ma rimangono tre problemi:

- I dati per il lavoro del modulo successivo finiscono nella View, non nel Presenter,
- Ogni modulo sa della struttura di un altro modulo,
- Ogni router sa che lavora con ViewController e lo schema funziona solo per questo.

## Transizioni tra Moduli - Molti Protocolli

Per risolvere i problemi rimanenti, sono stati utilizzati protocolli. Molti protocolli. Così come swizzling e implementazione promise personalizzata. Di conseguenza, abbiamo ottenuto un sistema di trasferimento dati tra moduli senza gli svantaggi elencati, i dati dal presenter vengono dati al router e configura il presenter del modulo successivo con essi. Ma sono apparsi due nuovi problemi:
- Ci volevano circa 2 giorni per un nuovo sviluppatore per padroneggiarlo,
- I dati venivano trasmessi solo in una direzione.

## Transizioni tra Moduli - Variante con ModuleInput

La variante attuale, disponibile sul nostro Github sotto il nome [ViperMcFlurry](https://github.com/rambler-digital-solutions/ViperMcFlurry) è diventata molto più facile da padroneggiare. Ogni modulo ora ha un punto di ingresso - ModuleInput, che permette di configurare il modulo o chiamare metodi. Questo moduleInput può essere utilizzato all'interno del router per la configurazione del modulo, può essere restituito al presenter per connessione permanente con il sottomodulo.
Ogni modulo può avere ModuleOutput impostato per restituire dati dal modulo. ModuleInput/Output sono protocolli che vengono impostati all'interno del modulo, cioè memorizzano il contratto per la comunicazione con esso. Nella maggior parte dei moduli, il presenter di questo modulo funge da ModuleInput, e il presenter del modulo chiamante funge da ModuleOutput.

## Embed Segue

Poiché il metodo `-performSegue` richiede solo il nome della transizione, `-prepareForSegue:` è stato sottoposto a swizzling, e Typhoon configura il modulo per ViewController, possiamo utilizzare qualsiasi classe Segue e il meccanismo di transizione tra moduli funzionerà.

Pertanto, per incorporare moduli è stato creato un tipo speciale `UIStoryboardSegue` EmbedSegue. All'interno del `-performSegue` il SourceViewController chiama un metodo che restituisce View per l'identificatore Segue. Il modulo viene incorporato in questa View.