# Basics

## EsLint / CoffeeLint

### Install EsLint / CoffeeLint plugin in your Code Editor

Use EsLint/CoffeeLint for linting your JavaScript/CoffeeScript.

- Provides a first alert prior to committing any code to source control.

- Provides consistency across your team.

## Single Responsibility

### Rule of 1

Define 1 component per file, recommended to be less than 400 lines of code

- One component per file promotes easier unit testing and mocking.

- One component per file makes it far easier to read, maintain, and avoid collisions with teams in source control.

- One component per file avoids hidden bugs that often arise when combining components in a file where they may share variables, create unwanted closures, or unwanted coupling with dependencies.

The following example defines the `app` module and its dependencies, defines a controller, and defines a service all in the same file.

```coffee
# avoid
angular
  .module('app', ['ngRoute'])
  .controller('SomeController' , SomeController)
  .service('someService' , someService)

SomeController = () ->

someService = () ->
```

  The same components are now separated into their own files.

```coffee
# recommended

# app.module.coffee
angular
  .module('app', ['ngRoute'])
```

```coffee
# recommended

# someController.coffee
(->
  SomeController = () ->

  angular
    .module('app')
    .controller('SomeController' , SomeController)
)()
```

```coffee
# recommended

# someService.coffee
(->
  someService = () ->

  angular
    .module('app')
    .service('someService' , someService)
)()
```

### Small Functions

Define small functions, no more than 75 LOC (less is better).

- Small functions are easier to test, especially when they do one thing and serve one purpose.

- Small functions promote reuse.

- Small functions are easier to read.

- Small functions are easier to maintain.

- Small functions help avoid hidden bugs that come with large functions that share variables with external scope, create unwanted closures, or unwanted coupling with dependencies.

## IIFE

### JavaScript Scopes

Wrap Angular components in an Immediately Invoked Function Expression (IIFE).

- An IIFE removes variables from the global scope. This helps prevent variables and function declarations from living longer than expected in the global scope, which also helps avoid variable collisions.

- When your code is minified and bundled into a single file for deployment to a production server, you could have collisions of variables and many global variables. An IIFE protects you against both of these by providing variable scope for each file.

```coffee
# avoid
# loggerService.coffee
angular
  .module('app')
  .service('loggerService', loggerService)

# logger function is added as a global variable
loggerService = () ->

# storageService.coffee
angular
  .module('app')
  .service('storageService', storageService)

# storage function is added as a global variable
storageService = () ->
```

```coffee
# recommended

# logger.service.coffee
(->
  loggerService = () ->

    return

  angular
  .module('app')
  .service('loggerService', loggerService)
)()

# storage.service.coffee
(->
  storageService = () ->

    return

  angular
  .module('app')
  .service('storageService', storageService)
)()
```

- Note: For brevity only, the rest of the examples in this guide may omit the IIFE syntax.

- Note: IIFE's prevent test code from reaching private members like regular expressions or helper functions which are often good to unit test directly on their own. However you can test these through accessible members or by exposing them through their own component. For example placing helper functions, regular expressions or constants in their own service or constant.

## Modules

### Avoid Naming Collisions

Use unique naming conventions with separators for sub-modules

- Unique names help avoid module name collisions. Separators help define modules and their submodule hierarchy. For example `app` may be your root module while `app.dashboard` and `app.users` may be modules that are used as dependencies of `app`.

### Definitions (aka Setters)

Declare modules without a variable using the setter syntax

- With 1 component per file, there is rarely a need to introduce a variable for the module.

```coffee
# avoid

app = angular.module('app', [
  'ngAnimate'
  'ngRoute'
  'app.shared'
  'app.dashboard'
])
```

  Instead use the simple getter syntax.

```coffee
# recommended

angular
  .module('app', [
  'ngAnimate'
  'ngRoute'
  'app.shared'
  'app.dashboard'
])
```

### Getters

When using a module, avoid using a variable and instead use chaining with the getter syntax

- This produces more readable code and avoids variable collisions or leaks.

```coffee
# avoid
app = angular.module('app')
app.controller('SomeController' , SomeController)

SomeController = () ->
```

```coffee
# recommended
(->
  SomeController = () ->

  return

  angular
  .module('app')
  .controller('SomeController' , SomeController)
)()
```

