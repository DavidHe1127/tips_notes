## Node

* Internals
  * [Node internals](./node_internals.md)
  * [nextTick vs setImmediate](./nexttick_vs_setimmediate.md)
* Fundamental
  * [Scaling and load balancing](./scaling_load_balancing.md)
  * [Streams](./stream.md)
  * [HTTP Agent](./http_agent.md)
* Miscellaneous
  * [Module Loading](#module-loading)
  * [Exec script from command line](#exec-script-from-command-line)
  * [Non-blocking intensive computation with Fork](#non-blocking-computation-with-fork)
* Best Practice
  * [Production Deployment](./production_deployment_tips.md)
  * [Graceful shutdown](./graceful_shutdown.md)
  * [Manage Environment Variables](#manage-env-vars)
  * [Hoisting import](#hoisting-import)
  * [logging](#logging)
  * [Error handling](./error_handling.md)

### module-loading
```javascript
const foo = require('./foo');

// first look for foo.js in current directory
// if not found, search for index.js in ./foo/index.js
// or else throws out error 'module not found'
```

### manage-env-vars
* Use [dotenv](https://github.com/motdotla/dotenv) to load env vars for development **ONLY**. In other words, there should be only one `.env` rather than multiple ones - `.env.development, .env.test`.
  ```js
  if (process.env.NODE_ENV !== 'production') {
    require('dotenv').config();
  }
  ```
  Ideally, env-specific vars should be set by ci/cd pipeline/tool.
  ![env_config.png](./env_config.png)
* Have a `.env.example` file as a base to unfold potential params for other team members to share.
* `.env` needs to be shortlisted in `.gitignore`.

 [Good readings](https://www.twilio.com/blog/2017/08/working-with-environment-variables-in-node-js.html)

### hoisting-import
Import will be hoisted during evaluations!

Entry
```js
import dotenv from 'dotenv';

if (process.env.NODE_ENV !== 'production') {
  dotenv.config();
  console.log(11);
}

import firebase from './firebase';
```

module firebase
```js
console.log(1333)
const firebase = 12;

export default firebase;
```
Execution will print `1333` followed by `11`.

### logging

The logging output target should always be the standard output/error. It is not the responsibility of the application to route logs.

### exec_script_from_command_line

```shell
$ node -e 'require("./db").init()'

// or in npm script
"go": "node -e 'require(\"shelljs\")'",
```

### Non-blocking Computation with Fork

```js
// compute.js
const longComputation = () => {
  let sum = 0;
  for (let i = 0; i < 1e9; i++) {
    sum += i;
  };
  return sum;
};

process.on('message', message => {
  const result = longComputation();
  process.send(result);
});

// main.js
const http = require('http');
const { fork } = require('child_process');

const server = http.createServer();

server.on('request', (request, res) => {
  if (request.url === '/compute') {
    const compute = fork('compute.js');
    compute.send('start');

    compute.on('message', result => {
      res.end(`Long computation result: ${result}`)
    });
  } else {
    res.end('Route not found')
  }
});

server.listen(3000);
```
