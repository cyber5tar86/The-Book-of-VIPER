**VIPER** - Approccio architetturale per lo sviluppo di applicazioni (in particolare iOS), basato su [Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) di [Robert C. Martin (Uncle Bob)](http://blog.cleancoder.com/).

![Clean Architecture](../Resources/clean-architecture.png)

**Obiettivi principali di VIPER**:

- Aumentare la copertura dei test del livello di Presentazione, solitamente costituito da *Massive View Controllers*.
- Conformarsi al Principio di Singola Responsabilità.

È importante notare che VIPER non è un elenco di regole e template. È un elenco di raccomandazioni su come costruire un'architettura flessibile, testabile e riutilizzabile. Noi, il Team iOS di **Rambler&Co**, abbiamo adottato alcuni principi canonici e formato le nostre migliori pratiche per alcuni casi.

VIPER sembra difficile all'inizio, specialmente per sviluppatori senza esperienza di lavoro in team su progetti di grandi dimensioni. Potrebbe essere difficile comprenderne i benefici se i moduli indipendenti e l'alta copertura dei test non sono le vostre priorità. Ma VIPER è utile anche per applicazioni piccole.

**Pro e contro di VIPER:**

Pro:

- **Aumento della testabilità** per il livello di presentazione dell'applicazione.
- **I moduli sono indipendenti** l'uno dall'altro. Questo separa l'ambiente di sviluppo e aumenta la riutilizzabilità del codice.
- **Gli approcci architetturali principali sono definiti**. Quindi è molto più facile aggiungere un nuovo sviluppatore al team o spostare il progetto a un altro team.

Contro:

- **Aumenta notevolmente il numero di classi** nel progetto, così come la complessità di creare un nuovo modulo.
- Alcuni principi **non funzionano con UIKit** out of the box.
- **Mancanza di raccomandazioni**, migliori pratiche ed esempi di applicazioni complesse.

Copriremo ciascuno di questi concetti in dettaglio e ci concentreremo su come risolvere questi problemi attraverso il libro.

**Timeline della storia di VIPER:**

- **08.2012** - Articolo [The Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) di Robert Martin
- **12.2013** - Articolo [Introduction to VIPER](http://mutualmobile.github.io/blog/2013/12/04/viper-introduction/) di [MutualMobile](http://mutualmobile.github.io/)
- **06.2014** - Issue #13. objc.io [Architecting iOS Apps with VIPER](https://www.objc.io/issues/13-architecture/viper/) di MutualMobile
- **07.2014** - [iPhreaks Show podcast](https://itunes.apple.com/ru/podcast/the-iphreaks-show/id634022060?mt=2&i=316803444) di MutualMobile. Storia di VIPER, obiettivi e utilizzo.
- **04.2015** - L'hackathon VIPER di Rambler&Co porta alla prima applicazione.
- **12.2015** - Rambler&Co ha dozzine di applicazioni VIPER in sviluppo e [rilasciate nell'AppStore](https://itunes.apple.com/ru/developer/rambler-internet-holdings/id395455934).