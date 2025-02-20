# Pistolet

[![Build Status](https://travis-ci.org/FrenchHipster/pistolet.svg?branch=master)](https://travis-ci.org/FrenchHipster/pistolet)
[![npm version](https://badge.fury.io/js/pistolet.svg)](https://badge.fury.io/js/pistolet)
[![lerna](https://img.shields.io/badge/maintained%20with-lerna-cc00ff.svg)](https://lernajs.io/)

Pistolet (pronounced pistol-eh) is a Javascript testing tool to create mock API responses.


## Installation

    $ npm install pistolet --save-dev

### Available Plugins

Pistolet does not come with any integration out-of-the-box, but has plugins to do so:

* [Angular](https://www.npmjs.com/package/pistolet-angular) unit tests
* [Express](https://www.npmjs.com/package/pistolet-express) backend


## General Usage

```javascript
describe('your test suite', () => {
  beforeAll(() => {
    // Start Pisolet, once per test suite
    server = new Pistolet([
      'scenario/from/json/file',
      new ClassScenario(),
      objectScenario
    ]);
  });

  // Required, otherwise the server keeps running and occupies the TCP port
  afterAll(() => server.stop());

  // Clears the request history, and resets scenarios (in case they are stateful)
  afterEach(() => server.reset());
});
```

**Note:** This example uses Jasmine, although this is not a requirement.  
You are more than welcome to use it with other test suites, and contributing examples would be greatly appreciated.


### Override existing scenarios

Pistolet allows to override the scenarios previously defined.  
These overrides have a shorter lifespan and will be discarded after calling `reset()`.

```javascript
beforeAll(() => server = new Pistolet(['default/scenario']));

it('should test an edge case', () => {
  server.override('additional/scenario');
});
```


## Writing scenarios

### JSON Files

JSON scenarios are the easiest to write and use.

Make sure to set the `dir` property in the configuration for Pistolet to know where to find these files.

```json
{
  "name": "The name is purely optional, for developers convenience",
  "request": {
    "method": "GET",
    "path": "/your/api/path"
  },
  "response": {
    "status": 200,
    "data": {
      "some": "response"
    }
  }
}
```

### Javascript/Typescript Scenarios

```typescript
import { Mock, Request, Response, Scenario } from 'pistolet';

export const SampleObjectScenario: Scenario = {
  mocks: [/* ... */],
  next(request: Request, response: Response, match: Mock) {
    // Simply accept and send the mock
    response.status(200).send(match);
    return true;
  }
};

export class SampleClassScenario implements Scenario {
  mocks: Mock[] = [/* ... */];
  next(request: Request, response: Response, match: Mock) {
    // Wait a second, and fail
    setTimeout(() => {
      response.status(503).send({ errorMessage: 'Some Error' });
    }, 1000);
    return true;
  }
}
```

At the end of `next()`, you can return:

* A `Mock` object (same type as the `match` parameter), which will be sent immediately
* A `Promise<Mock>` object, which will be sent when the promise resolves
* `true`, to indicate the scenario has a custom logic and will handle the response
* `false` or `undefined`, to indicate that there was no match

### JSON format

#### URL vs path & query

URL matching will perform a simple string comparison:

```json
{
  "method": "GET",
  "url": "/api/search?q=criteria"
}
```

You can also use `path` and `query` to the same effect:

```json
{
  "method": "GET",
  "path": "/api/search",
  "query": {
    "q": "criteria"
  }
}
```

Using query parameters will perform an object comparison, which is not sensitive to the order in which the parameters come in the URL.

Query parameters can also use regular expressions:

```json
{
  "method": "GET",
  "path": "/api/search",
  "query": {
    "q": "/\\w+/g"
  }
}
```

#### Delayed responses

Pistolet will return the reponse after `delay` milliseconds if the property is present:
```json
{
  "response": {
    "delay": 200,
    "data": { "delayed": "response", "after": "200ms" }
  }
}
``` 
