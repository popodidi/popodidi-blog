+++
date = "2017-04-13T10:35:41+08:00"
tags = ["javascript", "REST", "documentation"]
description = "Document generation integrated in testing process with duckdoc"
title = "duckdoc - REST API documentation"

+++

REST API documentation is always a nightmare of backend developer. Especially when the project is under development and the API is evolving every build, the complaints of inconsistency between document and realistic from frontend / mobile teammates are endless.

[duckdoc][duckdoc] is a npm package that aims to solve this problem.

----
### Table of contents

- Prepare minimum project with [expressjs][expressjs], [mocha][mocha] and [chai][chaijs]
- Integrate [duckdoc-jsoner][duckdoc-jsoner] in test
- Generate document site with [duckdoc][duckdoc]

### Prerequisite

- Familiar with javascript, npm, backend development, node

### Environment

- Macbook Pro Retina, 13″, Early 2015
- macOS Sierra, 10.12.4
- node v7.7.1
- npm v4.1.2
- express-generator v4.15.0
----

## Introduction

duckdoc consists of [duckdoc][duckdoc] and [duckdoc-jsoner][duckdoc-jsoner].

[duckdoc-jsoner][duckdoc-jsoner] prepares `.json` files of each endpoint for [duckdoc][duckdoc], which parses those files and renders to static  document sites (i.e., `.html`).

## Preparation

First, we have to prepare a node server with a few endpoints and the corresponding tests. In the post, we are using [expressjs][expressjs] and [mocha][mocha] approach.

### Generate express project

With [express-generator][express-generator],

```bash
$ npm install express-generator -g  # install if needed
$ express duckdoc-example           # generate example project
$ cd duckdoc-example && npm install # npm install
$ npm start                         # test
```

You should have the following at http://localhost:3000

```text
Express
Welcom to Express
```

You have it? Nice!

### Create endpoints

Create `/routes/duck.js` with contents
```js
var express = require('express');
var router = express.Router();

// GET /duck
router.get('/', function(req, res, next) {
  var resBody = {
    color: "white",
    sound: "quack",
    canFly: false,
    canSwim: true,
    foots: [
      "left", "right"
    ],
    eyeNumber: 2
  }

  res.status(200).json(resBody);
});

// POST /duck
router.post('/alwaysSuccessPost', function(req, res, next) {
  res.sendStatus(200);
});

module.exports = router;
```

and add the a few lines of codes in `app.js`
```js
// ...
// ...
var users = require('./routes/users');
var duck = require('./routes/duck');    // add this line

// ...
// ...
// ...
app.use('/users', users);
app.use('/duck', duck);                 // add this line

// ...
// ...
```

and run `$ npm start` again.
You can test the endpoints with tools like [Postman][postman]. If everything works, we now have a backend server with `GET /duck` and `POST /duck/alwaysSuccessPost` endpoints.

### Test endpoints with [mocha][mocha] and [chai][chaijs]

install mocha and chai
```bash
$ npm install mocha chai chai-http --save-dev
```

add test script in `package.json`
```js
{
    // ...
    // ...
    "scripts": {
        "start": "node ./bin/www",
        "test": "mocha"
    }
    // ...
    // ...
}
```

create `test/test.js` file
```bash
$ mkdir test && touch test/test.js
```

with contents in `test/test.js`
```js
var chai = require('chai');
var chaiHttp = require('chai-http');
var app = require('../app');
var should = chai.should();
chai.use(chaiHttp)

describe('duckdoc-example', () => {
  describe('/GET duck', () => {
    it('it should GET a duck', (done) => {
      chai.request(app)
        .get('/duck')
        .end((err, res) => {
          res.should.have.status(200);
          done();
        });
    });
  });

  describe('/POST duck/alwaysSuccessPost', () => {
    it('it should always succeed', (done) => {

      let reqBody = {
        "hello": "I am",
        "a": "duck"
      };

      chai.request(app)
        .post('/duck/alwaysSuccessPost')
        .send(reqBody)
        .end((err, res) => {
          res.should.have.status(200);
          done();
        });
    });
  });
});
```

