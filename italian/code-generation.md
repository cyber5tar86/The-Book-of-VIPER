Creare un nuovo modulo VIPER può essere piuttosto macchinoso e comporta almeno la fornitura di quanto segue:

- **Cinque nuove classi** (Assembly, ViewController, Presenter, Interactor, Router),
- **Cinque nuovi protocolli** (ViewInput, ViewOutput, InteractorInput, InteractorOutput, RouterInput),
- **Cinque nuovi test** (AssemblyTests, ViewControllerTests, PresenterTests, InteractorTests, RouterTests).

Inoltre, tutte le relazioni necessarie devono essere stabilite, i protocolli devono essere implementati, la dependency injection deve essere configurata - e molte altre cose in casi particolari. Questa complessità porta due problemi principali:

- Troppo tempo speso in lavoro di routine
- Gli errori di battitura diventano più frequenti, il che influisce negativamente sullo stile del codice e sulla logica dell'applicazione

Un modo per affrontare il problema è tramite la creazione di template Xcode (stavamo seguendo questo approccio tempo fa in Rambler&Co). Risolve i problemi menzionati sopra, pur avendo alcuni svantaggi:

* Costruire un nuovo template è un processo complesso a causa della sintassi macchinosa
* I template potrebbero smettere di funzionare dopo un aggiornamento di Xcode
* Nessun modo conveniente per aggiungere nuovi template a Xcode (Alcatraz non conta)
* Impossibile aggiungere file template a target diversi (ad es. quando si generano test automatici)
* I template Xcode sono altamente dipendenti dallo sviluppatore, non dal progetto

Per essere indipendenti dall'IDE Xcode e dai suoi dettagli di implementazione, abbiamo deciso di separare il processo di generazione del codice e abbiamo creato una piccola applicazione utility - [Generamba](https://github.com/rambler-digital-solutions/Generamba).

![Generamba](http://s24.postimg.org/gej9cg1cl/generamba.jpg)

---

### Installazione

```
gem install generamba
```

### Utilizzo

#### Configurazione del progetto

Prima di creare un modulo è necessario configurare il `Rambafile` chiamando `generamba setup` (il file si trova nella cartella root del progetto). Contiene nome del progetto, prefisso, azienda, modulo, percorsi alle cartelle di test e un elenco di template utilizzati. Il file è versionato in git e dovrebbe essere utilizzato da tutti gli sviluppatori che mantengono il progetto.

#### Creazione di un nuovo modulo

La generazione di un nuovo modulo viene eseguita chiamando `generamba gen ModuleName TemplateName`. Di conseguenza, creerà tutti i file descritti nel template dato - e saranno aggiunti al progetto Xcode, così come al file system.

#### Lavorare con i template

Tutti i template che vengono utilizzati nel progetto corrente sono descritti nel `Rambafile`. La chiamata a `generamba template install` installerà i template forniti uno per uno - sia da una cartella locale, da un repository git remoto o anche da un catalogo di template ([inclusi quelli condivisi](https://github.com/rambler-digital-solutions/generamba-catalog)). Tutti i template sono memorizzati nella cartella `Templates` del progetto.

L'elenco completo dei comandi e delle loro opzioni può essere trovato nella nostra [wiki](https://github.com/rambler-digital-solutions/Generamba).

Il progetto è open source, quindi tutti coloro che lo desiderano [possono aiutarci nello sviluppo, inviarci segnalazioni di bug e condividere le loro idee](https://github.com/rambler-digital-solutions/Generamba/issues). Inoltre, saremo felici di aggiungere nuovi template al [nostro catalogo](https://github.com/rambler-digital-solutions/generamba-catalog) tramite Pull Request.

### Altri generatori di codice
- [vipergen](https://github.com/teambox/viper-module-generator)
- [boa](https://github.com/team-supercharge/boa)