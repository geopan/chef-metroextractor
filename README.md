Metroextractor Chef Cookbook
===================
![Build Status](https://circleci.com/gh/mapzen/chef-metroextractor.png?circle-token=143ea95327e7591cf64082ca9a790c5014529c3b)

Attribution
-----------
Heavily influenced by Mike Migurski's [Extractotron](https://github.com/migurski/Extractotron/)

What does it do?
----------------
Downloads the latest planet in pbf format.
Produces, via osmosis, a number of metro extracts in both pbf and bz2 formats.
Produces shape files in both imposm and osm2pgsql formats.

What hardware do I need?
------------------------
On AWS, using a c3.2xlarge, it takes about 30 hours to process all the extracts and shapes should you want to
produce your own. You'll need roughly 8GB of heap allocated to osmosis for the extracts portion (by default,
heap size is set to half the available RAM, so if you're tight on memory you can adjust this to suite), and
to be safe, ~75GB of disk space.

If you'd rather not go that route, you're in luck. We'll be producing extracts weekly, and you can find
then here: [Mapzen Metro Extracts](http://mapzen.com/metro-extracts/)

Pull requests for new cities are more than welcome.

Contributing
------------
Extract creation is driven off the file `files/default/cities.json`. If you'd like to add an extract, you can either
update the file directly if you feel comfortable doing so and submit a pull request, or open a GitHub issue and
we'll look into doing it for you.

If you'd like to update any other code, please fork, update, write specs for your changes and submit a pull.

Usage
-----
    include_recipe 'metroextractor'

Supported Platforms
-------------------
Tested on Ubuntu12.04LTS. You can probably get away with distributions similar to those, but as yet
they have not been tested.

Requirements
------------
* Chef >= 11.4

Attributes
----------
### metroextractor.setup

#### basedir
Base working directory
* type: string
* default: /mnt/metro

#### scriptsdir
Where to install scripts
* type: string
* default: /opt/metroextractor-scripts

### metroextractor.user

#### id
User to run everything as
* type: string
* default: metro

#### shell
User shell
* type: string
* default: /bin/bash

#### manage_home
Private
* type: string
* default: false

#### create_group
Create the user group
* type: boolean
* default: true

#### ssh_keygen
Generate ssh keys for the user
* type: boolean
* default: false

### metroextractor.postgres

#### dbs
What postgres dbs to create
* type: array
* default: %w(osm)

#### user
What postgres user to create
* type: string
* default: osmuser

#### password
Postgres user password
* type: string
* default: password


### metroextractor.planet

#### url
Where to download the planet
Currently required to be pbf (not validated)
* type: string
* default: http://ftp.heanet.ie/mirrors/openstreetmap.org/pbf/planet-latest.osm.pbf

#### file
Derive the name of the planet download from the url
Currently required to be pbf (not validated)
* type: string
* default: ```planeturl.split('/').last```

### metroextractor.extracts

#### osmosis_timeout
Set the timeout for processing of pbf/bz extracts
* type: int
* default: 172_800 (2 days)

#### osmosis_lock
Location of osmosis lockfile
* type: string
* default: basedir/.osmosis.lock

#### osmosis_jvmopts
JVM options to pass to osmosis. Xmx is calculated automatically and 
set to total available RAM: ```#{(node[:memory][:total].to_i * 0.6).floor / 1024}M```
* type: string
* default: '-server -XX:SurvivorRatio=8 -Xms#{heap} -Xmx#{heap}'

### metroextractor.shapes

#### osm2pgsql_timeout
Set the timeout for processing of shape files
* type: int
* default: 172_800 (2 days)

#### osm2pgsql_lock
Location of osm2pgsql lockfile
* type: string
* default: basedir/.osm2pgsql.lock

Upstream cookbook overrides

### osmosis

#### symlink
Create a symlink into /usr/bin
* default: true

#### install_type
Install from source
* default: tgz

Dependencies
-----------
apt, osm2pgsql, osmosis postgresql

Vagrant Environment
===================

Installation
------------
    vagrant plugin install vagrant-berkshelf 
    bundle install
    berks install
    vagrant up
    vagrant ssh

#### Note
* running a planet download and then processing extracts on your local machine might be hit or miss

#### I don't like Vagrant
* well then sir, provision an Ubuntu12.04 LTS system with the provider of your choice, and then bootstrap with chef-solo:
    `knife solo bootstrap root@${host} -r 'recipe[metroextractor]'`
* and re-cook with the following:
    `knife solo cook root@${host} -r 'recipe[metroextractor]'`
* alternatively, you can add the metroextractor cookbook to your chef server and wrap it as you see fit

License and Authors
-------------------
License: GPL
Authors: grant@mapzen.com
