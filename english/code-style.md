Using the same code style by all team developers is no less important than a unified point of view on application architecture and responsibilities of different objects. Having agreements for working with VIPER stack is no exception.

### General Module Rules

- Module name should fully reflect its purpose. The *Module* suffix should not be included in the name.
  
  **Example:** `MessageFolder`, `PostList`, `CacheSettings`.
  
- All module elements are divided into subfolders within one module folder.

  **Example:**
  
  ```
  /NewPostUserStory
      /NewPostModule
          /Assembly
          /Interactor
          /Presenter
          /Router
          /View
      /ChooseAvatarModule
          /Assembly
          /Interactor
          /Presenter
          /Router
          /View
  ```
  
- If after writing the module some of its elements remain unused, be it classes, protocols or methods, they are removed.
- All helpers are located in the subfolder of their layer.

  **Example:**

  ```
/Interactor
      /UserInputValidator
        UserInputValidator.h
        UserInputValidator.m
      /PlainObjectMapper
        PlainObjectMapper.h
        PlainObjectMapperImplementation.h
        PlainObjectMapperImplementation.m
  ```
- All methods through which layers communicate with each other should be synchronous.

  **Example:**
  
  ```objc
  @interface InteractorInput
  - (void)obtainDataFromNetwork;
  ...
  
  @interface InteractorOutput
  - (void)didObtainDataFromNetwork:(NSArray *)data;
  ...
  ```
  
- All protocol methods that cover module elements should start with verbs - this helps explicitly indicate that each component has behavior, not state.
- Methods denoting the start of action should start with verbs expressing command or request (imperative mood verbs).
- Methods denoting completion of action or process, stating fact should start with past tense verb.

  **Example:**
  
  ```objc
  - (void)obtainImageForPostId:(NSString *)postId;
  - (void)processUserInput:(NSString *)userInput;
  - (void)invalidateCurrentCache;
  ```
  
- In public interfaces of all classes and protocols we try to use forward-declaration, using `#import` only when necessary.

  **Example:**
  
  ```objc
  #import <Foundation/Foundation.h>

  #import "PostListViewOutput.h"
  #import "PostListModuleInput.h"
  #import "PostListInteractorOutput.h"

  @protocol PostListViewInput;
  @protocol PostListRouterInput;
  @protocol PostListInteractorInput;
  @class PostListViewModelMapper;

  @interface PostListPresenter : NSObject <PostListModuleInput, PostListViewOutput, PostListInteractorOutput>

  @property (nonatomic, weak) id<PostListViewInput> view;
  @property (nonatomic, strong) id<PostListRouterInput> router;
  @property (nonatomic, strong) id<PostListInteractorInput> interactor;
  @property (nonatomic, strong) PostListViewModelMapper *postListViewModelMapper;

  @end
  ```

-----

### Interactor Layer
#### `Interactor` Class

##### Naming
`<ModuleName>Interactor.h / <ModuleName>Interactor.m`

##### Additional Rules

- Interactor doesn't hold state, only dependencies located in its public interface.

  **Example:**
  
  ```objc
  @interface PostListInteractor : NSObject <PostListInteractorInput>

  @property (nonatomic, weak) id<PostListInteractorOutput> output;
  @property (nonatomic, strong) id<AccountService> accountService;

  @end
  ```
  
- Interactor holds weak reference to presenter. Variable is called `output`.

  **Example:** 
  
  `@property (nonatomic, weak) id<PostListInteractorOutput> output;`

#### Service Facades
##### Naming
`<Feature>Facade.h / <Feature>Facade.m` 

##### Description
If interactors of several modules have repeating logic for using services in a certain way, it's encapsulated in a separate facade over services. As an example, an object implementing pagination logic for different post feeds - it can request a list of new elements, compare them with previously cached ones, calculate offsets and gaps - and much more. Thanks to extracting these connections into a separate entity, pagination can be quite easily connected for any element list module.

**Example:** `PagingFacade`.

#### `<InteractorInput>` Protocol
##### Naming
`<ModuleName>InteractorInput.h`

##### Description
Contains methods for communicating with interactor. Interactor is covered by this protocol from presenter's point of view.

##### Method Examples

```objc
- (NSArray *)obtainNewsFromCache;
- (void)obtainMessageWithId:(NSString *)messageId;
- (void)performLoginWithUsername:(NSString *)username password:(NSString *)password;
```

##### Common Method Patterns

- If in this module we need to independently decide when to request data from cache and when from network, it's acceptable to explicitly indicate this to interactor (`obtainFromNetwork/-fromCache`).

#### `<InteractorOutput>` Protocol
##### Naming
`<ModuleName>InteractorOutput.h`

##### Description
Contains methods through which interactor communicates with the upper layer of the module. Presenter is usually covered by this protocol.