now run
```bash
$ npm test
```

If everything is good, you should have this at your console.
```
duckdoc-example
  /GET duck
    ✓ it should GET a duck (61ms)
  /POST duck/alwaysSuccessPost
    ✓ it should always succeed


2 passing (109ms)

```

Now, we've got a `duck` backend running at `http://localhost:3000` with a few tests that make sure every endpoint return status code `200`. Finally, we can start integrating [duckdoc][duckdoc] to generate document.

## Document generation

### Install
```bash
$ npm install duckdoc duckdoc-jsoner --save-dev
```

### Prepare `.json` files

We have to first prepare `.json` file, which contains information of each endpoint, from each test.


import `duckdoc-jsoner`, set `outputPath`
```js

// ...
// ...
var should = chai.should();
chai.use(chaiHttp);
/**
 * begin
 */

 var path = require('path');
 var jsoner = require('duckdoc-jsoner');
 jsoner.outputPath = path.join(__dirname, '../doc/json');

let api = {
  method: "??",
  url   : "https://??",
  req   : {
    headers: {},
    body   : {}
  },
  res   : {
    status: {
      code   : 10000,
      message: "????"
    },
    body  : {}
  }
};
/**
 * end
 */
// ...
// ...
```

create `.json` after each test is done.
```js
// ...
// ...

describe('duckdoc-example', () => {
  describe('/GET duck', () => {
    it('it should GET a duck', (done) => {
      chai.request(app)
        .get('/duck')
        .end((err, res) => {
          res.should.have.status(200);

          /**
           * begin
           */
          let theApi = Object.assign({}, api, {
            method: "GET",
            url   : "https://localhost:3000/duck",
            req   : {
              headers: res.request.header
            },
            res   : {
              status: {
                code   : res.statusCode,
                message: "OK"
              },
              body  : JSON.stringify(res.body)
            }
          })
          jsoner.createFromAPI(theApi);
          /**
           * end
           */

          done();
        });
    });
  });

  describe('/POST duck/alwaysSuccessPost', () => {
    it('it should always succeed', (done) => {

      let reqBody = {
        "hello": "I am",
        "a": "duck"
      };

      chai.request(app)
        .post('/duck/alwaysSuccessPost')
        .send(reqBody)
        .end((err, res) => {
          res.should.have.status(200);

          /**
           * begin
           */
          let theApi = Object.assign({}, api, {
            method: "POST",
            url   : "https://localhost:3000/duck/alwaysSuccessPost",
            req   : {
              headers: res.request.header,
              body: reqBody
            },
            res   : {
              status: {
                code   : res.statusCode,
                message: "OK"
              },
              body  : JSON.stringify(res.body)
            }
          })
          jsoner.createFromAPI(theApi);
          /**
           * end
           */

          done();
        });
    });
  });
});
```

and run test again
```bash
$ npm test
```

If jsoner is working, it should create some `.json` files at `doc/json/`
```text
doc
- json
    - GET_duck.json
    - POST_duck+alwaysSuccessPost.json
```
### Generate document

add test script in `package.json`
```js
{
    // ...
    // ...
    "scripts": {
        "start": "node ./bin/www",
        "test": "mocha",
        "doc": "duckdoc -o doc/site -p duckdoc-example doc/json"
    }
    // ...
    // ...
}
```

and run
```bash
$ npm run doc
```
<br/>
#### BOOM! Your document site is READY!
Just go checkout `doc/site/index.html` or [Demo][demo].


[demo]: https://popodidi.github.io/duckdoc
[duckdoc]: https://github.com/popodidi/duckdoc
[duckdoc-jsoner]: https://github.com/popodidi/duckdoc-jsoner
[mocha]: https://mochajs.org
[chaijs]: http://chaijs.com/
[expressjs]: http://expressjs.com
[express-generator]: https://expressjs.com/en/starter/generator.html
[postman]: https://www.getpostman.com
