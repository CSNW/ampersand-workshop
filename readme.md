# Objectives

* Get comfortable:
  * Developing with [Ampersand](http://ampersandjs.com/)
  * Writing client-side modules w/ [Browserify](http://browserify.org/)
* Practice TDD, MVC & mini-module development
* Write code as quickly as possible
* Get unstuck as quickly as possible

Instead of using large, general-purpose libraries like jQuery and Underscore, we'll practice using standards-compliant JS with polyfills where necessary, [`lodash-node`](https://www.npmjs.org/package/lodash-node) and other mini-modules.

Larger libraries are more convenient at first, but have fun trying to update the jQuery dependency in a large app. Using a proper module system like npm helps a little because each module can have its own version of jQuery, but then you're delivering and using multiple versions of jQuery on the client. Mini-modules (along with thoughtful dependency versioning and [`npm dedupe`](https://www.npmjs.org/doc/cli/npm-dedupe.html)) are a more maintainable solution.

Large sections of the app can be separated out into standalone modules, with their own test suites and versioned dependencies. This breaks us away from the feel of a monolithic app and allows us to be more agile. It also facilitates re-use of sub-components within other apps and isolates the potential impact of changes (for example, you can update a mini-module dependency used by the task subsystem of an app, it won't break other parts of the app because they would continue using the older version of that mini-module). This sort of thing can be done with CSS as well -- less, for example, has a namespacing feature so you can use CSS from one namespace in one part of the app (say "bootstrap-3.2.0") and CSS from another namespace ("boostrap 3.0.0") in another.

*Note:* Originally, the &yet version of this workshop used hapi for a server and jade for a templating engine, but we'll use a simple static [`http-server`](https://www.npmjs.org/package/http-server) with [mustache](https://github.com/janl/mustache.js/) templating.

### Changes from previous versions

* Changed the naming conventions. &yet likes to put all client-side code in a `client` folder, but since we have no server-side code in this project, we'll keep things flatter & keep most things at the root level or one level deep. Also, &yet uses identical filenames, relying solely on folder names to keep things straight. We'll go ahead and use folders (`/views/`, `/pages/`, `/tests/`, etc) but we prefer to also put a "_view.js" suffix on views, a "_tests.js" suffix on tests and a "_page.js" suffix on top-level views (but we don't add suffixes to models/collections).
* Using ECMAScript 5-6, polyfills & `lodash-node` mini-modules instead of underscore (Ampersand has recently been moving in this direction as well)
* Using HTML 4-5 (for instance `el.textContent`), polyfills & mini-modules instead of jQuery

## 1. Writing a Mocha Test

- create a new project folder, name it whatever you want and put `{"name": "wolves-client"}` in a `package.json` file in the root (this project is the main part of the &yet training and is part of a futuristic drama involving mad scientists and wolf attacks)
- npm install mocha using the `--save-dev` to autopopulate package.json (see [npm install docs](https://www.npmjs.org/doc/cli/npm-install.html))
- create a `tests` folder w/ a `main_view_tests.js` file that describes a "main view" (we haven't created this yet) -- the main view 'should render the text "Some Text"' (see [Mocha: Getting Started](http://mochajs.org/#getting-started) for an example)
- the test should require the `MainView` class from `../views/main_view.js` (Node.js [require docs](http://nodejs.org/api/modules.html#modules_modules))
- inside the `it()` test, it should create an instance of the class (instantiating with no arguments), and finally, use Node's [assert](http://nodejs.org/api/assert.html#assert_assert) to verify that the text in the view's `el` equals "Some Text" (you'll need to require assert at the top of your file, it's included in node so you don't have to `npm install` it).

*Note:* Instead of installing jQuery for `$(el).text()`, use `el.textContent`, it's in the HTML standard & is supported by IE9+ (IE's `el.innerText` is non-standard & isn't supported by FF). If you really need IE8 support, there's a robust `textContent` polyfill at https://github.com/shawnbot/aight.

*Note 2:* Be sure to instantiate the view in the test itself (inside the `it`), not the `describe` or the top level of the test module.

## 2. Watching Your Test Fail

It's good to watch your test fail before you implement the functionality (the "red" in the "red, green, refactor" TDD mantra). This is so you can:

* make sure it *does* fail
* make sure it fails the way you expect (not a syntax error in the test code, for instance)

To get the tests to fail:
- Verify that mocha is not globally installed (that you can't run it via `mocha tests`) -- it should be installed locally, not globally.
  - if you *can* run mocha directly, run `where mocha` (win) or `which mocha` (mac) to see where mocha is installed globally and remove it
  - you want a locally versioned & configured test framework, not a global one. One of the big advantages of `npm` and of node's module system is the priority given to locally-versioned and locally-configured dependencies.
- to run mocha & see your tests fail, you can run `node_modules/.bin/mocha tests`
- this should fail with `Error: Cannot find module '../views/main_view.js'` (b/c we haven't written this file yet)
- the standard way to run the tests for any npm/node module, regardless of what test framework is used, is `npm test`. To register mocha with `npm test`, just add it to a new `scripts` section in your package.json like this (the path for npm scripts includes `node_modules/.bin`):

```json
"scripts": {
  "test": "mocha tests"
}
```

## 3. Making Your Test Pass

- create a `views` folder and, inside it, `main_view.js` which requires `ampersand-view` (you'll need to npm install `--save` this)
- this main_view.js module should export a view that extends from AmpersandView (`module.exports = AmpersandView.extend({...});`
- set a `template` property to `'<body><h1>Some Text</h1></body>'` and an `autoRender` property to `true`.

Run the tests again and you should get a different failure:

```
ReferenceError: document is not defined
```

This is because everything is running in node and there is no global `document` object.

- install browserify as a devDependency (`--save-dev`) (Note: I got repeated ERR's and an infinite loop when I did this, running `npm cache clean` and retrying fixed it).
- add a "build" script that calls browserify to package.json:

```json
"scripts": {
  "build": "browserify tests/main_view_tests.js -o tests/tests.js"
}
```

- run your new build script via `npm run build` (`npm test` is unique in that it can be run without the `run` command, all other scripts need to be run with `npm run {{script_name}}`). You'll notice that `npm run build` will echo back the name of the  If there is no output, it succeeded.
- create a `test.html` file in the root that looks like this:

```html
<html>
  <head>
    <link rel="stylesheet" href="node_modules/mocha/mocha.css" />
    <script src="node_modules/mocha/mocha.js"></script>
    <script>mocha.setup('bdd')</script>
    <script src="tests/tests.js"></script>
    <script>
      onload = function() {
        mocha.checkLeaks();
        var runner = mocha.run();
      };
    </script>
  </head>
  <body>
    <div id="mocha"></div>
  </body>
</html>
```

- change your test script to `start test.html` (win) or `open test.html` (mac):

```json
"scripts": {
  "test": "start test.html"
}
```

- run `npm test`, if the test doesn't pass, fix any bugs (Note that the files & line numbers in the error stack are untranslated -- they refer to the single bundle file instead of using the sourcemap. Apparently Chrome uses the sourcemap for the interactive debugger, but not for the stacktrace in error messages. We can and should convert the stack using the sourcemap, but I still need to streamline the process, currently it's a bit of work, see the very bottom of this readme ("Aside: Converting error stacks using sourcemaps").

## 4. Testing Manually

- create an `app.js` file in the root folder and in it, set `window.app` to an object w/ an `init()` function

- inside the init function, call [`domready`](https://www.npmjs.org/package/domready) (you'll need to npm install `domready`) and in the callback, set the app's view property to a `new MainView({el: document.body})` (you'll need to require MainView via `require('./views/main_view.js')`. Notice that in Ampersand the template *includes* the view's element (`<body>`).
- at the bottom of app.js, run `window.app.init()`

- run browserify on your app file as part of the build script by adding: `... && browserify app.js -o wolves-client.js`
- write an `index.html` that consists of a single script tag (don't forget the end tag) pointing to `wolves-client.js` (yes, it is browser-supported and standards-compliant to imply `<html>`, `<head>`, `<body>`, etc)
- you can fire up the browser from the command line with `start index.html` (win) or `open index.html` (mac)
- Verify that "Some Text" is displayed in the browser window

## 5. Smoothing the process with watchify

- install watchify to watch & build your compiled files
- add scripts to package.json's "scripts" property, here's the example from the browserify handbook:

```json
"scripts": {
  "build": "browserify browser.js -o static/bundle.js",
  "watch": "watchify browser.js -o static/bundle.js --verbose",
}
```

You can call browserify with `-d` or `--debug` to generate embedded source maps, so the code in the dev tools looks like the source files, not the single generated file. However, if you want to be able to open the generated file in sublime, you may want to spin out the source maps to a separate file using the `exorcist` module (sublime froze up on me when I had embedded source maps, probably due to the super-long line length, but UltraEdit handled it just fine). Optionally, you can install exorcist and change your browserify commands to pipe through exorcist and redirect the output to your file like this: `browserify app.js -d | exorcist wolves-client.js.map > wolves-client.js`. Note, however, that watchify doesn't work with exorcist and result piping, since watchify runs multiple times every time a file changes, so it can't just be piped to a file.

Use ` && ` as a separator to run multiple commands on the same line. Now you can run these with `npm run build` and `npm run watch` (alternatively, you could do this as part of your server process). However, you can't use ` && ` to run two watchify commands because watchify never returns. Instead you can do `start watchify ...` (win) or `watchify ... &` (mac). On windows, this opens a couple new command windows, but on the mac it opens a background process with no UI -- this could lead to a frustrating situation where watchify is stuck in an error condition and you have no way of seeing that -- I'm not sure how common that is on the mac, but I've already run into a situation like that on Windows (see: https://github.com/substack/watchify/issues/83).

## 6. Add a home page

- Write a `tests\home_view_tests.js` file that requires `../pages/home_page.js` (the ampersand convention is to put sub-views in `/views/` and top-level views in `/pages/`) and instantiates it and verifies that it has the text "Hey there, wolves" (you'll want to require node's built-in `assert` module again to do the assertion).
- Write an `tests\index.js` file that requires both test files (be sure to begin the paths with `./`, otherwise node will think they're `npm install`ed modules, not relative file references)
- Change the browserify/watchify test commands to point to this central file instead of `main_view_tests.js`

Now if you run watchify or browserify, you should get an error: `Cannot find module '../pages/home_page.js'` -- this is the "red" failing test. Now let's make it pass:

- Create a `pages/home.js` view that inherits from `ampersand-view` and sets the `template` property to `<section class="page"><h1>Hey there, wolves</h1></section>`
- Add an `autoRender: true` property
- Note that creating this file won't cause watchify to re-compile -- the file isn't being watched because it hadn't been created previously. You'll need to touch a file that watchify is watching (the test file) or manually close & re-watch.
- Run `npm test` to verify the test passes (you won't be able to test manually until we put in a router and a View Switcher)

*Note: Sometimes watchify will [hit an EPERM error](https://github.com/substack/watchify/issues/83), apparently it happens when a file is changed while watchify is still building from the previous time. This EPERM error will happen every time from then on; the command window needs to be closed & re-opened for it to reset.*

## 7. Add a list view

- Write a `howls_view_tests.js` test that requires `../pages/howls_page.js` and instantiates the view & verifies that it has the text "Howls! Awoooooooo!".
- Write a `pages/howls_page.js` that inherits from `ampersand-view`, with a template of `<section class="page"><h1>Howls! Awoooooooo!</h1></section>`

## 8. Add a router
- npm install ampersand-router
- (Peter's TODO: figure out how to test the router, see http://stackoverflow.com/questions/9215737/testing-routers-in-backbone-js-properly)
- In `router.js`, inherit from `ampersand-router` and set the routes property of the router like this:

```javascript
routes: {
  '': 'home',
  'howls': 'howls'
}
```

- Add a `home()` method to the router that does a `this.trigger('page', new HomePage());` and a `howls()` method that triggers the page event with a new `HowlsPage` (you'll need to `require` 3 things, `ampersand-router`, `./pages/home` and `./pages/howls`).
- @sdkester found that if you try to run this from file://, you'll get `pushState is insecure in file:// mode`. To serve it locally, you can globally install http-server via `npm install http-server -g` and then run it from `http-server -c-1` (in the same folder where your index.html file is) and view it at `http://localhost:8080/index.html`.


## 9. Adding ViewSwitcher and turning on the router

- in `app.js`, require `./router.js` and in `init()` set `this.router` to a new router (no arguments), and after instantiating the new MainView (still inside domready), set `this.router.history.start({pushState: true});`.
- in `main_view.js`, require `ampersand-view-switcher` (you'll need to npm install this) and add the following 3 methods:

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

## 10. Use the router on click

- in `main_view.js`, add handling of the link click, making clicks trigger navigate actions:

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

## 11. Use mustache for templates

- npm install `mustache`
- save the template above into `views/main.mustache` using UTF-8 encoding
- npm install `brfs`, which will statically change fs.readFileSync() into strings
- add the `-t brfs` option to the browserify command, this adds the brfs module as a transform plugin
  - Note: there's a brfs bug (https://github.com/substack/brfs/issues/28) where `require('fs')` as part of a comma-delimited multiple variable declaration statement fails to parse, so you need to put `var fs = require('fs');` on its own line.
- at the top of `main_view.js`, set a `template` variable to `fs.readFileSync('main.mustache', 'utf8');` and require `fs` at the top of the file
- in `main_view.js`, set the `template` property to a function: `function(ctx) { return Mustache.render(template, ctx); }` and require `mustache` at the top of the file.

## 12. Add a Model

- In `models/howl.js`, define a model that extends from `ampersand-model`
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

## 13. Add a Collection

- in `models/howls.js`, extend `ampersand-rest-collection` (and npm install it)
- set the url property to `http://wolves.technology/howls`, set the model property to `Howl` (and define this at the top of the file to `require('./howl.js');`
- define an initialize method that does `this.fetch();`

## 14. Render the collection on the view

- in `app.js`, set `this.howls = new Howls();` (and require it at the top of the file)
- in `pages/howls_page.js`, add a render method that calls `renderWithTemplate()` and `renderCollection()` (and require `../views/howl_view.js` at the top of the file):

```javascript
        this.renderWithTemplate();
        this.renderCollection(app.howls, HowlView, this.queryByHook('howls-container'));
```

- add `<div data-hook="howls-container"></div>` to the howls template
- create a new howl view in `views/howl_view.js` that extends from `ampersand-view` and sets the template to a new mustache file:

```html
<div class="well">
  <p>{{model.niceDate}}</p>
  <p>{{model.user.username}}</p>
  <pre>{{model.content}}</pre>
</div>
```

# 15. 


## Aside: Converting error stacks using sourcemaps

Apparently Chrome uses source maps for interactive debugging/stepping, but it doesn't use them for converting or displaying error stacks. Here's how to convert error stacks manually:

* npm install `exorcist` and add it to your browserify commands to generate separate source map files:

```json
"scripts": {
  "build": "browserify tests/main_view_tests.js -d | exorcist tests\\tests.js.map > tests/tests.js"
}
```

* The source map file doesn't know what file you piped the output to, so it has a default `file` property set to "generated.js". You'll need to update the `file` property and JSON.parse the map into an object:

```javascript
var map = JSON.parse(fs.readFileSync('tests\\tests.js.map', 'utf8'));
map.file = "tests/tests.js";
```

* Convert chrome's err.stack property from a string to an array and use `shift()` to pop off the first item in the array which is the error message itself and not a property stack line:

```javascript
var stack_arr = err.stack.split('\r\n');
stack_arr.shift();
```

* Feed this array into a function that converts it into what stackMapper uses (this code is originally from https://github.com/thlorenz/stack-mapper/blob/master/test/util/frames-fromstr.js, but I modified the regex slightly to allow a trailing `)` b/c Chrome stacks now includes that if the line is in a named function):

```javascript
function fromStr(str_arr) {
  var frames = [];
  for (var i=0 ; i<str_arr.length ; ++i) {
    var frame_str = str_arr[i];

    var matched = frame_str.match(/^(.*)[:](\d+)[:](\d+)\)?$/);
    if (!matched)
      throw new Error('no match: "' + frame_str + '"');
    frames.push({
      filename: matched[1],
      line: matched[2] - 0,
      column: matched[3] - 0
    });
  }

  return frames;
}
```

* Finally, instatiate the stack-mapper with the map and run the frames into it to get the modified frames:

```javascript
var stackMapper = require('stack-mapper');
var sm = stackMapper(map);
var inframes = fromStr(stack_arr);
console.log(sm.map(inframes));
```

Note: the zuul testing framework uses https://github.com/stacktracejs/stacktrace.js/ to get a compatible stack trace in a cross-browser way.