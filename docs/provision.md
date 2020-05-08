# Provision service

Provisioning is a process of configuration of an IoT platform in which system operator creates and sets-up different entities used in the platform - users, channels and things. It is part of process of setting up IoT applications where we connect devices on edge with platform in cloud.

For provisioning we can use [Mainflux CLI][cli] for creating users and for each node in the edge (eg. gateway) required number of things, channels, connecting them and creating certificates if needed.

Provision service is used to set up initial application configuration once user is created. Provision service creates  things, channels, connections and certificates. Once user is created we can use provision to create a setup for edge node in one HTTP request instead of issuing several CLI commands. 

Provision service provides an HTTP API to interact with [Mainflux][mainflux].  

For gateways to communicate with [Mainflux][mainflux] configuration is required (MQTT host, thing, channels, certificates...). Gateway will send a request to [Bootstrap][bootstrap] service providing `<external_id>` and `<external_key>` in HTTP request to get the configuration. To make a request to [Bootstrap][bootstrap] service you can use [Agent][agent] service on a gateway.  

To create bootstrap configuration you can use [Bootstrap][bootstrap] or `Provision` service. [Mainflux UI][mfxui] uses [Bootstrap][bootstrap] service for creating gateway configurations. `Provision` service should provide an easy way of provisioning your gateways i.e creating bootstrap configuration and as many things and channels that your setup requires.  

Also, you may use provision service to create certificates for each thing. Each service running on gateway may require more than one thing and channel for communication. 
If, for example, you are using services [Agent][agent] and [Export][exp] on a gateway you will need two channels for `Agent` (`data` and `control`) and one thing for `Export`. 
Additionally, if you enabled mTLS each service will need its own thing and certificate for access to [Mainflux][mainflux]. 
Your setup could require any number of things and channels, this kind of setup we can call `provision layout`.

Provision service provides a way of specifying this `provision layout` and creating a setup according to that layout by serving requests on `/mapping` endpoint. Provision layout is configured in [config.toml][conftoml].

## Configuration

The service is configured using the environment variables presented in the following [table][config]. Note that any unset variables will be replaced with their default values.


By default, call to `/mapping` endpoint will create one thing and two channels (`control` and `data`) and connect it. If there is a requirement for different provision layout we can use [config][conftoml] file in addition to environment variables. 

For the purposes of running provision as an add-on in docker composition environment variables seems more suitable. Environment variables are set in [.env][env].  

Configuration can be specified in [config.toml][conftoml]. Config file can specify all the settings that environment variables can configure and in addition `/mapping` endpoint provision layout can be configured.

In `config.toml` we can enlist an array of things and channels that we want to create and make connections between them which we call provision layout.

Things Metadata can be whatever suits your needs. Thing that has metadata with `external_id` will have bootstrap configuration created, `external_id` value will be populated with value from [request](#example)). 
Bootstrap configuration can be fetched with [Agent][agent]. For channel's metadata `type` is reserved for `control` and `data` which we use with [Agent][agent].

Example of provision layout below
```toml
[[things]]
  name = "thing"

  [things.metadata]
    external_id = "xxxxxx"


[[channels]]
  name = "control-channel"

  [channels.metadata]
    type = "control"

[[channels]]
  name = "data-channel"

  [channels.metadata]
    type = "data"

[[channels]]
  name = "export-channel"

  [channels.metadata]
    type = "data"
```

## Authentication
In order to create necessary entities provision service needs to authenticate against Mainflux. 
To provide authentication credentials to the provision service you can pass it in as an environment variable or in a config file as Mainflux user and password or as API token (that can be issued on `/users` or `/keys` endpoint of [authn][authn]. 

Additionally, users or API token can be passed in Authorization header, this authentication takes precedence over others.

* `username`, `password` - (`MF_PROVISION_USER`, `MF_PROVISION_PASSWORD` in [.env][env], `mf_user`, `mf_pass` in [config.toml][conftoml]
* API Key - (`MF_PROVISION_API_KEY` in [.env][env] or [config.toml][conftoml]
* `Authorization: Token|ApiKey` - request authorization header containing either users token or API key. Check [authn][authn].

## Running
Provision service can be run as a standalone or in docker composition as addon to the core docker composition.

Standalone:
```bash
MF_PROVISION_BS_SVC_URL=http://localhost:8202/things \
MF_PROVISION_THINGS_LOCATION=http://localhost:8182 \
MF_PROVISION_USERS_LOCATION=http://localhost:8180 \
MF_PROVISION_CONFIG_FILE=docker/addons/provision/configs/config.toml \
build/mainflux-provision
```

Docker composition:
```bash
docker-compose -f docker/addons/provision/docker-compose.yml up
```

## Provision

For the case that credentials or API token is passed in configuration file or environment variables, call to `/mapping` endpoint doesn't require `Authentication` header:
```bash
curl -s -S  -X POST  http://localhost:8888/mapping  -H 'Content-Type: application/json' -d '{"external_id": "33:52:77:99:43", "external_key": "223334fw2"}'
```

In the case that provision service is not deployed with credentials or API key or you want to use user other than one being set in environment (or config file):
```bash
curl -s -S  -X POST  http://localhost:8091/mapping -H "Authorization: <token|api_key>" -H 'Content-Type: application/json' -d '{"external_id": "<external_id>", "external_key": "<external_key>"}'
```

Or if you want to specify a name for thing different than in `config.toml` you can specify post data as:

```json
{"name": "<name>", "external_id": "<external_id>", "external_key": "<external_key>"}
```

Response contains created things, channels and certificates if any:
```json
{
  "things": [
    {
      "id": "c22b0c0f-8c03-40da-a06b-37ed3a72c8d1",
      "name": "thing",
      "key": "007cce56-e0eb-40d6-b2b9-ed348a97d1eb",
      "metadata": {
        "external_id": "33:52:79:C3:43"
      }
    }
  ],
  "channels": [
    {
      "id": "064c680e-181b-4b58-975e-6983313a5170",
      "name": "control-channel",
      "metadata": {
        "type": "control"
      }
    },
    {
      "id": "579da92d-6078-4801-a18a-dd1cfa2aa44f",
      "name": "data-channel",
      "metadata": {
        "type": "data"
      }
    }
  ],
  "whitelisted": {
    "c22b0c0f-8c03-40da-a06b-37ed3a72c8d1": true
  }
}
```

[mainflux]: https://github.com/mainflux/mainflux
[bootstrap]: https://github.com/mainflux/mainflux/tree/master/bootstrap
[export]: https://github.com/mainflux/export
[agent]: https://github.com/mainflux/agent
[mfxui]: https://github.com/mainflux/mainflux/ui
[config]: https://github.com/mainflux/mainflux/tree/master/provision#configuration
[env]: https://github.com/mainflux/mainflux/blob/master/.env
[conftoml]: https://github.com/mainflux/mainflux/blob/master/docker/addons/provision/configs/config.toml
[authn]: https://github.com/mainflux/mainflux/blob/master/authn/README.md
[exp]: https://github.com/mainflux/export
[cli]: https://github.com/mainflux/mainflux/tree/master/cli