# Elm on Heroku

*June 14, 2015*

## tl;dr

I wrote a [buildpack](https://github.com/srid/heroku-buildpack-elm) to simplify deployment of [Elm](http://elm-lang.org) apps on Heroku. You only need to provide `elm-package.json` and `app.json`, the later specifying the elm-make command for building your app.

## What is Heroku?

[Heroku](http://heroku.com) is an application hosting platform that allows you to just "focus on the app" and not fret about the details of hosting, servers and maintenance. This is a win for us developers whose time is now freed up to work on ideas that matter. 

Applications deployed to Heroku generally use one of the available buildpacks that specify how to "build" and prepare the app for running.

## Elm buildpack

You can use my [buildpack](https://github.com/srid/heroku-buildpack-elm) for deploying your Elm apps. Behind the scenes, it runs `elm package` and `elm make` before serving the generated files using the [warp](https://hackage.haskell.org/package/wai-app-static) static web server (or [spas](https://github.com/srid/spas)). 

Since different Elm apps invoke `elm make` in different manner, you are required to configure this value in the Heroku-specific `app.json` file. Take a look at the [app.json of elm-todomvc](https://github.com/evancz/elm-todomvc/blob/master/app.json) containing two specific config points:

* `ELM_COMPILE`: This specificies the `elm make` command to build your Elm app. Example: `"elm make Todo.elm"`
* `ELM_STATIC_DIR`: This specifies the directory, containing html/js/css, that the web server should use. Example: `"."`

[Chronicle's app.json](https://github.com/srid/chronicle/blob/master/app.json) is a little more non-trivial in that the compile step is delegated to Makefile, and the static directory also contains other assets copied over during the compile step.

## One-click deployment

All of this buildpack business further facilitates deploying Elm apps to Heroku with a single click. Take a look at the README of [elm-todomvc](https://github.com/evancz/elm-todomvc), and click the purple "Deploy to Heroku" button. If you already have a Heroku account, this should take you straight to the form where you can deploy and open the application in a matter of seconds. 

[Give this a try](https://github.com/evancz/elm-todomvc) with elm-todomvc now!

## Inside the buildpack

Internally the buildpack [pulls](https://github.com/srid/heroku-buildpack-elm/blob/master/bin/compile#L1) versioned elm (and warp/spas) binaries from Amazon S3. These binaries were built once, using docker, before being uploaded to S3 for re-use. Furthermore, Heroku provides a caching mechanism thereby making later deployments of the same app not re-download the same binaries again. The caching mechanism is also used to cache elm-package installs. This leads to quicker deployment time ... essentially running only `elm make ...`.

## Deploy your Elm app on Heroku

Take a look the [elm-todomvc](https://github.com/evancz/elm-todomvc) repo; you specifically need the `app.json` file. That's all. Just deploy [from the terminal](https://devcenter.heroku.com/articles/git) or using the [one-click button](https://devcenter.heroku.com/articles/heroku-button).
