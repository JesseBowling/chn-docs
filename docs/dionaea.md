Dionaea Honeypot
================
The CommunityHoneyNetwork dionaea honeypot is an implementation of [@DinoTools's Dionaea](https://github.com/DinoTools/dionaea), configured to report logged attacks to the CommunityHoneyNetwork management server.

> "Dionaea's intention is to trap malware exploiting vulnerabilities exposed by services offerd to a network, the ultimate goal is gaining a copy of the malware."

## Prerequisites

The default deployment model uses Docker and Docker Compose to deploy containers for the project's tools, and so, require the following:

* Docker >= 1.13.1
* Docker Compose >= 1.15.0

**Please ensure the user on the system installing the honeypot is in the local
 docker group**
 
 Please see your system documentation for adding a user to the docker group.

## Important Note!
The env files, as well as the docker-compose.yml files below are intended 
to help you understand the various options. While they may serve as a basis 
for users with advanced deployment needs, most users should default to the 
configuration files provided by the deployment scripts in the CHN web interface.

## Example dionaea docker-compose.yml
```dockerfile
version: '3'
services:
  dionaea:
    image: stingar/dionaea:1.9.1
    restart: always
    volumes:
      - configs:/etc/dionaea/
    ports:
      - "21:21"
      - "23:23"
      - "69:69"
      - "123:123"
      - "135:135"
      - "445:445"
      - "1433:1433"
      - "1723:1723"
      - "1883:1883"
      - "1900:1900"
      - "3306:3306"
      - "5000:5000"
      - "5060:5060"
      - "5061:5061"
      - "11211:11211"
      - "27017:27017"
    env_file:
      - dionaea.env
    cap_add:
      - NET_ADMIN
volumes:
    configs:
```

Please note that while Dionaea will run without the `NET_ADMIN` capability set, it will consume 100% of a CPU and not
 function optimally.

## Example dionaea.env file


Prior to starting, Dionaea will parse options passed to the container via the `env_file` specification in the docker-compose file. The following is an example env file:

```
# This can be modified to change the default setup of the unattended installation

DEBUG=false

# IP Address of the honeypot
# Leaving this blank will default to the docker container IP
IP_ADDRESS=

CHN_SERVER=https://{fqdn_or_ip_of_chn_server}
DEPLOY_KEY={deploy_key}

# Network options
LISTEN_ADDRESSES=0.0.0.0
LISTEN_INTERFACES=eth0

# Service options
# blackhole, epmap, ftp, http, memcache, mirror, mongo, mqtt, mssql, mysql, pptp, sip, smb, tftp, upnp
SERVICES=(blackhole epmap ftp http memcache mirror mongo mqtt pptp sip smb tftp upnp)

DIONAEA_JSON=/etc/dionaea/dionaea.json

# Logging options
HPFEEDS_ENABLED=true
FEEDS_SERVER={fqdn_or_ip_of_chn_server}
FEEDS_SERVER_PORT=10000

# Comma separated tags for honeypot
TAGS=

# A specific "personality" directory for the dionaea honeypot may be specified
# here. These directories can include custom dionaea.cfg and service configurations
# files which can influence the attractiveness of the honeypot.
PERSONALITY=""
```

### Configuration Options

The following options are supported in the `/etc/default/dionaea` files:

* DEBUG: (boolean) Enable more verbose output to the console
* IP_ADDRESS: IP address of the host running the honeypot container
* CHN_SERVER: (string) The hostname or IP address of the CHN Server to register honeypot.
* DEPLOY_KEY: (string; REQUIRED) The deploy key provided by the feeds server administration for registration during the first startup. This key is **required** for registration.
* DIONAEA_JSON: (string) The path to write out the Dionaea registration information.
* LISTEN_ADDRESSES: (string) The IP address of the Dionaea network listener
* LISTEN_INTERFACES: (string) The hardware interfaces to listen on for Dionaea network connectivity
* SERVICES: (array of strings) The network services to enable for the Dionaea honeypot.
* HPFEEDS_ENABLED: (boolean) Enable the hpfeeds handler module
* FEEDS_SERVER: (string) The hostname or IP address of the HPFeeds server to send logged events.
* FEEDS_SERVER_PORT: (integer) The HPFeeds port. Default is 10000.
* TAGS: (string) Comma delimited string for honeypot-specific tags. Tags must be separated by a comma to be parsed properly. **TAGS** string must be enclosed in double quotes if string contains spaces.


# Service Configuration

Note: Any time a configuration file is added, removed, or modified, either the dionaea service or the docker container must be restarted.

## Configuring Dionaea services

The honeypot services running on Dionaea are highly configurable. The default configurations can be used to detect that the server is acting as a honeypot. Therefore, modifying these configuration files is recommended.

After Dionaea has been started, these services can be edited by modifying the yaml files in `./dionaea/services-enabled/`

There are many options for each service, so we recommend referencing the official [Dionaea documentation for services](http://dionaea.readthedocs.io/en/latest/service/index.html).

## Enabling Dionaea services

To enable a new service after Dionaea has been built, you can run the following command to enable the service:

    $ docker-compose exec dionaea cp /opt/dionaea/etc/dionaea/services-available/<service_config> /opt/dionaea/etc/dionaea/services-enabled/

For example, to enable FTP:

    $ docker-compose exec dionaea cp /opt/dionaea/etc/dionaea/services-available/ftp.yaml /opt/dionaea/etc/dionaea/services-enabled/

Current Dionaea services available:

* blackhole.yaml
* epmap.yaml
* ftp.yaml
* http.yaml
* memcache.yaml
* mirror.yaml
* mongo.yaml
* mqtt.yaml
* mssql.yaml
* mysql.yaml
* pptp.yaml
* sip.yaml
* smb.yaml
* tftp.yaml
* upnp.yaml


## Disabling Dionaea services

If you wish to remove a service from the dionaea honeypot, you can simply delete the corresponding yaml file from `./dionaea/services-enabled/`, and remove the relevant service from dionaea.env.

## Adding a custom dionaea "personality"

You can add files to your dionaea honeypot in order to customize it's behavior. Currently the code supports custom 
versions of `dionaea.cfg`, and custom service configuration via a directory structure. See [here](https://github.com/CommunityHoneyNetwork/dionaea/tree/master/personalities/debian) for an example personality. 

Once you have the custom files on the honeypot host, volume mount a directory containing these files to the container, 
and specify the directory name in the `PERSONALITY` env option.

For a more concrete example: let's say I want to include a `dionaea.cfg` file in a personality called 'sneakydionaea'. 

First I'll create a directory called "sneakydionaea" on my honeypot VM with the `dionaea.cfg` file in it. It might 
look like this:
 
```bash
$ ls -l
total 12
-rw-r--r-- 1 me  me  1535 Apr 25 10:30 dionaea.env
-rw-rw-r-- 1 me  me  2115 Apr 25 10:29 deploy.sh
-rw-r--r-- 1 me  me   256 Apr 25 10:30 docker-compose.yml
drwxrwxr-x 2 me  me    42 Apr 25 10:43 sneakydionaea

$ ls -l sneakydionaea/
total 8
-rw-rw-r-- 1 me me 310 Apr 25 10:43 dionaea.cfg
```

Then make the following change to the `docker-compose.yml`:

```
<snip>
    volumes:
      - configs:/etc/dionaea
      - ./sneakydionaea:/opt/personalities/sneakydionaea
<snip>
```
and then modify the `dionaea.env` to specify the directory name in the `PERSONALITY` variable:
```
# A specific "personality" directory for the dionaea honeypot may be specified
# here. These directories can include custom fs.pickle, dionaea.cfg, txtcmds and
# userdb.txt files which can influence the attractiveness of the honeypot.
PERSONALITY=sneakydionaea
```
You should then be able to `docker-compose down` and `docker-compose up -d` at this point and the personality should take effect.

# Acknowlegements

CommunityHoneyNetwork Dionaea is an adaptation of [@DinoTools' Dionaea](https://github.com/DinoTools/dionaea/) Dionaea software and [Threatstream's Modern Honey Network](https://threatstream.github.io/mhn/) Dionaea & HPFeeds work, among other contributors and collaborators.

# License

Dionaea is licensed under the [GNU GENERAL PUBLIC LICENSE Version 2.0](https://raw.githubusercontent.com/DinoTools/dionaea/master/LICENSE)

The ThreatStream implementation of Dionaea with HPFeeds, upon which CommunityHoneyNetwork is based is licensed under the [GNU LESSER GENERAL PUBLIC LICENSE Version 2.1](https://raw.githubusercontent.com/threatstream/mhn/master/LICENSE)

The [CommunityHoneyNetwork Dionaea deployment model and code](https://github.com/CommunityHoneyNetwork/dionaea) is therefore also licensed under the [GNU LESSER GENERAL PUBLIC LICENSE Version 2.1](https://raw.githubusercontent.com/CommunityHoneyNetwork/dionaea/master/LICENSE)
