<h1 align="center">
    Setting up <a href="https://traefik.io/traefik/">Traefik</a>
</h1>

>[!NOTE]
> Take a look at the files and replace the necessary fields.<br>
> You will need a `DNS` server for this to work correctly.

## How to setup.

* Create docker network
```bash
sudo docker network create proxy
```

* Create the dir & Clone the repo

```bash
mkdir -pv /opt/docker; git clone https://github.com/JershBytes/traefik-config.git /opt/docker/traefik
```
>[!CAUTION]
> After cloning the repo run a chmod 600 on /data/acme.json.

* Generate dashboard credentials

  * Install apache2-utils
  ```bash
  sudo apt install apache2-utils
  ```
  * Run this command.
  ```bash
    echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g # Replacing user with your user.
  ```

* Make a .env file (if you're using plain docker compose.)
  * In the root of the folder.

```bash
touch .env; nano .env
```
* Paste these contents.
```
CF_API_EMAIL=
CF_DNS_API_TOKEN=
TRAEFIK_DASHBOARD_CREDENTIALS=
```

## Deploying Traefik

After this is all done. We can go to the traefik folder and bring up the instance with a `docker compose up -d`

If all goes well you should have an SSL cert for the dashboard. If you do you can switch the `caServer` to use the prod endpoint.


### Links

* [Cloudflare API Token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)


