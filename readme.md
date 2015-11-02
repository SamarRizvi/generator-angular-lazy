# Yeoman generator for AngularJS projects

> Opinionated Yeoman generator for creating Angular applications which lazy load components as needed at runtime.

## Table of contents
- [What's included](#what-s-included-)
    - [SystemJS](#systemjs)
    - [JSPM](#jspm)
    - [AngularJS](#angularjs)
    - [UI Router](#ui-router)
    - [UI Router Extras](#ui-router-extras)
    - [ocLazyLoad](#oclazyload)
- [Structure](#structure)
    - [Application component](#application-component)
    - [State component](#state-component)
    - [General component](#general-component)

## What's included?
These are the main tools and libraries the project stack relies on.

### [SystemJS](https://github.com/systemjs/systemjs)
We're using the recently, in ECMAScript 2015, standardized module system. SystemJS builds up on these APIs to make it easier for us to modularize out code properly and to load those modules as they're needed. Since most browsers don't implement the module system natively SystemJS uses the [ES2015 Module Loader Polyfill](https://github.com/ModuleLoader/es6-module-loader) under the hood to close the gap.

Since the ES2015 module loader system is farly new most of the existing JavaScript libraries didn't have the chance yet to migrate to the new syntax. AMD and CommonJS are still the most used systems. SystemJS implements adapters for those module systems so that we're not blocked when it comes to use popular libraries like AngularJS or Lodash which do not yet use the new import / export syntax.

### [JSPM](https://jspm.io)
[NPM](https://www.npmjs.com) is a great package manager but it was initially designed to be used on the server side. There is no straight forward way to load NPM packages in the browser at runtime. JSPM eases that process and also overwrites some package.json properties for certain packages where necessary, e.g. the main file.

### [AngularJS](https://angularjs.org)
If you're here then you should know what Angular is.

### [UI Router](http://angular-ui.github.io/ui-router/)
Angular's integrated router has very limited capabilities, e.g. it doesn't support nested views. UI Router gives you much more flexibility and has become the de-facto standard router for Angular applications.

### [UI Router Extras](http://christopherthielen.github.io/ui-router-extras)
UI Router Extras adds even more functionality to the router on top of UI Router. Most important, [Future States](http://christopherthielen.github.io/ui-router-extras/#/future) which enable us to describe, in an abstract way, what states our application has and where the code for those resides, without actually loading the JavaScript code itself. It is then lazy loaded at runtime when the uset accesses the state for the first time.

### [ocLazyLoad](https://oclazyload.readme.io)
By default Angular requires us to load all application code upfront before it boots the application. That works well for smaller applications. For large scale applications this introduces long loading times an impacts the user experience negatively. ocLazyLoad allows us to add modules to Angular applications at runtime.

### Angular Translate
TODO

### Karma
TODO

### Protractor
TODO

### SASS
TODO

## Structure
Angular Lazy follows a component based approach. Everything is a component, even the application itself. Components should be self-contained and should be easily reusable for multiple projects. Of course there will be cases where a component is very specific to a project and might not be reusable. But always have the goal of reusability in focus.

### General
Each component has a `index.js` file which is the main access point of it. Other components should not directly reference resources than `index.js` from each other. Within there the component should expose / export all of it's public API. Other common files across all components are the `*-spec.js` and `*-test.js` files. Spec-files contain the unit tests and test-files the end-to-end tests. For larger components tests can also be split across multiple files. The Karma test runner will scan the project for all `*-spec.js` files and Protractor will load all `*-test.js` files to run the end-to-end tests.

#### i18n
If you choose to activate `i18n` while generating the application, each component will have a `i18n` folder which contains it's translations.

### Application component

> $: yo angular-lazy

```text
+src
|  +components
|  |  +application
|  |  |  +config
|  |  |  |  constants.json
|  |  |  |  default-locale.js
|  |  |  |  error-handling.js
|  |  |  |  routes.json
|  |  |  |  routing.js
|  |  |  i18n
|  |  |  stylesheets
|  |  |  application.html
|  |  |  application-controller.js
|  |  |  application-route.js
|  |  |  application-spec.js
|  |  |  application-test
|  |  |  index.js
|  index.js
```

#### constants.json
As the filename suggests, this is the place where you define application wide constants. Those can be imported where necessary and are also available as injectable values within the application.

#### default-locale.js
In here, the default locale is configured. This file will only be present if you choose to activate `i18n` while generating the application structure.

#### error-handling.js
By default, this file only contains a small code piece which logs state transition errors. UI Router swallows transition error by default and we're left on our own to figure out what happened. This file can  be extend with additional error handling functionality as needed, e.g. network errors.

#### routes.json
This is where we define our lazy loaded routes. For the Future States feature from UI Router Extras work properly, we need to tell it what routes exist and where they can be loaded from.

```javascript
[
  {
    "name": "app",
    "url": "/",
    "type": "load",
    "src": "components/application/index"
  },
  {
    "name": "home",
    "url": "home",
    "type": "load",
    "src": "components/home-state/index"
  }
]
```

The name and url properties must match those we use in the `$stateProvider.state(...)` call. If a route is not yet loaded, UI Router Extras will catch the `$stateNotFound` event and look it up in the list of routes defined in `routes.json`. If it finds a match it will load the specific component and then resume the state transition. The type property is a helper to distinguish state types. Abstract states must be defined here too, like the `app` state in the example above. There is only one value `load` by default. It is used within `routing.js` to know which states should be handled by the default loader. If we have states which need to be handled specially, we can introduce new types and loaders. In most cases the default loader will be sufficient. And finally, the `src` property tells the loader where to load the state component from. This is always relative to the `baseURL` configured within SystemJS.

#### routing.js
This file contains the configuration for the state factory which lazy loads our code based on the definitions in `routes.json`.

#### application.html
This is the template for our basic layout, common for all states. By default it only contains a `ui-view` element.

#### application-controller.js
This file contains our application controller which is accessible for all components. It's mainly used for handling data which is needed throughout the whole application, e.g. information about the currently logged in user. Be careful to not overload it. Functionality like loading the actual user data should always be within a service.

#### application-route.js
This file contains the application state. This is only an abstract state and each other state within our application should be a direct or indirect descendant of it. This enables us to load application wide data before any of the actual states get loaded.

### State component
> $: yo angular-lazy:state name
> $: yo angular-lazy:state parent.child

```text
+src
|  +components
|  |  +[name]-state
|  |  |  i18n
|  |  |  [name]-route.js
|  |  |  [name]-state.html
|  |  |  [name]-state.scss
|  |  |  [name]-state-controller.js
|  |  |  [name]-state-spec.js
|  |  |  [name]-state-test.js
|  |  |  index.js
```

When running the state component generator it will automatically add the new state to `routes.json` within the application component.

#### [name]-route.js
This file contains the state definition for UI Router. If you change the URL or the state name at some point in time don't forget to also update it in `routes.json`. Otherwise the state will not be loaded properly when lazy loaded.

#### [name]-state-controller.js
This file contains the controller for the newly generated state.

### General component
> $: yo angular-lazy:component name

```text
+src
|  +components
|  |  +[name]
|  |  |  i18n
|  |  |  [name].html
|  |  |  [name].scss
|  |  |  [name]-controller.js
|  |  |  [name]-directive.js
|  |  |  [name]-spec.js
|  |  |  [name]-test.js
|  |  |  index.js
```

By default a component corresponds to a directive within AngularJS. Accordingly, if you run the component generator it will create a directive.