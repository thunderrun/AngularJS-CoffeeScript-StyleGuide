# Advanced

this part of guide is still working in progress

## Exception Handling

### decorators

Use a [decorator](https://docs.angularjs.org/api/auto/service/$provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/$provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/$exceptionHandler) service to perform custom actions when exceptions occur.

- Provides a consistent way to handle uncaught Angular exceptions for development-time or run-time.

Note: Another option is to override the service instead of using a decorator. This is a fine option, but if you want to keep the default behavior and extend it a decorator is recommended.

```coffee

(->

exceptionConfig = ($provide)->
    $provide.decorator
        '$exceptionHandler', ['$delegate', '$log', extendExceptionHandler]


extendExceptionHandler = ($delegate, $log)->
    return (exception, cause)->
        $delegate(exception, cause)
        errorData =
            exception: exception,
            cause: cause

        msg = 'ERROR PREFIX' + exception.message
        $log.error(msg, errorData)

        # Log during dev with http://toastrjs.com
        # or any other technique you prefer
        toastr.error(msg)


angular
    .module('app.exception')
    .config(['$provide', exceptionConfig])
)()
```

### Exception Catchers

Create a service that exposes an interface to catch and gracefully handle exceptions.

- Provides a consistent way to catch exceptions that may be thrown in your code (e.g. during XHR calls or promise failures).

  Note: The exception catcher is good for catching and reacting to specific exceptions from calls that you know may throw one. For example, when making an XHR call to retrieve data from a remote web service and you want to catch any exceptions from that service and react uniquely.

  ```javascript
  /* recommended */
  angular
      .module('blocks.exception')
      .service('exception', exception);

  exception.$inject = ['logger'];

  function exception(logger) {
      var service = {
          catcher: catcher
      };
      return service;

      function catcher(message) {
          return function(reason) {
              logger.error(message, reason);
          };
      }
  }
  ```

### Route Errors

Handle and log all routing errors using [`$routeChangeError`](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError)

- Provides a consistent way to handle all routing errors.

- Potentially provides a better user experience if a routing error occurs and you route them to a friendly screen with more details or recovery options.

  ```javascript
  /* recommended */
  var handlingRouteChangeError = false;

  function handleRoutingErrors() {
      /**
        * Route cancellation:
        * On routing error, go to the dashboard.
        * Provide an exit clause if it tries to do it twice.
        */
      $rootScope.$on('$routeChangeError',
          function(event, current, previous, rejection) {
              if (handlingRouteChangeError) { return; }
              handlingRouteChangeError = true;
              var destination = (current && (current.title ||
                  current.name || current.loadedTemplateUrl)) ||
                  'unknown target';
              var msg = 'Error routing to ' + destination + '. ' +
                  (rejection.msg || '');

              /**
                * Optionally log using a custom service or $log.
                * (Don't forget to inject custom service)
                */
              logger.warning(msg, [current]);

              /**
                * On routing error, go to another route/state.
                */
              $location.path('/');

          }
      );
  }
  ```

## Startup Logic

### Configuration

Inject code into [module configuration](https://docs.angularjs.org/guide/module#module-loading-dependencies) that must be configured before running the angular app. Ideal candidates include providers and constants.

- This makes it easier to have less places for configuration.

```javascript
angular
    .module('app')
    .config(configure);

configure.$inject =
    ['routerHelperProvider', 'exceptionHandlerProvider', 'toastr'];

function configure (routerHelperProvider, exceptionHandlerProvider, toastr) {
    exceptionHandlerProvider.configure(config.appErrorPrefix);
    configureStateHelper();

    toastr.options.timeOut = 4000;
    toastr.options.positionClass = 'toast-bottom-right';

    ////////////////

    function configureStateHelper() {
        routerHelperProvider.configure({
            docTitle: 'NG-Modular: '
        });
    }
}
```

### Run Blocks

Any code that needs to run when an application starts should be declared in a service, exposed via a function, and injected into the [run block](https://docs.angularjs.org/guide/module#module-loading-dependencies).

- Code directly in a run block can be difficult to test. Placing in a service makes it easier to abstract and mock.

```javascript
angular
    .module('app')
    .run(runBlock);

runBlock.$inject = ['authenticator', 'translator'];

function runBlock(authenticator, translator) {
    authenticator.initialize();
    translator.initialize();
}
```

## Animations

### Usage

Use subtle [animations with Angular](https://docs.angularjs.org/guide/animations) to transition between states for views and primary visual elements. Include the [ngAnimate module](https://docs.angularjs.org/api/ngAnimate). The 3 keys are subtle, smooth, seamless.

- Subtle animations can improve User Experience when used appropriately.

- Subtle animations can improve perceived performance as views transition.

### Sub Second

Use short durations for animations. I generally start with 300ms and adjust until appropriate.

- Long animations can have the reverse effect on User Experience and perceived performance by giving the appearance of a slow application.

### animate.css

Use [animate.css](http://daneden.github.io/animate.css/) for conventional animations.

- The animations that animate.css provides are fast, smooth, and easy to add to your application.

- Provides consistency in your animations.

- animate.css is widely used and tested.

  Note: See this [great post by Matias Niemelä on Angular animations](http://www.yearofmoo.com/2013/08/remastered-animation-in-angularjs-1-2.html)

## Routing

Client-side routing is important for creating a navigation flow between views and composing views that are made of many smaller templates and directives.

Use the [AngularUI Router](http://angular-ui.github.io/ui-router/) for client-side routing.

- UI Router offers all the features of the Angular router plus a few additional ones including nested routes and states.

- The syntax is quite similar to the Angular router and is easy to migrate to UI Router.

Note: You can use a provider such as the `routerHelperProvider` shown below to help configure states across files, during the run phase.

  ```javascript
  // customers.routes.js
  angular
      .module('app.customers')
      .run(appRun);

  /* @ngInject */
  function appRun(routerHelper) {
      routerHelper.configureStates(getStates());
  }

  function getStates() {
      return [
          {
              state: 'customer',
              config: {
                  abstract: true,
                  template: '<ui-view class="shuffle-animation"/>',
                  url: '/customer'
              }
          }
      ];
  }
  ```

  ```javascript
  // routerHelperProvider.js
  angular
      .module('blocks.router')
      .provider('routerHelper', routerHelperProvider);

  routerHelperProvider.$inject = ['$locationProvider', '$stateProvider', '$urlRouterProvider'];
  /* @ngInject */
  function routerHelperProvider($locationProvider, $stateProvider, $urlRouterProvider) {

      this.$get = RouterHelper;

      $locationProvider.html5Mode(true);

      RouterHelper.$inject = ['$state'];
      /* @ngInject */
      function RouterHelper($state) {
          var hasOtherwise = false;

          var service = {
              configureStates: configureStates,
              getStates: getStates
          };

          return service;

          ///////////////

          function configureStates(states, otherwisePath) {
              states.forEach(function(state) {
                  $stateProvider.state(state.state, state.config);
              });
              if (otherwisePath && !hasOtherwise) {
                  hasOtherwise = true;
                  $urlRouterProvider.otherwise(otherwisePath);
              }
          }

          function getStates() { return $state.get(); }
      }
  }
  ```

Define routes for views in the module where they exist. Each module should contain the routes for the views in the module.

- Each module should be able to stand on its own.

- When removing a module or adding a module, the app will only contain routes that point to existing views.

- This makes it easy to enable or disable portions of an application without concern over orphaned routes.

## Filters

Avoid using filters for scanning all properties of a complex object graph. Use filters for select properties.

- Filters can easily be abused and negatively affect performance if not used wisely, for example when a filter hits a large and deep object graph.

## Angular $ Wrapper Services

### $document and $window

Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

- These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

### $timeout and $interval

Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

- These services are wrapped by Angular and more easily testable and handle Angular's digest cycle thus keeping data binding in sync.

## Resolving Promises

### Controller Activation Promises

Resolve start-up logic for a controller in an `activate` function.

- Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

- The controller `activate` makes it convenient to re-use the logic for a refresh for the controller/View, keeps the logic together, gets the user to the View faster, makes animations easy on the `ng-view` or `ui-view`, and feels snappier to the user.

  ```coffee
  # avoid
  (->
    Avengers = (dataservice)->
          init = ()=>
          @avengers = []
          @title = 'Avengers'

          dataservice
          .getAvengers()
          .then (data)=>
              @avengers = data

          init()
          return

    angular
      .module('app')
      .controller('Avengers', Avengers)
  )()
  ```

  ```coffee
  # recommended
  (->
    Avengers = (dataservice)->

      init = ()=>
          @avengers = []
          @title = 'Avengers'

          activate()

      ##############

      activate = ()=>
          dataservice
          .getAvengers()
          .then (data)=>
              @avengers = data

      init()
      return

    angular
      .module('app')
      .controller('Avengers', Avengers)
  )()
  ```

### Handling Exceptions with Promises

The `catch` block of a promise must return a rejected promise to maintain the exception in the promise chain.

Always handle exceptions in services.

- If the `catch` block does not return a rejected promise, the caller of the promise will not know an exception occurred. The caller's `then` will execute. Thus, the user may never know what happened.

- To avoid swallowing errors and misinforming the user.

  Note: Consider putting any exception handling in a function in a shared module and service.

```javascript
/* avoid */

function getCustomer(id) {
    return $http.get('/api/customer/' + id)
        .then(getCustomerComplete)
        .catch(getCustomerFailed);

    function getCustomerComplete(data, status, headers, config) {
        return data.data;
    }

    function getCustomerFailed(e) {
        var newMessage = 'XHR Failed for getCustomer'
        if (e.data && e.data.description) {
          newMessage = newMessage + '\n' + e.data.description;
        }
        e.data.description = newMessage;
        logger.error(newMessage);
        // ***
        // Notice there is no return of the rejected promise
        // ***
    }
}

/* recommended */
function getCustomer(id) {
    return $http.get('/api/customer/' + id)
        .then(getCustomerComplete)
        .catch(getCustomerFailed);

    function getCustomerComplete(data, status, headers, config) {
        return data.data;
    }

    function getCustomerFailed(e) {
        var newMessage = 'XHR Failed for getCustomer'
        if (e.data && e.data.description) {
          newMessage = newMessage + '\n' + e.data.description;
        }
        e.data.description = newMessage;
        logger.error(newMessage);
        return $q.reject(e);
    }
}
```

## Directives

### Limit 1 Per File

Create one directive per file. Name the file for the directive.

- It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module.

- One directive per file is easy to maintain.

  > Note: "**Best Practice**: Directives should clean up after themselves. You can use `element.on('$destroy', ...)` or `scope.$on('$destroy', ...)` to run a clean-up function when the directive is removed" ... from the Angular documentation.

```coffee
# avoid
angular
    .module('app.widgets')

    # order directive that is specific to the order module
    .directive('orderCalendarRange', orderCalendarRange)

    # sales directive that can be used anywhere across the sales app
    .directive('salesCustomerInfo', salesCustomerInfo)

    # spinner directive that can be used anywhere across apps
    .directive('sharedSpinner', sharedSpinner)

```

```coffee
# recommended

# order directive that is specific to the order module at a company named Acme
angular
    .module('sales.order')
    .directive('acmeOrderCalendarRange', orderCalendarRange)

# spinner directive that can be used anywhere across the sales app at a company named Acme
angular
    .module('sales.widgets')
    .directive('acmeSalesCustomerInfo', salesCustomerInfo)

# spinner directive that can be used anywhere across apps at a company named Acme
angular
    .module('shared.widgets')
    .directive('acmeSharedSpinner', sharedSpinner)

```

  Note: There are many naming options for directives, especially since they can be used in narrow or wide scopes. Choose one that makes the directive and its file name distinct and clear. Some examples are below, but see the [Naming](#naming) section for more recommendations.

### Manipulate DOM in a Directive

When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hides and shows, use ngHide/ngShow.

- DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templates)

### Provide a Unique Directive Prefix

Provide a short, unique and descriptive directive prefix such as `acmeSalesCustomerInfo` which would be declared in HTML as `acme-sales-customer-info`.

- The unique short prefix identifies the directive's context and origin. For example a prefix of `cc-` may indicate that the directive is part of a CodeCamper app while `acme-` may indicate a directive for the Acme company.

  Note: Avoid `ng-` as these are reserved for Angular directives. Research widely used directives to avoid naming conflicts, such as `ion-` for the [Ionic Framework](http://ionicframework.com/).

### Restrict to Elements and Attributes

When creating a directive that makes sense as a stand-alone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when it's stand-alone and as an attribute when it enhances its existing DOM element.

- It makes sense.

- While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

  Note: EA is the default for Angular 1.3 +

  ```pug
  //avoid
  div.my-calendar-range
  ```

  ```coffee
  # avoid
  (->
      myCalendarRange = () ->
          link = (scope, element, attrs)->
          # ... #

          directive =
          link: link,
          templateUrl: '/template/is/located/here.html',
          restrict: 'C'

          return directive

      angular
          .module('app.widgets')
          .directive('myCalendarRange', myCalendarRange)
  )()
  ```

  ```html
  //recommended
  div
    my-calendar-range
  ```

  ```coffee
  # recommended
  (->

      myCalendarRange = () ->

          link = (scope, element, attrs)->
          # ... #

          directive =
              link: link,
              templateUrl: '/template/is/located/here.html',
              restrict: 'EA'

          return directive

      angular
          .module('app.widgets')
          .directive('myCalendarRange', myCalendarRange)
  )()
  ```

### Directives and ControllerAs

Use `controller as` syntax with a directive to be consistent with using `controller as` with view and controller pairings.

- It makes sense and it's not difficult.

  Note: The directive below demonstrates some of the ways you can use scope inside of link and directive controllers, using controllerAs. I in-lined the template just to keep it all in one place.

  Note: Regarding dependency injection, see [Manually Identify Dependencies](#manual-annotating-for-dependency-injection).

  Note: Note that the directive's controller is outside the directive's closure. This style eliminates issues where the injection gets created as unreachable code after a `return`.

<!--todo: change this-->

```html
<div my-example max="77"></div>
```

```coffee
myExample = () ->
  directive =
    restrict: 'EA'
    templateUrl: 'app/feature/example.directive.html'
    scope: max: '='
    link: linkFunc
    controller: ExampleController
    controllerAs: 'vm'
    bindToController: true

  linkFunc = (scope, el, attr, ctrl) ->
    console.log 'LINK: scope.min = %s *** should be undefined', scope.min
    console.log 'LINK: scope.max = %s *** should be undefined', scope.max
    console.log 'LINK: scope.vm.min = %s', scope.vm.min
    console.log 'LINK: scope.vm.max = %s', scope.vm.max

  return directive

ExampleController = ($scope) ->
  # Injecting $scope just for comparison
  vm = this
  vm.min = 3
  console.log 'CTRL: $scope.vm.min = %s', $scope.vm.min
  console.log 'CTRL: $scope.vm.max = %s', $scope.vm.max
  console.log 'CTRL: vm.min = %s', vm.min
  console.log 'CTRL: vm.max = %s', vm.max

angular.module('app').directive 'myExample', myExample
ExampleController.$inject = [ '$scope' ]
```

```pug
div hello world
div
  | max={{vm.max}}
  input(ng-model='vm.max')
div
  | min={{vm.min}}
  input(ng-model='vm.min')
```

Use `bindToController = true` when using `controller as` syntax with a directive when you want to bind the outer scope to the directive's controller's scope.

- It makes it easy to bind outer scope to the directive's controller scope.

  Note: `bindToController` was introduced in Angular 1.3.0.

```pug
div(my-example='', max='77')
```

```coffee
(->
  myExample = () ->
    directive =
      restrict: 'EA'
      templateUrl: 'app/feature/example.directive.html'
      scope: max: '='
      controller: ExampleController
      controllerAs: 'vm'
      bindToController: true
    return directive

angular
  .module('app')
  .directive('myExample', myExample)
```

```pug
div hello world
div
  | max={{vm.max}}
  input(ng-model='vm.max')
div
  | min={{vm.min}}
  input(ng-model='vm.min')
```
