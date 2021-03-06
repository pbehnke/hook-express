# hook-express README.md

![Editor](https://raw.githubusercontent.com/billroy/hook-express/master/doc/editor.png "Using the editor")

![Command line](https://raw.githubusercontent.com/billroy/hook-express/master/doc/cli.png "Using the API")

## What is it?

hook-express a micro-service host and laboratory for expressjs.  

The hook-express server is an expressjs web app that can live-edit its routing and per-route behavior on the fly.  You can add and edit routes and route functions (which are called here "hooks") through the hooks API or a web-based editor interface, without restarting the server to change configurations.

The server exposes an API that provides for manipulation of hooks, and an editor interface for editing hooks through-the-web.


## What is a hook?

A hook is a set of javascript statements that implement the business logic of an API endpoint for a micro-service.

The code for a hook is the core of an express route dispatch function.  For example, considering this express route dispatch function:

    app.use('/time', function(req, res) {
        res.send(new Date());
    });

The "hook" part is just the guts of the function:

    res.send(new Date());

## Install

    git clone https://github.com/billroy/hook-express.git
    cd hook-express
    npm install
    node hook-express

Open a browser on http://localhost:3000/editor

## Default username and password

The editor and API require a username and password for access.  It is important to be aware that the code running in a hook has access to the local file system!

The default username is 'hook' and the default password is 'express'.

Please change the username and password or risk becoming a security statistic.

You can change the username and password by setting these environment variables before starting hook-express:

    HOOK_EXPRESS_API_USERNAME
    HOOK_EXPRESS_API_PASSWORD

You can also add usernames to the 'apiUsers' table in hook-express.js.

## An Important Security Note

Code running in a hook has complete access to node modules like 'fs', so it can erase your file system, modify files, and so forth.  Change the default password, and keep your password safe.

## Command Line Options

### --port=3000

Binds the server to a chosen port.  Default port is 3000.

### --load=[filepath || url]

Loads a set of hooks (of the form produced by GET /hx/hooks) from a file or url.  Runs immediately after the server starts up, so it is possible to load from a static file served under /public.

### --ssl and --certs

Specify --ssl=true and --certs=/path/to/certs to engage ssl.

### --logfile=[]

Logging via winston to the console is always enabled, but logging to file is disabled by default.  Use --logfile=filepath to turn on logging to file.

### --loglevel=[info]

By default, the log level is 'info'.  Use --loglevel to change it.

### --capture

Use --capture to capture all 404 requests as new hooks on the 404'ed route.

When --capture is engaged, the first time an unknown route (path/method pair) is called, a 404 is returned and a new hook is created on the route with simple default contents.  The second call to the same route will get the generated hook response instead of 404.

By default, --capture is not engaged, and a 404 is just a 404.

## Static files

You can add files to be static-served to the public/ directory and hook-express will serve them at '/'.  So, the file public/index.html is the home page for the site.

## Limitations and caveats

Limitations of the current version:

    - bound dispatch function names of the form "hook_<number>", like hook_1, are reserved
    - nothing is persisted - the server starts up with no hooks defined
    - the hooks and static files in public/ are served without authentication

Caveats:

    - anyone with the API username and password can write a hook that can modify your file system


## The HttPie Tool

The examples below use the excellent "http" utility from HttPie: https://httpie.org -- it is well-suited for this sort of use because, unlike curl and wget, it has json-friendly defaults.

## API Example: creating a web hook to return the time

    $ http -b -a hook:express :3000/time
    Cannot GET /time

    $ http -b -a hook:express :3000/hx/hooks path=/time hook='res.send(new Date());'
    {
        "hook": "res.send(new Date());",
        "hookId": "hook_1",
        "method": "get",
        "path": "/time"
    }

    $ http -b -a hook:express :3000/time
    "2016-06-29T00:21:58.973Z"


## API Example: fetching all hooks

    $ http -a hook:express :3000/hx/hooks
    {
        "hook_1": {
            "hook": "res.send(new Date());",
            "hookId": "hook_1",
            "method": "get",
            "path": "/time"
        }
    }

## API Example: fetching a single hook by hookId

        $ http -a hook:express :3000/hx/hooks/hook_1
        {
            "hook_1": {
                "hook": "res.send(new Date());",
                "hookId": "hook_1",
                "method": "get",
                "path": "/time"
            }
        }


## API Example: deleting a hook

    $ http delete -a hook:express :3000/hx/hooks/hook_1
    deleted

    $ http -a hook:express :3000/hx/hooks
    {}


## API Example: deleting all hooks

    $ http delete -a hook:express :3000/hx/hooks/*
    deleted

    $ http -a hook:express :3000/hx/hooks
    {}
