---
layout: post
title: How to containerize your Django application with Docker and compose on OSX
section: developer
summary: |
    Docker has lots of documentation, but not much in the way of practical examples of moving your stack over to Docker. This post looks at setting up your development Docker environment from start to finish.
---
I've been hearing good things about Docker for a while now -- and certainly the premise made a lot of sense. I develop on a Mac and deploy to Heroku so having my development environment match production is ideal.

Getting your development environment "close enough" is not so much of a problem these days, but I still carry the scars earned in hard lost battles compiling binaries against incompatible BSD libraries and the accompanying inexplicable segfaults and premature jubilance.

Reccently we've been spoiled with apps like <i>Postgress.app</i> and the first OSX package manager to actually get it right, <i>Homebrew</i>.

Still, the dream lives on ... mirroring a production environment on your dev machine without going the full monty and devoting half your expensive SSD to Vagrant and some Chef/Puppet/Ansible recipes.

Suffice to say, I was keen to make it work. But Docker did not make this easy, it was a whole new world and the documentation lacked accessibility, tutorials contradictory, and although I parsed individual concepts a holistic solution seemed elusive. What was this "compose" thing, what's boot2docker, why can't I use <i>--volumes</i> on OSX, what's Dockerhub, and why when someone talks about "dockerizing" Django why is it completely unhelpful?

So in an attempt to avoid being overly loquacious, here's the quick answers:
<ul>
<li><em>compose</em> is a tool you can use to describe all the containers your app needs, as well start and stop them. This functionality may in the future be integrated into Docker as "container groups".</li>
<li><em>boot2docker</em> is a light weight virtual machine running Tiny Core Linux via VirtualBox. This is necessary because OSX doesn't support for something that Docker needs (Linux Containers). Once you've installed this, you don't need to know much more about it.</li>
<li>You'll want your container (your "vm") to be able to access stuff on your host (your laptop), for example your app's source code. The <em>--volumes</em> argument didn't work on OSX making this difficult but <a href="https://blog.docker.com/2014/10/docker-1-3-signed-images-process-injection-security-options-mac-shared-directories/">this was fixed in Docker 1.3</a> for anything inside the /Users folder.</li>
<li>Dockerhub is a place to put your compiled containers to make it easier to share with people. You can also put your Dockerfile up (put it into a GitHub repository and link it via an "automated build" on Dockerhub).</li>
<li>"Dockerizing" tutorials are mostly building up the containers from scratch, this is not useful for us. We'll use the library of "official" pre-built images on Dockerhub and extend them to "containerize" our app.</li>
</ul>

We'll now look at what it takes to get a simple hello world application (with Redis and Postgres w/ hstore) up and running.

If you want to jump straight to the source, it's <a href="https://github.com/aidanlister/blog-docker-walkthrough">all on github</a>.

I'm going to assume you've installed Docker, if not <a href="https://docs.docker.com/installation/mac/">do that first</a>.

<h3>Setting up your database container</h3>
This part is pretty simple as we could use the stock standard Postgres docker from Dockerhub (<a href="https://registry.hub.docker.com/_/postgres/">library/postgres</a>).

However, I needed the <strong>hstore</strong> extension in Postgres and I couldn't find a Dockerfile that enabled that on Dockerhub, so I created one: a simple <a href="https://registry.hub.docker.com/u/aidanlister/postgres-hstore/">Dockerfile that enables hstore just before the server is launched</a>. You can simply reference it from your docker-compose.yml or Dockerfile.

We create a docker-compose.yml, a document that describes our infrastructure holistically, in our project root. It should contain the following:

{% highlight yaml %}
dbdata:
  image: postgres:latest
  volumes:
    - /var/lib/postgresql
  command: true

db:
  image: aidanlister/postgres-hstore
  volumes_from:
    - dbdata
  ports:
    - "5432"
{% endhighlight %}

This is creating two containers, one for the data and one for the server. This is the <a href="https://docs.docker.com/userguide/dockervolumes/">recommended approach</a>, so that you can trash your server container but keep your data persisted. This has other benefits for example easily distributing a ready-to-go container to colleagues.

We'll step through loading data into your database for the first time later in the article.

Aside: Why use "postgres:latest" as the base for our data volume when we could use Tiny Linux or something else? Because Docker caches changes to the filesystem, it would actually use more disk space to use anything other than the same base image.

Type "docker-compose up" to watch everything download and build, and you should see a message that your server is up and running. ctrl-c to close.

<h3>Setting up your redis server</h3>
This is exactly the same. We add this to our docker-compose.yml:

{% highlight yaml %}
redisdata:
  image: redis:latest
  volumes:
    - /var/lib/redis
  command: true

redis:
  image: redis:latest
  volumes_from:
    - redisdata
  ports:
    - "6379"
{% endhighlight %}

Type "docker-compose up" to watch postgres spin up instantly without building (the image has been cached in your VM), and redis build and spin up. You're now running four containers with a single command.


<h3>Setting up the python container</h3>
Next we want to get our python container up and running. This is where it gets tricker ... create a Dockerfile in your project root (same folder as manage.py and your docker-compose.yml).

{% highlight yaml %}
FROM python:2.7.8
MAINTAINER Your Name
EXPOSE 8000

RUN mkdir -p /usr/src/app
COPY requirements.txt /usr/src/requirements.txt

WORKDIR /usr/src/python
RUN pip install -r /usr/src/requirements.txt

ENV DJANGO_SETTINGS_MODULE myproject.settings.local
ENV DATABASE_URL postgres://postgres@db/postgres
ENV REDISTOGO_URL redis://redis:6379

