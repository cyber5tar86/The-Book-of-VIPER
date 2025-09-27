## Modulo Semplice vs Modulo Complesso

Quando abbiamo iniziato a utilizzare VIPER nel nostro lavoro, utilizzavamo il concetto "Una schermata - un modulo". Questo funzionava perfettamente perché le schermate erano principalmente tabelle semplici. Ma con l'apparizione della prima schermata "complessa", sono iniziati i problemi.

## Esempi di moduli complessi da Mail - schermata delle impostazioni e visualizzazione messaggi
![submodules.001](../Resources/submodules/submodules.001.png)
Le schermate complesse sono di varie forme. Ad esempio, la schermata delle impostazioni gestisce molti elementi non correlati tra loro.
La schermata di visualizzazione messaggi contiene un'intestazione, collezioni per contatti e allegati, così come la visualizzazione del messaggio, che richiede trasformazioni speciali.

## Problemi dei Moduli Complessi
![submodules.002](../Resources/submodules/submodules.002.png)
Quali problemi creano i moduli complessi?
- Dati eterogenei in un modulo
- Logica di business complessa
- Testing difficoltoso
- Impossibilità di riutilizzo
- Difficile modificare funzioni e configurazione

Ad esempio il modulo delle impostazioni: dovrebbe memorizzare informazioni su nome con firma, elenco di caselle di posta connesse, stato delle notifiche. Ogni sezione delle impostazioni può influenzare i suoi vicini nella sezione, ma teoricamente non dovrebbe toccare gli altri. Anche se ha questa possibilità. È impossibile riutilizzare un tale modulo, tutto è adattato alle opzioni specifiche dell'applicazione. Aggiungere o modificare impostazioni richiede di studiare il funzionamento dell'intero modulo.

## Suddividere Impostazioni e Visualizzazione Messaggi in Sottomoduli
![submodules.003](../Resources/submodules/submodules.003.png)
La divisione delle impostazioni in sottomoduli per sezioni è piuttosto ovvia. Questo è logico e chiaro. Il primo modulo - dati utente, il secondo modulo - elenco di caselle di posta connesse e così via.

Con il modulo di visualizzazione messaggi, tutto è anche abbastanza semplice - intestazione, contatti, allegati (e possono anche essere compressi) e visualizzazione del corpo del messaggio.

## Sottomoduli: Pro e Contro

### Pro dei sottomoduli:
- Singola responsabilità - ogni modulo può, e idealmente dovrebbe, essere responsabile di una funzione.
- Testabilità - i moduli piccoli sono più facili da testare.
- Riutilizzabilità - il modulo contatti o allegati può essere utilizzato nelle schermate di composizione messaggi e anche in un'altra applicazione, come un messenger.
- Per aggiungere nuove funzionalità, puoi aggiungere un nuovo sottomodulo.
- Possibilità di creare diverse configurazioni, ad esempio aggiungere un elemento di impostazioni aggiuntivo solo per gli sviluppatori.

### Contro:
- Codice aggiuntivo. Fornisce inizializzazione e lavoro coordinato dei sottomoduli. Deve essere ben testato.
- Pericolo di separazione eccessiva. Un sistema suddiviso in moduli troppo piccoli si sfalda. Ad esempio, nelle impostazioni non dovresti creare un modulo per ogni elemento.
- Complicazione dei flussi di dati. Se il sottomodulo di un sottomodulo deve restituire alcuni dati al modulo principale, la catena di chiamate sembrerà molto più complessa del caso di un modulo monolitico.

Pertanto, la suddivisione in sottomoduli richiede una buona analisi.

## Opzioni per Dividere Moduli Complessi in Sottomoduli - Panoramica
![submodules.004](../Resources/submodules/submodules.004.png)
Nel lavorare sui progetti, sono state utilizzate 4 varianti di moduli compositi:
- Modulo contenitore
- Scroll View Container
- Tabella con gruppi di celle
- Tabella con celle-moduli
- Modulo-View

Esaminiamoli in dettaglio

## Modulo Contenitore
![submodules.005](../Resources/submodules/submodules.005.png)
Questo è un analogo di Container e EmbedSegue, ma gestito dal modulo. Il presenter chiede al router di aggiungere un modulo figlio. Il router inizializza il modulo figlio, gli fornisce dati per lavorare, aggiunge il ViewController del sottomodulo come figlio al controller del modulo. E similmente, la View del controller del sottomodulo viene aggiunta al View-container del controller del modulo.

