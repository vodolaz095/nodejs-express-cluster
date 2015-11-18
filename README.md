# nodejs-express-cluster
Example of usage of native Nodejs cluster with express application


# Code of Example

```javascript

'use strict';


//simple express application
var express = require('express'),
  app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.get('/boom', function (req, res) {
  res.send('As you wish...');
  process.exit(1);
});

//simple net echo server
var net = require('net'),
  server = net.createServer(function (socket) {
    socket.write('Echo server\r\n');
    socket.pipe(socket);
  });

//clustered background process

function doThings(payload) {
  this.payload = payload;
}

doThings.prototype.listen = function (port,callback) {
  //port is ignored(
  var payload = this.payload;
  setInterval(function () {
    console.log('Worker: I am background service with PID#%d doing this:%s', process.pid, payload);
  }, 1000);
  setTimeout(function () {
    console.log('Worker: I am background service with PID#%d doing this:%s. I want to sleep!', process.pid, payload);
    process.exit(1);
  }, 5000);
  process.nextTick(callback);
};

var
  bg1 = new doThings('route1'),
  bg2 = new doThings('route2');



//making function to start cluster
var
  cluster = require('cluster'),
  os = require('os'),
  assert = require('assert');

function startAsCluster(server, port, numberOfWorkers) {
  assert(typeof server.listen === 'function', 'first argument need to have method `listen`!');

  if (cluster.isMaster) {
    var
      numCPUs = Math.min(os.cpus().length, numberOfWorkers);

    for (var i = 0; i < numCPUs; i++) {
      cluster.fork();
    }

    cluster.on('online', function (worker) {
      console.log('Cluster : Worker of %s PID#%d is online!', server.constructor.name, process.pid, worker.process.pid);
    });

    cluster.on('exit', function (worker) {
      console.log('Cluster : Worker #%d died (%d)! Trying to spawn spare one...', worker.process.pid, worker.process.exitCode);
      cluster.fork();
    });

  } else {
    server.listen(port, function () {
      switch (server.constructor.name) {
      case 'EventEmitter':
        console.log('Worker: I am ExpressJS PID#%d listening on port %d', process.pid, port);
        break;
      case 'server':
        console.log('Worker: I am net server PID#%d listening on port %d', process.pid, port);
        break;
      default:
        console.log('Worker: I am %s PID#%d listening on port %d', server.constructor.name, process.pid, port);
      }
    });
  }
}

startAsCluster(app, 3000, 1); //for express application
startAsCluster(server, 3001, 1); //for telnet server


startAsCluster(bg1, 3001, 1); //for background process, port is ignored
startAsCluster(bg2, 3001, 1); //for background process, port is ignored


```


# Usage

```shell

  # npm install
  # npm start

```