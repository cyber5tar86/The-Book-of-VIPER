## Simple vs Complex Modules

When we started using VIPER in our work, we used the concept "One screen - one module". This worked perfectly because screens were mainly simple tables. But with the appearance of the first "complex" screen, problems began.

## Examples of complex modules from Mail - Settings screen and Message viewing
![submodules.001](../Resources/submodules/submodules.001.png)
Complex screens come in different forms. For example, the settings screen manages many unrelated elements.
The message viewing screen contains a header, collections for contacts and attachments, as well as message viewing, which requires special transformations.

## Problems with Complex Modules
![submodules.002](../Resources/submodules/submodules.002.png)
What problems do complex modules create?
- Heterogeneous data in one module
- Complex business logic
- Difficult testing
- Inability to reuse
- Difficult to change functions and configuration

For example, the settings module: it should store information about name with signature, list of connected mailboxes, notification status. Each settings section can affect its neighbors in the section, but theoretically shouldn't touch the others. Although it has this possibility. It's impossible to reuse such a module, everything is tailored to specific application options. Adding or changing settings requires studying the work of the entire module.

## Breaking Settings and Message Viewing into Submodules
![submodules.003](../Resources/submodules/submodules.003.png)
The division of settings into submodules by sections is quite obvious. This is logical and clear. The first module - user data, the second module - list of connected mailboxes and so on.

With the message viewing module, everything is also quite simple - header, contacts, attachments (and they can also be collapsed) and message body viewing.

## Submodules: Pros and Cons

### Pros of submodules:
- Single responsibility - each module can, and ideally should, be responsible for one function.
- Testability - small modules are easier to test.
- Reusability - contacts or attachments module can be used on message composition screens and even in another application, like a messenger.
- To add new functionality, you can add a new submodule.
- Ability to create different configurations, for example adding an additional settings item only for developers.

### Cons:
- Additional code. It provides initialization and coordinated work of submodules. It needs to be well tested.
- Danger of excessive separation. A system broken into too small modules falls apart. For example, in settings you shouldn't make a module for each item.
- Complication of data flows. If a submodule's submodule should return some data to the main module, the call chain will look much more complex than in the case of a monolithic module.

Therefore, breaking into submodules requires good analysis.

## Options for Dividing Complex Modules into Submodules - Overview
![submodules.004](../Resources/submodules/submodules.004.png)
In working on projects, 4 variants of composite modules were used:
- Container module
- Scroll View Container
- Table with cell groups
- Table with cell-modules
- Module-View

Let's examine them in detail

## Container Module
![submodules.005](../Resources/submodules/submodules.005.png)
This is an analog of Container and EmbedSegue, but managed from the module. The presenter asks the router to add a child module. The router initializes the child module, gives it data to work with, adds the submodule's ViewController as a child to the module's controller. And similarly, the submodule controller's View is added to the module controller's View-container.

This approach is good to use when separate business logic is required inside the module, for example for tables that have a complex header.

## Container Module - Example
Imagine a list of user posts, above them a header with avatar and the ability to send a message.
The entire posts module only deals with posts - it loads them, displays them and processes tapping on a post cell with transition to post viewing. If you add profile loading with transition to message writing here, the module will noticeably become more complex, so it's convenient to extract them into a separate embeddable submodule. To work, it only needs the user identifier.

## Scroll View Controller
![submodules.006](../Resources/submodules/submodules.006.png)
A complex case of container module. This is how the message viewing screen is implemented in the Rambler/Mail client. Inside the Scroll View there are several containers, each independently managed by its submodule. The contacts module only works with loading and displaying contacts, handles hiding/expanding the list. The attachments module divides attachments into images and documents, handles hiding and expanding the list. The message display module processes emails, adds line breaks to long strings, loads and caches inline attachments with images.

Moreover, as proper in VIPER, everything related to loading, processing and business logic is performed by submodule interactors. For this they have references to services. Technically, each such submodule can be expanded to the full screen.

## Table with Cell Groups
![submodules.007](../Resources/submodules/submodules.007.png)
This is a way to build a settings table. The interactor is given a list of submodules for display during initialization. It queries each submodule and asynchronously receives an array of view-models for each submodule, glues them into a common array and passes them to its table for display. The cell factory gets all necessary data from cell-model for creating and configuring the cell, so it's universal for all submodules.

Submodules use cell-model factories as Views, they transform data from presenter into a form suitable for display as cells, translate events from cells to presenter, that is, they fully perform all View work.

This allows the notifications module to work only with notifications, and the connected mailboxes module to load the list of mailboxes from its service for display.

## Table with Cell-Modules
![submodules.008](../Resources/submodules/submodules.008.png)
There are situations when content displayed in a table is very complex, the cell processes many user actions, it needs to load additional data to work or needs to display a CollectionView inside itself. In such cases, you can make each cell a separate module.

The complexity is that you need to make not only cells reusable, but also VIPER modules. The cell factory should configure the module state when displaying the cell. For example, in the case of CollectionView inside a cell, you need to pass it not only the list of objects to display, but also set the appropriate ContentOffset.

But this allows processing all action events inside such a submodule, including connecting to the server.

## Module View
![submodules.009](../Resources/submodules/submodules.009.png)
The most obvious way is the module View. This approach integrates well with other variants. For example, we can have a cell module, inside which there is a gallery submodule or video player. Such submodules can be easily reused on other screens.

## When do Submodules Help?
![submodules.010](../Resources/submodules/submodules.010.png)
- Reduces complexity of the main module
- Easy reuse of submodules
- Simplifies adding new functionality
- Simplifies testing

## When do Submodules Interfere?
![submodules.011](../Resources/submodules/submodules.011.png)
- Significantly increases code volume
- Complicates logic