##### Method Examples

```objc
- (void)didObtainMessage:(Message *)message;
- (void)didPerformLoginWithSuccess;
```

##### Common Method Patterns:

- In most cases we use `did` as prefix for each method - this indicates the passive role of interactor, which can perform a range of actions on request and notify about their completion.
- In case of absence of unified error handling system, method pairs are created (`didObtainWithSuccess/-withFailure`).

### Presenter Layer
#### `Presenter` Class
##### Naming
`<ModuleName>Presenter.h / <ModuleName>Presenter.m`

##### Additional Rules

- Unlike all other elements, presenter has state. It's located in private extension.
- Presenter holds weak reference to view. Variable is called `view`.

  **Example:**
  
  `@property (nonatomic, weak) id<PostListViewInput> view;`
  
- Presenter holds strong reference to router. Variable is called `router`.

  **Example:**
  
  `@property (nonatomic, strong) id<PostListRouterInput> router;`
  
- Presenter holds strong reference to interactor. Variable is called `interactor`.

  **Example:**
  
  `@property (nonatomic, strong) id<PostListInteractorInput> interactor;`
  
- If presenter needs to hold an object implementing `ModuleInput` protocol of child module, variable is called `<OtherModuleName>ModuleInput`.

#### `State` Class
##### Naming
`<ModuleName>State.h / <ModuleName>State.m`

##### Description
In case the current module has some state, it can be extracted into a separate object that has no behavior and acts as simple data storage.

**Example:**

```objc
@interface PostListState

@property (nonatomic, assign) FeedType feedType;
@property (nonatomic, assign) BOOL hasHeader;
@property (nonatomic, strong) NSString *feedId;

@end
```

#### `<ModuleInput>` Protocol
##### Naming
`<ModuleName>ModuleInput.h`

##### Description
Contains methods through which other modules or its container can communicate with the module.

##### Additional Rules
- When using ViperMcFlurry library, inherits from `<RamblerViperModuleInput>` protocol.

##### Method Examples

```objc
- (void)configureWithPostId:(NSString *)postId;
- (void)updateContentInset:(CGFloat)contentInset;
```

#### `<ModuleOutput>` Protocol
##### Naming
`<ModuleName>ModuleOutput.h`

##### Description
Contains methods through which module communicates with its container or other modules.

##### Additional Rules
- When using ViperMcFlurry library, inherits from `<RamblerViperModuleOutput>` protocol.

##### Method Examples

```objc
- (void)didSelectMenuItem:(NSString *)menuItem;
- (void)didPerformLoginWithSuccess;
```

### Router Layer
#### `Router` Class
##### Naming
`<ModuleName>Router.h / <ModuleName>Router.m`

##### Additional Rules
- When using ViperMcFlurry library, holds weak reference to `ViewController` responsible for this module's transitions. Reference is a property covered by `<RamblerViperModuleTransitionHandlerProtocol>` protocol. Usually this variable is called `transitionHandler`

#### `Route` Class
##### Naming
`<ModuleName>Route.h / <ModuleName>Route.m`

##### Description
If several routers within one application implement repeating logic for transitions to one screen - it can be encapsulated in a separate route object that will be connected to needed modules. For example, this might be useful in case of authorization screen, which can be accessed from different modules - settings, profile, side menu. Thanks to encapsulating this logic in a separate object, we get rid of the need to write the same code in each of these modules' routers.

##### Method Examples

```objc
- (void)openAuthorizationModuleWithTransitionHandler:(id<RamblerViperModuleTransitionHandlerProtocol>)transitionHandler;
```

#### `<RouterInput>` Protocol
##### Naming
`<ModuleName>RouterInput.h`

##### Description
Contains transition methods to other modules that can be called by presenter.

##### Method Examples

```objc
- (void)openDetailNewsModuleWithNewsId:(NSString *)newsId
- (void)closeCurrentModule;
```

##### Common Method Patterns

- For consistency, all methods of this protocol start with either `open-` (opening some module), `close-` (closing module) or `embed-` (embedding child module in container).

### View Layer
#### Display Classes (ViewController, View, Cell)
##### Naming
`<ModuleName>View.h / <ModuleName>View.m`, `<ModuleName>ViewController.h / <ModuleName>ViewController.m`, `<ModuleName>Cell.h / <ModuleName>Cell.m`.

##### Additional Rules

- All `IBOutlet`s and `IBAction`s (i.e., all dependencies and interface) of View are moved to its public interface.
- View holds strong reference to presenter. Variable is called `output`.

  **Example:**
  
  `@property (nonatomic, strong) id<PostListViewOutput> output;`

#### DataDisplayManager Class
##### Naming
`<ModuleName>DataDisplayManager.h / <ModuleName>DataDisplayManager.m`

