## Module Transitions - Introduction

The approach to creating modules through factory, described in articles about canonical VIPER, is quite inconvenient. We decided to try native `UIStoryboardSegue` for transitions between modules. This approach opened tempting prospects - after all, to transition to another module it would only be necessary to specify SegueID and pass data for work to the module. Moreover, for module configuration we use Typhoon, so all module ViewControllers after initialization through Segue already have connections with other module components.

## Module Transitions - through ViewController

This is the simplest variant. The calling module's router has a reference to its ViewController, when transitioning to another module the ViewController's `-prepareForSegue:` method is called, where data for the next module is passed in `sender`. Inside the calling ViewController's `-prepareForSegue:` this data is passed to the next module.

This approach works, but there are some drawbacks:
- Logic for setting up the next module is placed inside View, not Router,
- No universality and reuse, this method needs to be implemented in each module,
- Data for the next module's work gets into View, not Presenter,
- Each module knows about another module's structure,
- Each router knows it works with `UIViewController` class, and the scheme only works for this variant.

## Module Transitions - through ViewController with Configuration Block

To solve the first two problems, method-swizzling and blocks were used. In `-prepareForSegue:` a block is sent in `sender`, in which module setup is performed through `destinationViewController`. In the alternative `-prepareForSegue:` method the block is called with destinationViewController from segue as parameter.

This works, the logic for setting up the next module is entirely inside Router, each module no longer requires adding `-prepareForSegue:` method to ViewController, but three problems remain:

- Data for the next module's work gets into View, not Presenter,
- Each module knows about another module's structure,
- Each router knows it works with ViewController and the scheme only works for this.

## Module Transitions - Many Protocols

To solve the remaining problems, protocols were used. Many protocols. As well as swizzling and custom promise implementation. As a result, we got a data transfer system between modules without the listed drawbacks, data from presenter is given to router and it configures the next module's presenter with them. But two new problems appeared:
- It took about 2 days for a new developer to master,
- Data was transmitted only in one direction.

## Module Transitions - ModuleInput Variant

The current variant, available on our Github under the name [ViperMcFlurry](https://github.com/rambler-digital-solutions/ViperMcFlurry) became much easier to master. Each module now has an entry point - ModuleInput, which allows configuring the module or calling methods. This moduleInput can be used inside the router for module setup, can be returned to presenter for permanent connection with submodule.
Each module can have ModuleOutput set to return data from the module. ModuleInput/Output are protocols that are set inside the module, that is, they store the contract for communication with it. In most modules, the presenter of this module acts as ModuleInput, and the calling module's presenter acts as ModuleOutput.

## Embed Segue

Since the `-performSegue` method only requires the transition name, `-prepareForSegue:` was subjected to swizzling, and Typhoon configures the module by ViewController, we can use any Segue classes and the module transition mechanism will work.

Therefore, for embedding modules, a special `UIStoryboardSegue` type EmbedSegue was created. Inside `-performSegue` the SourceViewController calls a method that returns View for the Segue identifier. The module is embedded into this View.