WORKDIR /usr/src/app
CMD [ "python", "manage.py", "runserver", "0.0.0.0:8000" ]
{% endhighlight %}

This is pretty simple: we've extended the official Python docker with FROM, copied our requirements.txt into the container image, run <em>pip install</em> on it, set up our environment variables and then <em>runserver'd</em>.

You can test that it builds correctly with <code>docker build -t yourapp .</code> which will download the base images, then run with <code>docker run -it --rm yourapp</code>. If you want to open it in your browser, we'll need to link the three containers together in our docker-compose.yml, add:

{% highlight yaml %}
web:
  build: .
  command: python manage.py runserver 0.0.0.0:8000
  volumes:
    - .:/usr/src/app/
  ports:
    - "8000:8000"
  links:
    - db
    - redis
  environment:
    - INSTANCE_TYPE=web
    - DEBUG=1
    - DJANGO_SETTINGS_MODULE=yourapp.settings.local
    - DATABASE_URL=postgres://postgres@db/postgres
{% endhighlight %}

You'll note that we've specified a "volumes" key. This will map the source code in the "." folder on the host (the current working folder which should also be location of the Dockerfile and docker-compose.yml) to the /usr/src/app folder on the container. This, combined with using "python manage.py runserver" as the "command" means when you edit and re-save a source file on the host, <strong>runserver will automatically reload your code changes</strong>. <em>In fact, it's even faster than you'd normally be used to on OSX because the linux container will have inotify support.</em>

Type <code>boot2docker ip</code> into a fresh terminal window to get the IP address of the Docker VM.

We can now type "docker-compose up" again, this will build and launch your containers. Open the IP of your VM in your browser eg. <code>http://192.168.59.103:8000/</code>. If everything has gone well, you'll see a Hello World showing that we have connected to both Postgres and Redis.

To connect up some python rqworkers to the redis server, you would add:

{% highlight yaml %}
worker:
  build: .
  command: python manage.py rqworker
  links:
    - db
    - redis
  volumes:
    - .:/usr/src/app/
  environment:
    - INSTANCE_TYPE=worker
    - DEBUG=1
    - DJANGO_SETTINGS_MODULE=abas.settings.local
    - DATABASE_URL=postgres://postgres@db/postgres
{% endhighlight %}

We're essentially done. We've got all our containers talking, our app is running ... there's just a few extra things worth discussing:

<h3>Running scripts that interact directly with the database</h3>

You could connect to your Postgres database inside the container directly from your host, but that's un-docker and you might not want postgres installed on your host. Instead, we're going to create a Docker image for these types of tasks.

Create a new folder to hold your dockers in your project root, e.g. "dockers". In that folder create a folder for your "database job" docker, "dbjob". In that folder add a Dockerfile that looks like:

{% highlight yaml %}
FROM postgres:latest
MAINTAINER Your Name
RUN mkdir /usr/scripts
ADD scripts/ /usr/scripts/
WORKDIR /usr/scripts
ENTRYPOINT ["sh"]
{% endhighlight %}

You'll then need to create a folder "scripts", where you'll put scripts that could be executed against the database. For example, here's a reset.sh which drops and restores the database:

{% highlight yaml %}
#!/bin/bash
if [ ! -e /tmp/hostvar/database.dump ]; then
    echo "reset.sh: /tmp/database.dump does not exist!"
    exit 0
fi

dropdb \
  -h "$POSTGRES_PORT_5432_TCP_ADDR" \
  -p "$POSTGRES_PORT_5432_TCP_PORT" \
  -U postgres \
  postgres \
  || { echo 'reset.sh: unable to drop DB'; exit 1; }

createdb \
  -h "$POSTGRES_PORT_5432_TCP_ADDR" \
  -p "$POSTGRES_PORT_5432_TCP_PORT" \
  -U postgres \
  postgres \
  || { echo 'reset.sh: unable to recreate DB'; exit 1; }

pg_restore \
  -h "$POSTGRES_PORT_5432_TCP_ADDR" \
  -p "$POSTGRES_PORT_5432_TCP_PORT" \
  -U postgres \
  --no-acl --no-owner --verbose \
  -d postgres < /tmp/hostvar/database.dump

echo "reset.sh: success"
{% endhighlight %}

To run this, we make sure only the database is running with <code>docker-compose up db</code> and then execute <code>docker run -it --link=abas_db_1:postgres --volume=$PWD/var/:/tmp/hostvar/ dbjob reset.sh</code>.

This code builds our docker, mounts the "var" folder on our host (which should contain your database.dump dump), and then runs the "reset.sh" command inside the scripts folder on the container.

<h3>Running management commands</h3>

You're probably thinking "I hope I don't need to create a dockerfile just to run management commands". Nope, luckily you can just use <code>docker-compose run web python manage.py shell_plus</code>.

<h3>Using python debugger</h3>

You can now use pdb with docker (see <a href="https://github.com/docker/fig/issues/359">this</a> and <a href="https://github.com/docker/fig/pull/485">this</a> about why you couldn't). To start your project in a way that exposes a TTY for docker-compose:

{% highlight sh %}
$ docker-compose run --service-ports web
{% endhighlight %}

<h3>Other useful commands</h3>

Docker one-liners:
<ul>
<li>Delete all containers: <code>docker rm -f $(docker ps -aq)</code></li>
<li>Delete all images: <code>docker rmi -f $(docker images -q)</code></li>
<li>Delete dangling images: <code>docker rmi $(docker images -q -f dangling=true)</code></li>
</ul>

So this is a summation of my knowledge thus far, I hope it is useful. Remember all of the code is <a href="https://github.com/aidanlister/blog-docker-walkthrough">available on GitHub</a>. Please leave any feedback in the comments below.
