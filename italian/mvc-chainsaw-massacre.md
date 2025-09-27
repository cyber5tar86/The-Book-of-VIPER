Non è difficile iniziare un nuovo progetto con l'architettura VIPER come base. Ma costantemente gli sviluppatori devono supportare e migliorare applicazioni che sono state inizialmente sviluppate con una base di codice caotica, senza considerare regole rigorose di design e architettura. Spesso accade che la prima versione di un progetto non abbia molti requisiti e possano stare su un singolo foglio di carta. Pertanto, non viene prestata abbastanza attenzione all'architettura dell'applicazione in questa fase. Ma i termini di riferimento per la seconda versione possono avere tante pagine quante ne ha "Guerra e Pace" (di Leo Tolstoy). Ecco perché, un compito più complesso, ma interessante allo stesso tempo, è migrare un'applicazione iOS esistente da una base debole a una solida, basata su un'architettura VIPER flessibile e robusta.

### Perché MVC diventa Massive-View-Controller

**Model-View-Controller (MVC)** è un pattern architetturale di base per le applicazioni iOS offerto da Apple. Solitamente questo pattern viene seguito esplicitamente o implicitamente da tutti gli sviluppatori novizi. Model-View-Controller è ovviamente un pattern architetturale a tre livelli. Quando lo si utilizza, tutti gli oggetti dell'applicazione, a seconda dello scopo, appartengono a uno di tre livelli: Model (Modello), presentation (View) o control (Controller). Ogni livello architetturale è separato dagli altri da confini astratti, che vengono utilizzati per la comunicazione tra oggetti di livelli diversi. Lo scopo principale di questo concetto architetturale è la separazione della logica di business (modello) dalla sua visualizzazione (rappresentazione).

**Il livello model** descrive i dati di un'applicazione e determina la logica di elaborazione e archiviazione. Gli oggetti modello non dovrebbero avere un collegamento chiaro con gli oggetti del livello di presentazione; non contengono informazioni su come i dati potrebbero essere visualizzati. Come esempio di oggetti di questo livello possono essere oggetti di archiviazione dati, parser, client di rete, ecc.

**Il livello Presentation** contiene oggetti che un utente può vedere. Gli oggetti di questo livello sono solitamente riutilizzabili. Questi includono classi come UILabel e UIButton.

**Il livello Controller** media l'interazione tra gli oggetti del livello di presentazione e gli oggetti modello: controlla l'input dell'utente e usa un modello e una vista per implementare la reazione necessaria. Inoltre, gli oggetti di questo livello sono utilizzati per la definizione e coordinazione dei problemi dell'applicazione, gestione del ciclo di vita di altri oggetti.

![Model-View-Controller](../Resources/MVCMassacre/MVC.png)

> **Nota**. Nonostante il fatto che Apple nella sua documentazione si riferisca al pattern architetturale descritto come classico Model-View-Controller, tuttavia, il suo nome corretto è **Model-View-Adapter (MVA)** o **mediating-controller MVC**.

Secondo le raccomandazioni Apple, i **presentation controllers (view controllers)** sono il nucleo del livello di controllo in un'app iOS. Ogni tale oggetto è responsabile di:

* gestione della gerarchia delle viste;
* ridimensionamento delle viste di presentazione per adattarsi a una dimensione specifica del display del dispositivo;
* aggiornamento del contenuto di presentazione in risposta a un cambiamento dei dati;
* elaborazione dell'input dell'utente e trasmissione dei dati al livello del modello;
* rilascio di risorse correlate in caso di memoria insufficiente;

Il modo raccomandato per descrivere un view controller e la vista correlata è un editor Storyboard. Utilizzando questo strumento, è possibile non solo specificare quali oggetti di presentazione appartengono a un view controller specifico, ma anche definire relazioni gerarchiche e transizioni tra controller diversi.

![Storyboard editor](../Resources/MVCMassacre/Storyboard editor.png)

Apple fornisce alcuni suggerimenti per creare un view controller:

* **Usa le classi view controller fornite con l'SDK standard.**
* **Rendi un View Controller il più indipendente possibile.**
* **Non memorizzare dati in un view controller.**
* **Usa un view controller per reagire agli eventi esterni.**

