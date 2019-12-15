My own steps followed when sutting up my pub. Based on https://github.com/ahdinosaur/ssb-pub , but with some changes and some stuff removed that I haven't tested myself.


# ssb-pub

easily host your own [Secure ScuttleButt (SSB)](https://www.scuttlebutt.nz) pub in a docker container

if you feel like sharing your pub, please add it to [the informal registry of pubs](https://github.com/ssbc/scuttlebot/wiki/Pub-Servers) as a private pub with your contact details so newbies may request an invite from you!

(if you are running a v1 pub, [migrate to the latest v2!](#migrating-from-v1-to-v2) :tada: )

:heart:

## manual setup

### install docker

### install `ssb-pub` image

#### build image from source

from GitHub:

```shell
git clone https://github.com/zozs/ssb-pub.git
cd ssb-pub
docker build -t zozs/ssb-pub .
```

### create `sbot` container

#### step 1. create a directory on the docker host for persisting the pub's data

```shell
mkdir ~/ssb-pub-data
chown -R 1000:1000 ~/ssb-pub-data
```

#### step 2. setup ssb config

```shell
EXTERNAL=<hostname.yourdomain.tld>

cat > ~/ssb-pub-data/config <<EOF
{
  "connections": {
    "incoming": {
      "net": [
        {
          "scope": "public",
          "host": "0.0.0.0",
          "external": ["${EXTERNAL}"],
          "transform": "shs",
          "port": 8008
        }
      ]
    },
    "outgoing": {
      "net": [
        {
          "transform": "shs"
        }
      ]
    }
  }
}
EOF
```

#### step 3. run the container

create a `./create-sbot` script:

```shell
cat > ./create-sbot <<EOF
#!/bin/bash

memory_limit="\$((\$(free -b --si | awk '/Mem\:/ { print \$2 }') - 200*(10**6)))"

docker run -d --name sbot \
   --init \
   -v ~/ssb-pub-data/:/home/node/.ssb/ \
   -p 8008:8008 \
   --restart unless-stopped \
   --memory "\$memory_limit" \
   zozs/ssb-pub
EOF
```

where

- `--memory` sets an upper memory limit of your total memory minus 200 MB (for example: on a 1 GB server this could be simplified to `--memory 800m`)

then

```shell
# make the script executable
chmod +x ./create-sbot
# run the script
./create-sbot
```

#### step 4. create `./sbot` script

we will now create a shell script in `./sbot` to help us command our Scuttlebutt server running:

```shell
# create the script
cat > ./sbot <<EOF
#!/bin/sh

docker exec -it sbot sbot \$@
EOF
```

then

```shell
# make the script executable
chmod +x ./sbot
# test the script
./sbot whoami
```

#### step 5. announce your pub

On your regular SSB client (i.e. on your desktop, laptop, etc.) *NOT* on the pub, announce that your pub exists and start syncing with it.
Your followers can now also see the pub.

```
ssb-server publish --type pub --address.key "{feedIdOfPub}" --address.host {hostnameOfPub} --address.port {portOfPub}
```

Example, but ensure you replace everything with your own correct values.

```
ssb-server publish --type pub --address.key "@Dg0PUvvq+IsEkD+iQpKfMiDh9/LkqaabHgQJU05O6z0=.ed25519" --address.host ssb.zozs.se --address.port 8008
```


## command and control

### create invites

from your server:

```shell
./sbot invite.create 1
```

### start, stop, restart containers

for `sbot`

- `docker stop sbot`
- `docker start sbot`
- `docker restart sbot`


