# Objectives

* Get comfortable
  * Developing with Ampersand
  * Writing Client-Side Modules (browserify)
* Practice MVC
* Write Code as Quickly as Possible
* Get Unstuck as Quickly as Possible

# Notes

The ampersand folks use hapi for a server and jade for a templating engine. We'll use express and mustache. 

## 1. Writing a Test

- put `node_modules` in a `.gitignore`
- put `{"name": "wolves-client"}` in a `package.json`
- in the spirit of TDD, npm install mocha. Use `--save` to autopopulate package.json while you're at it (`npm install mocha --save`)
- write a mocha test in `tests\main_view_tests.js` that `describe()`s the main view -- `it()` should render the text "Some Text"
- the test should require `../views/main.js`, instantiate it and verify that the view's `el` has "Some Text" in it (you'll probably want to add jQuery as a devDependency, `--save-dev`, and use it's `.text()` for this...)

## 2. Making it Pass
- in `client/views/main.js`, require `ampersand-view` and export a view that extends from it (via `module.exports = AmpersandView.extend({...});`, setting a `template` property to `<body><h1>Some Text</h1></body>` and an `autoRender` property to `true`.
- in `client/app.js` set `window.app` to an object w/ an init function that uses `domready` (you'll need to npm install `domready`) to set its view property to a `new MainView({el: document.body})` (you'll need to require the view via `require('./views/main.js')`. Notice that the template *includes* the view's element (`<body>`).
- at the bottom of app.js, run `window.app.init()`
- install browserify as a devDependency (`--save-dev`)
- in order to run browserify from the command line, you'll need to run `node_modules\.bin\browserify` or add the `.bin` folder to your path
- run browserify on your app file: `browserify app.js -o wolves-client.js` and on your tests file: `browserify tests\main_view_tests.js -o tests\tests.js`
- copy `test\browser\index.html` from mocha's github repo (for some reason this folder isn't downloaded as part of the npm install) as `test.html` (it makes things simpler to have this in the root) and modify it for your project
- use `http-server -c-1` to server the content without caching it (-1) (if http-server's not already installed, it can be installed with `npm install http-server -g`

## 3. Testing it Manually
- write an `index.html` that consists of a single script tag pointing to `wolves-client.js` (yes, it is browser-supported and standards-compliant to imply all the other tags)
- use `http-server -c-1` to server the content without caching it (-1) (if http-server's not already installed, it can be installed with `npm install http-server -g`

## 4. Smoothing the process
- install watchify to watch & build your compiled files
- add scripts to package.json's "scripts" property, here's the example from the browserify handbook:

```json
"scripts": {
  "build": "browserify browser.js -o static/bundle.js",
  "watch": "watchify browser.js -o static/bundle.js --debug --verbose",
}
```

You can call browserify with `-d` or `--debug` to generate source maps, so the code in the dev tools looks like the source files, not the single generated file.

Use ` && ` as a separator to run multiple commands on the same line. Now you can run these with `npm run build` and `npm run watch` (alternatively, you could do this as part of your server process).

## 5. Add a home page

- Write a `tests\home_view_tests.js` file that requires `../client/pages/home.js` (the ampersand convention is to put sub-views in `/views/` and top-level views in `/pages/`) and instantiates it and verifies that it contains the text "Hey there, wolves"
- Write an `tests\index.js` file that requires both test files (be sure to begin the paths with `./`, otherwise node will think they're `npm install`ed modules, not relative file references)
- Change the browserify/watchify test commands to point to this central file instead of `main_view_tests.js`
- Create a `client/pages/home.js` view that inherits from `ampersand-view` and sets the `template` property to `<section class="page"><h1>Hey there, wolves</h1></section>`
- Verify the test passes (you won't be able to test manually until we put in a router and a View Switcher)

## 6. Add a list view

- Write a `howls_view_tests.js` test that requires `../client/pages/howls.js` and instantiates the view & verifies that it has the text "Howls! Awoooooooo!".
- Write a `client/pages/howls.js` that inherits from `ampersand-view`, with a template of `<section class="page"><h1>Howls! Awoooooooo!</h1></section>`

## 7. Add a router
- npm install ampersand-router
- I'm not sure how to test the router in the context of mocha's browser tests. If someone figures it out, could you fork this repo & add it here?
- In `client/router.js`, inherit from `ampersand-router` and set the routes property of the router like this:

```javascript
routes: {
  '': 'home',
  'howls': 'howls'
}
```

- Add a `home()` method to the router that does a `this.trigger('page', new HomePage());` and a `howls()` method that triggers the page event with a new `HowlsPage` (you'll need to `require` 3 things, `ampersand-router`, `./pages/home` and `./pages/howls`).

## 8. Adding ViewSwitcher and turning on the router

- in `client/app.js`, require `./router.js` and in `init()` set `this.router` to a new router (no arguments), and after instantiating the new MainView (still inside domready), set `this.router.history.start({pushState: true});`.
- in `client/views/main.js`, require `ampersand-view-switcher` (you'll need to npm install this) and add the following 3 methods:

```javascript
    initialize: function () {
        this.listenTo(app.router, 'page', this.handleNewPage);
    },
    render: function () {
        this.renderWithTemplate(this);
        this.pages = new PageSwitcher(this.queryByHook('page-container'));
    },
    handleNewPage: function (page) {
        this.pages.set(page);
    }
```

- set the template to the following string: (we'll pull it out into a mustache file in the next step):

```
<body>
  <nav class="navbar navbar-default">
    <div class="container-fluid">
      <div class="navbar-header">
        <a class="navbar-brand" href="/">Wolves</a>
      </div>
      <ul class="nav navbar-nav">
        <li>
          <a href="/howls">Howls</a>
        </li>
      </ul>
    </div>
  </nav>
  <div class="container">
    <main data-hook="page-container"></main>
  </div>
</body>
```

## 9. Use the router on click

- in `client/views/main.js`, add handling of the link click, making clicks trigger navigate actions:

```javascript
    events: {
        'click a[href]': 'handleLinkClick'
    },
    handleLinkClick: function (e) {
        var aTag = e.target;
        var isLocal = aTag.host === window.location.host;

        if (isLocal && !e.altKey && !e.ctrlKey && !e.metaKey && !e.shiftKey) {
            e.preventDefault();
            app.router.history.navigate(aTag.pathname, {trigger: true});
        }
    }
```

## 10. Use mustache for templates

- npm install `mustache`
- save the template above into `client/views/main.mustache` using UTF-8 encoding
- at the top of `client/views/main.js`, set a `template` variable to `fs.readFileSync('main.mustache', 'utf8');` and require `fs` at the top of the file
- in `client/views/main.js`, set the `template` property to a function: `function(ctx) { return Mustache.render(template, ctx); }` and require `mustache` at the top of the file.

## 11. Add a Model

- In `client/models/howl.js`, define a model that extends from `ampersand-model`
- Add props to the model like this (this creates getters, similar to what we do with our entities metadata):

```javascript
    props: {
        id: 'string',
        content: 'string',
        createdAt: 'date',
        user: 'object'
    },
```

- Add a derived property of `niceDate` that formats using the `moment` library (which you'll need to npm install):

```javascript
    derived: {
        niceDate: {
            deps: ['createdAt'],
            fn: function () {
                return moment(this.createdAt).fromNow();
            }
        }
    }
```

## 12. Add a Collection

- in `client/models/howls.js`, extend `ampersand-rest-collection` (and npm install it)
- set the url property to `http://wolves.technology/howls`, set the model property to `Howl` (and define this at the top of the file to `require('./howl');`
- define an initialize method that does `this.fetch();`

