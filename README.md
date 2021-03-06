GIT-Push Server
==================

This node module makes it easy to build an HTTP stateless git-push server, to build stuff like Heroku or GitBook.io without keeping the repositories content.

### Installation

```
$ npm install git-push-server
```

### Example

```js
var GitPush = require("git-push-server");
var express = require("express");
var path = require("path");

// Create the http application
var app = express();

// Create the git-push server
var git = new GitPush();

// Create a router for the git-push server
var router = express.Router();

// Start the git server on the router
git.start(router);

// Bind the router to the app
app.use('/:author/:repo.git', function(req, res, next) {
    // Needed to identify the repository
    req.repoId = [req.params.author, req.params.repo].join("/");
    next();
});
app.use('/:author/:repo.git', router);

// Start the http server
var server = app.listen(3000, function() {
    console.log('Listening on port %d', server.address().port);
});
```

You can now run from a git repository:

```
$ git push http://localhost:3000/test/test.git master
```

### Run an operation on each push

If you need to run an operation on the folder resulting from the push, you need to override ```push```. For async operaiton, this method can return a **promise**.

```js
git.push = function(pushInfos) {
    // pushInfos.repoId
    // pushInfos.auth.username
    // pushInfos.auth.password
    // pushInfos.content
    // pushInfos.bare

    // do some build or deployment operation
};
```

### Authentication

You need to override ```authenticate``` to make authentication work. It should return a **boolean** or a **promise** for async authentication.

```js
git.authenticate = function(infos) {
    // infos.repoId -> repository id set with in req.repoID
    // infos.username
    // infos.password

    return doSomethingWithDatabase(infos.username, infos.password, infos.repoId);
};
```

### Options

```js
var git = new GitPush(options);
```

`options.debug`: enable log messages (default false)
`options.root`: root directory for the repositories (default os.tmpdir)

