# api documentation for  [portscanner (v2.1.1)](https://github.com/baalexander/node-portscanner)  [![npm package](https://img.shields.io/npm/v/npmdoc-portscanner.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-portscanner) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-portscanner.svg)](https://travis-ci.org/npmdoc/node-npmdoc-portscanner)
#### Asynchronous port scanner for Node.js

[![NPM](https://nodei.co/npm/portscanner.png?downloads=true)](https://www.npmjs.com/package/portscanner)

[![apidoc](https://npmdoc.github.io/node-npmdoc-portscanner/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-portscanner_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-portscanner/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-portscanner/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-portscanner/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": "",
    "bugs": {
        "url": "https://github.com/baalexander/node-portscanner/issues"
    },
    "dependencies": {
        "async": "1.5.2",
        "is-number-like": "^1.0.3"
    },
    "description": "Asynchronous port scanner for Node.js",
    "devDependencies": {
        "ava": "^0.4.2",
        "eslint": "^3.10.2",
        "eslint-config-standard": "^6.2.1",
        "standard": "^8.5.0"
    },
    "directories": {
        "lib": "./lib"
    },
    "dist": {
        "shasum": "eabb409e4de24950f5a2a516d35ae769343fbb96",
        "tarball": "https://registry.npmjs.org/portscanner/-/portscanner-2.1.1.tgz"
    },
    "engines": {
        "node": ">=0.4",
        "npm": ">=1.0.0"
    },
    "gitHead": "47889e0c6a4ef449420e90eb59a5100a11eab6db",
    "homepage": "https://github.com/baalexander/node-portscanner",
    "keywords": [
        "portscanner",
        "port",
        "scanner",
        "checker",
        "status"
    ],
    "license": "MIT",
    "main": "./lib/portscanner.js",
    "maintainers": [
        {
            "name": "baalexander",
            "email": "baalexander@gmail.com"
        },
        {
            "name": "laggingreflex",
            "email": "laggingreflex@gmail.com"
        },
        {
            "name": "shinnn",
            "email": "snnskwtnb@gmail.com"
        },
        {
            "name": "smassa",
            "email": "endangeredmassa@gmail.com"
        }
    ],
    "name": "portscanner",
    "optionalDependencies": {},
    "preferGlobal": false,
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/baalexander/node-portscanner.git"
    },
    "scripts": {
        "lint": "standard",
        "test": "ava"
    },
    "version": "2.1.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module portscanner](#apidoc.module.portscanner)
1.  [function <span class="apidocSignatureSpan">portscanner.</span>checkPortStatus (port)](#apidoc.element.portscanner.checkPortStatus)
1.  [function <span class="apidocSignatureSpan">portscanner.</span>findAPortInUse ()](#apidoc.element.portscanner.findAPortInUse)
1.  [function <span class="apidocSignatureSpan">portscanner.</span>findAPortNotInUse ()](#apidoc.element.portscanner.findAPortNotInUse)



# <a name="apidoc.module.portscanner"></a>[module portscanner](#apidoc.module.portscanner)

#### <a name="apidoc.element.portscanner.checkPortStatus"></a>[function <span class="apidocSignatureSpan">portscanner.</span>checkPortStatus (port)](#apidoc.element.portscanner.checkPortStatus)
- description and source-code
```javascript
function checkPortStatus(port) {
  var args, host, opts, callback

  args = [].slice.call(arguments, 1)

  if (typeof args[0] === 'string') {
    host = args[0]
  } else if (typeof args[0] === 'object') {
    opts = args[0]
  } else if (typeof args[0] === 'function') {
    callback = args[0]
  }

  if (typeof args[1] === 'object') {
    opts = args[1]
  } else if (typeof args[1] === 'function') {
    callback = args[1]
  }

  if (typeof args[2] === 'function') {
    callback = args[2]
  }

  if (!callback) return promisify(checkPortStatus, arguments)

  opts = opts || {}

  host = host || opts.host || '127.0.0.1'

  var timeout = opts.timeout || 400
  var connectionRefused = false

  var socket = new Socket()
  var status = null
  var error = null

  // Socket connection established, port is open
  socket.on('connect', function () {
    status = 'open'
    socket.destroy()
  })

  // If no response, assume port is not listening
  socket.setTimeout(timeout)
  socket.on('timeout', function () {
    status = 'closed'
    error = new Error('Timeout (' + timeout + 'ms) occurred waiting for ' + host + ':' + port + ' to be available')
    socket.destroy()
  })

  // Assuming the port is not open if an error. May need to refine based on
  // exception
  socket.on('error', function (exception) {
    if (exception.code !== 'ECONNREFUSED') {
      error = exception
    } else {
      connectionRefused = true
    }
    status = 'closed'
  })

  // Return after the socket has closed
  socket.on('close', function (exception) {
    if (exception && !connectionRefused) { error = error || exception } else { error = null }
    callback(error, status)
  })

  socket.connect(port, host)
}
```
- example usage
```shell
...

A brief example:

'''javascript
var portscanner = require('portscanner')

// Checks the status of a single port
portscanner.checkPortStatus(3000, '127.0.0.1', function(error, status) {
  // Status is 'open' if currently in use or 'closed' if available
  console.log(status)
})

// Find the first available port. Asynchronously checks, so first port
// determined as available is returned.
portscanner.findAPortNotInUse(3000, 3010, '127.0.0.1', function(error, port) {
...
```

#### <a name="apidoc.element.portscanner.findAPortInUse"></a>[function <span class="apidocSignatureSpan">portscanner.</span>findAPortInUse ()](#apidoc.element.portscanner.findAPortInUse)
- description and source-code
```javascript
function findAPortInUse() {
  var params = [].slice.call(arguments)
  params.unshift('open')
  return findAPortWithStatus.apply(null, params)
}
```
- example usage
```shell
...
// determined as available is returned.
portscanner.findAPortNotInUse(3000, 3010, '127.0.0.1', function(error, port) {
  console.log('AVAILABLE PORT AT: ' + port)
})

// Find the first port in use or blocked. Asynchronously checks, so first port
// to respond is returned.
portscanner.findAPortInUse(3000, 3010, '127.0.0.1', function(error, port) {
  console.log('PORT IN USE AT: ' + port)
})

// You can also pass array of ports to check
portscanner.findAPortInUse([3000, 3005, 3006], '127.0.0.1', function(error, port) {
  console.log('PORT IN USE AT: ' + port)
})
...
```

#### <a name="apidoc.element.portscanner.findAPortNotInUse"></a>[function <span class="apidocSignatureSpan">portscanner.</span>findAPortNotInUse ()](#apidoc.element.portscanner.findAPortNotInUse)
- description and source-code
```javascript
function findAPortNotInUse() {
  var params = [].slice.call(arguments)
  params.unshift('closed')
  return findAPortWithStatus.apply(null, params)
}
```
- example usage
```shell
...
portscanner.checkPortStatus(3000, '127.0.0.1', function(error, status) {
// Status is 'open' if currently in use or 'closed' if available
console.log(status)
})

// Find the first available port. Asynchronously checks, so first port
// determined as available is returned.
portscanner.findAPortNotInUse(3000, 3010, '127.0.0.1', function(error, port) {
console.log('AVAILABLE PORT AT: ' + port)
})

// Find the first port in use or blocked. Asynchronously checks, so first port
// to respond is returned.
portscanner.findAPortInUse(3000, 3010, '127.0.0.1', function(error, port) {
console.log('PORT IN USE AT: ' + port)
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
