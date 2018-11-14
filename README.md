# loopback-decorators

Add remote method decorators to loopback.

# Installation

## Requirements

- [Node.js](https://nodejs.org/en/download/) >= 6.9.1

Within your node project, install the package from npm using:

```shell
npm install loopback-decorators
```

## Special Providers

Loopback decorators provides several special provider tokens for common remote method dependencies. These are:

| Token                 | Provides                                                                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `$app`                | The loopback application (similar to `require('./server')`)                                                                                 |
| `$instance`           | The model instance for non-static remote methods                                                                                            |
| `$model`              | The model being operated on (for both static and non-static methods)                                                                        |
| `$ctx`                | The loopback http remoting context                                                                                                          |
| `$req`                | The loopback http request                                                                                                                   |
| `$res`                | The loopback http response                                                                                                                  |
| `$headers`            | The incoming http request headers                                                                                                           |
| `$optionsFromRequest` | See [using options from request](https://loopback.io/doc/en/lb3/Using-current-context.html#annotate-options-parameter-in-remoting-metadata) |
| `$options`            | Alias for `$optionsFromRequest`                                                                                                             |
| `^SomeModelName`      | Any model registered in your loopback model registry                                                                                        |

You can also setup your own providers (even with their own dependencies) as follows (see examples for further info):

```
{
  provide: 'tokenName',
  useFactory: (list, of, deps) => someValue,
  deps: ['$app', 'someDep', '^MyModel']
}
```

# Example Remote Method

A basic controller:

```ts
import { RemoteMethod } from 'loopback-decorators';

@RemoteMethod({
  selector: 'getStuff',
  providers: [
    '$app',
    '$instance',
    {
      provide: 'customEntity',
      useFactory: function(modelName) {
        return modelName.customEntityÏ;
      },
      deps: ['modelName'],
    },
  ],
  meta: {
    accessType: 'EXECUTE',
    isStatic: false,
    description: 'Get some stuff from a models remote method',
    accepts: [
      {
        arg: 'accessToken',
        type: 'any',
        http: function(ctx) {
          return ctx.req.accessToken;
        },
      },
    ],
    returns: { arg: 'res', type: 'stuff', root: true },
    http: { path: '/get-stuff', verb: 'get' },
  },
})
export class GetStuffRemote {
  constructor(public app, public instance, public customEntity) {}
  async onRemote($accessToken) {
    // This is where you put the remote method logic
  }
}
```

# Example ModelEvent

A basic controller:

```ts
import {ModelEvent} from 'loopback-decorators';

@ModelEvent({
  <!-- the loopback event -->
  selector: 'create',
  providers: [
    '$app', '$model', '^User'
    }
  ],
})
export class DoSomethingOnCreate {
  constructor(public app, public Model, public User) {}
  async onEvent(inst) {
    // This is where you put the event method logic
  }
}

```

# Setting up your remote module

```ts
import { RemoteMethodModule } from 'loopback-decorators';

@RemoteMethodModule({
  remotes: [GetStuffRemote],
  events: [DoSomethingOnCreate],
  proxyFor: 'ModelInternal',
  proxyMethods: ['find', 'findById'],
})
export class ModelAPI {
  constructor(public Model: any) {}
}

export = function(Model: any) {
  return new ModelAPI(Model);
};
```

# Validating Remote Inputs

```ts
@RemoteMethod({
  // ...
  meta: {
    accessType: 'WRITE',
    isStatic: false,
    description: 'Create a thing',
    accepts: [
      {
        arg: 'payload',
        type: 'RequestModelType',
        http: { source: 'body' },
      },
    ],
    returns: { arg: 'res', type: 'stuff', root: true },
    http: { path: '/things', verb: 'post' },
  },
})
export class GetStuffRemote {
  constructor(public app, public instance, public customEntity) {}
  async onRemote(@Validate payload) {
    // Error will be thrown before `onRemote` is called if the payload is not valid
  }
}
```

# Returning an instance of another model as a response

```ts
export class GetStuffRemote {
  constructor(public app, public instance, public customEntity) {}

  @Response('ModelContructorName')
  async onRemote(payload: any) {
    // Result returned will be an instance of app.models.ModelContructorName
  }
}
```

or, if you are returning an array

```ts
export class GetStuffRemote {
  constructor(public app, public instance, public customEntity) {}

  @Response(['ModelContructorName'])
  async onRemote(payload: any) {
    // Result returned will be an array of app.models.ModelContructorName
  }
}
```

You can also pass the config directly:

```ts
  @Response({ responseClass: 'MyResponseClass' })
  // Or, for an array
  @Response({ responseClass: 'MyResponseClass', isMulti: true })
```

# License

MIT