##### Description
Object covering `UITableViewDataSource` and `UITableViewDelegate` implementation logic. Works only with data, knows nothing about specific `UIView`s of the screen. Usually not covered by protocol, because a specific screen most often corresponds to one specific DataDisplayManager implementation.

##### Method Examples

```objc
- (id<UITableViewDataSource>)dataSourceForTableView:(UITableView *)tableView;
- (id<UITableViewDelegate>)delegateForTableView:(UITableView *)tableView
                               withBaseDelegate:(id <UITableViewDelegate>)baseTableViewDelegate;
```

#### CellObjectFactory Class
##### Naming
`<ModuleName>CellObjectFactory.h / <ModuleName>CellObjectFactory.m`

##### Description
Often it's convenient to extract cell model creation logic from DataDisplayManager to a separate object that essentially transforms regular models into CellObjects.

#### <ViewInput> Protocol
##### Naming
`<ModuleName>ViewInput.h`

##### Description
Contains methods through which presenter can control display or get user-entered data.

##### Method Examples

```objc
- (void)updateWithTitle:(NSString *)title;
- (NSString *)obtainCurrentUserInput;
```

#### <ViewOutput> Protocol
##### Naming
`<ModuleName>ViewOutput.h`

##### Description
Contains methods through which View notifies presenter about changes in its state.

##### Additional Rules
- This protocol also contains methods through which View notifies presenter about its lifecycle events.

##### Method Examples

```objc
- (void)didTapLoginButton;
- (void)didModifyCurrentInput;
```

##### Common Method Patterns
- In most cases we use `did` as prefix for each method - this indicates the passive role of View, which can perform a range of actions on request and notify about their completion.

**Examples:** 

```objc
- (void)didTriggerViewWillAppearEvent;
- (void)didTriggerMemoryWarningEvent;
```

### Assembly Layer
#### Assembly Class
##### Naming
`<ModuleName>Assembly.h / <ModuleName>Assembly.m`

##### Additional Rules

- Only the method configuring View is moved to Assembly interface. Setting up the entire VIPER stack are implementation details of assembly that shouldn't be known to the outside world.

##### Method Examples

```objc

- (PostListViewController *)viewPostListModule;
- (PostListPresenter *)presenterPostListModule;

```

##### Common Method Patterns

- For more convenient autocompletion, all methods creating standard components have `<ComponentName><ModuleName>Module` prefix.

-----

### Comments

- All protocol methods and concrete class methods are necessarily covered by detailed javadoc comments.

  
  **Example:**
  
  ```objc
  @protocol PostListInteractorInput <NSObject>

  /**
   Method returns post model by specific index
 
   @param index Post index within viewed category
 
   @return Post
   */
  - (PostModelObject *)obtainPostAtIndex:(NSUInteger)index;

  /**
   Method returns total number of posts for current feed
 
   @return Number of posts
   */
  - (NSUInteger)obtainOverallPostCount;

  @end
  ```
  
- Comments are written for interfaces of all unique module components (various helpers, cells).

  **Example:**
  
  ```objc
  /**
   Cell used for brief post display in list
   */
  @interface PostListCell : UITableViewCell

  ...

  @end
  ```

- For interfaces of all non-unique components (interactor, presenter, view), as well as their protocols, the same comment is written describing the purpose of the entire module.

  **Example:**
  
  ```objc
  /**
   Module is responsible for displaying post list in any of the application feeds.
   Used as embedded module.
   */
  @interface PostListAssembly : TyphoonAssembly
  ...
  @end
  ```

- In class implementations, method groups of different protocols are separated using `#pragma mark -`.

  **Example:**
  
  ```objc
  @implementation PostListPresenter

  #pragma mark - PostListModuleInput

  - (void)configureModuleWithPostListConfig:(PostListConfig *)config  {
       ...
  }

  #pragma mark - PostListViewOutput

  - (void)didTriggerPullToRefreshEvent {
       ...
  }

  #pragma mark - PostListInteractorOutput

  - (void)didProcessCacheTransaction:(CacheTransactionBatch *)transaction {
       ...
  }

  ```

-----

### Tests

- For each module component, a separate test case is created with name like `<ModuleName>ViewControllerTests.m`.
- We try to follow the rule one test - one check.
- To separate test implementation into logical blocks, we use *given/when/then* notation.
- Tests of methods from different protocols are separated using `#pragma mark -`.
- Since not all methods are declared in Assembly interface, a separate extension `_Testable.h` is created for their verification. In this case we test private methods, but this approach is due to *Typhoon* library implementation details. In case of writing factory manually, it's possible to check component work results through its public interface.
- Module file structure in tests maximally repeats project file structure.