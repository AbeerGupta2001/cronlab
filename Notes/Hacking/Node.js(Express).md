
**Node.js** applications behave differently from Apache and Python HTTP servers. They are not serving static files from a configured document root. They are running application code, and that code decides what to return for every request. That flexibility is powerful, but it is also where many mistakes occur.

Attackers look at **Node.js Express** applications specifically because developers often leave development-mode features enabled when they push to production. Debug endpoints, verbose error responses, and exposed environment variables all stem from the same habit: code that worked in development went live as-is. The result is an application that tells you exactly how it is built and, sometimes, what credentials it uses.

## Triggering Verbose Errors

Express's built-in error handler suppresses stack traces when `NODE_ENV` is set to `production`. In development mode, it passes them through. But developers often write their own error handlers, and those custom handlers expose stack traces regardless of `NODE_ENV`. In the attached VM, the web app uses a custom error handler. The detailed JSON response you see is not part of Express behaviour; it is the result of application code written for debugging and never hardened. Both patterns appear on real engagements: the `NODE_ENV=production` check is only meaningful if no custom error handler overrides it. A `500 Internal Server Error` from an API endpoint is worth investigating further. In development mode, the response body often contains the error message, stack trace, and context about what the application was trying to do when it failed. The stack trace is the key finding here, as it reveals internal file paths, module names, and, sometimes, the database query that triggered the failure. Each of those details tells you something about the application's internals that a simple status code does not.

## Enumerating Routes via Debug Endpoints

One of the most useful things a misconfigured Express application can do is tell you all of its own routes. Developers sometimes build a debug endpoint that lists every registered route for convenience during development, then forget to remove it before going live.

```shell-session
root@ip-10-81-64-63:~# curl -s http://10.48.159.160:3000/api/routes
[{"method":"GET","path":"/"},{"method":"GET","path":"/api/users"},{"method":"GET","path":"/api/routes"},{"method":"GET","path":"/api/debug/env"}]
```

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861)

**Note:** The `/api/routes` endpoint works by reading Express's internal `app._router.stack` property to enumerate registered routes. This is a recognised Express misconfiguration pattern, but `app._router.stack` is an internal property whose structure can vary between Express versions, and Express 5 changed the router internals enough to break implementations that relied on the Express 4 structure. If you encounter a route endpoint that returns an unexpected format or an error, the application may be running a different Express version than the one this lab uses.

## Exposed Environment Variables

Environment variables in Node.js applications often contain database credentials, API keys, and configuration flags. A debug endpoint that returns `process.env` is a significant finding:

```shell-session
root@ip-10-81-64-63:~# curl -s http://10.48.159.160:3000/api/debug/env
{"NODE_ENV":"development","DB_PASSWORD":"NodeDBPass2024!","PORT":"3000","DB_HOST":"localhost:5432","APP_NAME":"company-portal"}r
```

If the response includes `DB_PASSWORD`, `SECRET_KEY`, or similar fields, document them. The `NODE_ENV` value is also telling. `NODE_ENV=development` on a production server is a signal that the application was deployed without proper hardening.


## Static File Serving

Express applications commonly use the `express.static()` middleware to serve front-end assets like JavaScript files, stylesheets, and configuration. The static files route, if it exists, serves everything in a directory. Client-side JavaScript files sometimes contain API endpoint URLs, internal hostnames, or debug flags embedded as constants.

Once you know the routes from the `/api/routes` endpoint, check what is being served statically:

```shell-session
root@ip-10-81-64-63:~# curl -s http://10.48.159.160:3000/static/config.js
// Client-side configuration
const API_BASE = 'http://internal-api.company.local:8080';
const DEBUG = true;
const VERSION = '1.2.0';
```

![](https://cdn-images.tryhackme.com/room-icons/68d2c1e7ab94268f6271de1d-1771765213861)

**Note:** `express.static()` silently returns a 404 for dotfiles (files starting with `.`) by default. This is the opposite of Python's HTTP server, which serves dotfiles like any other file. If you are looking for a `.env` or similar file on an Express static route and get a 404, that does not mean the file is absent; it means the middleware is blocking it. You would need shell access or a different route to reach it.