We acknowledge that this resource is made possible by Professor Barbara Lerner, Mount Holyoke College.


1.  To create a simple Web page that displays Hello, World, create a file named HelloWorld1.html that contains:

<!DOCTYPE HTML>
<html>

<body>
  <!-- Basic HTML -->
  <p>Hello world!</p>
</body>

</html>

Then, load this page into a browser using a File Open command.  You should see a Web page that just contains Hello world!

2. To add some javascript to the page, change the HTML to:

<!DOCTYPE HTML>
<html>

<body>
  <!-- Basic HTML -->
  <p>Using a script file</p>

  <p>Before the script...</p>

  <!-- Run a script -->
  <script src="HelloWorld.js"></script>

  <p>...After the script.</p>

</body>

</html>

Put the following in HelloWorld.js:

alert( 'Hello, world from a script file!' );

Now, a pop-up box should say Hello World.

## Hello World Server
1. Create a server that returns Hello, world

a. Install node.js from https://nodejs.org/

b. Put this in a file called HelloServer.js:

var http = require('http');

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.end('Hello World!');
}).listen(8080);

c. Start the server by typing this into a Terminal / Shell:  node HelloServer.js

d. In a browser, enter the following URL:   http://localhost:8080

You should get a page that says Hello World.

2.  Get the client to talk to the Web server.  Add this line to your html file:

 <a href="http://localhost:8080">Say hi to the server</a>

Now, if you reload the html page and click on the link, it should take you to the page that displays hello world.

3.  Have the server say hello in different languages based on user action.

a. Modify the html file to have several links with different languages.  The link will indicate which language to use like this:  http://localhost:8080?lang=english

b. Modify HelloServer.js to check which language was requested:

// Parses the URL to return a different result based on the language

var http = require('http');
var url = require('url');

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  var q = url.parse(req.url, true).query;
  console.log(q.lang);
  if (q.lang == 'french') {
     res.end ('Bonjour monde!');
  }
  else if (q.lang == 'chinese') {
     res.end ('Ni hao shijie');
  }
  else {
     res.end('Hello World!');
  }
}).listen(8080);

c.  Restart the server and load your html page.  Depending on which link the user clicks on, they should see a different page.

## Hello World MongoDB

1. Put Hello World into the database and retrieve it.

a. Install MongoDB.  Look online for instructions as they vary based on platform.  If you are on a Mac, I recommend installing homebrew and using that to install MongoDB.

b. Start MongoDB.  Again, look online.  The instructions vary by platform.

c. Start the Mongo shell.

d.  Add Hello World to the db with key english:

db.inventory.insertOne(
      {item: "english", value: "hello, world"}
    )

e. Look up hello world.

 db.inventory.find ({ item: "english"})

2.  Change the server to look up how to say hello world.

a.  Add more entries to the database for other languages.

b. Change HelloServer.js to use the database:

// Parses the URL to return a different result based on the language
// Looks up the value in the mongo db

var http = require('http');
var url = require('url');
var mongo = require('mongodb');
var MongoClient = mongo.MongoClient;
var mongoURL = "mongodb://localhost:27017/test";

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  var q = url.parse(req.url, true).query;
  console.log(q.lang);

  // Connect to the MongoDB
  MongoClient.connect(mongoURL, function(err, db) {
    if (err) throw err;
    console.log ("Looking up ", q.lang);
    db.db("test").collection("inventory").find ({item: q.lang}).toArray(function (err, result) {
      if (err) throw err;
      console.log (result)
      if (result[0] == undefined) {
        res.end ("Unknown language")
      }
      else {
        res.end (result[0].value)
      }
      db.close();
    });
  });

}).listen(8080);

## Hello World MariaDB

1. Put Hello World into the database and retrieve it.

a. Install MariaDB.  Look online for instructions as they vary based on platform.  If you are on a Mac, I recommend installing homebrew and using that to install MariaDB.

b. Start MariaDB.  Again, look online.  The instructions vary by platform.

c. Start the mysql shell.

d.  Add Hello World to the db:

use test;

create table hello (
    Language VARCHAR(100) NOT NUll PRIMARY KEY,
    Hello VARCHAR(100) NOT NULL);

insert into hello (Language,Hello)
    -> values('English','Hello, world'),
    -> ('French','Bonjour, monde'),
    -> ('Chinese','Ni hao, shijie');


e. Look up hello world for English.

select Hello from hello where Language='English';

2.  Change the server to look up how to say hello world.

a.  Change HelloServer.js to use the database:

// Parses the URL to return a different result based on the language
// Looks up the value in the Maria db

var http = require('http');
var url = require('url');

var mariadb = require('mariadb');

