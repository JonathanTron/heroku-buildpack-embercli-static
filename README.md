Heroku buildpack: heroku-buildpack-embercli-static
=======================

This is an example [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks).

Usage
-----

The buildpack will detect that your app has a `package.json` containing `"ember build"` in the root. If this file has the expected contents, it will try to `ember build` the app.
