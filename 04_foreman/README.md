# The Setup for Foreman

## The Goal

Using The Setup effectively, especially as part of a team, means coordinating
the way in which applications are run, even in development.
Environment-specific options should be determined by environment variables, and
port mappings for applications across an organization should be consistent.

[Foreman](http://ddollar.github.io/foreman/) provides a useful approach to both
of these considerations, by specifying the commands used to start an app's
processes and environment variables in dedicated files: `Procfile` and `.env`,
respectively.

## The Installation

Foreman is a Ruby gem. This means that installation is just:

    gem install foreman

## The Configuration

As mentioned above, there are two parts to setting up an app under Foreman: the
`Procfile` and the `.env` file.

### The Procfile

A `Procfile` is just a YAML file containing the names of processes that should
be launched and the command to launch them. A simple `Procfile` for a Rails app
running under puma might look like this:

    app: bundle exec rails s puma -p 3001 --binding 127.0.0.1

You can add additional key/value pairs to start multiple processes, such as
background workers, a [MailCatcher](http://mailcatcher.me/) server, and so on.
From that point, instead of running `rails s` manually, you run `foreman start`.

### The .env file

Environment-specific configuration (especially of the sensitive kind) belongs in
(surprise!) your environment. Foreman includes
[dotenv](https://github.com/bkeepers/dotenv), which provides great functionality
for loading environment variables from a `.env` file in a project's directory.

Let's say you have a Rails app that needs database credentials. Your
`config/database.yml` file might contain the following section...

```yaml
development:
  <<: *default
  database: accounts_development
  username: <%= ENV['DB_USERNAME'] %>
  password: <%= ENV['DB_PASSWORD'] %>
  host: <%= ENV['DB_HOSTNAME'] %>
```

...and the corresponding `.env` file might contain...

```sh
DB_HOSTNAME=localhost
DB_USERNAME=my_db_user
DB_PASSWORD=my_db_password
```

For consistency's sake, if the configuration information is sensitive in one
application environment, it should be injected via environment variables in
**all** applicaton environments. This way, anyone cloning the application's repo
can quickly see a list of all of the environment-specific settings that are
required by the app (across all environments).

When it comes to local machine environments, though, there's a good chance other
developers will want to have some say in how they are running things locally. As
such, I recommend against committing a `.env` file to version control.  Add
`.env` to your `.gitignore` (or equivalent) file, and instead commit a
`.env.example` file, with reasonable default values for development filled in.
This will prevent churn in version control at the cost of occasionally having
to sync up `.env` with `.env.example`. It's a good idea to check for the
presence of all required environment variables at app startup and bail out if
any are missing, to prevent a failure to update a local `.env` from exhibiting
weird bugs at runtime.

One final suggestion: if you have a large number of related configuration
settings that differ between environments but are not sensitive information, you
might wish to consider setting an `APP_ENV` environment variable and using it to
determine configuration files (or sections of files) to load. This can be useful
for setting up integrations to point to different hosts or use different API
keys depending on environment.

You might ask, "why don't you overload `RACK_ENV` to serve this purpose, in
Ruby?" This is because I prefer that everything but my app code think that a
"staging" environment is actually production.

The goal is to prevent a bunch of environment-based branching. Stuff like...

```ruby
do_something_special if Rails.env.production?
```

...is the source of many a subtle bug. You shouldn't be writing code like this,
but chances are, some code like this is already in your app, whether you wrote
it or not.

Since we're on the topic of staging environments, there's no time like the
present to discuss [setting up Docker](../05_docker/)!
