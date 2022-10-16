Simple Book Register web form using MEAN stack
The MEAN stack is a full-stack, JavaScript-based framework for developing web applications. MEAN stands for MongoDB, Express, Angular & Node, after the four key technologies that make up the different layers of the stack.
MongoDB - document database
Express(.js) - Node.js web framework
Angular(.js) - a client-side JavaScript framework
Node(.js) - the premier JavaScript web server
There are variations to the MEAN stack such as MERN (replacing Angular.js with React.js) and MEVN (replacing Angular.js with Vue.js), but regardless of which client-side framework is your favorite, MEAN is the ideal JavaScript stack, from top to bottom.
The MEAN architecture is designed to make building web applications in JavaScript, and handling JSON, incredibly easy.

Angular.js Front End
At the very top of the MEAN stack is Angular.js, the self-styled “Superheroic JavaScript MVW Framework” (MVW stands for “Model View and Whatever”).
Angular.js allows you to extend your HTML tags with metadata in order to create dynamic, interactive web experiences much more powerfully than, say, building them yourself with static HTML and JavaScript (or jQuery).
Angular has all of the bells and whistles you’d expect from a front-end JavaScript framework, including form validation, localization, and communication with your back-end service. So how do you build the service it talks to?

Express.js and Node.js Server Tier
The next level down is Express.js, running on a Node.js server. Express.js calls itself a “fast, unopinionated, minimalist web framework for Node.js,” and that is indeed exactly what it is.
Express.js has powerful models for URL routing (matching an incoming URL with a server function), and handling HTTP requests and responses. By making XML HTTP Requests (XHRs) or GETs or POSTs from your Angular.js front-end, you can connect to Express.js functions that power your application.
Those functions in turn use MongoDB’s Node.js drivers, either via callbacks for using Promises, to access and update data in your MongoDB database.

MongoDB Database Tier
If your application stores any data (user profiles, content, comments, uploads, events, etc.), then you’re going to want a database that’s just as easy to work with as Angular, Express, and Node.
That’s where MongoDB comes in: JSON documents created in your Angular.js front end can be sent to the Express.js server, where they can be processed and (assuming they’re valid) stored directly in MongoDB for later retrieval.

This project demonstrates how to build a simple Book Register web form using MEAN stack.

Prerequisites
Ubuntu Server 20.04 LTS (this project is executed on a Microsoft Azure Ubuntu VM)
Putty (for Windows OS)
.ppk private key file to connect to the Ubuntu Server (The .ppk key is gotten from the conversion of the .pem file created when the Ubuntu VM was created. The .pem file is converted to .ppk using Puttygen).

Step 1 - Install NodeJs
Update Ubuntu
$ sudo apt update
Upgrade Ubuntu
$ sudo apt upgrade
Get the location of Node.js software from Ubuntu repositories
$ curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
Install Node.js on the server
$ sudo apt-get install -y nodejs
Note that the command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.
Verify the node installation with the command below
$ node -v 
Verify the node installation with the command below
$ npm -v


Step 2 - Install MongoDB
MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For this application, book records to MongoDB that contain book name, isbn number, author, and number of pages are added.
Import the public key used by the package management system
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
Create a list file for MongoDB
$ echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

Install MongoDB
$ sudo apt install -y mongodb
Start the server
$ sudo service mongodb start
Verify that the service is up and running
$ sudo systemctl status mongodb


Install ‘body-parser’ package. The ‘body-parser’ package helps to process JSON files passed in requests to the server.
$ sudo npm install body-parser
Create a folder named ‘Books’ and cd into it
$ mkdir Books && cd Books
Create a file in it named server.js and open it with a text editor
$ sudo nano server.js
Copy and paste the web server code below into the server.js file
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});



Step 3 - Install Express and set up routes to the server
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. Express will be used to pass book information to and from the MongoDB database.
Mongoose package will also be used. This provides a straight-forward, schema-based solution to model the application data. It will be used to establish a schema for the database to store data of the book register.
Install ExpressJs and Mongoose
$ sudo npm install express mongoose
In ‘Books’ folder, create a folder named apps and cd into it
$ mkdir apps && cd apps
Create a file in it named routes.js and open it with a text editor
$ sudo nano routes.js
Copy and paste the web server code below into the routes.js file
     var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};

In the ‘apps’ folder, create a folder named models and cd into it
$ mkdir models && cd models
Create a file in it named book.js and open it with a text editor
$ sudo nano book.js
Copy and paste the code below into book.js file.
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);

Step 4 - Access the routes with AngularJS
AngularJS provides a web framework for creating dynamic views in web applications. In this project, AngularJS is used to connect the web page with Express and perform actions on the book register.
Change the directory back to ‘Books’
$ cd ../..
Create a folder named public and cd into it
$ mkdir public && cd public
Create a file in it named script.js and open it with a text editor
$ sudo nano script.js
Copy and paste the Code below (controller configuration defined) into the script.js file
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});

In ‘public’ folder, create a file in it named index.html and open it
$ sudo nano index.html
Copy and paste the code below into index.html file.
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

Change the directory back up to ‘Books’
$ cd ..
Start the server by running this command:
$ node server.js
The server is now up and running, we can connect it via port 3300:


Add an inbound security rule to the Network Security Group of the VM to open inbound connection through port 3300:


Access the Book Register web application from the Internet with a browser using the VM’s Public IP address or Public DNS name, followed by port 3300:
http://<PublicIP-or-PublicDNS>:3300
The Book Register Application looks like in a browser:

As books are registered and deleted, there will be activities in the running ExpresJS server as seen below:


Conclusion
If you’re building a JavaScript application, particularly in Node.js, then you should give MEAN a serious look. Whether you’re building a high-throughput API, a simple web application, or a microservice, MEAN is the ideal stack for building Node.js applications.
While MEAN is particularly suited to real-time applications, particularly those running natively in the cloud, and single-page (dynamic) web applications built in Angular.js, it can be used many ways:
Workflow management tools
News aggregation sites
Todo and Calendar applications
Interactive forums
And much much more.

Credits
https://www.mongodb.com/mean-stack
https://darey.io
