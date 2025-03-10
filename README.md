# DomIoT
A Home Assistant for dependent or hospitalized people at home.

> This repository is part of the DomIoT project. This project was initiated as a school project at the Institut National des Sciences Appliquées de Rennes (INSA Rennes). 

## Getting started
### Setup a local domain name
with dnsmasq, resolve home.internal to 127.0.0.1

> [!WARNING]  
> TODO
Few links to help you :
* https://doc.ubuntu-fr.org/configuration_serveur_dns_dhcp
* https://stackoverflow.com/questions/22313142/wildcard-subdomains-with-dnsmasq

### Setup the project
The first step is to clone the repository and setup an `.env` file. Run the following command to generate a password and secret key and write them to your `.env` file :

```bash
echo "PG_PASS=$(openssl rand -base64 36 | tr -d '\n')" >> .env
echo "AUTHENTIK_SECRET_KEY=$(openssl rand -base64 60 | tr -d '\n')" >> .env
```

Once it's done, you can start the project with the following command :
```bash
docker compose up -d
```

- login to http://home.internal:81 with the default credentials (`admin@example.com:changeme`) set new credentials and set-up the following proxies :
    - auth.home.internal -> `http://server:9000`
    - hass.home.internal -> `http://authentik_proxy:8123`
    - manager.home.internal -> `http://home.internal:81`

Please activate Websockets support for each proxy.

### Authentik setup
Go to http://auth.home.internal/if/flow/initial-setup/ setup your credentials. Go then to "Admin Interface" and then Applications > Providers. Create a new "Proxy Provider" with the following settings :
- Name: `hass-proxy-provider`
- Authorization flow : `default-provider-authorization-explicit-consent`
- External host: `http://hass.home.internal`
- Internal host: `http://homeassistant:8123`  

No we will create a new Application. Go to Applications > Applications and create a new Application with the following settings :
- Name: `Home Assistant`
- Slug: `homeassistant`
- Provider: `hass-proxy-provider`

Then go to Applications > Outposts and create a new Outpost with the following settings :
- Name: `hass-proxy-outpost`
- Type: `Proxy`
- Applications : Choose the Home Assistant application you created earlier

Once the outpost is created, click on "View Deployment Info", and "Click to copy token". This will put the token in the clipboard. If not, a popup will show the token. Copy it and replace in the `docker-compose.yml` file the `AUTHENTIK_TOKEN` value with the token you just copied.

Here you go! You can now access Home Assistant at http://hass.home.internal.

> [!NOTE]
> Make sure you connect with a user that has a username that already exists in Home Assistant (otherwise, you will not be able to access Home Assistant).