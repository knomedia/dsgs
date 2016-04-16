# dsgs

[d]ocker [s]tatsd [g]raphite [s]etup

A small script to help you run
[hopsoft/docker-graphite-statsd](https://github.com/hopsoft/docker-graphite-statsd)
with a different statsd config file. hopsoft's docker image for statsd and
graphite are great, and really flexible. I generally run it with a different
config file for statsd (to match closer to what we use in production at the day
job).

This script will do the following:

* create a directory (you choose the name) within your home directory and
    populate it with other directories to be used as mounted volumes in the docker
    container
* copy a given statsd config file (you supply it, or it will use a default one)


## Usage

Clone this repo, or download the `dsgs` file. Move it somewhere that is in your PATH (ala `/usr/local/bin`). For example:

```bash
git clone git@github.com:knomedia/dsgs.git
cd dsgs
cp dsgs /usr/local/bin/dsgs
```


```
usage:

dsgs <dir> [config_url]

<dir> is the name of a directory that will be created in your home folder
and updated with a statsd config file, and then used as a local volume for the
docker container that you spin up.

[config_url] optional url to a statsd config file to use
```


For example:

```bash
dsgs local_statsd_config
```

this would create a `~/local_statsd_config` directory with some sub
directories, and copy a default statsd `config.js` file into
`~/local_statsd_config/statsds/`

Then follow the prompt to start up your container locally. It will look
something like:

```
VIRTUAL_HOST=graphite.docker docker run -d
  --name graphite\
  -p 81:80  -p 2003:2003\
  -p 8125:8125/udp\
  -v ~/local_statsd_config/graphite/configs:/opt/graphite/conf\
  -v ~/local_statsd_config/graphite/data:/opt/graphite/storage\
  -v ~/local_statsd_config/statsd:/opt/statsd\
  hopsoft/graphite-statsd
```

From there you can follow the directions at [hopsoft/docker-graphite-statsd](https://github.com/hopsoft/docker-graphite-statsd). But the td/dr; is:

```
# stop the container from running
docker stop graphite

# start the container up
docker start graphite

# remove the container
docker rm graphite
```

Assuming you are using [https://github.com/codekitchen/dinghy](dinghy) you will be able to get access to the graphite web ui at `http://graphite.docker:81`, and can send statsd metrics over UDP to `graphite.docker:8125` something like:

```bash
echo "some.key.bucket:1|c" | nc -w 1 -u graphite.docker 8125
```

