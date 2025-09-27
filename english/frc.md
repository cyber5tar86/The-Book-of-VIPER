## NSFetchedResultsController in VIPER

One of the most convenient features that **CoreData** provides for working with object graphs is `NSFetchedResultsController`. Within the MVC architecture, its use is quite obvious - the screen controller implements the `NSFetchedResultsControllerDelegate` protocol:

```objc
- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(id)anObject atIndexPath:(nullable NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(nullable NSIndexPath *)newIndexPath;
- (void)controllerWillChangeContent:(NSFetchedResultsController *)controller;
- (void)controllerDidChangeContent:(NSFetchedResultsController *)controller;
```

These methods are indeed extremely convenient to directly update state and insert/reload/delete some cells in them. Of course, you have to pay for such simplicity:

- Another responsibility for the controller,
- Knowledge about CoreData goes beyond the model/service layer,
- We tightly bind to the chosen mechanism for database state change notifications,
- *+100 lines* in an already large class.

Some of the problems are partially solved by decomposing the controller into composite objects, including VIPER stack elements.

In VIPER, unlike MVC, the role and place for `NSFetchedResultsController` are not so clearly defined. Definitely not *View* - first, it cannot receive business entities from someone from its own layer, second - in most cases you should try not to use `NSManagedObject`s in pure form at the top level of architecture.

*Presenter* doesn't fit either - its responsibility is to connect module elements and hold state. *FRC* is a separate data source, not connected to the interactor.

Placing it on the service layer is also wrong, since services are passive objects that can only react to commands from high-level components, and don't contain any additional state.

The optimal variant for placing *FRC* responsibility - and by that we mean tracking the object graph state, is the interactor:

- It already works with other business entities,
- In most cases it knows that we use CoreData,
- It is the data source of the current module - it doesn't matter what caused their appearance - direct request from presenter or receiving notification from database.

We came to two different variants of working with FRC at the interactor level. The first suits tables with limited content. The second - for infinite scroll.

#### Working with Tables with Limited Content

##### CacheTracker Protocol

`CacheTracker` - protocol of an object whose task is to receive notifications about database state changes and form a set of transactions based on them.

```objc
@protocol CacheTracker <NSObject>

/**
 Method configures the cache tracker
 
 @param cacheRequest Request describing tracker behavior
 */
- (void)setupWithCacheRequest:(CacheRequest *)cacheRequest;

/**
 Method forms transaction batch based on current cache state
 
 @return CacheTransactionBatch
 */
- (CacheTransactionBatch *)obtainTransactionBatchFromCurrentCache;

@end
```

##### Example `CacheTracker` Implementation

The provided implementation variant is built precisely on working with `NSFetchedResultsController`. An alternative protocol implementation could, for example, be built on working with `NSNotification`s received from CoreData.

```objc
#pragma mark - Public methods

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

- (NSFetchRequest *)fetchRequestWithCacheRequest:(CacheRequest *)cacheRequest {
    NSFetchRequest *fetchRequest = [NSFetchRequest fetchRequestWithEntityName:[cacheRequest.objectClass entityName]];
    [fetchRequest setPredicate:cacheRequest.predicate];
    [fetchRequest setSortDescriptors:cacheRequest.sortDescriptors];
    return fetchRequest;
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

#pragma mark - NSFetchedResultsControllerDelegate methods

- (void)controllerWillChangeContent:(NSFetchedResultsController *)controller {
    self.transactionBatch = [CacheTransactionBatch new];
}

- (void)controller:(NSFetchedResultsController *)controller didChangeObject:(NSManagedObject *)anObject atIndexPath:(NSIndexPath *)indexPath forChangeType:(NSFetchedResultsChangeType)type newIndexPath:(NSIndexPath *)newIndexPath {
    id plainObject = [self.objectsFactory plainNSObjectForObject:anObject];
    CacheTransaction *transaction = [CacheTransaction transactionWithObject:plainObject
                                                               oldIndexPath:indexPath
                                                           updatedIndexPath:newIndexPath
                                                                 objectType:NSStringFromClass(self.cacheRequest.objectClass)
                                                                 changeType:changeType];
    [self.transactionBatch addTransaction:transaction];
}

- (void)controllerDidChangeContent:(NSFetchedResultsController *)controller {
    if ([self.transactionBatch isEmpty]) {
        return;
    }
    
    [self.delegate didProcessTransactionBatch:self.transactionBatch];
}
```

##### CacheRequest

`CacheRequest` - object containing complete description of database state tracking parameters necessary for `CacheTracker`. Essentially, this is a request based on which `CacheTracker` forms its behavior.

```objc
@interface CacheRequest : NSObject

@property (strong, nonatomic, readonly) NSPredicate *predicate;
@property (strong, nonatomic, readonly) NSArray *sortDescriptors;
@property (assign, nonatomic, readonly) Class objectClass;
@property (strong, nonatomic, readonly) NSString *filterValue;

+ (instancetype)requestWithPredicate:(NSPredicate *)predicate
                     sortDescriptors:(NSArray *)sortDescriptors
                         objectClass:(Class)objectClass
                         filterValue:(NSString *)filterValue;

@end
```

##### Transaction Classes

`CacheTransaction` - object describing change of one `NSManagedObject`.

