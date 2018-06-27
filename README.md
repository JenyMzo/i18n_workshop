# i18n Workshop
Internationalization workshop for MedellinJS

**Before starting**

1. Install [Node.js LTS](http://nodejs.org/)
2. Clone this repository
3. Install express `npm install express --s`
4. Install cookie-parser `npm install cookie-parser --s`
5. Install express-session `npm install express-session --s`
6. Install i18n `npm install i18n --s`
7. Install jade `npm install jade --s`

## Creating server file

Let's start creating a `server.js` file.

```js
'use strict';

const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.listen(port);

console.log('Server started on: ' + port);
```

run `npm run start on the terminal` to test the server.


As we are using Jade as template engline, we need to set that.

```js
app.set('view engine', 'jade');
```
Also, we need to serve static files (css and images) inside the assets folder.

```js
app.use(express.static('assets'));
```

## App configuration

Now, we need to create a config file to set up the express session and cookie parser, the function should receive the app `express()` as well as cookieParse, session and i18n.

```js
'use strict'

module.exports = (app, cookieParser, session, i18n) => {
  i18n.configure({
    //define how many languages we would support
    locales:['en', 'es'],
    //define the path to language json files, default is /locales
    directory: __dirname + '/locales',
    //define the default language
    defaultLocale: 'en',
    // define a custom cookie name to parse locale settings from
    cookie: 'i18n'
  });

  app.use(cookieParser("i18n_demo"));

  app.use(session({
    secret: "i18n_demo",
    resave: true,
    saveUninitialized: true,
    cookie: { maxAge: 60000 }
  }));

  app.use(i18n.init);
}
```

Now, we are going to require and call this in our server file like this:

```js
  const session = require('express-session');
  const cookieParser = require('cookie-parser');
  const i18n = require('i18n');
  const app = express();
  const config = require('./config.js');

  config(app, cookieParser, session, i18n);
```
## Creating Locale files

Add the language json files inside locales folder, Since we have configured english and spanish (en, es), we need to add 2 files: en.json ad es.json.

Example:

**en.json**
```json
{
	"Title": "Internationalization with Node.js",
	"Welcome": "Hi, welcome to the i18n demo for Medellín JS",
	"Button": "I'm a button"
}
```

**es.json**
```json
{
	"Title": "Internacionalización con Node.js",
	"Welcome": "Hola, bienvenido al demo de i18n para Medellín JS",
	"Button": "Soy un botón"
}
```

## Routing
Finally, set up the routes. In this case we have 3 routes:

* / ---> get main page*
* /en ---> change language to English
* /es ---> change language to Spanish

The routes function should receive the app (`express()`) and the i18n like this:

```js
'use strict';

module.exports = (app, i18n) => {
  app.get('/', function (req, res) {
    //let's validate that the cookie exists
    if (!!req.cookies.i18n){
      res.setLocale(req.cookies.i18n);
    }
    res.render('main', {
    i18n: res
    })
  });

  app.get('/es', function (req, res) {
    res.cookie('i18n', 'es');
    res.redirect('/')
  });

  app.get('/en', function (req, res) {
    res.cookie('i18n', 'en');
    res.redirect('/')
  });
}
```

Now, we are going to require and call this in our server file like this:

```js
const routes = require('./routes.js');

routes(app, i18n);
```

The way in wich we can reference text in our view is `#{i18n.__("Object key")}`

For example:

**es.json**
```json
{
	"Title": "Internacionalización con Node.js",
	"Welcome": "Hola, bienvenido al demo de i18n para Medellín JS",
	"Button": "Soy un botón"
}
```

**view (main.jade)**
```jade
h1(class='heading') #{i18n.__("Title")}
```

![that's it!](https://media.makeameme.org/created/we-did-it-sw5z69.jpg)
