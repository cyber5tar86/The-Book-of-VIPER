## NSFetchedResultsController in VIPER

Una delle caratteristiche più convenienti che **CoreData** fornisce per lavorare con i grafi di oggetti è `NSFetchedResultsController`. All'interno dell'architettura MVC, il suo uso è abbastanza ovvio - il controller della schermata implementa il protocollo `NSFetchedResultsControllerDelegate`:

```objc
- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(nullable NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(nullable NSIndexPath *)newIndexPath;
- (void)controllerWillChangeContent:(NSFetchedResultsController *)controller;
- (void)controllerDidChangeContent:(NSFetchedResultsController *)controller;
```

Questi metodi sono infatti estremamente convenienti per aggiornare direttamente lo stato e inserire/ricaricare/eliminare alcune celle in essi. Ovviamente, devi pagare per tale semplicità:

- Un'altra responsabilità per il controller,
- La conoscenza su CoreData va oltre il livello model/service,
- Ci leghiamo strettamente al meccanismo scelto per le notifiche di cambiamento di stato del database,
- *+100 righe* in una classe già grande.

Alcuni dei problemi sono parzialmente risolti decomponendo il controller in oggetti compositi, inclusi gli elementi dello stack VIPER.

In VIPER, a differenza di MVC, il ruolo e il posto per `NSFetchedResultsController` non sono così chiaramente definiti. Definitivamente non *View* - primo, non può ricevere entità di business da qualcuno del suo stesso livello, secondo - nella maggior parte dei casi dovresti cercare di non usare `NSManagedObject` in forma pura al livello superiore dell'architettura.

Neanche *Presenter* si adatta - la sua responsabilità è collegare gli elementi del modulo e mantenere lo stato. *FRC* è una fonte di dati separata, non connessa all'interactor.

Posizionarlo sul livello di servizio è anche sbagliato, poiché i servizi sono oggetti passivi che possono solo reagire ai comandi dai componenti di alto livello, e non contengono alcuno stato aggiuntivo.

La variante ottimale per posizionare la responsabilità *FRC* - e per questo intendiamo il monitoraggio dello stato del grafo degli oggetti, è l'interactor:

- Lavora già con altre entità di business,
- Nella maggior parte dei casi sa che utilizziamo CoreData,
- È la fonte di dati del modulo corrente - non importa cosa ha causato la loro apparizione - richiesta diretta dal presenter o ricezione di notifica dal database.

Siamo arrivati a due diverse varianti di lavoro con FRC al livello dell'interactor. La prima è adatta per tabelle con contenuto limitato. La seconda - per scroll infinito.

#### Lavorare con Tabelle con Contenuto Limitato

##### Protocollo CacheTracker

`CacheTracker` - protocollo di un oggetto il cui compito è ricevere notifiche sui cambiamenti di stato del database e formare un insieme di transazioni basate su di essi.

```objc
@protocol CacheTracker <NSObject>

/**
 Il metodo configura il tracker della cache
 
 @param cacheRequest Richiesta che descrive il comportamento del tracker
 */
- (void)setupWithCacheRequest:(CacheRequest *)cacheRequest;

/**
 Il metodo forma un batch di transazioni basato sullo stato corrente della cache
 
 @return CacheTransactionBatch
 */
- (CacheTransactionBatch *)obtainTransactionBatchFromCurrentCache;

@end
```

##### Esempio di Implementazione `CacheTracker`

La variante di implementazione fornita è costruita precisamente sul lavoro con `NSFetchedResultsController`. Un'implementazione alternativa del protocollo potrebbe, ad esempio, essere costruita sul lavoro con `NSNotification` ricevute da CoreData.

```objc
#pragma mark - Metodi pubblici

- (void)setupWithCacheRequest:(CacheRequest *)cacheRequest {
    NSManagedObjectContext *defaultContext = [NSManagedObjectContext MR_defaultContext];
    NSFetchRequest *fetchRequest = [self fetchRequestWithCacheRequest:cacheRequest];
    self.controller = [[NSFetchedResultsController alloc] initWithFetchRequest:fetchRequest
                                                          managedObjectContext:defaultContext
                                                            sectionNameKeyPath:nil
                                                                     cacheName:nil];
    self.controller.delegate = self;
    [self.controller performFetch:nil];
}

- (CacheTransactionBatch *)obtainTransactionBatchFromCurrentCache {
    CacheTransactionBatch *batch = [CacheTransactionBatch new];
    for (NSUInteger i = 0; i < self.controller.fetchedObjects.count; i++) {
        id object = self.controller.fetchedObjects[i];
        NSIndexPath *indexPath = [self.controller indexPathForObject:object];
        id plainObject = [self.objectsFactory plainNSObjectForObject:object];
        CacheTransaction *transaction = [CacheTransaction transactionWithObject:plainObject
                                                                   oldIndexPath:nil
                                                               updatedIndexPath:indexPath
                                                                     objectType:NSStringFromClass(self.cacheRequest.objectClass)
                                                                     changeType:NSFetchedResultsChangeInsert];
        [batch addTransaction:transaction];
    }
    
    return batch;
}
```

##### Classi delle Transazioni

`CacheTransaction` - oggetto che descrive il cambiamento di un `NSManagedObject`.

`CacheTransactionBatch` combina un insieme di transazioni in un oggetto destinato al trasferimento tra livelli direttamente alla tabella.

##### Esempio di Interactor

L'interactor fa praticamente nessun lavoro - ha solo bisogno di preconfigurare correttamente `CacheTracker` e diventare il suo delegato.

```objc
@implementation PostListInteractor

- (void)setupCacheTrackingWithCacheRequest:(CacheRequest *)cacheRequest {
    [self.cacheTracker setupWithCacheRequest:cacheRequest];
    CacheTransactionBatch *initialBatch = [self.cacheTracker obtainTransactionBatchFromCurrentCache];
    [self.output didProcessCacheTransaction:initialBatch];
}

#pragma mark - Metodi del protocollo CacheTrackerDelegate

- (void)didProcessTransactionBatch:(CacheTransactionBatch *)transactionBatch {
    [self.output didProcessCacheTransaction:transactionBatch];
}

@end
```

#### Lavorare con Scroll Infinito

Quando si lavora con feed infiniti, tenere tutti gli oggetti ricevuti in memoria può essere molto costoso. In tali casi, è meglio usare una variante modificata della prima. Le modifiche sono le seguenti:

- `CacheTracker` non aggiunge oggetti model alle transazioni, ma passa solo gli indici.
- La tabella può richiedere un oggetto per indice attraverso la catena `Presenter -> Interactor -> CacheTracker`.