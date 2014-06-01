Heroku buildpack: heroku-buildpack-embercli-static
=======================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks).
The buildpack will detect that your app has a `package.json` containing
`"ember build"` in the root. If this file has the expected contents,
it will try to `ember build` the app.

Usage
-----

Update the `package.json` file of your `ember-cli` app to have an entry for
`dependencies` which match what you have for the `devDependencies` entry:

```json
//...
  "devDependencies": {
    "body-parser": "^1.2.0",
    "broccoli-asset-rev": "0.0.5",
    "broccoli-coffee": "^0.1.0",
    "broccoli-ember-hbs-template-compiler": "^1.5.0",
    "broccoli-merge-trees": "^0.1.3",
    "broccoli-static-compiler": "^0.1.4",
    "ember-cli": "0.0.28",
    "express": "^4.1.1",
    "glob": "^3.2.9",
    "loom-generators-ember-appkit": "^1.1.1",
    "originate": "0.1.5"
  },
  "dependencies": {
    "body-parser": "^1.2.0",
    "broccoli-asset-rev": "0.0.5",
    "broccoli-coffee": "^0.1.0",
    "broccoli-ember-hbs-template-compiler": "^1.5.0",
    "broccoli-merge-trees": "^0.1.3",
    "broccoli-static-compiler": "^0.1.4",
    "ember-cli": "0.0.28",
    "express": "^4.1.1",
    "glob": "^3.2.9",
    "loom-generators-ember-appkit": "^1.1.1",
    "originate": "0.1.5"
  }
//...
```

This buildpack is meant to be used via the
[heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi) in
combination with the
[heroku-buildpack-nodejs](https://github.com/heroku/heroku-buildpack-nodejs)
and my fork of the
[nginx-buildpack](https://github.com/JonathanTron/nginx-buildpack).

First install the `heroku-buildpack-multi` to your app:

```bash
$ heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
```

then create a `.buildpacks` file in the root of your app with the following
content:

```
https://github.com/heroku/heroku-buildpack-nodejs.git
https://github.com/JonathanTron/heroku-buildpack-embercli-static.git
https://github.com/JonathanTron/nginx-buildpack.git#no-app
```

Create a custom nginx config file for the `nginx-buildpack` in
`config/nginx.conf.erb` in you app root directory with a content similar to the
following:

```nginx
daemon off;
#Heroku dynos have at least 4 cores.
worker_processes <%= ENV['NGINX_WORKERS'] || 4 %>;

events {
  use epoll;
  accept_mutex on;
  worker_connections 1024;
}

http {
  gzip on;
  gzip_comp_level 2;
  gzip_min_length 512;

  server_tokens off;

  log_format l2met 'measure#nginx.service=$request_time request_id=$http_x_request_id';
  access_log logs/nginx/access.log l2met;
  error_log logs/nginx/error.log;

  include mime.types;
  default_type application/octet-stream;
  sendfile on;

  #Must read the body in 5 seconds.
  client_body_timeout 5;

  server {
    listen <%= ENV["PORT"] %>;
    server_name _;
    keepalive_timeout 5;

    root <%= ENV["HOME"] %>/dist;

    location / {
      try_files $uri /index.html;

      // uncomment the following lines if you want to force https
      //if ( $http_x_forwarded_proto != 'https' ) {
      //  return 301 https://$host$request_uri;
      //}
      //add_header Strict-Transport-Security "max-age=31536000; includeSubDomains;";
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|eot|svg|ttf)$ {
      expires max;

      // uncomment the following lines if you want to force https
      //if ( $http_x_forwarded_proto != 'https' ) {
      //  return 301 https://$host$request_uri;
      //}
      //add_header Strict-Transport-Security "max-age=31536000; includeSubDomains;";

      break;
    }
  }
}
```

Commit and push your changes to Heroku.

It will do the following:

1. Use the `heroku-buildpack-nodejs` to install your `ember-cli` app's dependencies.
2. Use the `heroku-buildpack-embercli-static` (this buildpack) to fetch you `bower`
   dependencies and then `ember build` you app to the `dist/` directory.
3. Use the `nginx-buildpack` to serve your app as static assets.

License
=======

Copyright (c) 2014 Jonathan Tron

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