var pool = mariadb.createPool({
  host: "localhost",
  user: "blerner",
  password: "cs316",
  database: "test"
});

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  var q = url.parse(req.url, true).query;
  console.log(q.lang);

  pool.getConnection ()
    .then (con => {
      console.log ("Connected!");

      sql = "select Hello from hello where Language = '" + q.lang + "'";

      con.query (sql)
        .then ((rows) => {
          console.log (rows);
        console.log (rows[0]);
      console.log (rows[0].Hello);
      res.end (rows[0].Hello)
      con.end();
        })
    }).catch (err => {
      con.end();
      throw err;
    });

}).listen(8080);

## Hello World AJAX
HelloWorld5.html:

<!DOCTYPE HTML>
<html>

<body>

  <p>Select your language:</p>
  <form>
    <input type="radio" name="language" value="chinese"> Chinese
    <input type="radio" name="language" value="english"> English
    <input type="radio" name="language" value="french"> French

    <p><input type="button" id="btn" value="Say hi!"></p>
  </form>

  <div id="hello"></div>

  <script>
     // Get the button
     const btn = document.querySelector('#btn');

     // Attach a listener to the button.
     btn.onclick = function () {
     // Get the radio buttons.
         const rbs = document.querySelectorAll('input[name="language"]');

     // Check the radio buttons to see which is clicked.
         let selectedValue;
         for (const rb of rbs) {
              if (rb.checked) {
                  selectedValue = rb.value;
                  break;
              }
         }

     // Display a pop-up message
         getHelloFromServer(selectedValue);
    };

    // Holds the request and response
    var ajaxRequest;

    // Called when the user clicks the Say hi button.
    // It creates the request and sends it to the server.
    function getHelloFromServer(language) {
    ajaxRequest = new XMLHttpRequest();

    // Register a callback function for when the response arrives.
    ajaxRequest.onreadystatechange = displayHello;

    // Put the GET URL into the request
    ajaxRequest.open("GET", "http://localhost:8080?lang=" + language, true);

    // Send it to the server
    ajaxRequest.send();
    }

    // This function is called whenever the state of the request
    // changes.  The only state change that results in any action is
    // when a response is received.  At that point, it will display
    // the hello translation in the original web page.
    function displayHello() {
    // readyState == 4 means that the response has arrived
    // status == 200 means the server response is OK
    if (ajaxRequest.readyState == 4 && ajaxRequest.status == 200)
    {
        // Get the text that was in the response.  This will be
        // "hello, world" in the desired language.
        var helloText = ajaxRequest.responseText;

        // Create the node to put into the document containing the
        // desired text.
        var helloBody = document.createTextNode (helloText);

        // Find the portion of the Web page where you want to put
        // the translation.
        var helloElement = document.getElementById("hello");

        // If there was already a translation of hello there,
        // replace it.  Otherwise, add it.
        if (helloElement.childNodes[0]) {
        helloElement.replaceChild (helloBody,
                       helloElement.childNodes[0]);
        }
        else {
        helloElement.appendChild(helloBody);
        }
    }
    }

  </script>

</body>

</html>

helloServer5.js:

// Parses the URL to return a different result based on the language
// Looks up the value in the mongo db
// Load the necessary modules
var http = require('http');
var url = require('url');
const fs = require('fs');
var mongo = require('mongodb');

// Initialize Mongo info
var MongoClient = mongo.MongoClient;
var mongoURL = "mongodb://localhost:27017/test";

// Create the server, telling it the name of the function to call
// when a request arrives.
var server = http.createServer(handleRequest);

// Get it listening to port 8080.
server.listen(8080);

// This is called whenever a request arrives at the server.
// req contains the request.
// res is the object that will contain the response.
function handleRequest (req, res) {
  // Initialize the response to be html
  res.writeHead(200, {'Content-Type': 'text/html'});

  // Parse the query.
  var q = url.parse(req.url, true).query;
  console.log(q);
  console.log(q.lang);

  // No language requested.  Serve the hello world Web page.
  if (q.lang == undefined) {
       fs.createReadStream('HelloWorld5.html').pipe(res)
  }

  // Language query provided.  
  else {
    // Connect to the database.  Pass in a function to call when the
    // connection completes.  
    MongoClient.connect(mongoURL,
            function (err, db) {
               if (err) throw err;
               dbLookup (db, q.lang, res)
            });
  }
}

// Look up the language in the database.
function dbLookup (db, lang, response) {
  console.log ("Looking up ", lang);

  // Find all items in the inventory collection of the test database
  // where the item field is the desired language.  We are expecting
  // there to be at most one match.
  var items = db.db("test").collection("inventory").find ({item: lang})

  // Convert the returned items to an array.  On completion, the
  // response is returned.
  items.toArray(function (err, result) {
              if (err) throw err;
            returnResponse(response, result);
            db.close();
        });
}

// Put the value found in the database into the response.
function returnResponse (response, result) {
    console.log (result)
    if (result[0] == undefined) {
      response.end ("Unknown language")
    }
    else {
      response.end (result[0].value)
    }
}

