Talkyard for Docker Swarm and Compose
=====================================

Here are Docker Swarm and Docker-Compose template files that you can use to integrate Talkyard into your already existing Docker Swarm or Docker-Compose installation.

There are `${REPO_NAME}` placeholders so you can use your own Docker repository with your own images.

Currently only Docker-Compose; Swarm not yet tested (but should be fairly similar?).
Would maybe need to run `envsubst` to update variables in a Swarm stack config file,
before deploying? Like this?:  `export $(grep -v '^#' .env | xargs -d '\n') envsubst < .env`, see: https://stackoverflow.com/questions/19331497/set-environment-variables-from-file-of-key-pair-values.
Or like this?: `sh -ac ' . ./.env; envsubst < .env'`, see https://serverfault.com/a/540484/44112.

If you'd like to install on a new server, dedicated to Talkyard only — then
probably it's simpler for you to use https://github.com/debiki/talkyard-prod-one instead.

**Missing:** Automatic backups. Probably there'll be an image that
backups the database and uploaded files regularly, to the talkyard-backups Docker volume.
It'd be your responsibility to mirror (e.g. via rsync) this volume's contents to a safe
place.

**Missing:** Automatic upgrades. Do you want that? Or you'd rather run `git pull`
and `docker-compose restart` (se below) yourself?


Usage example
---------------

To prepare Ubuntu and install Docker-Compose, you can do this:

```
# Setup a firewall (its name is "ufw")
ufw allow OpenSSH
ufw allow http
ufw allow https
ufw enable

# Make your server Redis and ElasticSearch friendly, and configure automatic
# operating system security upgrades.
wget https://raw.githubusercontent.com/debiki/talkyard-prod-one/master/scripts/prepare-ubuntu.sh
vi prepare-ubuntu.sh  # have a look at what the script does, maybe comment out some things
chmod u+x prepare-ubuntu.sh
./prepare-ubuntu.sh

# Install Docker-Compose if you haven't already.
wget https://raw.githubusercontent.com/debiki/talkyard-prod-one/master/scripts/install-docker-compose.sh
vi install-docker-compose.sh  # have a look at the script
chmod u+x install-docker-compose.sh
./install-docker-compose.sh
```

Download:

```
sudo -i
cd /opt/
git clone https://github.com/kajmagnus/talkyard-prod-swarm.git
cd talkyard-prod-swarm
git submodule update --init
```

Use:

```
# Start a reverse proxy with HTTPS.
docker network create proxy_net
pushd .
cd traefik/
vi traefik.toml  # Fill in your email address, the acme.email field.
                 # Optionally, enable the dashboard.
touch acme.json
chmod 600 acme.json
docker-compose up -d
popd

# Configure your hostname, PostgreSQL password, and optionally a Docker repository.
vi talkyard.env  # softlinks to .env

# Configure memory.
cp mem/2g.yml docker-compose.override.yml

# Edit email server settings.
vi conf/play-framework.conf

# Upgrade to the latest version.
pushd .
cd talkyard-versions/
git checkout master
git pull origin
popd

# Start Talkyard
docker-compose up -d
docker-compose logs -f --tail 33
```

Upgrading to newer versions: `git pull` new versions of the
Compose and Swarm config files:

```
cd /opt/talkyard-prod-swarm/talkyard-versions
git pull
cd ..
docker-compose restart
```

You can override the default settings in `docker-compose.override.yml`.


License
---------------

```
Copyright (c) 2019 Debiki AB and Kaj Magnus Lindberg

License: MIT (this repository only — the Talkyard source code is
elsewhere under a different license)
```
