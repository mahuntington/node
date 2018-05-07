# Node.js

## NPM

Whenever creating a new node project, use to set it up

```
npm init
```

[NPM](https://www.npmjs.com/) is a package manager for node.js and js libraries and frameworks.  You can look up packages there, and install them for your project with:

```
npm install package-name --save
```

## module.exports/require

Traditionally, importing JS files into other JS files has not been supported... until now!  There are several ways to do this, but node allows you to add things the to module.exports object.

```javascript
module.exports = 'foo';
```

Whatever is on there, will be the return value of

```javascript
require('yourJSFile.js');
```

You can assign this to a variable to use later

## Instantation

First install express with `npm install express --save` and require it

```javascript
var express = require('express'); //include express package
```

`express` has lots of abilities, but we just want to start up a server app:

```javascript
var app = express(); // create an express app
```

Then make it listen on a port

```javascript
app.listen(3000, function(){ //start the server
	console.log('listening on port ' + PORT);
});
```

## CRUD and HTTP Verbs

In comp sci, there are different kinds of action you can take on data

- create
- read
- update
- destroy

There are different kinds of 'methods' you can use to make a request to a server that map to these actions:

- post (create)
- get (read)
- put/patch (update)
- delete (delete)

PUT is for updating an entire model, PATCH is for changing just a few attributes

## Routing

Basic routing can be done at the app level

```javascript
app.get('/', function(req, res){ // route for /
	res.send('hi'); //respond with 'hi'
});
```

This sets up an event listener for any GET request that comes into the server with the path of `/`.  Give it a callback function which takes a request variable and a respond variable.  Each of these have different methods/properties which we'll learn about

## Middleware

We can define a callback function that is called or all requests and then continues on to other request handlers

```javascript
app.use(function(req, res, next){
	console.log('middleware');
	next();
})
```

`next();` tells express to continue processing the request.  `app.get(),app.post()`, etc have this ability too, but it's rarely used.

Just like with `app.get(),app.post()`, etc we can add the ability to handle just those requests that match a url pattern

```javascript
app.use('/foo', function(req, res, next){
	console.log('middleware');
	next();
})
```

## MVC

One of the main goals of express is to cleanly separate data from the view layer.  To do this, it follows the Model View Controller pattern, which creates a middleman (Controller) which passes data (Models) off to the presentation/html (View) layer

In addition, putting all your routes inside server.js can become difficult to maintain with a lot of variables.  We can group similar urls together into separate "controller" files.

```javascript
var runsController = require('./controllers/runs.js'); //require our own runsController
app.use('/runs/', runsController); //use it for anything starting with /runs
```

Now, create a controllers directory, create an appropriately named file, and use:

```javascript
var controller = require('express').Router(); //require express and create a router (controller)
controller.get('/', function(req, res){ //route for finding all routes by a the session user
	res.send('runs index');
});
module.exports = controller;
```

## Views

Let's separate the views from the controller.  Install express with `npm install ejs --save`

Create a directory called 'views' and create an appropriately named file with the extension .ejs.  Now we can reference it.  Express will assume the file path starts with the 'views' directory you created.

```javascript
var controller = require('express').Router(); //require express and create a router (controller)
controller.get('/', function(req, res){ //route for finding all routes by a the session user
	res.render('viewFile.ejs');
});
module.exports = controller;
```

EJS files are just html, but you can use javascript to dynamically create HTML:

```html
<ul>
	<% for(var i = 0; i < 4; i++) { %>
		<li>
			<%= i %><!-- add = to write a value to the html.  Omit the = and it will just run the JS, but not show anything visually -->
		</li>
	<% } %>
</ul>
```

You can pass data into the view file by adding a second parameter that's an object.  The properties of the object will become the variable name that's accessible in the view file.

```javascript
var controller = require('express').Router(); //require express and create a router (controller)
controller.get('/', function(req, res){ //route for finding all routes by a the session user
	res.render('viewFile.ejs', {
		variable1Name: 'variable 1 value'
	});
});
module.exports = controller;
```

Now inside the .ejs file, we can access `variable1Name` like so:

```html
Value: <%= variable1Name; %>
```

## URL Params/Query Strings

Data can be passed to the server through query strings (a.ka. GET parameters) which look like ?param1=value1&param2=value2

They can be accessed like so:

```javascript
app.get('/', function(req, res){
	var param1 = req.query.param1;
	var param2 = req.query.param2;
});
```

Your route can also have parameters in the path section

```javascript
app.get('/:id', function(req, res){
	var id = req.params.id;
});
```

## Body Parser

We can also pass data to server in the body of the request.

### Form data

```html
<form action="/runs" method="POST">
	<input type="text" name="param1"/>
</form>
```

`npm install body-parser --save` and require it

```javascript
var bodyParser = require('body-parser');
```

Next, tell express to expect form data

```javascript
app.use(bodyParser.urlencoded({ extended: false })) //tell body parser that we'll be passing in form data
```

Then, inside any request handler, we can access the form data, formatted as a JS object:

```javascript
app.post('/run', function(req, res){
	var param1 = req.body.param1;
});
```

### JSON

Install and require `body-parser` as before, but the middleware changes:

```javascript
controller.use(bodyParser.json()); //anything handled by this controller is expecting JSON data, not form data
```

Now instead of using form data, we can use Postman.  Create a new tab, choose method, select "body", choose "raw", click on Text dropdown and choose JSON(application/json).  Make sure Headers have a line for "Content-Type" set to "application/json"

We can capture the `req.body` params the same way as with form data.

## Method Override

Unfortunately, forms can only use methods that are GET or POST.  To fake DELETE, or PUT/PATCH we use the package `method-override`.  Install it using `npm install method-override --save` and require it

```javascript
var methodOverride = require('method-override');
```

Use it:

```javascript
controller.use(methodOverride('_method')); //tell method override to expect ?method=PUT/DELETE attached to POST requests
```

Then in our form we add an extra get param (the param name must match what's passed to the methodOverride function):

```html
<form action="/runs?_method=DELETE" method="POST">
	<input type="text" name="param1"/>
</form>
```

Now the appropriate route handler will handle the request

```javascript
app.delete('/runs', function(req, res){

});
```

## REST

There are seven routes which control basic HTTP operations for data:

https://gist.github.com/alexpchin/09939db6f81d654af06b

| **URL**         |**HTTP Verb**|**Action**|
|-----------------|-------------|----------|
| /photos/        | GET         | index    |
| /photos/new     | GET         | new      |
| /photos         | POST        | create   |
| /photos/:id     | GET         | show     |
| /photos/:id/edit| GET         | edit     |
| /photos/:id     | PATCH/PUT   | update   |
| /photos/:id     | DELETE      | destroy  |

## Session

Cookies are used in browsers to store data on a user's computer.  This way, when the user returns, the site can use that data to customize the app.  Cookies are stored in plain-text though, so typically we use sessions, which are encrypted cookies that only last as long as the user's browser is open.

`npm install express-session --save` and require it:

```javascript
var session = require('express-session'); //include express sessions for session work
```

Now we "use" it:

```javascript
app.use(session({ //setting up session encryption info
	secret: "seakrett", //unique keyword for encrypting session data
	resave: false, // don't resave session if nothing changed
	saveUninitialized: false //even if no data, set a cookie
}));
```

To set data, simply run:

```javascript
req.session.userData = 'some value';
```

To get data, simply run:

```javascript
var userData = req.session.userData;
```

To destroy a session:

```javascript
req.session.destroy(function(){
	//do something once session destroy succeeds
});
```

## Bcrypt

To encrypt information, we `npm install bcrypt --save` and require it:

```javascript
var bcrypt = require('bcrypt');
```

To encrypt a value:

```javascript
var encryptedValue = bcrypt.hashSync('value to encrypt', bcrypt.genSaltSync(10));
```

genSaltSync determines how difficult it is to crack the encryption.  The higher the number, the harder it is, but the longer it takes.  10 is usually good.

Using this technique, the same starting value will not be encrypted the same way twice.  This prevents from a hacker from guessing other users' passwords based on matching encrypted values.  This means that we can never decrypt a salted value.  Instead, we can only compare it with another salted value to see if they so close that they can't be anything but the same.

```javascript
var valuesMatch = bcrypt.compareSync('does salted value match this?', saltedValue);
```

## Static

If we have static files (JS, CSS, HTML, etc), we can put them in a directory used just for these kinds of files.  Use this near the top of server.js:

```javascript
app.use(express.static('public')); //set up a static asset dir in /public
```

Create a directory that matches the name passed to `express.static()`.  Express will try to match requests with files in that directory

- The route `/plain.html` would match the file `/public/plain.html`
- The route `/css/app.css` would match the file `/public/css/app.css`
- The route `/js/app.js` would match the file `/public/js/app.js`

## Database

- Install postgres driver `npm install pg --save`
- Install sequelize package `npm install sequelize --save`

Set up a database connection in `models/db.js`

```javascript
var Sequelize = require('sequelize'); //require sequelize package

var DB_URL = process.env.DATABASE_URL || 'postgres://matthuntington:password@localhost:5432/sedstack'; //use either environment variable or static url

var db = new Sequelize(DB_URL); //create the connection.  Will not run multiple times, due to require cacheing the file

module.exports = db;
```

Now we can create a model:

```javascript
var Sequelize = require('sequelize'); //require sequelize package
var db = require('./db.js'); //require connection to the db

var Runs = db.define('run', { //set up model variables
	date: Sequelize.DATE, //use date data type
	distance: Sequelize.FLOAT, //float for distance
});

db.sync(); //if table does not exist, create it

module.exports = Runs;
```

We can now require our run model elsewhere

```javascript
var Runs = require('./models/runs.js');
```

and create a run (all sequelize db query functions return a promise.  This is an object that has a `then()` function that takes two params, a success callback for when the query succeeds and a fail callback for when it doesn't.  Only the success callback is shown below):

```javascript
Runs.create({
	date: new Date('2016-1-1'),
	distance: 5.5
}).then(function(createdRun){
	//createdRun is the object representation of the row created in the DB
});
```

find all runs matching a criteria (more on querying: http://docs.sequelizejs.com/en/v3/docs/querying/)

```javascript
Runs.findAll({
	where: {
		distance: 5.5
	}
}).then(function(foundRuns){
	//an array of run objects that represent rows in the db
});
```

update all runs matching a criteria

```javascript
Runs.update(
	{
		date: new Date('2017-1-1'),
		distance: 6.1
	}, //change the selected runs to match this object
	{
		where: {
			id: 1 //only update rows that have the column id set to 1
		}
	}
).then(function(didSucceed){
	res.json(didSucceed); //respond with success status
});
```

delete all runs matching a criteria

```javascript
Runs.destroy({ //destroy the run as specified by id in the url
	where: {
		id: 1 //only delete rows that have the column id set to 1
	}
}).then(function(didSucceed){
	res.json(didSucceed); //send back if it succeeded
});
```

We can set up relationships between models easily:

```javascript
var Sequelize = require('sequelize'); //require sequelize package
var Runs = require('./run.js'); //require our Runs model
var db = require('./db.js'); //require connection to the DB

var Users = db.define('user', { //set up model for Users
	username: {
		type: Sequelize.STRING, //string data type
		unique: true //each value must be unique in the DB
	},
	password: Sequelize.STRING //string for password
});

Users.hasMany(Runs, { as: 'Runs' }); //set up the relationship that Users have many runs.  Will create a user_id column in the Runs table

db.sync(); //if table does not exist yet, create it

module.exports = Users;
```

Now, when we have a specific user object we have methods like `.getRuns()` and `.addRun()`:

```javascript
Users.findById(1).then(function(user){ //find the user in the DB who's ID is 1
	user.getRuns().then(function(runs){ //get that user's runs
		//do something with 'runs' param
	});
});
```

```javascript
Users.findById(1).then(function(foundUser){ //get the user from the DB
	Runs.create({
		date: new Date('2017-1-1'),
		distance: 6.1		
	}).then(function(createdRun){ //create a run from req.body data (JSON)
		foundUser.addRun(createdRun).then(function(){ //add the run to the user
			//do something here
		});
	});;
});
```