Queste raccomandazioni e consigli, nonostante la loro utilità, sono troppo generali, descritti senza specificare definizioni concrete, pertanto hanno una serie di svantaggi. Il principale di essi è la crescita incontrollata della complessità dei view controllers. Il loro stretto accoppiamento con il ciclo di vita della vista, che è la caratteristica specifica dell'SDK iOS, richiede la generazione di una risposta agli eventi di questo ciclo direttamente nel view controller.

Di conseguenza, i view controllers diventano il nucleo di quasi tutto ciò che accade in un'applicazione e, di conseguenza, crescono fino a dimensioni gigantesche, trasformandosi in qualcosa chiamato **Massive-** o **Mega-View-Controller**.

![Massive-View-Controller](../Resources/MVCMassacre/MassiveViewControllerScheme.png)

I principali svantaggi dell'uso del Massive-View-Controller sono:

* **Alta complessità di supporto e sviluppo.**
* **Una soglia di ingresso elevata.**
* **Codice scarsamente testabile.**
* **È quasi impossibile riutilizzare il codice.**

Per evitare questi inconvenienti e impedire che un processo di sviluppo dell'applicazione risulti in uno stallo, è necessario seguire rigorosamente i requisiti di un pattern architetturale ben progettato. Model-View-Controller, nonostante le debolezze e le soluzioni non ovvie ad alcuni problemi di design, a causa della facilità d'uso, è ancora adatto per applicazioni più piccole o applicazioni che richiedono alta velocità di sviluppo. Tuttavia, per progetti grandi e complessi, con ciclo di supporto e sviluppo a lungo termine è meglio usare un'architettura più flessibile - VIPER.

### Da una implementazione povera a una buona. Refactoring del Massive-View-Controller.

Di fronte al compito di sviluppare un'applicazione la cui base di codice è gravata dal Massive-View-Controller, prima di tutto devi migliorare la struttura del progetto. Di conseguenza, questo richiede la migrazione da un Massive-View-Controller grande e difficilmente mantenibile in qualcosa di più conveniente e flessibile, ad esempio, un modulo VIPER.

Nel VIPER, a differenza dell'MVC, il view controller è il nucleo del livello view, che consente di non gravarlo con logica aggiuntiva e utilizzarlo solo per l'implementazione dei compiti direttamente correlati alla visualizzazione dei dati.

Ci sono tre esempi di transizione da un Massive-View-Controller a un modulo VIPER descritti di seguito. Un'implementazione ViewController da un progetto reale (una guida a una delle città del mondo) è utilizzata come punto di partenza. Questo view controller contiene l'implementazione del menu di navigazione principale.

![Massive-View-Controller Example](../Resources/MVCMassacre/MassiveViewControllerExample.png)

#### Esempi di Refactoring

**Transizione tra moduli**

Questo frammento di codice implementa una transizione a una sezione interna dell'app. La direzione della transizione è determinata dall'indice della cella della tabella del menu selezionata.

```objective-c
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    long index = indexPath.row;
    switch (index) {
        case MainMenuCellsCity: {
            UIStoryboard *sb = [TyphoonStoryboard storyboardWithName:@"Quarters"
                                                             factory:AGAppDelegateMacros.storyboardFactory
                                                              bundle:nil];
            UIViewController *vc = [sb instantiateViewControllerWithIdentifier:@"agquarterslistviewcontroller"];
            [self.navigationController pushViewController:vc animated:YES];
```

**Risultato del refactoring:**

Nel modulo VIPER, la logica di transizione viene estratta dal livello View e spostata nel Router. Il View Controller informa semplicemente il Presenter dell'evento di selezione della cella, e il Presenter chiede al Router di eseguire la transizione appropriata.

![First Example Result](../Resources/MVCMassacre/FirstExampleResult.png)

Questo approccio fornisce una migliore separazione delle responsabilità e rende il codice più testabile e mantenibile. La logica di navigazione è ora incapsulata nel Router, rendendo facile modificare o estendere il comportamento di navigazione senza influenzare altri componenti del modulo.

Il refactoring da MVC a VIPER richiede un investimento iniziale di tempo e sforzo, ma fornisce benefici significativi a lungo termine in termini di manutenibilità, testabilità e scalabilità del codice.