### Setting vs Getting

Only set once and get for all other instances

- A module should only be created once, then retrieved from that point and after.

```coffee
# recommended

# to set a module
angular.module('app', [])

# to get a module
angular.module('app')
```

### Named vs Anonymous Functions

Use named functions instead of passing an anonymous function in as a callback

- This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

```coffee
# avoid
angular
  .module('app')
  .controller('Dashboard', () ->)
  .service('logger', () ->)
```

```coffee
# recommended

# dashboard.controller.coffee
(->
  DashboardController = () ->

    # logic goes here -->

    return

  angular
    .module('app')
    .controller('DashboardController', DashboardController)
)()
```

```coffee
# logger.service.coffee

(->
  loggerService = () ->

    # logic goes here -->

    return

  angular
    .module('app')
    .service('loggerService', loggerService)
)()
```

## Controllers

### controllerAs View Syntax

Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.

- Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`.

- It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

- Helps avoid using `$parent` calls in Views with nested controllers.

```pug
//- avoid
div(ng-controller="CustomerController")
    {{ name }}
```

```pug
//- recommended
div(ng-controller="CustomerController as customer")
    {{ customer.name }}
```

### controllerAs Controller Syntax

Use the `controllerAs` syntax over the `classic controller with $scope` syntax.

- The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

- `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.

- Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move the method to a service, and reference them from the controller. Consider using `$scope` in a controller only when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on).

```coffee
# avoid
(->
    Customer = () ->
      @name = {}
      @sendMessage = () ->

          # here @/this is not the same
          @stuff = "stuff"

      return

  angular
    .module('app')
    .controller('Customer', Customer)
)()
```

```coffee
# recommended
(->
    Customer = () ->
      vm = @
      vm.name = {}
      vm.sendMessage = () ->

      return

  angular
    .module('app')
    .controller('Customer', Customer)
)()
```

### controllerAs with vm

Use a capture variable for `this` when using the `controllerAs` syntax. Choose a consistent variable name such as `vm`, which stands for ViewModel.

- The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

```coffee
# avoid
(->
    Customer = () ->
      @name = {}
      @sendMessage = () ->

          # here @/this is not the same
          @stuff = "stuff"

      return

  angular
    .module('app')
    .controller('Customer', Customer)
)()
```

```coffee
# recommended
(->
    Customer = () ->
      vm = @
      vm.name = {}
      vm.sendMessage = () ->

      return

  angular
    .module('app')
    .controller('Customer', Customer)
)()
```

  Note: When creating watches in a controller using `controller as`, you can watch the `vm.*` member using the following syntax. (Create watches with caution as they add more load to the digest cycle.)

```pug
input(ng-model="vm.title")
```

```coffee
SomeController = ($scope, $log)->
  vm = @
  vm.title = 'Some Title'

  $scope.$watch 'vm.title', (current, original)->
    $log.info('vm.title was %s', original)
    $log.info('vm.title is now %s', current)
```

  Note: When working with larger codebases, using a more descriptive name can help ease cognitive overhead & searchability. Avoid overly verbose names that are cumbersome to type.

```pug
//- avoid
input(ng-model="customerProductItemVm.title")
```

```pug
//- recommended
input(ng-model="productVm.title")
```

### Defer Controller Logic to Services

Defer logic in a controller by delegating to services.

- Logic may be reused by multiple controllers when placed within a service and exposed via a function.

- Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

- Removes dependencies and hides implementation details from the controller.

- Keeps the controller slim, trim, and focused.

```coffee
# avoid
(->
  Order = ( $http, $q )->

    @checkCredit = checkCredit
    @total = 0

    checkCredit = ()=>
        orderTotal = @total
        $http.get('api/creditcheck').then (data)=>
            remaining = data.remaining
            return $q.when(!!(remaining > orderTotal))
  angular
    .module('app')
    .controller('Order', Order)
)()
```

```coffee
# recommended
(->
  OrderController = (creditService)->

    vm = @

    init = () ->

        vm.checkCredit = checkCredit
        vm.total = 0

    checkCredit = () ->

        creditService.check()

    init()

    return

  OrderController
    .$inject = [
      'creditService'
    ]

  angular
    .module('app')
    .controller('OrderController', OrderController)
)()
```

### Keep Controllers Focused

