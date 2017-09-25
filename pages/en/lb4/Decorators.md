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

As a default, LoopBack comes with some pre-defined decorators:

- [route](#route-decorators)
- [inject](#dependency-injection)
- [authentication](#authentication-decorators)
- [repository](#repository-decorators)

## Route Decorators

Route decorators are used to expose controller methods as REST API operations.
If you are not familiar with the concept Route or Controller, please see [LoopBack Route](routes.htm)
and [LoopBack Controller](controllers.htm) to learn more about them.

By calling a decorator, you provide OpenAPI specification to describe the endpoint 
which the decorated method maps to:

### api

  Syntax: `@api`

  `@api` is a decorator for controller constructor, it's called before a controller
  class. `@api` is used when you have a `basePath` and a Paths Object, which
  contain multiple path definitions. Please note the api specs defined with `@api`
  will override other api specs defined inside the controller. For example:

  ```ts
    @api({
      basePath: '/',
      paths: {
        '/greet': {
          get: {
            'x-operation-name': 'greet',
            'x-controller-name': 'MyController',
            parameters: [{name: 'name', type: 'string', in: 'query'}],
            responses: {
              '200': {
                description: 'greeting text',
                schema: {type: 'string'},
              }
            }
          }
        }
      }
    })
    class MyController {
      // The operation endpoint defined here will be overriden!
      @get('/foo')
      @param.query.number('limit')
      greet(name) {
      }
    }
    app.controller(MyController);
  ```

  A more detailed explanation can be found in [Specifying Controller APIs](controllers.htm#Specifying-Controller-APIs)

### operation

  Syntax: `@operation`

  `@operation` is a controller method decorator. It exposes a Controller method as 
  a REST API operation. You can specify the verb, path, parameters and response 
  as specification of your endpoint, for example:

  ```ts
  const spec = {
    parameters: [{name: 'name', type: 'string', in: 'query'}],
    responses: {
      '200': {
        description: 'greeting text',
        schema: {type: 'boolean'},
      }
    }
  };
  class MyController {
    @operation('HEAD', '/checkExist', spec)
    checkExist(name) {
    }
  }
  ```
  
### commonly-used operations
  
  Syntax: `@get`, `@post`, `@put`, `@patch`, `@del`

  You can call these suger operation decorators as a shortcut of `@operation`, for example:
  ```ts
  class MyController {
    @get('/greet', spec)
    greet(name) {
    }
  }
  ```

  is equivalent to

  ```ts
  class MyController {
    @operation('GET', '/greet', spec)
    greet(name) {
    }
  }
  ```
  
  For more usage, refer to [Routing to Controllers](controllers.htm#Routing-to-Controllers)

### param

  Syntax: `@param`

  `@param` can be applied to method itself or specific parameters, it describes
  an input parameter of a Controller method.

  For example:

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
  
  It follows this pattern to make it quicker to define params: `@param.${in}.${type}(${name})`
  
  - in: one of the following values: query, header, path, formData, body
  - type: one of the following values:  string, number, boolean, integer
  - name: a string, name of the parameter

  So an example would be `@param.query.number('offset')`.

  You can find the specific usage in [Writing Controller methods](controller.htm#Writing-Controller-methods)
  
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

  Syntax: `@authenticate`

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

### Model Decorators

#### model

  Syntax: `@model`

  Define the decorated class as a Model for use with the Legacy Juggler.
  For usage examples, see [Define Models](Repositories.html#define-models)

#### property

  Syntax: `@property`

  Define the metadata for a Model property. For usage examples, see [Define Models](Repositories.html#define-models)

### Relation Decorators

  *This feature has not yet been released in alpha form. Documentation will be* 
  *added here as this feature progresses.*

- `@relation`

  Register a general relation

- `@belongsTo` `@hasOne` `@hasMany` `@embedsOne` `@embedsMany` `@referencesOne` `@referencesMany`

  Register a specific relation

### Repository Decorator

  Syntax: `@repository`
  
  A repository represents a collection of data and the means to access that data, 
  the decorator either injects an existing repository or creates a repository 
  from a model and a datasource.

  The injection example can be found in [Repository#controller-configuration](Repositories.html#controller-configuration)

  To create a repository in a controller, you can define your model and datasource 
  first, then import them in your controller file:

  *Know more about how to create model and datasource, please see example in [Thinking in LoopBack](Thinking-in-LoopBack.htm#define-product-model-repository-and-data-source)*
  
  ```ts
  // my-controller.ts
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
    // etc
  }
  ```
