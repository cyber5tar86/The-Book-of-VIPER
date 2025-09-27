## TDD and VIPER

### Brief Overview of TDD

TDD - Test Driven Development. It assumes that tests are written first, then implementation. If we cover real classes with protocols, they come first. After all components are written - they can be integrated.

Thanks to this approach, complete description of expected class behavior is achieved in tests. Tests can serve as examples of class usage. Also TDD allows us to look at the class impartially from the user's point of view before writing it.

The topic of TDD in iOS was covered in more detail in the [article](http://habrahabr.ru/company/rambler-co/blog/263087/) by Andrey Rezanov.

## VIPER
Usually separation into main application layers is enough to test the service and Core layer quite well, but with a "fat" ViewController problems arise in testing the Presentation layer. Therefore, many leave it without sufficient test coverage. Let's understand how VIPER modules can help us in covering View with tests.

The general approach for testing is as follows: surround the test class object with protocol mocks of dependencies. Call interface methods/manipulate properties, check mock method calls.

### View Testing
There are some libraries that help in testing the UI layer, and recently we also got UI tests. Is this enough for complete View coverage? In our opinion - no.

UI tests can serve as a good way to write acceptance tests. The entire production application acts as a black box, and we try to get one or another result through the application interface.

The problem with UI tests is that in case of an overloaded VC, the entire View is a black box with many states for us, and testing requires too many tests.

How can VIPER help us here? Logic for data preparation and presentation is moved from ViewController to Presenter. Accordingly, View is only responsible for processing and forwarding events to Presenter, as well as displaying UI. Special attention should be paid to the fact that View is not necessarily UI, but a class through which the external world interacts with the module. What does this mean for us? IBOutlet and IBAction **must** be made public. As a result, we get the ability to test all kinds of clicks, field filling and more without simulating clicks, searching for the right buttons by text on them and other unreliable things.

Let's summarize by highlighting 2 main types of tests:

- Interact with IBOutlet/call IBAction -> check that corresponding Presenter mock methods were called
- Call protocol methods through which Presenter communicates -> check that IBOutlet/View changes

Testing ViewController/View lifecycle methods can be highlighted separately, which we need to orient ourselves to anyway, since often Presenter cannot start View setup before `-viewDidLoad` or `-viewWillAppear`.

### Router Testing
Router methods are called when it's necessary to make a transition from the current ViewController. Accordingly, tests cover transition/closing methods of the current controller. We test calls of various transition animators.

### Interactor Testing
Interactor is a connecting link in working with various services. Work with Storage goes through it, PONSO objects are created in it.

Most tests concern checking calls of some services depending on responses from others.

### Presenter Testing
Presenter can be called the connecting link of the module, since it proxies requests from one part of the module to another. Tests for such forwarding make up the lion's share of Presenter tests.

It should be noted separately that Presenter is the entry point for the module, data from the previous controller is passed to it. Tests need to be written for this too.

Presenter is the place where the maximum number of logical branches is most often located. This also needs to be taken into account, and all possible fundamentally different combinations of values should be fed to the input.

### Assembly Testing
Assembly configures module component dependencies. Its tests are responsible for checking that the module consists of the right parts and all dependencies are filled.

Fortunately, with strict module structure, these tests can be created automatically.