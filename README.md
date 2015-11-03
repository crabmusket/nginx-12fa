# nginx for a 12 factor app

One of the tenets of the [12 factor app](http://12factor.net/) is that services should be configurable using environment variables.
Currently, there's not really a clean way to parameterise your nginx config file using environment variables.
This Docker image uses a small script to generate an nginx config file when you run the container by substituting environment variables into your nginx config file.

## Using this image

This image works the same as the regular `nginx` image available on the Docker Hub.
To bring your own config file, mount it to `/etc/nginx/nginx.template.conf`.
On container start, environment variables in the config file will be replaced, and the config file used to run nginx as usual.
For example:

    # nginx.conf
    worker_processes 1;

    events {
      worker_connections 1024;
    }

    http {
      server {
        listen 80;
        return 301 http://${REDIRECT_URL};
      }
    }

Run:

    $ docker run -d --name nginx \
        -v $(pwd)/nginx.conf:/etc/nginx/nginx/template.conf
        -e "REDIRECT_URL=google.com" \
        eightyeight/nginx-12fa:1.9

## Config file substitution

The substitution script _only_ accepts environment variables enclosed in `{}`, because regular nginx variables look like `$var`. Here's an example of some substitutions:

    $ cat test
    where am I? ${HOME}
    Escaping \${HOME}
    This variable ${IS_NOT_IN_THE_ENV}
    $ ./subenv test test.out && cat test.out
    Where am I? /home/daniel
    Escaping \${HOME}
    This variable 
