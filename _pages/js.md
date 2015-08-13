---
layout: default
title: Javascript standards and conventions
id: JS
permalink: /js/
---

# JS

## Principals and Morals

We care about all users no matter what device, browser, internet connection and capability. We love having a boring site and appreciate the saracasm in this post:

[Is your website too boring, functional and usable?](http://tommorris.org/posts/2547)

We embrace the constraints of the browser by understanding that [Progressive Enhancement is still important](http://jakearchibald.com/2013/progressive-enhancement-still-important/).

We don't attempt to cram all the pages into one single document because we are aware of [The disdvantages of Single Page Applications](http://adamsilver.github.io/articles/the-disadvantages-of-single-page-applications/).

### 3rd Party Script

3rd party script should be treated with caution. When introducing into the code base we are inheriting several potential problems including but not limited to support and maintenance costs, learning costs, upgrade costs, bug fixing and depending on the impact of the script the pain of removing it if that ever becomes necessary.

Vendor should house *all* third party scripts. Don’t store the minified and non-minified version of the same thing - pick one. Don’t put version number in filename. More work when upgrading.

## Avoid Polyfills

[The disadvantages of Javascript polyfills](http://adamsilver.io/articles/the-disadvantages-of-javascript-polyfills/)

## Inheritance

[Javascript inheritance](http://adamsilver.github.io/articles/javascript-inheritance/)

## Architecture

Architecture should allow for the range of features we have across the site from the more simple (e.g. footer sites overlay) to the more complex (e.g. add to basket). With that in-mind a simple client-side MVC-type approach is defined as follows.

### Components

All components are constructors for testability and consistency. See testing docs for more on that.

#### Controllers

For every feature a controller must be defined and instantiated. Controllers implement views. Controllers have no knowledge of other controllers.

They communicate via custom events (these are NOT event emitters, at least not how we differentiate between them on this project). CustomEvents are defined against project.events (which is a mediator). Controllers can then publish or subscribe to these. Examples follow later.

#### Views

A view encapsulates DOM. Finding elements, interoggating elements, adding event listeners to elements. They don't have any idea of the existence of any other view or controller.

Views can optionally become event emitters (these are NOT to be confused with custom events as per their usage in controllers). They can become event emitters by inheriting from BaseView (which inherits from EventEmitter).

Examples follow later.

#### Services

Services encapsulate XHR calls. This is where the client and server contract is vital for enhanced UI behaviour.

#### Models

Models/collections are client-side objects that are useful for instant UI updates giving the user perception of speed while the XHR sync happens in the background. We don't have these yet, but likely useful for the menu page in the future.

#### Custom Events vs Event Emitters

Both custom events and event emitters are very similar and perhaps the naming for Custom Events needs to change - we will discuss.

Custom events should only be referenced at the controller level, NOT in the views or anywhere else for that matter. They are used to ensure controllers don't know anything about each other and are loosely coupled via the project.events mediator object.

Event Emitters are tied to components. It means any object that implements it can listen for events on it and be more tightly coupled.

### Simple example e.g. Footer sites overlay

Controller:

	project.controllers.FooterSiteMenuController = function() {
		this.view = new project.views.FooterSiteMenuView();
	};

View:

	project.views.FooterSiteMenuView = function() {
		this.container = $("...");
		this.container.hide();
		$("the link").on("click", $.proxy(this, "onLinkClicked"));
	};

	project.views.FooterSiteMenuView.prototype.onLinkClicked = function(e) {
		e.preventDefault();
		// logic to toggle between showing and hiding etc
	};

	project.views.FooterSiteMenuView.prototype.hide = function() {
		// stuff to make it hide
	};

	project.views.FooterSiteMenuView.prototype.show = function() {
		// stuff to make it hide
	};

Initialising:

	new project.controllers.FooterSiteMenuController();

### Complex example e.g. Remove item from basket

Controller:

	project.controllers.BasketController = function() {
		this.basketService = new project.services.BasketService();
		this.basketView = new project.views.BasketView();
		this.basketView.on("itemRemoved", $.proxy(this, "onItemRemoved"), this);
	};

	project.controllers.BasketController.prototype.onItemRemoved = function(id) {
		this.basketService.removeItem(id, $.proxy(this, "onItemRemovedSuccessfully"));
	};

	project.controllers.BasketController.prototype.onItemRemovedSuccessfully = function(response) {
		for(var i = 0; i < response.basket.removedItems.length; i++) {
			this.basketView.removeItem(response.basket.removedItems[i].id);
		}
		this.basketView.updateTotals(response.basket.summary);
	};

Service:

	project.services.BasketService = function() {

	};

	project.services.BasketService.prototype.removeItem = function(id, successFn) {
		$.ajax({ method: "post", url: "", success: successFn });
	};

Sample service response:

	{
		basket: {
			removedItems: [{
				id: 123
			}]
			summary: {
				subTotal: "",
				deliveryFee: "",
				total: "",
			}
		}
	}

View:

	project.views.BasketView = function(container) {
		container.on("click", ".remove", $.proxy(this, "onRemoveClicked"));
	};

	// Base view is an event emitter
	project.utilities.inherit(project.views.BasketView, project.views.BaseView);

	project.views.BasketView.prototype.onRemoveClicked = function(e) {
		e.preventDefault();
		var id = e.target.attr("data-item-id");
		this.fire("itemRemoved", id);
	};

	project.views.BasketView.prototype.removeItem = function(id) {
		$(/* item using id */).remove();
	};

	project.views.BasketView.prototype.updateTotals = function(totals) {
		$(/**/).html(totals.blah);
	};

Initialising:

	new project.controllers.BasketController();

### Utilising custom events

Controllers can raise events which is vital for communication across controllers. An example of this might be the menu page. The MenuController would be aware that a user has added an item to the basket. It might fire an event called "ItemAddedToBasket". The BasketController might listen for this event and do it's thing in order to update the basket appropriately. It might also raise its own notification to say "ItemSuccessfullyAddedToTheBasket" which the MenuController might listen for in order to update its view appropriately.

Only controllers should send custom events.

Events are defined in one "global" location project.events.js acting as a simple mediator.

	project.events = {
		itemAddedToBasket: new project.CustomEvent(),
		someOtherEvent: new project.CustomEvent()
	};

Publishing:

	project.events.itemAddedToBasket.publish();

Subscribing:

	project.events.itemAddedToBasket.subscribe();

## Bootstrapping

Bootstrapping should only take place in a separate bootstrap file.

### Imediately Invoked Function Expressions

Avoid the use of these as they cannot be tested and don't adhere to the first rule. i.e. they instantiate on their own.

### DOMContentLoaded etc

All scripts including the bootstrapping file(s) should be at the bottom of the page just before the end body tag. For this reason there is no reason to instantiate scripts based on this.

### Conditional bootstrapping v.s. bootstrapping per page/section

We have these two choices.

#### Option 1

Currently we do conditional bootstrapping due to having a single bootstrap file.

Postives:

* One HTTP request

Negatives:

* Loading more than necessary which means a big upfront hit
* Conditionality within the single bootstrap which slows down runtime

With this (current) solution, for components that aren't global there will need to be conditions before instantiating.

For example we don't want to create the sticky categories unless they are being shown, which only happens on the menu page. The page will need a small check as follows:

	if(document.getElementById("categories")) {
		var categoriesController = new project.controllers.CategoriesController();
	}

'Some check' might unfortunately have to find an element to check it exists first. Alternatively you could expose a plain object within the view template. Something like:

	project.categoriesPresent = true;

Then the bootstap could be:

	if(project.categoriesPresent) {
		var categoriesController = new project.controllers.CategoriesController();
	}

#### Option 2

The alternative is to be more granular and have fine grained control.

Positives:

* Only load the absolute neccessary script per page
* No conditionality required which optimises performance

Negatives:

* 1 extra HTTP request for some pages

## Namespacing and file organisation

[Javascript namespacing](http://adamsilver.io/articles/javascript-namespacing/)

### What does this look like?

Notice that components should be named and namespaced by what they are and *not* where they live.

	path/to/js/
        vendor/
            3rdPartyShiz.js
            ...
        project/
            project.js // namespace
            utilities/
            	project.utilities.js // namespace
            	project.utilities.FormValidator.js // at least wrap jquery plugin for now to have consistent app code
            	project.utilities.Toggler.js
            	project.utilities.Sticky.js
            	project.utilities.Dialog.js
                project.utilities.Customevent.js
            	...
            events/
                project.events.js // contains all the custom events on the singleton
            views/
                project.views.js // namespace
                project.views.BaseView.js // all views inherit from this to ensure they are evented and loosely coupled
                project.views.DialogView.js
                project.views.MenuView.js
                project.views.SideBasketView.js
                project.views.MiniBasketView.js
                project.views.LoginFormView.js
                ...
            controllers/
                project.controllers.js // namespace
                project.controllers.MenuController.js
                project.controllers.SideBasketController.js
                project.controllers.MiniBasketController.js
                project.controllers.LoginController.js // won't list all these out but basically for each form validator the controller will create it's form validator - these are one liners
                project.controllers.ForgotPasswordController.js // as previous
                ...
            services/
                project.services.js // namespace
                project.services.BasketService.js //ajax wrapper
                project.services.HelpService.js // ajax wrapper
                ...
            initalisers/
                project.initalisers.all.js

## Coding convention and style

### Host Objects

These should not be messed with. To start with the most basic example:

	// Good
	var myVariable = "hello";

	// Bad
	window.myVariable = "hello";

For more on host objects read: [H is for Host](http://www.cinsoft.net/host.html) and [Object creation by Kangax](http://perfectionkills.com/unnecessarily-comprehensive-look-into-a-rather-insignificant-issue-of-global-objects-creation/).

### Don't comma separate variables

This makes it easier to read and edit

	// good
	var a;
	var b;

	// bad
	var a,b;

### Atomic functions

Each function should do one thing only. If a function ends up larger than 5 lines it is very likely to be doing more than one thing.

### No anonymous functions

Anonymous functions are hard to debug and impossible to test. All functions should be named and accessible through the prototype chain.

### Event listeners

Event listeners (or handlers) should be named as if something has happened as follows:

	project.MyComponent = function() {
		project.events.itemAddedToBasket.subscribe(this.onItemAddedToBasket);
		$(".login-button").on("click", this.onLoginButtonClicked);
	};

	project.MyComponent.prototype.onItemAddedToBasket = function() {
	};

	project.MyComponent.prototype.onLoginButtonClicked = function() {
	};

Event listeners should hand off to other methods for the actual behaviour. As en example, let's say that when a button is clicked, an element is toggled. See code below

	project.SomeButtonToggler = function() {
		$(".some-button").on("click", this.onSomeButtonClicked);
	};

	project.SomeButtonToggler.prototype.onSomeButtonClicked = function(e) {
		e.preventDefault();
		this.toggle();
	};

	project.SomeButtonToggler.prototype.toggle = function() {
		/* code here */
	};

Always use $.fn.on instead of $.fn.click for example as it's more performant and provides flexibility.

Always delegate when possible.

### UpperCamelCase constructors

All constructors should be UpperCamelCase as follows:

	// good
	project.MyComponent = function() {};

	// bad
	project.myComponent = function() {};

### lowerCamelCase for everything else

All namespaces and variables should be lowerCamelCase.

	// good
	project.MyComponent = function() {
		this.someProperty;
	};

	// bad
	project.MyComponent = function() {
		this.SOMEPROPERTY;
		this.AnOtherProperty;
	};

### Don't use 'Use strict' on the client-side.

> "A word of caution, all you hard-charging programmers: applying "use strict" to existing code can be hazardous! This thing is not some feel-good, happy-face sticker that you can slap on the code to make it 'better'. With the "use strict" pragma, the browser will suddenly THROW exceptions in random places that it never threw before just because at that spot you are doing something that default/loose JavaScript happily allows but strict JavaScript abhors! You may have strictness violations hiding in seldom used calls in your code that will only throw an exception when they do eventually get run - say, in the production environment that your paying customers use!"

> "If you are going to take the plunge, it is a good idea to apply "use strict" alongside comprehensive unit tests and a strictly configured JSHint build task that will give you some confidence that there is no dark corner of your module that will blow up horribly just because you've turned on Strict Mode. Or, hey, here's another option: just don't add "use strict" to any of your legacy code, it's probably safer that way, honestly. DEFINITELY DO NOT add "use strict" to any modules you do not own or maintain, like third party modules"

Basically, the last thing we want to do is to have the same JS act differently depending on the UA which is exactly what use strict does. Instead lean on linting as per DFD which solves the exact same issues.

## Bundling and Minification and more

All files should be bundled and revved. Bundle file names should be in the following format:

	<name of bundle>.<the version>.js

e.g. paymentModules.123.js

## Styling and Javascript

Cross-browser scripting in terms of positioning, dimensions, viewport is very difficult. For this reason it is significantly more reliable to position, work out dimensions and animate elements if the element does not have padding, margin or borders. Always put those styles on the inner elements.

Dialogs and Accordions are vital they follow this technique.

## Testing

Jasmine is used for unit testing.

### Constructors

Any object that maintains state must use a constructor so that it can be tested reliably without state pollution across tests.

### File naming

Should reflect the source files in naming and directory structure except for two aspects:

The name of the file should include the word "Spec" and all the tests should live in a separate folder called Tests.

### Don't test dependencies

When testing a component (i.e. a constructor/function) you must not test the dependencies. For example if our function looks like this:

	function blah() {
		$(".blah").addClass("yo");
	}

We must utilise spies and mocks in order to test this. We must spy on $ and return a mock jquery object which contains an addClass mocked method on which to spy on.

As you can see our function does two things so our spec is straightforward:

	describe("Blah", function() {
		var mockBlah;
		beforeEach(function() {
			mockBlah = jasmine.createSpyObj("mockBlah", ["addClass"]);
			spyOn(window, $).and.callFake(function(arg) {
				if(arg === ".blah") {
					return mockBlah;
				}
			});
			blah();
		});
		it("Retrieves the blah element", function() {
			expect($).toHaveBeenCalledWith(".blah");
		});
		it("Adds a class of yo to the blah element", function() {
			expect(mockBlah.addClass).toHaveBeenCalledWith("yo");
		});
	});

### Quick guides

* The first describe should name the component e.g.

	describe("Accordion");

* All describe and it blocks should be in english using correct grammar.

* Don't repeat the word "should". Terse is good as follows:

	// good
	it("Returns true");

	// bad
	it("Should return true");

* Create a variable with the name of the component. That will be referenced throughout each test.

* All mocks should be prefixed with "mock" e.g. "mockSomething"

* Each scenario should have it's own describe block. Typical blocks would be "Creating X" for the construction, "Something clicked". Further nesting of describe blocks should be used for conditions based on scenario e.g. Within "Something clicked" might be "User logged in" and "User logged out" scenarios that require different test setup.