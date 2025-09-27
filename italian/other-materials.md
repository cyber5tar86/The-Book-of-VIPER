### Premessa

Ci sono molti articoli, presentazioni e video relativi a VIPER su Internet. Alcuni di essi sono buoni e utili, alcuni sono dannosi. Ma è meglio studiare la maggior parte di essi.

**Avviso importante:** I materiali in questa lista non significano che siamo d'accordo con gli autori e appoggiamo le loro idee. È possibile che ci piacciano semplicemente i font e abbiamo trovato alcuni scherzi divertenti.

### Articoli
- [objc.io #13 - Architecting iOS Apps with VIPER](http://www.objc.io/issues/13-architecture/viper/)

  **Autori:** Jeff Gilbert, Conrad Stoll.

  **Recensione:** *Classici senza tempo che dovreste aver già letto. Due cose lo rendono famoso: primo, è stato scritto ai tempi del vero objc.io, secondo, è stato scritto dall'autore di VIPER Jeff Gilbert.*

  *La realizzazione di VIPER in questo articolo non dovrebbe essere presa seriamente. Non è ottimale e ha bug in alcuni casi. Ma ha progetti di esempio in ObjC e SWIFT :)*

- [Introduction to VIPER](http://mutualmobile.github.io/blog/2013/12/04/viper-introduction/)

  **Autore:** Jeff Gilbert.

  **Recensione:** *Un altro articolo da leggere assolutamente, MutualMobile racconta del loro percorso verso VIPER. Dovreste prestare attenzione ai primi paragrafi. Lì Jeff dice che l'uso di tale architettura è stato forzato dalla necessità di test UI. C'è anche un bel punto sul naming - le prime lettere VIP e pensare fuori dalle E e R.*

  *Ma non siamo d'accordo con alcuni punti. Specialmente con i concetti di Wireframe, il divieto totale di uso di ManagedObject fuori dall'Interactor e l'uso diretto del DataStore.*

- [Brigade's Experience Using an MVC Alternative](https://medium.com/brigade-engineering/brigades-experience-using-an-mvc-alternative-36ef1601a41f)

  **Autore:** Ryan Quan.

  **Recensione:** *Secondo noi è il miglior candidato per il ruolo di "La migliore introduzione a VIPER". Buon testo, schemi semplici, descrizione chiara delle idee e principi principali. Uno dei tutorial popolari che consiglia di spostare la logica di business nel livello di servizio. Questo articolo è raccomandato come materiale motivazionale per il vostro team, famiglia e amici.*

  *Ovviamente, qui abbiamo il Wireframe. Inoltre, spostare DataManager (lo chiamiamo ServiceFacade) dall'Interactor è un caso così raro da consigliarne l'uso nella maggior parte dei moduli.*

- [The Clean Architecture](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)

  **Autore:** Robert Martin.

  **Recensione:** *Non riguarda VIPER stesso, ma è assolutamente da leggere. Uncle Bob parla di architettura pulita e DI, disegna cerchi e fa molte cose interessanti.*

  *È difficile usare tale architettura nella nostra dura realtà, ma è stata esattamente la cosa che ha portato MutualMobile a creare VIPER.*

### Podcast
- [iPhreaks Show - VIPER](https://itunes.apple.com/ru/podcast/the-iphreaks-show/id634022060?mt=2&i=316803444)

  **Partecipanti:** Conrad Stoll, Jeff Gilbert.

  **Recensione:** *Molte cose potrebbero essere diverse se le idee di questo podcast fossero state dette nell'articolo di Objc.io. Gli autori di VIPER raccontano della loro motivazione, approcci di refactoring, creazione di schermate composite, testing e molte altre cose. Questo podcast è una delle migliori cose che potete trovare su VIPER.*

### Video
- [250 Days Shipping With Swift and VIPER](https://realm.io/news/altconf-brice-pollock-250-days-shipping-with-swift-and-viper/)

  **Speaker:** Brice Pollock.

  **Recensione:** *Allegro, Divertente, SWIFTy. Lo sviluppatore di Coursera racconta dell'esperienza VIPER. Il team non era soddisfatto del modello canonico e ha aggiunto ViewModel, EventHandler e FlowController. Sembra interessante, ma il flusso di dati all'interno del modulo sembra spaventoso.*

- [Clean Architecture - VIPER](https://www.youtube.com/watch?v=OX4rLAJC7lw)

  **Speaker:** Sergi Gracia.

  **Recensione:** *Alcune informazioni sulle responsabilità degli elementi, qualcosa in più sui test, poche parole su SOLID e molto sull'ufficio e il team Redbooth. E bei font nella keynote. Niente di speciale, solo un'altra introduzione al concetto VIPER.*

  *La principale critica è la qualità del Video. Dovrebbe decisamente essere visto con le [slide in parallelo](https://speakerdeck.com/sergigracia/clean-architecture-viper).*