Questo approccio è buono da usare quando è richiesta logica di business separata all'interno del modulo, ad esempio per tabelle che hanno un'intestazione complessa.

## Modulo Contenitore - Esempio
Immagina un elenco di post dell'utente, sopra di essi un'intestazione con avatar e la possibilità di inviare un messaggio.
L'intero modulo dei post si occupa solo dei post - li carica, li visualizza e processa il tocco su una cella di post con transizione alla visualizzazione del post. Se aggiungi qui anche il caricamento del profilo con transizione alla scrittura di messaggi, il modulo diventerà notevolmente più complesso, quindi è conveniente estrarli in un sottomodulo incorporabile separato. Per lavorare, ha bisogno solo dell'identificatore utente.

## Scroll View Controller
![submodules.006](../Resources/submodules/submodules.006.png)
Un caso complesso di modulo contenitore. Così è implementata la schermata di visualizzazione messaggi nel client Rambler/Mail. All'interno dello Scroll View ci sono diversi contenitori, ognuno gestito indipendentemente dal suo sottomodulo. Il modulo contatti lavora solo con il caricamento e la visualizzazione dei contatti, gestisce nascondere/espandere l'elenco. Il modulo allegati divide gli allegati in immagini e documenti, gestisce nascondere ed espandere l'elenco. Il modulo di visualizzazione messaggi processa le email, aggiunge interruzioni di riga alle stringhe lunghe, carica e memorizza nella cache gli allegati inline con immagini.

Inoltre, come appropriato in VIPER, tutto ciò che è correlato al caricamento, elaborazione e logica di business è eseguito dagli interactor dei sottomoduli. Per questo hanno riferimenti ai servizi. Tecnicamente, ognuno di questi sottomoduli può essere espanso a schermo intero.

## Tabella con Gruppi di Celle
![submodules.007](../Resources/submodules/submodules.007.png)
Questo è un modo per costruire una tabella delle impostazioni. All'interactor viene dato un elenco di sottomoduli per la visualizzazione durante l'inizializzazione. Interroga ogni sottomodulo e riceve asincrono un array di view-model per ogni sottomodulo, li incolla in un array comune e li passa alla sua tabella per la visualizzazione. La factory delle celle ottiene tutti i dati necessari dal cell-model per creare e configurare la cella, quindi è universale per tutti i sottomoduli.

I sottomoduli usano factory di cell-model come Views, trasformano i dati dal presenter in una forma adatta per la visualizzazione come celle, traducono eventi dalle celle al presenter, cioè eseguono completamente tutto il lavoro della View.

Questo permette al modulo notifiche di lavorare solo con le notifiche, e al modulo caselle di posta connesse di caricare l'elenco delle caselle dal suo servizio per la visualizzazione.

## Tabella con Celle-Moduli
![submodules.008](../Resources/submodules/submodules.008.png)
Ci sono situazioni quando il contenuto visualizzato in una tabella è molto complesso, la cella processa molte azioni utente, ha bisogno di caricare dati aggiuntivi per lavorare o deve visualizzare una CollectionView al suo interno. In tali casi, puoi rendere ogni cella un modulo separato.

La complessità è che devi rendere riutilizzabili non solo le celle, ma anche i moduli VIPER. La factory delle celle dovrebbe configurare lo stato del modulo quando visualizza la cella. Ad esempio, nel caso di CollectionView all'interno di una cella, devi passargli non solo l'elenco di oggetti da visualizzare, ma anche impostare il ContentOffset appropriato.

Ma questo permette di processare tutti gli eventi di azione all'interno di tale sottomodulo, incluso collegarsi al server.

## Modulo View
![submodules.009](../Resources/submodules/submodules.009.png)
Il modo più ovvio è il modulo View. Questo approccio si integra bene con altre varianti. Ad esempio, possiamo avere un modulo cella, all'interno del quale c'è un sottomodulo galleria o video player. Tali sottomoduli possono essere facilmente riutilizzati su altre schermate.

## Quando i Sottomoduli Aiutano?
![submodules.010](../Resources/submodules/submodules.010.png)
- Riduce la complessità del modulo principale
- Facile riutilizzo dei sottomoduli
- Semplifica l'aggiunta di nuove funzionalità
- Semplifica il testing

## Quando i Sottomoduli Interferiscono?
![submodules.011](../Resources/submodules/submodules.011.png)
- Aumenta significativamente il volume del codice
- Complica la logica