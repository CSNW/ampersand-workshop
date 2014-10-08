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
- use `npm install <pkg> --save` to auto-populate `package.json`
- in the spirit of TDD, npm install mocha
- write a mocha test in `tests\main_view_tests.js` that `describe()`s the main view -- `it()` should render the text "Some Text"
- the test should require `../views/main.js`, instantiate it and verify that the view's `el` has "Some Text" in it (you'll probably want to add jQuery as a devDependency, `--save-dev`, and use it's `.text()` for this...)

## 2. Making it Pass
- in `client/views/main.js`, require `ampersand-view` and export a view that extends from it, setting a `template` property to `<body><h1>Some Text</h1></body>` and an `autoRender` property to `true`.
- in `client/app.js` set `window.app` to an object w/ an init function that uses `domready` to set its view property to a `new MainView({el: document.body})` (you'll need to require the view via `require('./views/main.js')`. Notice that the template *includes* the view's element (`<body>`).
- at the bottom of app.js, run `window.app.init()`
- install browserify as a devDependency (`--save-dev`) and run it on your app file: `browserify app.js -o wolves-client.js` and on your tests file: `browserify tests\main_view_tests.js -o tests\tests.js`
- copy `node_modules\mocha\test\browser\index.html` as `tests\test.html` and modify it for your project

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