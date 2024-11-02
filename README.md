

  <!-- <a href="https://traefikconfig.netlify.app">Demo</a> -->
</div>
  &#xa0;

<h1 align="center">Traefik Config</h1>

<iframe width="640" height="480" src="https://www.youtube.com/embed/n1vOfdz5Nm8" title="Traefik 3 and FREE Wildcard Certificates with Docker" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<div align="center">

 After watching [Techno Tims](https://github.com/timothystewart6) video on traefik v3, I thought why not make a repo for easy deployment. So Others don't have to go through some of the hassle of copy and pasting.

</div>

### Setup 
- First things first, clone the repo to where you keep your docker stacks. 

```shell
git clone https://github.com/JershBytes/traefik-config.git
```

### Create Docker Network

```shell
docker network create proxy
```

### Cloudflare API Token Secret

If you're following along in the video once you get to [here](https://youtu.be/n1vOfdz5Nm8?t=1017). paste that token in [cf_api_token.txt](./cf_api_token.txt) file.

### Traefik Dashboard Password & .env

make sure you have `htpasswd` installed.

To install on Linux

```shell
sudo apt update
sudo apt install apache2-utils
```

Mac OS (should already be installed)

Windows

htpasswd.exe Should already be installed on Windows

Generate credential pair

```shell
echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g
```

```shell
touch .env
nano .env
```
paste your credential pair:

e.g.

```shell
TRAEFIK_DASHBOARD_CREDENTIALS=user:$$2y$$05$$lSaEi.G.aIygyXRdiFpt7OqmUMW9QUG5I1N.j0bXoXxIjxQmoGOWu
```

### Start the stack

```shell
docker compose up -d --force-recreate
```

### Troubleshooting

Common ways to troubleshoot

```shell
docker ps
docker logs traefik
docker exec -it traefik /bin/sh
```

inside of container

```shell
top
ls
cat acme.json
cat traefik.yml
ls /run/secrets
cat /run/secrets/cf_api_token
echo ${CF_DNS_API_TOKEN_FILE}
echo ${TRAEFIK_DASHBOARD_CREDENTIALS}
```

### DNS

```shell
nslookup traefik-dashboard.local.example.com
```

### Switch to Production Acme Endpoints (once you verify that traefik-dashboard is working on staging.)

```yaml
...
      caServer: https://acme-v02.api.letsencrypt.org/directory # prod (default)
      #caServer: https://acme-staging-v02.api.letsencrypt.org/directory # staging
...
```

Clear out the existing staging certificates

```shell
cd data
nano acme.json
```

Clear and save

Restart the stack

```shell
docker compose up -d --force-recreate
```

### Adding Another Workload (Homepage Demo)

- Once again where you store your docker stacks. make a folder for homepage.

```shell
mkdir homepage
cd homepage
touch docker-compose.yaml
nano docker-compose.yaml
```

Contents of `docker-compose.yaml`

```yaml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    volumes:
      - /path/to/config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integrations, see alternative methods
    environment:
      PUID: $PUID
      PGID: $PGID
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.homepage.rule=Host(`homepage.local.example.com`)"
      - "traefik.http.routers.homepage.entrypoints=https"
      - "traefik.http.routers.homepage.tls=true"
      - "traefik.http.services.homepage.loadbalancer.server.port=3000"
    networks:
        - proxy

  networks:
    proxy:
      external: true
```

Check DNS

```shell
nslookup homepage.local.example.com
```

Start the new homepage Stack

```shell
docker compose up -d --force-recreate
```

### Adding External Routes

Uncomment a few things:

In `docker-compose.yaml`

```yaml
...
      - ./data/config.yml:/config.yml:ro
...
```

in `traefik.yml`

```yaml
...
  file:
    filename: /config.yml
...
```

edit config

```shell
vim data/config.yml
```

Contents of `config.yml`

```yaml
http:
 #region routers 
  routers:
    proxmox:
      entryPoints:
        - "https"
      rule: "Host(`proxmox.local.example.com`)"
      middlewares:
        - default-headers
        - https-redirectscheme
      tls: {}
      service: proxmox
    pihole:
  
#endregion
#region services
  services:
    proxmox:
      loadBalancer:
        servers:
          - url: "https://192.168.0.17:8006"
        passHostHeader: true
#endregion
  middlewares:
    https-redirectscheme:
      redirectScheme:
        scheme: https
        permanent: true
    default-headers:
      headers:
        frameDeny: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 15552000
        customFrameOptionsValue: SAMEORIGIN
        customRequestHeaders:
          X-Forwarded-Proto: https

    default-whitelist:
      ipAllowList:
        sourceRange:
        - "10.0.0.0/8"
        - "192.168.0.0/16"
        - "172.16.0.0/12"

    secured:
      chain:
        middlewares:
        - default-whitelist
        - default-headers
```

Restart the stack

```shell
docker compose up -d --force-recreate
```

Example of what the Final Product looks like can be found [here](./final-production-files.md)


&#xa0;

<a href="#top">Back to top</a>
