# cljs-lambda

A Lein plugin, template and small Clojurescript library for exposing functions
via [AWS Lambda](http://aws.amazon.com/documentation/lambda/).

 - Low instance warmup penalty
 - Specify execution roles and resource limits in project definition
 - Use [core.async](https://github.com/clojure/core.async) for deferred completion
 - Smaller zip files with `:optimizations` `:advanced` support
 - [Blog post/tutorial](https://nervous.io/clojure/clojurescript/aws/lambda/node/lein/2015/07/05/lambda/)

# Examples

## Get Started

```sh
$ lein new cljs-lambda my-lambda-project
$ cd my-lambda-project
$ lein cljs-lambda default-iam-role
$ lein cljs-lambda deploy
$ lein cljs-lambda invoke work-magic '{"variety": "black"}'
```

The above requires a recent [Node](https://nodejs.org/) runtime, and a properly-configured (`aws configure`) [AWS CLI](https://github.com/aws/aws-cli) installation **>= 1.7.31**.  Please run `pip install --upgrade awscli` if you're using an older version (`aws --version`).

Or, put:

[![Clojars
Project](http://clojars.org/io.nervous/lein-cljs-lambda/latest-version.svg)](http://clojars.org/io.nervous/lein-cljs-lambda)

In your Clojurescript project's `:plugins` vector.

## A Simple Function

```clojure
(ns cljs-lambda-example.cat
  (:require [cljs-lambda.util :refer [async-lambda-fn]]))

(def ^:export meow
  (async-lambda-fn
   (fn [{meow-target :name} context]
     (go
       (<! (async/timeout 1000))
       {:from "the-cat"
        :to meow-target
        :message "I'm  meowing at you"}))))
```

[And its associated project.clj](https://github.com/nervous-systems/cljs-lambda/blob/master/example/project.clj)

# The Plugin

## project.clj Excerpt
See [the example project](https://github.com/nervous-systems/cljs-lambda/blob/master/example/project.clj):

```clojure
{:cljs-lambda
 {:cljs-build-id "cljs-lambda-example"
  :aws-profile "XYZ"
  :defaults
  {:role    "arn:aws:iam::151963828411:role/lambda_basic_execution"}
  :functions
  [{:name   "dog-bark"
    :invoke cljs-lambda-example.dog/bark}
   {:name   "cat-meow"
    :invoke cljs-lambda-example.cat/meow}]}}
```

If `:aws-profile` is present, the value will be passed as `--profile` to all invocations of the AWS CLI, otherwise the CLI's default profile will be used.

## Function Configuration

The following keys are valid for entries in `[:cljs-lambda :functions]`.  The optional keys are listed at the end, alongside their defaults:

```clojure
{:name "the-lambda-function-name"
 :invoke my-cljs-namespace.module/fn
 :role "arn:..."
 [:description "I don't think this field sees much action"
  :create true
  :timeout 3 ;; seconds
  :memory-size 128]} ;; MB
```

Values in `[:cljs-lambda :defaults]` will be merged into each function map.  These values are written to Lambda on  `deploy`.  Alternatively:

```sh
$ lein cljs-lambda update-config
```

Will update the remote (Lambda) configuration of all of the functions listed in the project file.  The `:create` key, defaulting to `true`, determines whether a `create-function` Lambda command will be issued if an attempt is made to `deploy` a not-yet-existing Lambda function.

## cljsbuild

The plugin depends on `cljsbuild`, and assumes there is a `:cljsbuild` section
in your `project.clj`.  A deployment or build via `cljs-lambda` invokes
`cljsbuild` - it'll run either the first build in the `:builds` vector, or the
one identified by `[:cljs-lambda :cljs-build-id]`.

 - Source map support will be enabled if the `:source-map` key of the active build
is `true`.
 - If `:optimizations` is set to `:advanced` on the active build, the zip output will be structured accordingly (i.e. it'll only contain `index.js` and the single compiler output file).
 - With `:advanced`, `*main-cli-fn*` is required to be set (i.e. `(set! *main-cli-fn* identity)`)

## Limitations

 - `deploy` means "deploy all functions in this project".  It could be changed pretty easily.
 - I guess we ought to use the docstring for `:description`, if none is supplied.
 - PR's welcome.

# The Library

[![Clojars Project](http://clojars.org/io.nervous/cljs-lambda/latest-version.svg)](http://clojars.org/io.nervous/cljs-lambda)

It's pretty tiny - please see the [example project](https://github.com/nervous-systems/cljs-lambda/tree/master/example/src/cljs_lambda_example), or the project generated by `lein new cljs-lambda`.

# Invoking

## CLI

```sh
$ lein cljs-lambda invoke my-lambda-fn '{"arg1": "value" ...}'
```

## Programmatically

If you're interested in programmatically invoking Lambda functions from Clojure/Clojurescript, it's pretty easy with [eulalie](https://github.com/nervous-systems/eulalie):

```clojure
(eulalie.lambda.util/invoke!
 {:access-key ... :secret-key ... [:token :region etc.]}
 "my-lambda-fn"
 :request-response
 {:arg1 "value" :arg2 ["value"]})
```

## License

cljs-lambda is free and unencumbered public domain software. For more
information, see http://unlicense.org/ or the accompanying UNLICENSE
file.
