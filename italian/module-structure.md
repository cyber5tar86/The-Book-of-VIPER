## Modularizzazione dell'Applicazione

Qualsiasi progetto può essere diviso in diverse parti logiche con funzionalità chiaramente definite. Nell'app Instagram, la parte principale dell'applicazione è il news feed, in cui possiamo visualizzare le foto, navigare verso la schermata per creare un commento o inviare un messaggio. La scheda successiva è un elenco di like, dal quale possiamo passare a una foto particolare. Ciascuna di queste schermate è una parte autonoma dell'applicazione che svolge bene il compito e sa come avviare la transizione ad altre schermate se necessario. Chiamiamo questi tipi di elementi moduli.

Nel caso dell'app Instagram, specialmente nella schermata Profilo, ci sono il modulo foto (1), il modulo informazioni sull'utente (2), le schede per cambiare l'elenco foto ad altri moduli (3).

![Profile Instagram](../Resources/instagram_example_serkrapiv.png)

## Struttura del modulo VIPER

Per svolgere la sua missione, il modulo deve risolvere diversi problemi. È necessario implementare la logica di business del modulo, il networking, il database, renderizzare l'interfaccia utente. VIPER descrive il ruolo di ogni componente e come interagiscono tra loro. Quindi, il modulo VIPER è composto dai seguenti componenti:

- **View:** responsabile della visualizzazione dei dati sullo schermo e notifica al **Presenter** le azioni dell'utente. Ma **View** non chiede mai i dati. Li riceve semplicemente dal presenter.

- **Interactor:** contiene tutta la logica di business richiesta per questo modulo.

- **Presenter:** riceve informazioni sulle azioni dell'utente da **View** e le trasforma in richieste a **Router** e **Interactor**. Riceve dati da **Interactor**, li prepara e li invia a **View** per la visualizzazione.

- **Entity:** oggetti modello che non contengono alcuna logica di business.

- **Router:** responsabile della navigazione tra i moduli.

## Cosa abbiamo cambiato

VIPER appare nella sua forma originale su [Mutual Mobile](https://www.objc.io/issues/13-architecture/viper/). Abbiamo lavorato con questo approccio e presto ci siamo resi conto che ha alcuni problemi:

1) Nella versione originale di VIPER, **Wireframe** è responsabile del componente **Routing**. Ma è anche responsabile dell'assemblaggio del modulo, della transizione che esegue e dell'impostazione di tutte le dipendenze nel modulo. Questo è sbagliato perché viola il principio della [Singola Responsabilità](https://en.wikipedia.org/wiki/Single_responsibility_principle).

Quindi, abbiamo deciso di dividere **Wireframe** in due parti. Primo, **Router** è responsabile solo delle transizioni tra i moduli. Secondo, **Assembly** è responsabile dell'assemblaggio del modulo e delle dipendenze tra tutti i suoi componenti. Nei nostri progetti, usiamo [Typhoon](https://github.com/appsquickly/Typhoon) per questo scopo, una grande libreria per la Dependency Injection, e l'**Assembly** manuale del codice non viene nemmeno chiamato.

2) **Interactor** nasconde la logica di business. Suona piuttosto spaventoso, perché dietro di esso si nasconde molto lavoro, che è diviso tra le classi specializzate. Inoltre, lo stesso codice viene spesso riutilizzato all'interno dell'**Interactor**. Avevamo bisogno di una comprensione comune su come farlo per non reinventare la ruota in ogni progetto.

Abbiamo deciso di introdurre un livello aggiuntivo di servizi. **Service** - l'oggetto responsabile del lavoro con un tipo specifico di oggetti modello. Ad esempio, un servizio notizie per le notizie corrisponde a un elenco di categorie specifiche, così come informazioni dettagliate su ogni notizia. Il servizio di autorizzazione è responsabile, di fatto, dell'autorizzazione, del recupero password, dell'aggiornamento sessione e così via. Nei **Service**, a sua volta, ci sono dipendenze su oggetti di livello inferiore, responsabili del lavoro con la rete o il database.

I **Services** sono iniettati nell'**Interactor**. Di conseguenza, l'**Interactor** serve principalmente come facade, interagisce con il servizio e invia i dati risultanti ai **Presenters**. Noi come team abbiamo concordato di non andare ai livelli sopra l'**Interactor** quando utilizziamo Core Data NSManagedObject. Pertanto nell'**Interactor** avviene anche la conversione di NSManagedObject in Plain Old NSObject, cioè un semplice NSObject.

3) Nell'unità VIPER come **View** spesso agisce UIViewController. E a volte nel controller contiene codice che non è direttamente correlato ai compiti della View. Ad esempio, lavorare con tabelle e collezioni. Questi oggetti sono creati con protocolli. Quindi, è inteso che la loro implementazione dovrebbe essere spostata in oggetti separati. Ma, nell'esempio del codice Mutual Mobile, le tabelle sono direttamente nell'UIViewController.

Non ci piace, e abbiamo deciso che la View in generale non è un singolo oggetto e livello. Oltre al controller, questo livello può contenere oggetti aggiuntivi, che sono iniettati nel controller e si prendono carico di parte del suo lavoro. Un esempio di tale oggetto nei nostri progetti è DataDisplayManager, implementa i metodi UITableViewDatasource e UITableViewDelegate, o i loro codici per le collezioni.

4) La trasmissione dei dati tra i moduli non è coperta nel VIPER originale. La nostra soluzione a questo problema è descritta nel capitolo [transizioni tra i moduli](ModuleTransitions.md) in dettaglio ma non ci soffermeremo su di esso. È sufficiente dire che ogni modulo può avere interfacce di input e output - protocolli ModuleInput e ModuleOutput. Il primo è responsabile del trasferimento dei dati di input, come l'identificatore dell'articolo per visualizzare gli articoli. Il secondo è responsabile della trasmissione del risultato del modulo. Ad esempio, possiamo inviare l'elemento selezionato dall'utente quando lavoriamo con il modulo del menu delle impostazioni.

## Lo schema finale del modulo

Per illustrare tutto quanto sopra, suggeriamo di familiarizzare con lo schema finale dell'unità VIPER. Non abbiate paura. Non ogni modulo deve contenere un numero di oggetti. Lo scopo di questo schema è mostrare il più completamente possibile il nostro approccio all'architettura come esempio di un modulo complesso.

![VIPER Driving Module](../Resources/module_structure.png)