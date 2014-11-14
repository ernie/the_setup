# The Setup for Docker

## The Goal

Docker is amazing. It lets us control pretty much every aspect of the
environment in which our application runs except the kernel, with minimal
performance overhead and minimal hassle. A Docker image we build and test on our
Mac can run on our production server, and should behave identically. The image
can even have our app's code loaded and ready to go, if we like!

This makes Docker an ideal technology to use to spin up our own isolated staging
environment, and that's exactly what we want to use it for.

## The Installation

On OS X, we need to use boot2docker and VirtualBox in order to run a minimal
distro of Linux, since Docker works via the Linux kernel's containment features.
Fortunately, we can still do all of this via Homebrew, so we never have to see
an Oracle software installation window. We also want to use Fig, which can spin
up an entire fleet of docker containers.

    brew install caskroom/cask/brew-cask
    brew cask install virtualbox
    brew install boot2docker
    boot2docker upgrade
    brew install fig

## The Configuration

There are a few parts to this portion of the setup:

1. Shell
2. Dockerfile(s)
3. fig.yml

We'll tackle them in order. Please note that it isn't the intent of this guide
to be a comprehensive HOWTO on all things Docker and Fig, however. That's what
the [Docker docs](https://docs.docker.com/) and [Fig site](http://www.fig.sh/)
are for.

### The Shell

Running `boot2docker up` will start the Docker VM and spit out some instructions
about how to configure your environment so that the `docker` command works as
expected. That's helpful, but not terribly convenient. We could define the
environment variables statically in our `.bashrc`, `.zshrc`, or whatever, but I
find that I don't want the Docker VM running all the time, anyway. Consequently,
I opted to define a few simple functions in my `.zshrc` (this should work just
fine for Bash users as well):

```sh
function b2d() {
  if [ $(boot2docker status) = 'running' ]; then
    $(boot2docker shellinit 2> /dev/null)
  else
    echo 'Docker VM is not running. Starting...'
    boot2docker up > /dev/null 2>&1
    $(boot2docker shellinit 2> /dev/null)
  fi
}

function docker() {
  b2d
  /usr/bin/env docker $@
}

function fig() {
  b2d
  /usr/bin/env fig $@
}
```

By masking the `docker` and `fig` commands like this, we can ensure
the VM is up and running and the shell environment variables we need are
properly set, even if a future Docker upgrade changes them.

### The Dockerfile(s)

The specifics of your Dockerfiles will depend heavily on the type of app you're
setting up. For the purposes of this discussion, we're going to assume a simple
Rails application that requires access to a PostgreSQL database and Redis
server.

#### The Base Image

For the purposes of this guide, assume we have some opinions about the Linux
distro we want, the version of Ruby, and the standard configuration for an nginx
server that handles static assets in production. These will be the same for all
of our Ruby-based applications, so we want to build our own base image for Ruby
apps. We'll creatively name it `ruby-base`.

Let's create a new repository to contain our custom images, and copy over the
files included in this repository:

    mkdir docker-images
    cp -r ruby-base docker-images/ruby-base

By convention, we'll include a `config` subdirectory in any image's directory,
where we'll store various configuration files we intend to add to the image.

For this image, we've set up a simple [nginx.conf](ruby-base/config/nginx.conf)
that tries to serve static assets from `/app/public/`, and, that failing,
forwards HTTP requests to the unix socket file at `/tmp/app.sock`.

Taking a look at the [Dockerfile](ruby-base/Dockerfile) next, you'll see we are
basing the image on the official Ubuntu 14.04 LTS distribution, installing some
necessary packages to build Ruby, along with nginx and supervisor packages.
Supervisor is a Python app that's a bit like Foreman, but with some additional
features we'll find useful in production. It's designed to keep processes
running using various strategies, not unlike a supervisor in Erlang/OTP.

After installing packages, we build Ruby, update RubyGems and install Bundler,
then create an unprivileged account for use by our app (imaginatively named
"app").

Next we build and push our image to a registry:

    cd docker-images/ruby-base
    docker build -t <my-prefix>/ruby-base .
    docker push <my-prefix>/ruby-base

Your prefix is going to be either a user/organization name, if you're using the
official Docker registry, or the hostname of your private registry, if you've
got one of those going. In any case, once we've built and pushed this base
image, we're good to build our app's image on its foundation.

### The App Image

Here's where things get fun! We can bundle our app, precompile assets, and
basically create a version of our app that runs as if it's an appliance inside
our docker container.

    cp -r app-docker <my-rails-app>/docker
    cd <my-rails-app>
    ln -s docker/Dockerfile
    echo Dockerfile >> .gitignore

I like to create subdirectory in my apps named "docker" to hold all of the
Docker-related files (including the Dockerfile itself), symlink the Dockerfile
to the app root, and add the symlink to my `.gitignore` file. That's due to
specifics of my particular deployment configuration, though, so feel free to
adjust accordingly.

Next, we'll open up the Dockerfile and replace `<my-prefix>` with the prefix we
used earlier. Tweak the Dockerfile as desired. For instance, we might need to
add a few `ENV` declarations if our app's initializers do any environment
variable verification, to prevent our asset precompilation from bailing out.

Next up, take a look at `docker/supervisord.conf` used by the supervisord `CMD`
declaration in the Dockerfile. It's pretty simple. Note that we configure it
not to daemonize, so that the Docker container continues running, and that the
app is started as the user we created in the ruby-base image. By convention, we
place the actual commands used to start the app in the `docker/start` file,
being sure to `exec` the command itself so that the process supervisord is
monitoring is the one our app is running in.

Let's build our app image. There's no need to push it just yet, as we're only
testing:

    docker build -t <my-prefix>/<my-app-name> .

Running `docker images | grep <my-prefix>/<my-app-name>` should return the image
we just built.

### The fig.yml

Now that we have our images in place, we can stand up our staging environment.
We'll do this using [Fig](http://www.fig.sh/), because Fig is awesome.

Assuming you've been following along up until this point, copy
[fig.yml](fig.yml) from this directory to any location you choose. If you're
working on a SOA app, maybe this is the directory above all of the compenent
applications.

    cp fig.yml <some-directory-of-my-choosing>/

The `fig.yml` file is completely standalone, so if you like, you can totally
skip this step and figure it out later, just editing the `fig.yml` in place for
now. In any case, `cd` to the directory you chose to work in, open up `fig.yml`
in your editor of choice, and have a look.

A `fig.yml` file consists of one or more named services along with the
information necessary to start their containers. Minimally, this is an image
name and at least one exposed port to map. Note that port numbers are supplied
as strings by convention because a bare "xx:yy" format for numbers can lead to
some confusing interpretation in YAML.

Our `fig.yml` contains three services:

1. A PostgreSQL database, from the
   [official Docker postgres repo](https://registry.hub.docker.com/_/postgres/).
2. A Redis server, from the
   [official Docker redis repo](https://registry.hub.docker.com/_/redis/).
3. Our previously-built application image, under the unimaginative service name
   of "app".

We'll need to supply the repository name for the app image we built earlier, and
update the environment variables as necessary. Environment keys with no value
supplied will resolve to the value on your machine, if it's set. Note that we
set `RACK_ENV` to "production" here, because we want to exercise this staging
environment as though it's running in production mode. That's why it's really
important to do configuration via environment variables, and use something like
`APP_ENV` to control loading of specific lists of hostnames, etc., from any
application configuration files as necessary.

Since we're just running one application right now, we map the exposed port 80
from our container to port 4000 of the Docker VM. If we'd been running multiple
apps, we'd have used specific numbers that we would map to application names in
[nginx's proxy_ports.conf](../01_nginx/).

Lastly, we link the "app" service's container to both the "redis" and "db"
services. This creates entries in its `/etc/hosts` file for each instance of
those services pointing to their IP addresses on the internal network, which is
why we can simply specify "db" and "redis" for `DB_HOSTNAME` and
`REDIS_HOSTNAME`, respectively. It also means that any time we launch the "app"
service, the "redis" and "db" services will come up, as well.

We'll use that feature next:

    fig run app bundle exec rake db:setup

This will cause Fig to pull down postgres and redis images from the official
Docker registry, if you haven't already, and launch a new container for both,
followed by using the previously-tagged image for our app to run the command
that takes care of setting up our database.

Next, we'll bring up the whole stack to serve requests:

    fig up

At this point, we should be able to bring up a web browser and access either
http://my-app.local.staging or https://my-app.local.staging without issue.

## The Next Step

This was a minimal staging setup. It's probably the case that your app actually
needs other services, or maybe doesn't need the specific ones used in this
guide. That's OK. Hopefully at this point you have the general idea.

Try setting up a staging environment with more than one app working in concert.
Set up port mappings to 4001, 4002, etc, and map them to app names in
`/usr/local/etc/nginx/proxy_ports.conf`. Get the apps talking to each other, and
make sure you can access them from their respective names.

Hopefully The Setup will be a helpful first step toward developing with
production in mind each step of the way! You'll be happier for it in the long
run, I promise!

Happy coding!
