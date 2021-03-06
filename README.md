<p align="center">
  <img src="https://raw.githubusercontent.com/docelic/amber-introduction/master/amber.png">
  <h3 align="center"><strong>Introduction to the Amber Web Framework</strong><br>
  And its Out-of-the-Box Features</h3>
  <p align="center">
    <sup>
      <i>
				Amber makes building web applications easy, fast, and enjoyable.
      </i>
    </sup>
  </p>
  <p align="center">
  </p>
</p>

# Introduction

**Amber** is a web application framework written in [Crystal](http://www.crystal-lang.org). Homepage is at [amberframework.org](https://amberframework.org/), docs are on [Amber Docs](https://docs.amberframework.org), Github repository is at [amberframework/amber](https://github.com/amberframework/amber), and the chat is on [Gitter](https://gitter.im/amberframework/amber) or FreeNode IRC channel #amber.

Amber is inspired by Kemal, Rails, Phoenix and other frameworks. It is simple to get used to, and much more intuitive than frameworks like Rails. (But it does inherit some concepts from Rails that are good.)

This document is here to describe everything that Amber offers out of the box, sorted in a logical order and easy to consult repeatedly over time. The Crystal level is not described; it is expected that the readers coming here have a formed understanding of [Crystal and its features](https://crystal-lang.org/docs/overview/).

# Installation

```shell
git clone https://github.com/amberframework/amber
cd amber
make # The result of 'make' is one file -- command line tool bin/amber

# To install the file, or to symlink the system-wide executable to current directory, run one of:
make install # default PREFIX is /usr/local
make install PREFIX=/usr/local/stow/
make force_link # can also specify PREFIX=...
```

After installation or linking, `amber` is the command you will be using
for creating or managing Amber apps.

# Creating New Amber App

```shell
amber new <app_name> [-d DATABASE] [-t TEMPLATE_LANG] [-m ORM_MODEL] [--deps]
```

Supported databases are [PostgreSQL](https://www.postgresql.org/) (pg, default), [MySQL](https://www.mysql.com/) (mysql), and [SQLite](https://sqlite.org/) (sqlite).

Supported template languages are [slang](https://github.com/jeromegn/slang) (default) and [ecr](https://crystal-lang.org/api/0.21.1/ECR.html).

Slang is extremely elegant, but very different from the traditional perception of HTML.
ECR is HTML-like, very similar to Ruby ERB, and more than mediocre when compared to slang, but it may be the best choice for your application if you intend to use some HTML site template (from e.g. [themeforest](https://themeforest.net/)) whose pages are in HTML + CSS or SCSS. (Or you could also try [html2slang](https://github.com/docelic/html2slang/) which converts HTML pages into slang.)

In any case, you can combine templates in various languages in a project, and regardless of the language, have in mind that the templates are compiled into the application. There is no lookup on disk or choosing between available templates during runtime. This makes templates extremely fast, as well as read-only which is a very welcome side-benefit!

Supported ORM models are [granite](https://github.com/amberframework/granite-orm) (default) and [crecto](https://github.com/Crecto/crecto).

Granite is a very nice and simple, effective ORM model, where you mostly write your own SQL (i.e. all search queries typically look like YourModel.all("WHERE field1 = ? AND field2 = ?", [value1, value2])). But it also has belongs/has relations, and some other little things. (If you have by chance known and loved [Class::DBI](http://search.cpan.org/~tmtm/Class-DBI-v3.0.17/lib/Class/DBI.pm) for Perl, it might remind you of it in some ways.)

Supported migrations engine is [micrate](https://github.com/juanedi/micrate). Micrate is very simple and you basically write raw SQL in your migrations. There are just two keywords in the migration file which give instructions whether the SQLs that follow pertain to migrating up or down. These keywords are "-- +micrate Up" and "-- +micrate Down".

If argument --deps is provided, Amber will automatically run `crystal
deps` in the new directory to install shards.

# Running the App

The app can be started as soon as you have created it and ran `crystal deps` in the app directory.
(It is not necessary to run deps if you have invoked `amber new` with the argument --deps; in that case Amber did it for you.)

To run it, you can use a couple different approaches. Some are of course suitable for development, some for production, etc.:

```shell
# For development, clean and simple - compiles and runs your app:
crystal src/<app_name>.cr

# For development, clean and simple - compiles and runs your app, but
# also watches for changes in files and rebuilds/re-runs automatically.
amber watch

# For production, compiles app with optimizations and places it in bin/app.
# Crystal by default compiles using 8 threads (tune if needed with --threads NUM)
crystal build --no-debug --release --verbose -t -s -p -o bin/<app_name> src/<app_name>.cr
```

Please note that Granite currently has problems in edge cases. For example, if you create a new model but do not specify any fields for it, then until you add at least one field, Amber won't start due to a compile error ([#112](https://github.com/amberframework/granite-orm/issues/112)).

Amber by default uses a feature called "port reuse" available in newer Linux kernels. If you get an error "setsockopt: Protocol not available", it means your kernel does not have it. Please edit `config/environments/development.yml` and set "port_reuse" to false.

# Building the App and Troubleshooting

The application is always built, regardless of whether one is using the Crystal command 'run' (the default) or 'build'. It is just that in run mode, the resulting binary won't be saved to a file, but will be executed and later discarded.

For faster build speed, development versions are compiled without the --release flag. With the --release flag, the compilation takes noticeably longer, but the resulting binary has incredible performance.

Crystal caches partial results of the compilation (*.o files etc.) under `~/.cache/crystal/` for faster subsequent builds. This directory is also where temporary binaries are placed when one runs programs with `crystal [run]` rather than `crystal build`.

Sometimes building the app will fail on the C level because of missing header files or libraries. If Crystal doesn't print the actual C error, it will at least print the compiler line that caused it.

The best way to see the actual error from there is to copy-paste the command printed and run it manually in the terminal. The error will be shown and from there the cause will be determined easily.

There are some issues with the `libgc` library here and there. Crystal comes with built-in `libgc`, but it may conflict with the system one. In my case the solution was to install and then remove package `libgc-dev`.

# REPL

Often times, it is very useful to enter an interactive console (think of IRB shell) with all application classes initialized etc. In Ruby this would be done with IRB or with a command like `rails console`.

Due to its nature, Crystal does not have a free-form [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop), but you can save and execute scripts in the context of the application. One way to do it is via command `amber x [filename]`. This command will allow you to type or edit the contents, and then execute the script.

Another, more professional way to do it is via standalone REPL-like script tools [cry](https://github.com/elorest/cry) and [icr](https://github.com/crystal-community/icr). `cry` began as an experiment and a predecessor to `amber x`, but now offers additional functionality such as repeatedly editing and running the script if `cry -r` is invoked.

In any case, running a script "in application context" simply means requiring `config/application.cr` (and through it, `config/**`), Therefore, be sure to list all your requires in `config/application.cr` so that everything works as expected.

# File Structure

So, at this point you might be wanting to know what's placed where in an Amber application. The default structure looks like this:

```
./config/                  - All configuration
./config/application.cr    - Main configuration file
./config/initializers/     - Initializers (files loaded at very beginning)
./config/environments/     - Environment-specific YAML configurations
./config/webpack/          - Webpack (asset bundler) configuration
./config/routes.cr         - All routes
./db/migrations/           - All DB migration files (created with 'amber g migration ...')
./public/                  - The "public" directory for static files
./public/dist/             - Directory inside "public" for generated files and bundles
./public/dist/images/
./src/                     - Main source directory, with <app_name>.cr being the main/entry file
./src/controllers/         - All controllers
./src/models/              - All models
./src/views/layouts/       - All layouts
./src/views/               - All views
./src/views/home/          - Views for HomeController (path "/")
./src/assets/              - Static assets which will be bundled and placed into ./public/dist/
./src/assets/stylesheets/
./src/assets/fonts/
./src/assets/images/
./src/assets/javascripts/
./spec/                    - Tests (named *_spec.cr)
```

I prefer to have some of these directories accessible directly in the root directory of the application and to have the config directory named `etc`, so I run:

```
ln -sf config etc
ln -sf src/assets
ln -sf src/controllers
ln -sf src/models
ln -sf src/views
ln -sf src/views/layouts
```

# Database Commands

Amber provides a group of commands under the 'db' group to allow working with the database. The simple commands you will most probably want to run just to see basic things working are:

```shell
amber db create
amber db status
amber db version
```

Before these commands will work, you will need to configure database
credentials:

First, create a user to access the database. For PostgreSQL, this is done by invoking something like:

```shell
$ sudo su - postgres
$ createuser -dElPRS myuser
Enter password for new role: 
Enter it again: 
```

Then, edit `config/environments/development.yml` and configure "database_url:" to match your settings. If nothing else, the part that says "postgres:@" should be replaced with "yourusername:yourpassword@".

And then try the database commands from the beginning of this section.

Please note that for the database connection to succeed, all parameters must be correct (hostname, port, username, password, database name), the database server must be accessible, and the database must actually exist (unless you are invoking 'amber db create' to create it). In case of *any error in any of the stages* of connecting to the database, the error message will be very terse and just say "Connection unsuccessful: <database_url>". The solution is simple, though - simply use the printed database_url to manually attempt a connection to the database with the same parameters, and the problem will most likely quickly reveal itself.

Please note that the environment files for non-production environment are given in plain text. Environment file for the production environment is encrypted for additional security and can be seen or edited by invoking `amber encrypt`.

# Routes

Routes are very easy to understand. Routes connect HTTP methods (and the paths with which they were invoked) to controllers and methods on the Amber side.

Amber includes a wonderful command `amber routes` to display current routes. By default, the routes table looks like the following:

```shell
$ amber routes

╔══════╦═══════════════════════════╦════════╦══════════╦═══════╦═════════════╗
║ Verb | Controller                | Action | Pipeline | Scope | URI Pattern ║
╠──────┼───────────────────────────┼────────┼──────────┼───────┼─────────────╣
║ get  | Amber::Controller::Static | index  | static   |       | /*          ║
╠──────┼───────────────────────────┼────────┼──────────┼───────┼─────────────╣
║ get  | HomeController            | index  | web      |       | /           ║
╚══════╩═══════════════════════════╩════════╩══════════╩═══════╩═════════════╝
```

From this example, we see that a "GET /" request will instantiate
HomeController and then call method index() in it. The return value of
the method will be returned to the client.

Similarly, here's an example of a route that would route HTTP POST requests to "/registration" to the method create() in class RegistrationController:

```
post "/registration", RegistrationController, :create
```

Standard HTTP verbs (GET, HEAD, POST, PUT, PATCH, DELETE) by convention go to standard methods on the controllers (show, new, create, edit, update, destroy). However, there is nothing preventing you from routing URLs to any methods you want in the controllers, such as we've done with "index" above.

Websocket routes are supported too.

The DSL language specific to `config/routes.cr` file is defined in [dsl/router.cr](https://github.com/amberframework/amber/blob/master/src/amber/dsl/router.cr) and [dsl/server.cr](https://github.com/amberframework/amber/blob/master/src/amber/dsl/server.cr).

It gives you the following top-level commands/blocks:

```
# Define a pipeline
pipeline :name do
  ...
end

# Group a set of routes
routes :name, "path" do
  ...
end
```

Such as:

```crystal
Amber::Server.configure do |app|
  pipeline :web do
    # Plug is the method used to connect a pipe (middleware)
    # A plug accepts an instance of HTTP::Handler
    plug Amber::Pipe::Logger.new
  end

  routes :web do
    get "/", HomeController, :index    # Routes to HomeController::index()
    get "/test", PageController, :test # Routes to PageController::test()
  end
end
```

Within 'routes', the following commands are available:

```crystal
get, post, put, patch, delete, options, head, trace, connect, websocket, resources
```

`resources` is a macro defined as:

```crystal
    macro resources(resource, controller, only = nil, except = nil)
```

And unless it is confined with arguments `only` or `except`, it will automatically define get, post, put, patch, and delete routes for your resource and route them to the following methods in the controller:

```crystal
index, new, create, show, edit, update, destroy
```

Please note that it is not currently possible to define a different behavior for HEAD and GET methods ont he same path, because if a GET is defined it will also automatically add the matching HEAD route. That will result in two HEAD routes existing for the same path and trigger error `Amber::Exceptions::DuplicateRouteError`.

# Views

Information about views can be summarized in bullet points:

- Views in Amber are located in `src/views/`
- They are rendered using `render()`
- The first argument given to `render()` is the template name (e.g. `render("index.slang")`)
- If we are in the context of a controller, `render("index.slang")` will look for view using the path `src/views/<controller_name>/index.slang`
- If we are not rendering a partial, by default the template will be wrapped in a layout
- If the layout name isn't given, the default layout will be `views/layouts/application.slang`
- There is no unnecessary magic applied to template names &mdash; name given is the name that is looked up on disk
- Partials begin with "_" by convention, but that is not required
- To render a partial, use `render( partial: "_name.ext")`

# Variables in Views

In Amber, templates are compiled in the same scope as controller methods. This means you do not need instance variables for passing the information from controllers to views.

Any variable you define in the controller method is automagically visible in the template. For example, let's add the current date and time display to our /about page:

```shell
$ vi src/controllers/page_controller.cr

def about
	time = Time.now
	render "about.ecr"
end

$ vi src/views/page/about.ecr

Hello, World! The time is now <%= time %>.
```

Templates are actually executing in the controller class. If you do "<%= self.class %> in the above example, the response will be "PageController". So all the methods and variables you have on the controller are also available in views rendered from it.

# Starting the Server

It is important to explain exactly what is happening from when you run the application til Amber starts serving the aplication:

1. `crystal src/<app_name>.cr` - you or a script starts Amber
	1. `require "../config/*"` - as the first thing, `config/*` is required. Inclusion is in alphabetical order. Crystal only looks for *.cr files and only files in config/ are loaded (no subdirectories)
		1. `require "../config/application.cr"` - this is usually the first file in `config/`
			1. `require "./initializers/**"` - loads all initializers. There is only one initializer file by default, named `initializer/database.cr`. Here we have a double star ("**") meaning inclusion of all files including in subdirectories. Inclusion is always current-dir first, then depth
			1. `require "amber"` - Amber itself is loaded
				1. Loading Amber makes `Amber::Server` class available
				1. `include Amber::Environment` - already in this stage, environment is determined and settings are loaded from yml file (e.g. from `config/environments/development.yml`. Settings are later available as `settings`
			1. `require "../src/controllers/application_controller"` - main controller is required. This is the base class for all other controllers
				1. It defines `ApplicationController`, includes JasperHelpers in it, and sets default layout ("application.slang").
			1. `require "../src/controllers/**"` - all other controllers are loaded
			1. `Amber::Server.configure` block is invoked to override any config settings
		1. `require "config/routes.cr"` - this again invokes `Amber::Server.configure` block, but concerns itself with routes and feeds all the routes in
	1. `Amber::Server.start` is invoked
		1. `instance.run` - implicitly creates a singleton instance of server, saves it to `@@instance`, and calls `run` on it
		1. Consults variable `settings.process_count`
		1. If process count is 1, `instance.start` is called
		1. If process count is > 1, the desired number of processes is forked, while main process enters sleep
			1. Forks invoke Process.run() and start completely separate, individual processes which go through the same initialization procedure from the beginning. Forked processes have env variable "FORKED" set to "1", and a variable "id" set to their process number. IDs are assigned in reverse order (highest number == first forked).
		1. `instance.start` is called for every process
			1. It saves current time and prints startup info
			1. `@handler.prepare_pipelines` is called. @handler is Amber::Pipe::Pipeline, a subclass of Crystal's HTTP::Handler. `prepare_pipelines` is called to connect the pipes so the processing can work, and implicitly adds Amber::Pipe::Controller (the pipe in which app's controller is invoked) as the last pipe. This pipe's duty is to call Amber::Router::Context.process_request, which actually dispatches the request to the controller.
			1. `server = HTTP::Server.new(host, port, @handler)`- Crystal's HTTP server is created
			1. `server.tls = Amber::SSL.new(...).generate_tls if ssl_enabled?`
			1. Signal::INT is trapped (calls `server.close` when received)
			1. `loop do server.listen(settings.port_reuse) end` - server enters main loop

# Serving Requests

Similarly as with starting the server, is important to explain exactly what is happening when Amber is serving requests:

Amber's app serving model is based on Crystal's built-in, underlying functionality:

1. The server that is running is an instance of Crystal's
	 [HTTP::Server](https://crystal-lang.org/api/0.24.1/HTTP/Server.html)
2. On every incoming request, handler is invoked. As supported by Crystal, handler can be simple Proc or an instance of HTTP::Handler. HTTP::Handlers have a concept of "next" and multiple ones can be connected in a row. In Amber, these individual handlers are called "pipes", and the overall handler (complete chain of pipes) is Amber::Pipe::Pipeline, which, for convenience, is also a subclass of [HTTP::Handler](https://crystal-lang.org/api/0.24.1/HTTP/Handler.html). This handler, even though named Pipeline and implying a specific pipeline, is actually aware of all pipelines and can identify and trigger the appropriate one
3. In the pipeline, every Pipe (Amber::Pipe::*, ultimately subclass of Handler) is invoked, with one argument. That argument is
	 by convention called "context" and it is an instance of `HTTP::Server::Context`, which has two built-in methods &mdash; `request` and `response`, to access the request and response parts respectively. On top of that, Amber adds various other methods and variables, such as `router`, `flash`, `cookies`, `session`, `content`, `route`, and others as seen in [src/amber/router/context.cr](https://github.com/amberframework/amber/blob/master/src/amber/router/context.cr)
4. Please note that calling the chain of pipes is not automatic; every pipe needs to call `call_next(context)` at the appropriate point in its execution to call the next pipe in a row. It is not necessary to check whether the next pipe exists, because currently `Amber::Pipe::Controller` is always implicitly added as the last pipe, so at least one does exist. State between pipes is not passed via variables but via modifying `context` and the data contained in it

After that, pipelines, pipes, routes, and otherAmber-specific parts come into play.

So, in detail, from the beginning:

1. `loop do server.listen(settings.port_reuse) end` - main loop is running
	1. `spawn handle_client(server.accept?)` - handle_client() is called in a new fiber after connection is accepted
		1. `io = OpenSSL::SSL::Socket::Server.new(io, tls, sync_close: true) if @tls`
		1. `@processor.process(io, io)`
			1. `if request.is_a?(HTTP::Request::BadRequest); response.respond_with_error("Bad Request", 400)`
			1. `response.version = request.version`
			1. `response.headers["Connection"] = "keep-alive" if request.keep_alive?`
			1. `context = Context.new(request, response)` - this context is already extended with Amber's extensions in [src/amber/router/context.cr](https://github.com/amberframework/amber/blob/master/src/amber/router/context.cr)
			1. `@handler.call(context)` - `Amber::Pipe::Pipeline.call()` is called
				1. `raise ...error... if context.invalid_route?` - route validity is checked early
				1. `if context.websocket?; context.process_websocket_request` - if websocket, parse as such
				1. `elsif ...; ...pipeline.first...call(context)` - if reqular HTTP request, call the first handler in the appropriate pipeline
					1. `call_next(context)` - each pipe calls call_next(context) somewhere during its execution, and all pipes are executed
						1. `context.process_request` - the always-last pipe (Amber::Pipe::Controller) calls `process_request` to dispatch the action to controller. After that last pipe, the stack of call_next()s is "unwound" back to the starting position
					1. `context.finalize_response` - minor final adjustments to response are made (headers are added, and response body is printed unless action was HEAD)

# Static Pages

It can be pretty much expected that a website will need a set of simple, "static" pages. Those pages are served by the application, but mostly don't use a database nor any complex code. Such pages might include About and Contact pages, Terms of Conditions, etc. Making this work is trivial.

Let's say that, for simplicity and grouping, we want all "static" pages to be served by PageController. We will group all these pages under a common web-accessible prefix of /page/, and finally we will route page requests to PageController's methods. (Because these pages won't be objects, we won't need a model or anything else other than one controller method and one view per each page.)

Let's start by creating a controller:

```shell
amber g controller page
```

Afterwards, we edit `config/routes.cr` to link URL "/about" to method about() in PageController. We do this inside the "routes :web" block:

```
routes :web do 
  ...
  get "/about", PageController, :about
  ...
end
```

Then, we edit the controller and actually add method about(). This method can just directly return some string in response, or it can render a view, and then the expanded view contents will be returned as the response.

```shell
$ vi src/controllers/page_controller.cr

# Inside the file, we add:

def about
  # "return" can be omitted here. It is included only for clarity.
  return render "about.ecr"
end
```

Since this is happening in the "page" controller, the view directory for finding the templates defaults to `src/views/page/`. We will create the directory and the file "about.ecr" in it:

```shell
$ mkdir -p src/views/page/
$ vi src/views/page/about.ecr

# Inside the file, we add:

Hello, World!
```

Because we have called render() without additional arguments, the template will default to being rendered within the default application layout, `views/layouts/application.cr`.

And that's it! Visiting `/about` will go to the router, router will invoke `PageController::about()`, that method will render template `src/views/page/about.ecr` in the context of layout `views/layouts/application.cr`, and the result of rendering will be a full page with content `Hello, World!` in the body. That result will be returned to the controller, and from there it will be returned to the client.

# Assets Pipeline

In an Amber project, raw assets are in `src/assets/`:

```shell
src/assets/
src/assets/fonts
src/assets/images
src/assets/javascripts
src/assets/javascripts/main.js
src/assets/stylesheets/main.scss
```

At build time, all these are processed and placed under `public/dist/`.
The JS resources are bundled to `main.bundle.js` and CSS resources are bundled to `main.bundle.css`.

Currently, webpack is being used for asset management. I recommend replacing it with at least [Parcel](https://parceljs.org/). Finding a non-js/non-node/non-npm application for this purpose would be even better; please let me know if you know one.

This section will be expanded to include a full replacement procedure. (In general it seems it shouldn't be much more complex than replacing the command and development dependencies in project's `package.json` file.)

# Default Shards

By default, Amber project depends on just a few shards:

```
amberframework/amber          - Obviously, must depend on Amber
amberframework/granite-orm    - Database ORM
amberframework/quartz-mailer  - Sending and receiving emails
amberframework/jasper-helpers - Helpers for working with HTML in Amber/Crystal
will/crystal-pg               - PostgreSQL connector
amberframework/garnet-spec    - Extended Crystal specs for testing web applications
```

In turn, these depend on:

```
luislavena/radix                      - Radix Tree implementation
jeromegn/kilt                         - Generic template interface
jeromegn/slang                        - Slang template language
stefanwille/crystal-redis             - 
amberframework/cli                    - Building cmdline apps (based on mosop)
mosop/optarg                          - Parsing cmdline args
mosop/callback                        - Defining and invoking callbacks
mosop/string_inflection               - Word plurals, counts, etc.
amberframework/teeplate               - Rendering multiple template files
juanedi/micrate                       - Database migration tool
crystal-lang/crystal-db               - Common DB API
jwaldrip/shell-table.cr               - Creates textual tables in shell
askn/spinner                          - Spinner for the shell
crystal-lang/crystal-mysql            - 
crystal-lang/crystal-sqlite3          - 
amberframework/smtp.cr                - SMTP client (to be replaced with arcage/crystal-email)
ysbaddaden/selenium-webdriver-crystal - Selenium Webdriver client
```

And basic Crystal's build-in shards:

```
http
logger
json
colorize
random/secure
```

Only the parts that are used end up in the compiled project.

Now let's take a tour of all the important classes that exist in the Amber application and are useful for understanding the flow.

# Extensions

Amber adds some very convenient extensions to existing String and Number classes. The extensions are in the [extensions/](https://github.com/amberframework/amber/tree/master/src/amber/extensions) directory, but here's a listing of the current ones:

For String:

```crystal
      def str?
      def email?
      def domain?
      def url?
      def ipv4?
      def ipv6?
      def mac_address?
      def hex_color?
      def hex?
      def alpha?(locale = "en-US")
      def numeric?
      def alphanum?(locale = "en-US")
      def md5?
      def base64?
      def slug?
      def lower?
      def upper?
      def credit_card?
      def phone?(locale = "en-US")
      def excludes?(value)
      def time_string?
```

For Number:

```crystal
      def positive?
      def negative?
      def zero?
      def div?(n)
      def above?(n)
      def below?(n)
      def lt?(num)
      def self?(num)
      def lteq?(num)
      def between?(range)
      def gteq?(num)
```

# Support Routines

In [support/](https://github.com/amberframework/amber/tree/master/src/amber/support) directory there is a number of various support files that provide additional, ready made routines.

Currently, the following can be found there:

```
client_reload.cr      - Support for reloading developer's browser

file_encryptor.cr     - Support for storing/reading encrypted versions of files
message_encryptor.cr
message_verifier.cr

locale_formats.cr     - Very basic locate data for various, manually-added locales

mime_types.cr         - List of MIME types and helper methods for working with them:
                        def self.mime_type(format, fallback = DEFAULT_MIME_TYPE)
                        def self.zip_types(path)
                        def self.format(accepts)
                        def self.default
                        def self.get_request_format(request)
```

# Amber::Controller::Base

This is the base controller from which all other controllers inherit. Source file is in [src/amber/controller/base.cr](https://github.com/amberframework/amber/blob/master/src/amber/controller/base.cr).

On every request, the appropriate controller is instantiated and its initialize() runs. Since this is the base controller, this code runs on every request so you can understand what is available in the context of every controller.

The content of this controller and the methods it gets from including other modules are intuitive enough to be copied here and commented where necessary:

```crystal
module Amber::Controller
  class Base
    include Helpers::CSRF
    include Helpers::Redirect
    include Helpers::Render
    include Helpers::Responders
    include Helpers::Route
    include Callbacks

    protected getter context : HTTP::Server::Context
    protected getter params : Amber::Validators::Params

    delegate :cookies, :format, :flash, :port, :requested_url, :session, :valve,
      :request_handler, :route, :websocket?, :get?, :post?, :patch?,
      :put?, :delete?, :head?, :client_ip, :request, :response, :halt!, to: context

    def initialize(@context : HTTP::Server::Context)
      @params = Amber::Validators::Params.new(context.params)
    end
  end
end
```

[Helpers::CSRF](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/csrf.cr) provides these methods:

```crystal
    def csrf_token
    def csrf_tag
    def csrf_metatag
```

[Helpers::Redirect](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/redirect.cr) provides:

```crystal
    def redirect_to(location : String, **args)
    def redirect_to(action : Symbol, **args)
    def redirect_to(controller : Symbol | Class, action : Symbol, **args)
    def redirect_back(**args)
```

[Helpers::Render](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/render.cr) provides:

```crystal
    LAYOUT = "application.slang"
    macro render(template = nil, layout = true, partial = nil, path = "src/views", folder = __FILE__)
```

[Helpers::Responders](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/responders.cr) helps control what final status code, body, and content-type will be returned to the client.

[Helpers::Route](https://github.com/amberframework/amber/blob/master/src/amber/controller/helpers/route.cr) provides:

```crystal
    def action_name
    def route_resource
    def route_scope
    def controller_name
```

[Callbacks](https://github.com/amberframework/amber/blob/master/src/amber/dsl/callbacks.cr) provide:

```crystal
    macro before_action
    macro after_action
```

# Conclusion

We hope you have enjoyed this hands-on introduction to Amber!

Feel free to provide any feedback on content or additional areas you
would like to see covered in this guide. Thanks!
