# Template engines

Allow us to use static template file sin our application.
More info: https://expressjs.com/en/guide/using-template-engines.html

Example: `pug`

Install Pug: `npm install pug`

**IMPORTANT:** We don't have to require the module. Express loads the modules internally.

In your application file:

```js
app.set('view engine', 'pug');

// path to templates, default is ./views
app.set('views', './views');

// ... more code

// This will render the view and return HTML to the client
app.get('/', function(req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!' })
});
```

Then, create the `views/` folder and a template `index.pug`

```pug
html
  head
    title= title
  body
    h1= message
```

Then, if you go to http://localhost:9000/ you will see: `Hello there!`

And if you inspect the source code, you will see the html markup:

```html
<html><head><title>Hey</title></head><body><h1>Hello there!</h1></body></html>
```

**Remember:** At runtime, the template engine replaces variables in a template file with actual values, and transforms the template into an HTML file sent to the client. 