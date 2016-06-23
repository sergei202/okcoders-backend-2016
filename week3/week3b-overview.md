Overview for June 23rd
======================

So far we've built node applications that allow us to interact with them over the internet using routes and urls.  This is great, but not very user-friendly.  Today are going to add a *front-end* to our node applications.

We do this by building a front-end (html, css, images, etc) and telling express to serve these static elements.  They are called *static* because the files never change.

Express Static Middleware
-------------------------
We've seen `express.static()` once before, but let's review what it does and how it is used.

`express.static(path)` allows us to serve static content at a specific path (directory/folder).  It is an *express middleware*, so we have to tell express where we want our static content to show up, this is called the *mount point*, it is just another route, usually `'/'`.

`app.use(mount, express.static(path))`

Here is a simple app that will serve whatever files are in the current directory:
```js
var express = require('express');         // Don't forget to install the express module

var app = express();                      // Create our express app

app.use('/', express.static('./'));       // Tell express to mount whatever files are in the current directory at /

app.listen(8080);                         // Start our server
console.log('Listening at http://localhost:8080');

```

If you saved the file as `static.js`, you can go to `http://localhost:8080/static.js` to view it.

Create a new file called `index.html` and add some html code: `<h1>Hello World</h2>`.  You can access it at `http://localhost:8080/index.html`.  You can also access it at `http://localhost:8080/`, how can this be?  The file name `index.html` is special.  If you don't specify a filename, express will automatically try to serve `index.html`.

You add files to that directory and you can access them from the browser. Try adding some images.  You just made a web server!

Express Static Organization and Security
----------------------------------------
As we demonstrated above, serving the current directory also exposes your server-side code (node js files). For our simple/demo projects, it's not a big deal.  But in the real world you might have API keys and other sensitive information inside your code.  We don't people to have access to it.

Our directory will also get messy as we start adding more static files to our project.

We solve both problems with the same solution: We store our static files in a separate directory from our code and only expose that directory to `express.static()`.

If we put all of our static files inside a child directory called `public`, we can serve only it's contents:
```js
app.use('/', express.static('./public'));
```

Remember, `/` is the *mount point* for this static content in our express app.  `./public` is the relative path to the *public* folder from our node program.