Define a controller for a view, and try not to reuse the controller for other views. Instead, move reusable logic to services and keep the controller simple and focused on its view.

- Reusing controllers with several views is brittle and good end-to-end (e2e) test coverage is required to ensure stability across large applications.

## Services

### Singletons

Services are singletons and return an object that contains the members of the service.

- Services are instantiated with the `new` keyword, use `this` for public methods and variables. Since these are so similar to factories, use a service instead of factories for consistency.

Note: [All Angular services are singletons](https://docs.angularjs.org/guide/services). This means that there is only one instance of a given service per injector.

```coffee
# recommended
# service
(->
  loggerService = () ->

    return {
        logError: (msg) ->
        # ... #
    }

  angular
    .module('app')
    .service('loggerService', loggerService)
)()
```

### Services Single Responsibility

Services should have a [single responsibility](https://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a service begins to exceed that singular purpose, a new service should be created.

### Accessible Members Up Top

Expose the callable members of the service (its interface) at the top, using a technique derived from the [Revealing Module Pattern](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript).

- Placing the callable members at the top makes it easy to read and helps you instantly identify which members of the service can be called and must be unit tested (and/or mocked).

- This is especially helpful when the file gets longer as it helps avoid the need to scroll to see what is exposed.

- Setting functions as you go can be easy, but when those functions are more than 1 line of code they can reduce the readability and cause more scrolling. Defining the callable interface via the returned service moves the implementation details down, keeps the callable interface up top, and makes it easier to read.

*NOTE*: The javascript version of this style guide uses hoisted functions. CoffeeScript does not provide the ability do use function declarations (hoisted functions). So write functions in the return object.

```coffee
# avoid
(->
  dataService = () ->

    someValue = ''

    save = () ->
        # ... #

    validate = () ->
        # ... #

    return
        save: save,
        someValue: someValue,
        validate: validate

  angular
    .module('app')
    .service('dataService', dataService)
)()
```

```coffee
# recommended
(->
  dataService = () ->

    someValue = ''

    ##########

    return
        save: () ->
        # . #

        validate: () ->
        # . #
  angular
    .module('app')
    .service('dataService', dataService)
)()
```

## Data Services

### Separate Data Calls

Refactor logic for making data operations and interacting with data to a service. Make data services responsible for XHR calls, local storage, stashing in memory, or any other data operations.

- The controller's responsibility is for the presentation and gathering of information for the view. It should not care how it gets the data, just that it knows who to ask for it. Separating the data services moves the logic on how to get it to the data service, and lets the controller be simpler and more focused on the view.

- This makes it easier to test (mock or real) the data calls when testing a controller that uses a data service.

- Data service implementation may have very specific code to handle the data repository. This may include headers, how to talk to the data, or other services such as `$http`. Separating the logic into a data service encapsulates this logic in a single place hiding the implementation from the outside consumers (perhaps a controller), also making it easier to change the implementation.

```coffee
# recommended

# dataService
(->
  dataService = (
    $http,
    logger
  ) ->

    getAvengersComplete = (response)->
          response.data.results

    getAvengersFailed = (error)->
          logger.error('XHR Failed for getAvengers.' + error.data)

    return {
      getAvengers: () ->
        return $http.get('/api/maa')
            .then(getAvengersComplete)
            .catch(getAvengersFailed)
    }


  dataService
      .$inject = [
        '$http',
        'logger'
      ]

  angular
    .module('app.core')
    .service('dataService', dataService)
)()
```

Note: The data service is called from consumers, such as a controller, hiding the implementation from the consumers, as shown below.

```coffee
# recommended

# controller calling the dataservice service
(->
  AvengersController = (
    dataService,
    logger
  )->

    var vm = @
    vm.avengers = []

    activate = () ->
        getAvengers().then () ->
            logger.info('Activated Avengers View')

    getAvengers = () ->
        dataservice.getAvengers()
            .then (data)->
                vm.avengers = data

  AvengersController
    .$inject = [
      'dataService',
      'logger'
    ]

  angular
    .module('app.avengers')
    .controller('AvengersController', AvengersController)
)()
```

### Return a Promise from Data Calls

When calling a data service that returns a promise such as `$http`, return a promise in your calling function too.

- You can chain the promises together and take further action after the data call completes and resolves or rejects the promise.

```coffee
# recommended

activate = () ->

  # Step 1
  # Ask the getAvengers function for the avenger data and wait for the promise

  getAvengers().then () ->

    # Step 4
    # Perform an action on resolve of final promise

    logger.info('Activated Avengers View')


getAvengers = () ->

  # Step 2
  # Ask the data service for the data and wait for the promise

  return dataservice.getAvengers()
    .then (data)->

      # Step 3
      # set the data and resolve the promise

      vm.avengers = data
```

## Manual Annotating for Dependency Injection

### UnSafe from Minification

Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach.

- The parameters to the component (e.g. controller, service, etc) will be converted to mangled variables. For example, `common` and `dataservice` may become `a` or `b` and not be found by Angular.

```coffee
# avoid - not minification-safe ###
(->
  Dashboard = (common, dataservice)->

  angular
    .module('app')
    .controller('Dashboard', Dashboard)
)()
```

This code may produce mangled variables when minified and thus cause runtime errors.

```javascript
/* avoid - not minification-safe*/
angular.module('app').controller('DashboardController', d);function d(a, b) { }
```

### Manually Identify Dependencies

Use `$inject` to manually identify your dependencies for Angular components.

- This technique mirrors the technique used by [`ng-annotate`](https://github.com/olov/ng-annotate), which I recommend for automating the creation of minification safe dependencies. If `ng-annotate` detects injection has already been made, it will not duplicate it.

- This safeguards your dependencies from being vulnerable to minification issues when parameters may be mangled. For example, `common` and `dataservice` may become `a` or `b` and not be found by Angular.

- Avoid creating in-line dependencies as long lists can be difficult to read in the array. Also it can be confusing that the array is a series of strings while the last item is the component's function.

```coffee
# avoid
(->
  Dashboard = ($location, $routeParams, common, dataservice)->

  angular
    .module('app')
    .controller('Dashboard',
        ['$location', '$routeParams', 'common', 'dataservice', Dashboard])
)()
```

```coffee
# recommended
(->
  Dashboard = (
    $location,
    $routeParams,
    common,
    dataservice
  )->

    Dashboard
    .$inject = [
      '$location',
      '$routeParams',
      'common',
      'dataservice'
    ]

  angular
    .module('app')
    .controller('Dashboard', Dashboard)
)()
```

## Naming

### Naming Guidelines

Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. My recommended pattern is `feature.type.coffee`. There are 2 names for most assets:

1. the file name (`avengers.controller.coffee`)
1. the registered component name with Angular (`AvengersController`)

&nbsp;

- Naming conventions help provide a consistent way to find content at a glance. Consistency within the project is vital. Consistency with a team is important. Consistency across a company provides tremendous efficiency.

- The naming conventions should simply help you find your code faster and make it easier to understand.

### Feature File Names

Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. My recommended pattern is `feature.type.js`.

- Provides a consistent way to quickly identify components.

- Provides pattern matching for any automated tasks.

```javascript
// recommended

// controllers
avengers.controller.coffee
avengers.controller.spec.js

// services/services
logger.service.coffee
logger.service.spec.js

// constants
constants.coffee

// module definition
avengers.module.coffee

// routes
avengers.routes.coffee
avengers.routes.spec.js

// configuration
avengers.config.coffee

// directives
avenger-profile.directive.coffee
avenger-profile.directive.spec.js
```

### Test File Names

Name test specifications similar to the component they test with a suffix of `spec`.

- Provides a consistent way to quickly identify components.

- Provides pattern matching for [karma](http://karma-runner.github.io/) or other test runners.

```javascript
// recommended

avengers.controller.spec.js
logger.service.spec.js
avengers.routes.spec.js
avenger-profile.directive.spec.js
```

### Controller Names

Use consistent names for all controllers named after their feature. Use UpperCamelCase for controllers, as they are constructors.

- Provides a consistent way to quickly identify and reference controllers.

- UpperCamelCase is conventional for identifying object that can be instantiated using a constructor.

```coffee
# recommended

#avengers.controller.coffee
(->
  HeroAvengersController = ()->

  angular
    .module
    .controller('HeroAvengersController', HeroAvengersController)
)()
```

### Controller Name Suffix

Append the controller name with the suffix `Controller`.

- The `Controller` suffix is more commonly used and is more explicitly descriptive.

```coffee
# recommended

# avengers.controller.coffee
(->
  AvengersController = () ->

    return

  angular
    .module
    .controller('AvengersController', AvengersController)
)()
```

### Service Names

Use consistent names for all services named after their feature. Use camel-casing for services. Avoid prefixing services with `$`. Always suffix services with `Service`.

- Provides a consistent way to quickly identify and reference services.

- Avoids name collisions with built-in services that use the `$` prefix.

- Clear service names such as `logger` do not require a suffix.

- Service names such as `avengers` are nouns and require a suffix and should be named `avengersService`.

```coffee
# recommended

# logger.service.coffee
(->
  loggerService = () ->

  angular
    .module
    .service('loggerService', loggerService)
)()

# credit.service.coffee
(->
  creditService = () ->

  angular
  .module
  .service('creditService', creditService)
)()

# customer.service.coffee
(->
  customerService = () ->

  angular
    .module
    .service('customerService', customerService)
)()
```

### Directive Component Names

Use consistent names for all directives using camel-case. Use a short prefix to describe the area that the directives belong (some example are company prefix or project prefix).

- Provides a consistent way to quickly identify and reference components.

```coffee
# recommended

# avenger-profile.directive.coffee
(->
  xxAvengerProfile = () ->
    return

  angular
    .module
    .directive('xxAvengerProfile', xxAvengerProfile)
)()

# usage is <xx-avenger-profile> </xx-avenger-profile>
```

### Modules Names

When there are multiple modules, the main module file is named `app.module.coffee` while other dependent modules are named after what they represent. For example, an admin module is named `admin.module.coffee`. The respective registered module names would be `app` and `admin`.

- Provides consistency for multiple module apps, and for expanding to large applications.

- Provides easy way to use task automation to load all module definitions first, then all other angular files (for bundling).

### Configuration

Separate configuration for a module into its own file named after the module. A configuration file for the main `app` module is named `app.config.coffee` (or simply `config.coffee`). A configuration for a module named `admin.module.coffee` is named `admin.config.coffee`.

- Separates configuration from module definition, components, and active code.

- Provides an identifiable place to set configuration for a module.

### Routes

eparate route configuration into its own file. Examples might be `app.route.coffee` for the main module and `admin.route.coffee` for the `admin` module. Even in smaller apps I prefer this separation from the rest of the configuration.

## Application Structure LIFT Principle

### LIFT

Structure your app such that you can `L`ocate your code quickly, `I`dentify the code at a glance, keep the `F`lattest structure you can, and `T`ry to stay DRY. The structure should follow these 4 basic guidelines.

- Provides a consistent structure that scales well, is modular, and makes it easier to increase developer efficiency by finding code quickly. Another way to check your app structure is to ask yourself: How quickly can you open and work in all of the related files for a feature?

When I find my structure is not feeling comfortable, I go back and revisit these LIFT guidelines

1. `L`ocating our code is easy
1. `I`dentify code at a glance
1. `F`lat structure as long as we can
1. `T`ry to stay DRY (Don’t Repeat Yourself) or T-DRY

### Locate

Make locating your code intuitive, simple and fast.

- I find this to be super important for a project. If the team cannot find the files they need to work on quickly, they will not be able to work as efficiently as possible, and the structure needs to change. You may not know the file name or where its related files are, so putting them in the most intuitive locations and near each other saves a ton of time. A descriptive folder structure can help with this.

```text
/bower_components
/client
  /app
    /avengers
    /blocks
      /exception
      /logger
    /core
    /dashboard
    /data
    /layout
    /widgets
  /content
  index.pug
.bower.json
```

### Identify

When you look at a file you should instantly know what it contains and represents.

- You spend less time hunting and pecking for code, and become more efficient. If this means you want longer file names, then so be it. Be descriptive with file names and keeping the contents of the file to exactly 1 component. Avoid files with multiple controllers, multiple services, or a mixture. There are deviations of the 1 per file rule when I have a set of very small features that are all related to each other, they are still easily identifiable.

### Flat

Keep a flat folder structure as long as possible. When you get to 7+ files, begin considering separation.

- Nobody wants to search 7 levels of folders to find a file. Think about menus on web sites … anything deeper than 2 should take serious consideration. In a folder structure there is no hard and fast number rule, but when a folder has 7-10 files, that may be time to create subfolders. Base it on your comfort level. Use a flatter structure until there is an obvious value (to help the rest of LIFT) in creating a new folder.

### T-DRY (Try to Stick to DRY)

Be DRY, but don't go nuts and sacrifice readability.

- Being DRY is important, but not crucial if it sacrifices the others in LIFT, which is why I call it T-DRY. I don’t want to type session-view.html for a view because, well, it’s obviously a view. If it is not obvious or by convention, then I name it.

## Application Structure

### Overall Guidelines

Have a near term view of implementation and a long term vision. In other words, start small but keep in mind on where the app is heading down the road. All of the app's code goes in a root folder named `app`. All content is 1 feature per file. Each controller, service, module, view is in its own file. All 3rd party vendor scripts are stored in another root folder and not in the `app` folder. I didn't write them and I don't want them cluttering my app (`bower_components`, `scripts`, `lib`).

Note: Find more details and reasoning behind the structure at [this original post on application structure](http://www.johnpapa.net/angular-app-structuring-guidelines/).

### Layout

Place components that define the overall layout of the application in a folder named `layout`. These may include a shell view and controller may act as the container for the app, navigation, menus, content areas, and other regions.

- Organizes all layout in a single place re-used throughout the application.

### Folders-by-Feature Structure

Create folders named for the feature they represent. When a folder grows to contain more than 7 files, start to consider creating a folder for them. Your threshold may be different, so adjust as needed.

- A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there is no repetitive nor redundant names.

- The LIFT guidelines are all covered.

- Helps reduce the app from becoming cluttered through organizing the content and keeping them aligned with the LIFT guidelines.

- When there are a lot of files (10+) locating them is easier with a consistent folder structures and more difficult in flat structures.

```javascript
/**
* recommended
*/

app/
    app.module.coffee
    app.config.coffee
    components/
        calendar.directive.coffee
        calendar.directive.pug
        user-profile.directive.coffee
        user-profile.directive.pug
    layout/
        shell.pug
        shell.controller.coffee
        topnav.pug
        topnav.controller.coffee
    people/
        attendees.pug
        attendees.controller.coffee
        people.routes.coffee
        speakers.pug
        speakers.controller.coffee
        speaker-detail.pug
        speaker-detail.controller.coffee
    services/
        data.service.coffee
        localstorage.service.coffee
        logger.service.coffee
        spinner.service.coffee
    sessions/
        sessions.pug
        sessions.controller.coffee
        sessions.routes.coffee
        session-detail.pug
        session-detail.controller.coffee
```

![Sample App Structure](https://raw.githubusercontent.com/johnpapa/angular-styleguide/master/a1/assets/modularity-2.png)

Note: Do not structure your app using folders-by-type. This requires moving to multiple folders when working on a feature and gets unwieldy quickly as the app grows to 5, 10 or 25+ views and controllers (and other features), which makes it more difficult than folder-by-feature to locate files.

```javascript
/*
* avoid
* Alternative folders-by-type.
* I recommend "folders-by-feature", instead.
*/

app/
    app.module.coffee
    app.config.coffee
    app.routes.coffee
    directives.coffee
    controllers/
        attendees.coffee
        session-detail.coffee
        sessions.coffee
        shell.coffee
        speakers.coffee
        speaker-detail.coffee
        topnav.coffee
    directives/
        calendar.directive.coffee
        calendar.directive.pug
        user-profile.directive.coffee
        user-profile.directive.pug
    services/
        dataservice.coffee
        localstorage.coffee
        logger.coffee
        spinner.coffee
    views/
        attendees.pug
        session-detail.pug
        sessions.pug
        shell.pug
        speakers.pug
        speaker-detail.pug
        topnav.pug
```

## Modularity

### Many Small, Self Contained Modules

Create small modules that encapsulate one responsibility.

- Modular applications make it easy to plug and go as they allow the development teams to build vertical slices of the applications and roll out incrementally. This means we can plug in new features as we develop them.

### Create an App Module

Create an application root module whose role is to pull together all of the modules and features of your application. Name this for your application.

- Angular encourages modularity and separation patterns. Creating an application root module whose role is to tie your other modules together provides a very straightforward way to add or remove modules from your application.

### Keep the App Module Thin

Only put logic for pulling together the app in the application module. Leave features in their own modules.

- Adding additional roles to the application root to get remote data, display views, or other logic not related to pulling the app together muddies the app module and make both sets of features harder to reuse or turn off.

- The app module becomes a manifest that describes which modules help define the application.

### Feature Areas are Modules

Create modules that represent feature areas, such as layout, reusable and shared services, dashboards, and app specific features (e.g. customers, admin, sales).

- Self contained modules can be added to the application with little or no friction.

- Sprints or iterations can focus on feature areas and turn them on at the end of the sprint or iteration.

- Separating feature areas into modules makes it easier to test the modules in isolation and reuse code.

### Reusable Blocks are Modules

Create modules that represent reusable application blocks for common services such as exception handling, logging, diagnostics, security, and local data stashing.

- These types of features are needed in many applications, so by keeping them separated in their own modules they can be application generic and be reused across applications.

### Module Dependencies

The application root module depends on the app specific feature modules and any shared or reusable modules.

![Modularity and Dependencies](https://raw.githubusercontent.com/johnpapa/angular-styleguide/master/a1/assets/modularity-1.png)

- The main app module contains a quickly identifiable manifest of the application's features.

- Each feature area contains a manifest of what it depends on, so it can be pulled in as a dependency in other applications and still work.

- Intra-App features such as shared data services become easy to locate and share from within `app.core` (choose your favorite name for this module).

Note: This is a strategy for consistency. There are many good options here. Choose one that is consistent, follows Angular's dependency rules, and is easy to maintain and scale.

>
  My structures vary slightly between projects but they all follow these guidelines for structure and modularity. The implementation may vary depending on the features and the team. In other words, don't get hung up on an exact like-for-like structure but do justify your structure using consistency, maintainability, and efficiency in mind.
>
  In a small app, you can also consider putting all the shared dependencies in the app module where the feature modules have no direct dependencies. This makes it easier to maintain the smaller application, but makes it harder to reuse modules outside of this application.

## Testing

Unit testing helps maintain clean code, as such I included some of my recommendations for unit testing foundations with links for more information.

### Write Tests with Stories

Write a set of tests for every story. Start with an empty test and fill them in as you write the code for the story.

- Writing the test descriptions helps clearly define what your story will do, will not do, and how you can measure success.

```javascript
it('should have Avengers controller', function() {
    // TODO
});

it('should find 1 Avenger when filtered by name', function() {
    // TODO
});

it('should have 10 Avengers', function() {
    // TODO (mock data?)
});

it('should return Avengers via XHR', function() {
    // TODO ($httpBackend?)
});

// and so on
```

### Testing Library

Use [Jasmine](http://jasmine.github.io/) or [Mocha](http://mochajs.org) for unit testing.

- Both Jasmine and Mocha are widely used in the Angular community. Both are stable, well maintained, and provide robust testing features.

    Note: When using Mocha, also consider choosing an assert library such as [Chai](http://chaijs.com). I prefer Mocha.

### Test Runner

Use [Karma](http://karma-runner.github.io) as a test runner.

- Karma is easy to configure to run once or automatically when you change your code.

- Karma hooks into your Continuous Integration process easily on its own or through Grunt or Gulp.

- Some IDE's are beginning to integrate with Karma, such as [WebStorm](http://www.jetbrains.com/webstorm/) and [Visual Studio](https://visualstudiogallery.msdn.microsoft.com/02f47876-0e7a-4f6c-93f8-1af5d5189225).

- Karma works well with task automation leaders such as [Grunt](http://gruntjs.com/) (with [grunt-karma](https://github.com/karma-runner/grunt-karma)) and [Gulp](http://gulpjs.com/). When using Gulp, use [Karma](https://github.com/karma-runner/karma) directly and not with a plugin as the API can be called directly.

```javascript
/* recommended */

// Gulp example with Karma directly
function startTests(singleRun, done) {
    var child;
    var excludeFiles = [];
    var fork = require('child_process').fork;
    var Server = require('karma').Server;
    var serverSpecs = config.serverIntegrationSpecs;

    if (args.startServers) {
        log('Starting servers');
        var savedEnv = process.env;
        savedEnv.NODE_ENV = 'dev';
        savedEnv.PORT = 8888;
        child = fork(config.nodeServer);
    } else {
        if (serverSpecs && serverSpecs.length) {
            excludeFiles = serverSpecs;
        }
    }

    var karmaOptions = {
      configFile: __dirname + '/karma.conf.js',
      exclude: excludeFiles,
      singleRun: !!singleRun
    };

    let server = new Server(karmaOptions, karmaCompleted);
    server.start();

    ////////////////

    function karmaCompleted(karmaResult) {
        log('Karma completed');
        if (child) {
            log('shutting down the child process');
            child.kill();
        }
        if (karmaResult === 1) {
            done('karma: tests failed with code ' + karmaResult);
        } else {
            done();
        }
    }
}
```

### Stubbing and Spying

Use [Sinon](http://sinonjs.org/) for stubbing and spying.

- Sinon works well with both Jasmine and Mocha and extends the stubbing and spying features they offer.

- Sinon makes it easier to toggle between Jasmine and Mocha, if you want to try both.

- Sinon has descriptive messages when tests fail the assertions.

### Headless Browser

Use [PhantomJS](http://phantomjs.org/) to run your tests on a server.

- PhantomJS is a headless browser that helps run your tests without needing a "visual" browser. So you do not have to install Chrome, Safari, IE, or other browsers on your server.

Note: You should still test on all browsers in your environment, as appropriate for your target audience.

### Organizing Tests

Place unit test files (specs) side-by-side with your client code. Place specs that cover server integration or test multiple components in a separate `tests` folder.

- Unit tests have a direct correlation to a specific component and file in source code.

- It is easier to keep them up to date since they are always in sight. When coding whether you do TDD or test during development or test after development, the specs are side-by-side and never out of sight nor mind, and thus more likely to be maintained which also helps maintain code coverage.

- When you update source code it is easier to go update the tests at the same time.

- Placing them side-by-side makes it easy to find them and easy to move them with the source code if you move the source.

- Having the spec nearby makes it easier for the source code reader to learn how the component is supposed to be used and to discover its known limitations.

- Separating specs so they are not in a distributed build is easy with grunt or gulp.

```javascript
// recommended

src/client/app/customers/customer-detail.controller.coffee
                        /customer-detail.controller.spec.js
                        /customers.controller.coffee
                        /customers.controller.spec.js
                        /customers.module.coffee
                        /customers.route.coffee
                        /customers.route.spec.js
```

## Constants

### Vendor Globals

Create an Angular Constant for vendor libraries' global variables.

- Provides a way to inject vendor libraries that otherwise are globals. This improves code testability by allowing you to more easily know what the dependencies of your components are (avoids leaky abstractions). It also allows you to mock these dependencies, where it makes sense.

```coffee
# constants.coffee

# global toastr:false, moment:false

angular
  .module('app.core')
    .constant('toastr', toastr)
    .constant('moment', moment)
```

Use constants for values that do not change and do not come from another service. When constants are used only for a module that may be reused in multiple applications, place constants in a file per module named after the module. Until this is required, keep constants in the main module in a `constants.js` file.

- A value that may change, even infrequently, should be retrieved from a service so you do not have to change the source code. For example, a url for a data service could be placed in a constants but a better place would be to load it from a web service.

- Constants can be injected into any angular component, including providers.

- When an application is separated into modules that may be reused in other applications, each stand-alone module should be able to operate on its own including any dependent constants.

```coffee
# Constants used by the entire app
angular
  .module('app.core')
  .constant('moment', moment);

# Constants used only by the sales module
angular
  .module('app.sales')
  .constant('events', {
    ORDER_CREATED: 'event_order_created',
    INVENTORY_DEPLETED: 'event_inventory_depleted'
  })
```

Use task automation to list module definition files `*.module.coffee` before all other application JavaScript files.

- Angular needs the module definitions to be registered before they are used.

- Naming modules with a specific pattern such as `*.module.coffee` makes it easy to grab them with a glob and list them first.

```javascript
var clientApp = './src/client/app/';

// Always grab module files first
var files = [
  clientApp + '**/*.module.coffee',
  clientApp + '**/*.coffee'
];
```

## Angular Docs

For anything else, API reference, check the [Angular documentation](//docs.angularjs.org/api) and follow its code style if not mentioned in this guide.
