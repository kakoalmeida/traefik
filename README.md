### Traefik Installation

<p>First you need to install apache2-utils with the following command <b>sudo apt install apache2-utils</b>. After install, <b>htpasswd</b> will be used to generate a safe password for our Traefik admin.</p>
 
```htpasswd -nb admin your_password```

output:
```admin:$apr1$i0QHNdP1$E8nWtavDzrquH7QCRTstv.```

<p>The output of the command above, will be used in your <i> traefik_dynamic.toml</i> for user authentication in traefik management dashboard. And yes, the output may contain special characters in the end like ". or /"</p>

#### Configuration files.

<p> Now let's create ou traefik configuration file, <i> traefik.toml</i>. If u want you can use Yaml instead of toml.<br> Create the file: </br></p>

```vim traefik.yml```

After create the file, add the configuration from <i>traefik.toml</i> file in this repo. Remenber to change the email for the ACME challenge, which will be used to Lets-encrypt verify the certificate. It must be a valid email.

The entrypoints section tells traefik what ports should be listened. Also the redirections section will automatically redirect http connections on port 80 to port 443, the redirected traffic will be handled over TLS.

API section, gives us access to api and dashboard interface, that will be acessed through the URL defined in <i>traefik_dynamic.toml</i>.

certificateResolvers section tells traefik how we gonna manage our SSL certificates and how they can be validated. That's why we need a valid email.

Providers section allow traefik to work as a proxy for our docker containers, the tag watch is for new containers and the network is the same we gonna use in our containers.

The last section indicates what dynamic configurations we gonan use.

After all settings included, create <i>traefik_dynamic.toml</i> and edit according your scenario.

#### Running Traefik

Befora you start the container with traefik. Create acme.json and set it permissions to read and write for owner only.

```touch acme.json``` 
```chmod 600 acme.json```

Create the docker network:
```docker network create web```
If you are using docker in Swarm mode:
```docker network create --driver overlay --attachable web```

Now lets start our traefik container with:
```
docker run -d 
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.toml:/traefik.toml \
  -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
  -v $PWD/acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  --network web \
  --name traefik \
  traefik:v2.2 
```

#### How to use

In docker compose file or yaml files from Swarm mode, add the labels section. Your file should look like the following lines:
```
version: '3.7'
networks:
  web:
    external: true

services:
  my_app:
    image: your_image
    networks:
      - web
    labels:
      - traefik.http.routers.my_app.rule=Host(`app.your_domain`)
      - traefik.http.routers.my_app.tls=true
      - traefik.http.routers.my_app.tls.certresolver=lets-encrypt
      - traefik.port=80
```