## Hello World JSON
This example builds on the AJAX example to send an array of records, where each record has 2 fields representing a langauge name, and how to say hello in that language.

<!DOCTYPE HTML>
<html>

<body>

  <button type="button" id="sayHiBtn">Say hi in all langauges!</button>

  <div id="hello"></div>

  <script>
     // Get the button
     const btn = document.querySelector('#sayHiBtn');

     // Attach a listener to the button.
     btn.onclick = function () {
         getHelloFromServer();
    };

    // Holds the request and response
    var ajaxRequest;

    // Called when the user clicks the Say hi button.
    // It creates the request and sends it to the server.
    function getHelloFromServer(language) {
    ajaxRequest = new XMLHttpRequest();

    // Register a callback function for when the response arrives.
    ajaxRequest.onreadystatechange = displayHello;

    // Put the GET URL into the request
    ajaxRequest.open("GET", "http://localhost:8080?lang=all", true);

    // Send it to the server
    ajaxRequest.send();
    }

    // This function is called whenever the state of the request
    // changes.  The only state change that results in any action is
    // when a response is received.  At that point, it will display
    // the hello translation in the original web page.
    function displayHello() {
    // readyState == 4 means that the response has arrived
    // status == 200 means the server response is OK
    if (ajaxRequest.readyState == 4 && ajaxRequest.status == 200)
    {
        // Get the text that was in the response.  This will in
        // JSON.  When parsed, we expect to hae an array of objects.
        // The item field of the object is the language name and the
        // value field of the object is how to say hello in that language.
        var helloResponse = JSON.parse(ajaxRequest.responseText);

        // Find the portion of the Web page where you want to put
        // the translation.
        var helloElement = document.getElementById("hello");

        // If there is no translation there yet, add it.
        if (helloElement.childNodes[0] == null) {
          // For each entry in the array, build the document elements
          // for saying hello in that language.
          for (i = 0; i < helloResponse.length; i++) {
            var helloText = "In " + helloResponse[i].item + ", say " +
                        helloResponse[i].value;
               // Create the node to put into the document containing the
              // desired text.
              var helloBody = document.createTextNode (helloText);

              // Put the node inside a <p> element
             var innerElement = document.createElement("p");
             innerElement.appendChild (helloBody);

                // Put the <p> element into the existing div.
               helloElement.appendChild(innerElement);

            }
          }
      }
    }

  </script>

</body>

</html>

HelloJsonServer.js:

// Parses the URL to return a different result based on the language
// Looks up the value in the mongo db

var http = require('http');
var url = require('url');
var mongo = require('mongodb');
var MongoClient = mongo.MongoClient;
var mongoURL = "mongodb://localhost:27017/test";
const fs = require('fs');

http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'});
  var parsedURL = url.parse(req.url, true);
  var path = url.parse(req.url, true).pathname;
  var q = parsedURL.query;

  console.log(path);
  console.log(q);

  // If there is no lang query, serve the html page.
  if (q.lang == undefined) {
      var filepath = __dirname + path;
      fs.stat(filepath, function (err, stat) {
          if (err == null) {
              if (filepath.endsWith (".html")) {
                  // Initialize the response to be html
                  res.writeHead(200, {'Content-Type': 'text/html'});
                  fs.createReadStream(filepath).pipe(res)
          }
          }
      });
  }

  else {
      // Connect to the MongoDB.  In this example, we are ignoring
      // the value of lang from the query and serving all languages
      // to demonstrate JSON.
      MongoClient.connect(mongoURL, function(err, db) {
        if (err) throw err;
        console.log ("Looking up all languages");
        db.db("test").collection("inventory").find ().toArray(function (err, result) {
          if (err) throw err;
          console.log (result)

          // Turn the result returned by the database into a JSON string.
          jsonResult = JSON.stringify(result);
          console.log(jsonResult);

          // Put the JSON into the response and send it.
          res.end (jsonResult);
          db.close();
        });
      });
  }

}).listen(8080);


## JSON and Javascript Classes

class User {

  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  fromJson(json){
    this.firstName = json.firstName;
    this.lastName = json.lastName;
  }

  sayHi() {
    console.log("Hi " + this.firstName + "!");
  }

}

// Construct a User using the class
let user = new User("John", "Doe");
user.sayHi();
console.log ("\nThe user object as a class instance.");
console.log (user);

// Turn the user object into JSON
var jsonUser = JSON.stringify (user);
console.log ("\nThe user object as a JSON string.");
console.log (jsonUser);

// Parse the JSON to get back a Javascript object
var user2 = JSON.parse(jsonUser);
console.log ("\nA Javascript object derived from the JSON.");
console.log (user2);

// Construct a new object using the class and populate its fields
// based on the JSON.
var user3 = new User();
user3.fromJson(user2);
console.log ("\nA new class instance based on the Javascript object.");
console.log (user3);
user3.sayHi();
