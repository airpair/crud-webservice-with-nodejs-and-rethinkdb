![nodejs](http://roshiro.github.io/cdn/images/nodejs.jpg)

I have started the habit of looking for trending projects on GitHub every morning before starting work and it has been beneficial. I can see what other people are using and interested in. I usually tweet the projects that seem interesting and worth following. If you want to know what I have found so far, particularly projects related to Front-End development, [find me](http://twitter.com/roshiro) on twitter.

__One project that caught my eyes__ recently is [RethinkDB](http://www.rethinkdb.com/), a NoSQL database that stores JSON documents and as they say on their website it has an _"intuitive query language, automatically parallelized queries and simple administration"_. I really like how easy it is to query data and how it handles JSON docs.

I created a simple REST web service using Node.JS and Express to persist schema-free JSON data in RethinkDB so that I can have a taste of how easily I can integrate RethinkDB with javascript. I put [this web service up in GitHub](https://github.com/roshiro/crud_ws), so feel free to clone and play around with it. Below I explain how crud_ws web service was built.

__The crud_ws consists of basically 3 main files__

- package.json - Define the dependencies of the project
- server.js - The controller of the webservice
- modules/crud.js - The logic to persist the data on RethinkDB

####package.json
Here we are defining our dependency on express framework.
```javascript
{
    "name": "crud_ws",
    "description": "CRUD",
    "version": "0.0.1",
    "private": false,
    "dependencies": {
        "express": "3.x"
    }
}
```

####server.js

Here we define the routes and the respective http methods. For each route we say what method to execute.

```javascript
var express = require('express'),
    module = require('./modules/crud'),
    app = express();

app.use(express.bodyParser());
app.get('/cruds', module.findAll);
app.get('/cruds/:id', module.findById);
app.post('/cruds', module.create);
app.delete('/cruds/:id', module.delete);
app.put('/cruds/:id', module.update);

app.listen(3000);
console.log('Listening on port 3000...');
```

###modules/crud.js

Here we open the connection with the database and define all the methods that we call from `server.js`. `findAll()`, `findById()`, `create()`, `update()` and `delete()`.

```javascript
(function() {
  var r = require('rethinkdb'),
      connection;

  r.connect( {host: 'localhost', port: 28015}, function(err, conn) {
    if (err) throw err;
    connection = conn;
    r.db('test').tableCreate('cruds').run(conn, function(err, res) {
      if(err &amp;&amp; err.name === "RqlRuntimeError") console.log("Table already exist. Skipping creation.");
      else {
        console.log(res);
        throw err;
      }
    });
  });

  exports.findAll = function(req, res) {
    r.table('cruds').run(connection, function(err, cursor) {
        if (err) throw err;
        cursor.toArray(function(err, result) {
            if (err) throw err;
            res.send(JSON.stringify(result, null, 2));
        });
    });
  };

  exports.findById = function(req, res) {
    var id = req.params.id;
    r.table('cruds').get(id).
      run(connection, function(err, result) {
          if (err) throw err;
          res.send(JSON.stringify(result, null, 2));
      });
  };

  exports.create = function(req, res) {
    var presentation = req.body;
    console.log("cruds ", JSON.stringify(req.body));
    var presentation = req.body;
    console.log(JSON.stringify(req.body));
    r.table('cruds').insert(presentation).
      run(connection, function(err, result) {
        if (err) throw err;
        res.send(JSON.stringify({status: 'ok', location: '/cruds/'+result.generated_keys[0]}));
      });
  };

  exports.update = function(req, res) {
    var presentation = req.body,
        id = req.params.id;
    r.table('cruds').get(id).update(presentation).
      run(connection, function(err, result) {
        if (err) throw err;
        res.send(JSON.stringify({status: 'ok'}));
      });    
  };

  exports.delete = function(req, res) {
    var id = req.params.id;
    r.table('cruds').get(id).delete().
      run(connection, function(err, result) {
          if (err) throw err;
          res.send(JSON.stringify({status: 'ok'}));
      });
  };
})();
```

To make all this work you need to:

- Start RethinkDB service
```shell
$ rethinkdb
```

- Install dependencies
```shell
$ npm install
```

- Start crud_ws service

```shell
$ node server.js
```

- Done! Test away!

```shell
#Creating
$ curl -X POST -H "Content-Type: application/json" -d '{"title":"Hey, I'm using crud_ws", "slides": [{"1":"test"}, {"2": "Another test"}]}' http://localhost:3000/cruds

#Retrieving
$ curl -i -H "Accept: application/json" http://localhost:3000/cruds

#Updating
$ curl X PUT -i -H "Accept: application/json" -d 'title'='This is the updated title' http://localhost:3000/cruds/23cd6e44-d2b4-47d0-ba87-b788c496c82c

#Deleting
$ curl -X DELETE -i -H "Accept: application/json" http://localhost:3000/cruds/446a7bc3-d54c-4f1b-812f-b8daa9bc2016
```
