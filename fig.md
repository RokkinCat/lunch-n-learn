# RokkinCat Lunch & Learn: Docker + Fig

**Docker** - A tool for creating app-level containers to make distributed systems a bit easier

**Fig** - A YML-based tool for defining the way docker instances are started

## Docker on OSX
 * Uses boot2docker, an app that uses virtualbox and a virtual machine to emulate standard docker functionality on OSX
 * Required because OSX is **UNIX** and not **LINUX** and thus doesn't have all of the kernel-level features that Docker requires

## Container Images
 * Docker has a service for hosting containers that can be used to spin up app instances. Use this for:
   * Database
   * Redis
   * Anything that needs minimal configuration to be used
 * This service is just providing a host and shortname resolution for a `Dockerfile`
 * Can specify a git URI as well, so you can host your own pre-built `Dockerfiles`
 * Allows docker to cache it's images
 
## Dockerfile

The `Dockerfile` is an initialization script to prep the container for usage, this should include
all package installations and configuration needed to run the application hosted in the container.

**Example Dockerfile for padrino application**

```
FROM ruby
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev
RUN mkdir /application
WORKDIR /application
ADD Gemfile /application/Gemfile
RUN bundle install
ADD . /application
```

## fig.yml

The `fig.yml` file defines dependencies and runtime configuration for the applications running in containers. It can be used to tell the database to spin up when we start our application container, and it can mount a local folder into the container so we can make changes locally and have them reflected in our app, and it can define environment variables on the container itself.

**Example fig.yml for a padrino app**

```
db:
  image: postgres
  ports:
    - "5432"

redis:
  image: redis
  ports:
    - "6379"

web:
  build: .
  command: foreman start
  volumes:
    - .:/application
  ports:
    - "3000:3000"
  links:
    - db
    - redis
  environment:
    REDIS_URL: redis://redis:6379
    DATABASE_URL: postgres://postgres@db:5432/postgres
    PORT: 3000
    RACK_ENV: development
    AWS_ACCESS_KEY_ID: SUPER_SECRET_AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY: EVEN_MORE_SECRET_ACCESS_KEY
    S3_BUCKET_NAME: application-s3-bucket
    GOOGLE_API_KEY: please-let-me-use-google-maps
```

Then if we want to actually generate the padrino app (since we don't have padrino installed on our host machine), we run `fig run web padrino gen app docker-fig-example -r /application` to initialize the app. A quick run of `ls` in your host terminal will show the files got generated and copied to your host machine. Then a `fig build` will install your gems and cache the container with the gems installed. From then on, running `fig up` will make your application available at `http://localdocker-ip:3000`