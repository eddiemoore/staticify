staticify
===

A better static asset handler for node.js / express.js

Provides helpers to add a version identifier to your static asset's public URLs, and to remove the hash before serving the file from the file system.

How your URLs are transformed:

```
/home.css --> /home.<md5 hash of contents>.css
```

For example:

```
/home.css --> /home.ae2b1fca515949e5d54fb22b8ed95575.css
/js/script.js --> /js/script.3205c0ded576131ea255ad2bd38b0fb2.js
```

The version hashes are the md5 of the contents of the static asset. Thus, every file has it's own unique version identifier. When a file changes, only it's own hash changes. This lets you have a far-futures expires header for your static assets without worrying about cache-invalidation, while ensuring that the user only downloads the files that have changed since your last deployment.

With express.js
---

```javascript
var staticify = require("staticify")(path.join(__dirname, "public"));

...
app.use(staticify.middleware);

app.helpers({getVersionedPath: staticify.getVersionedPath});
```

And in your template:

```html
<link href="${getVersionedPath('/home.css')}" rel="stylesheet">
```

Usage
---

Install from npm:
```
npm install staticify
```

Initialise the staticify helper with the path of your public directory:

```javascript
var statificy = require("staticify")(path.join(__dirname, "public"));
```

This returns an object with the following helpers:

### .getVersionedPath(path)

Does the following transformation to the `path`, and returns immediately:
```javascript
staticify.getVersionedPath('/path/to/file.ext'); // --> /path/to/file.<md5 of the contents of file.ext>.ext
```

This method is meant to be used inside your templates.

This method is really fast (simply an in-memory lookup) and returns immediately. When you initialize this module, it crawls your public folder synchronously at startup, and pre-determines all the md5 hashes for your static files. This slows down application startup slightly, but it keeps the runtime performance at its peak.

### .middleware(req, res, next)

Convenience wrapper over `.serve` to handle static files in express.js.

```javascript
app.use(staticify.middleware)  // `app` is your express instance
```

### .replacePaths(string)

Takes the input string, and replaces any paths it can understand. For example:

```javascript
staticify.replacePaths("body { background: url('/index.js') }");
```
returns
```javascript
"body { background: url('/index.d766c4a983224a3696bc4913b9e47305.js') }"
```

Perfect for use in your build script, to modify references to external paths within your CSS files.

### .stripVersion(path)

Removes the md5 identifier in a path.

```javascript
staticify.stripVersion('/path/to/file.ae2b1fca515949e5d54fb22b8ed95575.ext'); // --> /path/to/file.ext
```

Note, this function doesn't verify that the hash is valid. It simply finds what looks like a hash and strips it from the path. 

### .refresh()

Rebuilds the md5 version cache described above. Use this method sparingly. This crawls your public folder synchronously (in a blocking fashion) to rebuild the cache. This is typically only useful when you are doing some kind of live-reloading during development.

### .serve(req)

Handles an incoming request for the file. Internally calls `.stripVersion` to strip the version identifier, and serves the file with a `maxage` of a year, using [send](https://github.com/tj/send). Returns a stream that can be `.pipe`d to a http response stream.

```javascript
staticify.serve(req).pipe(res)
```

License
---
MIT