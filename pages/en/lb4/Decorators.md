---
lang: en
title: 'Decorators'
keywords: LoopBack 4.0, LoopBack-Next
tags:
sidebar: lb4_sidebar
permalink: /doc/en/lb4/Decorators.html
summary:
---

A decorator allows you to annotate or modify your class declarations and members with metadata.

## Introduction

*If you're new to Decorators in TypeScript, see [here](https://www.typescriptlang.org/docs/handbook/decorators.html) for more info.*

Decorators give LoopBack the flexibility to modify your plain TypeScript classes 
and properties in a way that allows the framework to better understand how to 
make use of them, without the need to inherit base classes or add functions 
that tie into an API.

Currently, LoopBack core provides built-in decorator functions you can use directly
by `@myDecorator`:

- [route](#route-decorators)
- [inject](#dependency-injection)

As a default, LoopBack also comes with some pre-defined component decorators:

- [authentication](#authentication-decorators)
- [repository](#repository-decorators)

## Route Decorators
- `@get`, `@post`, `@put`, `@patch`, `@del`

  Register route for a controller by REST decorator, for their usage, refer to 
  [Routing to Controllers](controllers.htm#Routing-to-Controllers)

- `@api`

  Register route for a controller by api decorator, more details and example refer 
  to [Specifying Controller APIs](controllers.htm#Specifying-Controller-APIs)

- `@operation`

  Expose a Controller method as a REST API operation. Usage refer to [TBD in Controllers](controllers.htm#)

- `@param`

  Describe an input parameter of a Controller method. 
  
  `@param` can be applied to method itself or specific parameters. For example,

  ```ts
    class MyController {
      @get('/')
      @param(offsetSpec)
      @param(pageSizeSpec)
      list(offset?: number, pageSize?: number) {}
    }
  ```

  or
  
  ```ts
    class MyController {
      @get('/')
      list(
        @param(offsetSpec) offset?: number,
        @param(pageSizeSpec) pageSize?: number,
      ) {}
    }
  ```
  
  More example can be found in [Writing Controller methods](controller.htm#Writing-Controller-methods)
  
## Dependency Injection

`@inject` is a decorator to annotate method arguments for automatic injection by 
LoopBack IoC container. `@inject` can only be used on properties or method parameters.

The `@inject` decorator allows you inject dependencies bound to the application context, 
to the per-request contextor, to any implementation of the [Context](#context) object. 
You can bind values, class definitions and provider functions to those contexts and 
then resolve values (or the results of functions that return those values!) in other 
areas of your code.

```ts
// application.ts
import {Application} from '@loopback/core';
import 'fs-extra';
class MyApp extends Application {
  constructor() {
    super();
    const app = this;
    const widgetConf = JSON.parse(fs.readSync('./widget-config.json'));
    app.bind('config.widget').to(widgetConf);
  }
}
```
Now that we've bound the 'config.widget' key to our configuration object, we can
inject it in our WidgetController:

```ts
// widget-controller.ts
import {widgetSpec} from '../apispec';
@api(widgetSpec)
class WidgetController {
  constructor(
    @inject('config.widget') protected widget: JSON
    // This will be resolved at runtime!
  ) {}
  // etc...
}
```

For more information, see the [Dependency Injection](Dependency-Injection.htm) section under [LoopBack Core Concepts](Concepts.htm)

## Authentication Decorator

- @authenticate

  Mark a controller method as requiring authenticated user, takes a strategy name 
  as a parameter. An example is using 'BasicStrategy' to authenticate user in function
  `whoAmI`:

  ```ts
  // my-controller.ts
  import { authenticate } from '@loopback/authentication';
  import { inject } from '@loopback/context';

  class MyController {
    constructor(
      @inject(BindingKeys.Authentication.CURRENT_USER)
      private user: UserProfile,
    ) {}

    @authenticate('BasicStrategy')
    async whoAmI() : Promise<string> {
      return this.user.id;
    }
  }
  ```

## Repository Decorators

### Model decorators

- `@model`

  Define a model in a repository, see [define-models](Repositories.html#define-models)

- `@property`

  Define a property in a model, see [define-models](Repositories.html#define-models)

### Relation decorators

  *This feature has not yet been released in alpha form. Documentation will be* 
  *added here as this feature progresses.*

- `@relation`

  Register a general relation

- `@belongsTo` `@hasOne` `@hasMany` `@embedsOne` `@embedsMany` `@referencesOne` `@referencesMany`

  Register a specific relation

### Repository decorator

- `@repository`

  A repository represents a collection of data and the means to access that data, 
  the decorator either injects an existing repository or creates a repository 
  from a model and a datasource.

  The injection example can be found in [Repository#controller-configuration](Repositories.html#controller-configuration)

  ```ts
  // my-controller.ts
  // Know more about how to create model 'Todo' and datasource 'datasource', 
  // please see example in [Thinking in LoopBack](Thinking-in-LoopBack.htm#define-product-model-repository-and-data-source)
  import { Todo } from '{path_of_Todo_model}.ts';
  import { datasource } from '{path_of_datasource}.ts';

  export class TodoController {
    @repository(Todo, datasource)
    repository: EntityCrudRepository<Todo, number>;
    ... ...
  }
  ```
  
  If the model or datasource is already bound to the app, you can create the 
  repository by providing their names instead of the classes. For example:

  ```ts
  // with `datasource` and `Todo` already defined.
  app.bind('datasources.ds').to(datasource);
  app.bind('repositories.todo').to(Todo);

  export class TodoController {
    @repository('todo', 'ds')
    repository: EntityCrudRepository<Todo, number>;
    ... ...
  }
  ```