```objc
@interface CacheTransaction : NSObject

/**
 Changed object
 */
@property (strong, nonatomic, readonly) id object;

/**
 Object's IndexPath before its change
 */
@property (strong, nonatomic, readonly) NSIndexPath *oldIndexPath;

/**
 Object's IndexPath after its change
 */
@property (strong, nonatomic, readonly) NSIndexPath *updatedIndexPath;

/**
 Type of changed object
 */
@property (strong, nonatomic, readonly) NSString *objectType;

/**
 Change type
 */
@property (assign, nonatomic, readonly) NSFetchedResultsChangeType changeType;

+ (instancetype)transactionWithObject:(id)object
                         oldIndexPath:(NSIndexPath *)oldIndexPath
                     updatedIndexPath:(NSIndexPath *)updatedIndexPath
                           objectType:(NSString *)objectType
                           changeType:(NSUInteger)changeType;

@end

```

`CacheTransactionBatch` combines a set of transactions into one object intended for transfer between layers directly to the table.

```objc
@interface CacheTransactionBatch : NSObject

@property (strong, nonatomic, readonly) NSOrderedSet *insertTransactions;
@property (strong, nonatomic, readonly) NSOrderedSet *updateTransactions;
@property (strong, nonatomic, readonly) NSOrderedSet *deleteTransactions;
@property (strong, nonatomic, readonly) NSOrderedSet *moveTransactions;

/**
 Method adds new transaction to batch
 
 @param transaction Transaction
 */
- (void)addTransaction:(CacheTransaction *)transaction;

/**
 Method tells if batch contains at least one transaction
 
 @return YES/NO
 */
- (BOOL)isEmpty;

@end

```

##### Example Interactor

The interactor does practically no work - it only needs to properly preconfigure `CacheTracker` and become its delegate.

```objc
@implementation PostListInteractor

- (void)setupCacheTrackingWithCacheRequest:(CacheRequest *)cacheRequest {
    [self.cacheTracker setupWithCacheRequest:cacheRequest];
    CacheTransactionBatch *initialBatch = [self.cacheTracker obtainTransactionBatchFromCurrentCache];
    [self.output didProcessCacheTransaction:initialBatch];
}

#pragma mark - CacheTrackerDelegate protocol methods

- (void)didProcessTransactionBatch:(CacheTransactionBatch *)transactionBatch {
    [self.output didProcessCacheTransaction:transactionBatch];
}

@end

```

##### Working with Table

The final part of the scheme - table processing of the received transaction batch. In the current variant we work with `UITableView` not directly, but using the `Nimbus` framework - but this doesn't change the essence of actions.

```objc
- (void)updateDataSourceWithTransactionBatch:(CacheTransactionBatch *)transactionBatch {
    for (CacheTransaction *transaction in transactionBatch.insertTransactions) {
        PostListCellObject *cellObject = [self generateCellObjectForPost:transaction.object];
        
        NSUInteger numberOfObjects = [self.tableViewModel lj_numberOfObjectsInSection:PostListSectionIndex];
        NSUInteger updatedRow = transaction.updatedIndexPath.row;
        [self.tableViewModel insertObject:cellObject
                                    updatedRow
                                inSection:PostListSectionIndex];
    }
    
    for (CacheTransaction *transaction in transactionBatch.updateTransactions) {
        PostListCellObject *cellObject = [self generateCellObjectForPost:transaction.object];
        NSIndexPath *oldIndexPath = [NSIndexPath indexPathForRow:transaction.oldIndexPath.row
                                                       inSection:PostListSectionIndex];
        [self.tableViewModel removeObjectAtIndexPath:oldIndexPath];
        [self.tableViewModel insertObject:cellObject
                                    atRow:transaction.updatedIndexPath.row
                                inSection:PostListSectionIndex];
    }
    
    NSMutableArray *removeIndexPaths = [NSMutableArray array];
    for (CacheTransaction *transaction in transactionBatch.deleteTransactions) {
        NSIndexPath *removeIndexPath = [NSIndexPath indexPathForRow:transaction.oldIndexPath.row
                                                          inSection:PostListSectionIndex];
        [removeIndexPaths addObject:removeIndexPath];
    }
    [self.tableViewModel lj_removeObjectsAtIndexPaths:[removeIndexPaths copy]];
    
    for (CacheTransaction *transaction in transactionBatch.moveTransactions) {
        PostListCellObject *cellObject = [self generateCellObjectForPost:transaction.object];
        NSIndexPath *oldIndexPath = [NSIndexPath indexPathForRow:transaction.oldIndexPath.row
                                                       inSection:PostListSectionIndex];
        [self.tableViewModel removeObjectAtIndexPath:oldIndexPath];
        [self.tableViewModel insertObject:cellObject
                                    atRow:transaction.updatedIndexPath.row
                                inSection:PostListSectionIndex];
    }
}
```

#### Working with Infinite Scroll

When working with infinite feeds, keeping all received objects in memory can be very expensive. In such cases, it's better to use a modified first variant. Changes are as follows:

- `CacheTracker` doesn't add model objects to transactions, but passes only indices.
- Table can request object by index through the chain `Presenter -> Interactor -> CacheTracker`.