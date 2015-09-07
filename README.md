# Heroku buildpack: PHP

![buildpack_banner_php](https://cloud.githubusercontent.com/assets/51578/8779887/b85660f2-2ed3-11e5-9545-5694626ac85e.jpg)

This is the official [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for PHP applications.

It uses Composer for dependency management, supports PHP or HHVM (experimental) as runtimes, and offers a choice of Apache2 or Nginx web servers.

## Usage

You'll need to use at least an empty `composer.json` in your application.

    echo '{}' > composer.json
    git add composer.json
    git commit -m "add composer.json for PHP app detection"

If you also have files from other frameworks or languages that could trigger another buildpack to detect your application as one of its own, e.g. a `package.json` which might cause your code to be detected as a Node.js application even if it is a PHP application, then you need to manually set your application to use this buildpack:

    heroku buildpacks:set https://github.com/heroku/heroku-buildpack-php

Please refer to [Dev Center](https://devcenter.heroku.com/categories/php) for further usage instructions.

## Development

The following information only applies if you're forking and hacking on this buildpack for your own purposes.

### Compiling Binaries

You need an S3 bucket, referenced in `bin/compile`, to host your own binaries if you want custom ones.

The folder `support/build` contains [Bob](http://github.com/kennethreitz/bob-builder) build scripts for all binaries and dependencies.

To get started with it, create a Python app (*Bob* is a Python application) on Heroku inside a clone of this repository, and set your S3 config vars:

    $ heroku create --buildpack https://github.com/heroku/heroku-buildpack-python
    $ heroku ps:scale web=0
    $ heroku config:set WORKSPACE_DIR=/app/support/build
    $ heroku config:set AWS_ACCESS_KEY_ID=<your_aws_key>
    $ heroku config:set AWS_SECRET_ACCESS_KEY=<your_aws_secret>
    $ heroku config:set S3_BUCKET=<your_s3_bucket_name>
    $ heroku config:set S3_PREFIX=<optional_s3_subfolder_to_upload_to>

Then, shell into an instance and run a build by giving the name of the formula inside `support/build`:

    $ heroku run bash
    Running `bash` attached to terminal... up, run.6880
    ~ $ bob build php-5.5.11RC1
    
    Fetching dependencies... found 2:
      - libraries/zlib
      - libraries/libmemcached
    Building formula php-5.5.11RC1:
        === Building PHP
        Fetching PHP v5.5.11RC1 source...
        Compiling PHP v5.5.11RC1...

If this works, run `bob deploy` instead of `bob build` to have the result uploaded to S3 for you.

To speed things up drastically, it'll usually be a good idea to `heroku run bash --size PX` instead.

If the dependencies are not yet deployed, you can do so by e.g. running `bob deploy libraries/zlib`.


## Build sequence

Replace dependency version: 
    Find: Build Deps: php\-5\.[0-9a-z\.]+
    Replace: Build Deps: php-5.6.11

Build & Deploy:

bob deploy php
bob deploy nginx
bob deploy apache

bob deploy libraries/zlib --overwrite
bob deploy libraries/libmcrypt --overwrite
bob deploy libraries/icu --overwrite
bob deploy libraries/gettext --overwrite
bob deploy libraries/pcre --overwrite
bob deploy libraries/imagemagick --overwrite
bob deploy libraries/libmemcached --overwrite

bob deploy extensions/no-debug-non-zts-20131226/memcached-bare --overwrite
bob deploy extensions/no-debug-non-zts-20131226/memcached --overwrite
bob deploy extensions/no-debug-non-zts-20131226/redis --overwrite
bob deploy extensions/no-debug-non-zts-20131226/mongo --overwrite

bob deploy extensions/no-debug-non-zts-20131226/imagick-bare --overwrite
bob deploy extensions/no-debug-non-zts-20131226/imagick --overwrite

bob deploy extensions/no-debug-non-zts-20131226/phalcon --overwrite
bob deploy extensions/no-debug-non-zts-20131226/newrelic --overwrite

test2
