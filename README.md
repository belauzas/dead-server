[![view on npm](http://img.shields.io/npm/v/dead-server.svg)](https://www.npmjs.org/package/dead-server)
[![npm module downloads per month](http://img.shields.io/npm/dm/dead-server.svg)](https://www.npmjs.org/package/dead-server)

# Dead Server

This is a quick fix (updated version) for an original ["Live Server"](https://www.npmjs.com/package/live-server) package. Original version doesn't seem be maintained, and while there was a major braiking change ir the dependencies, I took to quick-fix it.

## Intro

This is a little development server with live reload capability. Use it for hacking your HTML/JavaScript/CSS files, but not for deploying the final site.

There are two reasons for using this:

1. AJAX requests don't work with the `file://` protocol due to security restrictions, i.e. you need a server if your site fetches content through JavaScript.
2. Having the page reload automatically after changes to files can accelerate development.

## Install

I **do not** recommend to install this package globally, because this might be an obstacle while working in a team. It's better to have it in your `package.json` file, so all team could use it without a hustle.

**Npm way**

```
npm i -D dead-server
```

**Manual way**

```
git clone https://github.com/belauzas/dead-server
cd dead-server
npm install # Local dependencies if you want to hack
npm install -g # Install globally
```

## Usage with `package.json`

`Package.json` part:

```json
  "scripts": {
    "dev": "dead-server --port=3000 --host=localhost"
  }
```

CLI command:

```shell
$ npm run dev
```

## Usage from command line

Issue the command `dead-server` in your project's directory. Alternatively you can add the path to serve as a command line parameter.

This will automatically launch the default browser. When you make a change to any file, the browser will reload the page - unless it was a CSS file in which case the changes are applied without a reload.

**Note**: `dead-server` has to be globally installed.

```shell
$ dead-server
```

Alternatively you can add the path to serve as a command line parameter.

```shell
$ dead-server ./index.html
```

General template:

```shell
$ dead-server [PATH] [OPTIONS...]
```

## Parameters

-   `--port=NUMBER` - select port to use, default: PORT env var or 8080
-   `--host=ADDRESS` - select host address to bind to, default: IP env var or 0.0.0.0 ("any address")
-   `--no-browser` - suppress automatic web browser launching
-   `--browser=BROWSER` - specify browser to use instead of system default
-   `--quiet | -q` - suppress logging
-   `--verbose | -V` - more logging (logs all requests, shows all listening IPv4 interfaces, etc.)
-   `--open=PATH` - launch browser to PATH instead of server root
-   `--watch=PATH` - comma-separated string of paths to exclusively watch for changes (default: watch everything)
-   `--ignore=PATH` - comma-separated string of paths to ignore ([anymatch](https://github.com/es128/anymatch)-compatible definition)
-   `--ignorePattern=RGXP` - Regular expression of files to ignore (ie `.*\.jade`) (**DEPRECATED** in favor of `--ignore`)
-   `--no-css-inject` - reload page on CSS change, rather than injecting changed CSS
-   `--middleware=PATH` - path to .js file exporting a middleware function to add; can be a name without path nor extension to reference bundled middlewares in `middleware` folder
-   `--entry-file=PATH` - serve this file (server root relative) in place of missing files (useful for single page apps)
-   `--mount=ROUTE:PATH` - serve the paths contents under the defined route (multiple definitions possible)
-   `--spa` - translate requests from /abc to /#/abc (handy for Single Page Apps)
-   `--wait=MILLISECONDS` - (default 100ms) wait for all changes, before reloading
-   `--htpasswd=PATH` - Enables http-auth expecting htpasswd file located at PATH
-   `--cors` - Enables CORS for any origin (reflects request origin, requests with credentials are supported)
-   `--https=PATH` - PATH to a HTTPS configuration module
-   `--https-module=MODULE_NAME` - Custom HTTPS module (e.g. `spdy`)
-   `--proxy=ROUTE:URL` - proxy all requests for ROUTE to URL
-   `--help | -h` - display terse usage hint and exit
-   `--version | -v` - display version and exit

### Default options:

If a file `~/.dead-server.json` exists it will be loaded and used as default options for dead-server on the command line. See "Usage from node" for option names.

## Usage from node

```javascript
var deadServer = require("dead-server");

var params = {
	port: 8181, // Set the server port. Defaults to 8080.
	host: "0.0.0.0", // Set the address to bind to. Defaults to 0.0.0.0 or process.env.IP.
	root: "/public", // Set root directory that's being served. Defaults to cwd.
	open: false, // When false, it won't load your browser by default.
	ignore: "scss,my/templates", // comma-separated string for paths to ignore
	file: "index.html", // When set, serve this file (server root relative) for every 404 (useful for single-page applications)
	wait: 1000, // Waits for all changes, before reloading. Defaults to 0 sec.
	mount: [["/components", "./node_modules"]], // Mount a directory to a route.
	logLevel: 2, // 0 = errors only, 1 = some, 2 = lots
	middleware: [
		function (req, res, next) {
			next();
		},
	], // Takes an array of Connect-compatible middleware that are injected into the server middleware stack
};
deadServer.start(params);
```

## HTTPS

In order to enable HTTPS support, you'll need to create a configuration module.
The module must export an object that will be used to configure a HTTPS server.
The keys are the same as the keys in `options` for [tls.createServer](https://nodejs.org/api/tls.html#tls_tls_createserver_options_secureconnectionlistener).

For example:

```javascript
var fs = require("fs");

module.exports = {
	cert: fs.readFileSync(__dirname + "/server.cert"),
	key: fs.readFileSync(__dirname + "/server.key"),
	passphrase: "12345",
};
```

If using the node API, you can also directly pass a configuration object instead of a path to the module.

## HTTP/2

To get HTTP/2 support one can provide a custom HTTPS module via `--https-module` CLI parameter (`httpsModule` option for Node.js script). **Be sure to install the module first.**
HTTP/2 unencrypted mode is not supported by browsers, thus not supported by `dead-server`. See [this question](https://http2.github.io/faq/#does-http2-require-encryption) and [can I use page on HTTP/2](http://caniuse.com/#search=http2) for more details.

For example from CLI(bash):

```
dead-server \
  --https=path/to/https.conf.js \
  --https-module=spdy \
  my-app-folder/
```

## Troubleshooting

-   No reload on changes
    -   Ensure that your index file has `html`, `head` and `body` tag
    -   Open your browser's console: there should be a message at the top stating that live reload is enabled. Note that you will need a browser that supports WebSockets. If there are errors, deal with them. If it's still not working, [file an issue](https://github.com/belauzas/dead-server/issues).
-   Error: watch <PATH> ENOSPC
    -   See [this suggested solution](http://stackoverflow.com/questions/22475849/node-js-error-enospc/32600959#32600959).
-   Reload works but changes are missing or outdated
    -   Try using `--wait=MS` option. Where `MS` is time in milliseconds to wait before issuing a reload.

## How it works

The server is a simple node app that serves the working directory and its subdirectories. It also watches the files for changes and when that happens, it sends a message through a web socket connection to the browser instructing it to reload. In order for the client side to support this, the server injects a small piece of JavaScript code to each requested html file. This script establishes the web socket connection and listens to the reload requests. CSS files can be refreshed without a full page reload by finding the referenced stylesheets from the DOM and tricking the browser to fetch and parse them again.

For more details, please refer to [live-server](https://www.npmjs.com/package/live-server) documentation.

## Version history

-   v1.0.6

    -   README update
    -   Support relative path specified for middleware
    -   Fixed issue regarding Folder Containing Exclamation Sign (!)

-   v1.0.5

    -   removed `colors` dependency and use `chalk` instead

-   v1.0.4

    -   `opn` package changed to `open`
    -   `cors`, `object-assign`, `send` and `proxy-middleware` versions changed from "latest" to respective latest stable versions

-   v1.0.3

    -   minor updates

-   v1.0.2

    -   mistype fix

-   v1.0.1

    -   even more renaming

-   v1.0.0

    -   Fork and `colors` package fix (revert to v1.4.0)
    -   rename to `dead-server`
