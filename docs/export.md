# Export
Mainflux Export service can send message from one Mainflux cloud to another via MQTT, or it can send messages from edge gateway to Mainflux Cloud.
Export service is subscribed to local message bus and connected to MQTT broker in the cloud.  
Messages collected on local message bus are redirected to the cloud.
When connection is lost, messages from the local bus are stored into `Redis` stream. Upon connection reestablishment `Export` service consumes messages from `Redis` stream and sends it to the Mainflux cloud.


## Install
Get the code:

```bash
go get github.com/mainflux/export
cd $GOPATH/github.com/mainflux/export
```

Make:
```bash
make
```

## Usage

```bash
cd build
./mainflux-export
```

## Configuration
By default `Export` service looks for config file at [`../configs/config.toml`][conftoml] if no env vars are specified.  

```toml
[exp]
  cache_pass = ""
  cache_url = "localhost:6379"
  cache_db = "0"
  log_level = "debug"
  nats = "localhost:4222"
  port = "8170"

[mqtt]
  username = "<thing_id>"
  password = "<thing_password>"
  ca = "ca.crt"
  cert = "thing.crt"
  mtls = "false"
  priv_key = "thing.key"
  retain = "false"
  skip_tls_ver = "false"
  url = "tcp://mainflux.com:1883"

[[routes]]
  mqtt_topic = "channel/<channel_id>/messages"
  subtopic = "subtopic"
  nats_topic = ".>"
  type = "plain"
  workers = 10
```
### Http port

- `port` - HTTP port where status of `Export` service can be fetched.
```bash
curl -X GET http://localhost:8170/version
{"service":"export","version":"0.0.1"}%
``` 

### Redis connection

To configure `Redis` connection settings `cache_url`, `cache_pass`, `cache_db` in `config.toml` are used.

### MQTT connection

To establish connection to MQTT broker following settings are needed:
- `username` - Mainflux <thing_id>
- `password` - Mainflux <thing_key>
- `url` - url of MQTT broker

Additionally, you will need MQTT client certificates if you enable mTLS. To obtain certificates `ca.crt`, `thing.crt` and key `thing.key` follow instructions [here](https://mainflux.readthedocs.io/en/latest/authentication/#mutual-tls-authentication-with-x509-certificates).

### Routes 

Routes are being used for specifying which subscriber's topic(subject) goes to which publishing topic.
Currently only MQTT is supported for publishing. To match Mainflux requirements `mqtt_topic` must contain `channel/<channel_id>/messages`, additional subtopics can be appended.

- `mqtt_topic` - `channel/<channel_id>/messages/<custom_subtopic>`
- `nats_topic` - `Export` service will be subscribed to NATS subject `export.<nats_topic>`
- `subtopic` - messages will be published to MQTT topic `<mqtt_topic>/<subtopic>/<nats_subject>`, where dots in nats_subject are replaced with '/'
- `workers` control number of workers that will be used for message forwarding.
- `type` - specifies message transformation, currently only `plain` is supported, meaning no transformation.

Before running `Export` service edit `configs/config.toml` and provide `username`, `password` and `url`
 * `username` - matches `thing_id` in Mainflux cloud instance
 * `password` - matches `thing_key`
 * `channel` - MQTT part of the topic where to publish MQTT data (`channel/<channel_id>/messages` is format of mainflux MQTT topic) and plays a part in authorization.

In order for `Export` service to listen on Mainflux NATS deployed on the same machine NATS port must be exposed.
Edit Mainflux [docker-compose.yml][docker-compose]. NATS section must look like below:
```
  nats:
    image: nats:1.3.0
    container_name: mainflux-nats
    restart: on-failure
    networks:
      - mainflux-base-net
    ports:
      - 4222:4222
```
  
## Environment variables

Service will look for `config.toml` first and if not found it will be configured with env variables and new config file specified with `MF_EXPORT_CONF_PATH` will be saved with values populated from env vars.  
The service is configured using the [environment variables](env). Note that any unset variables will be replaced with their default values.

For values in environment variables to take effect make sure that there is no `MF_EXPORT_CONF` file.

If you run with environment variables you can create config file:
```bash
MF_EXPORT_PORT=8178 \
MF_EXPORT_LOG_LEVEL=debug \
MF_EXPORT_MQTT_HOST=tcp://localhost:1883 \
MF_EXPORT_MQTT_USERNAME=<thing_id> \
MF_EXPORT_MQTT_PASSWORD=<thing_key> \
MF_EXPORT_MQTT_CHANNEL=<channel_id> \
MF_EXPORT_MQTT_SKIP_TLS=true \
MF_EXPORT_MQTT_MTLS=false \
MF_EXPORT_MQTT_CA=ca.crt \
MF_EXPORT_MQTT_CLIENT_CERT=thing.crt \
MF_EXPORT_MQTT_CLIENT_PK=thing.key \
MF_EXPORT_CONF_PATH=export.toml \
../build/mainflux-export&
```

Service will be subscribed to NATS `export.>` subject and send messages to `channels/<MF_EXPORT_MQTT_CHANNEL>/messages`.
Created `export.toml` you can edit to add different routes and use in next run.

## How to save config via agent

Configuration file for `Export` service can be send over MQTT using [Agent][agent] service.

```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<control_ch_id>/messages/req -h localhost -p 18831  -m  "[{\"bn\":\"1:\", \"n\":\"config\", \"vs\":\"<config_file_path>, <file_content_base64>\"}]"
```

`vs="config_file_path, file_content_base64"` - vs determines where to save file and contains file content in base64 encoding payload:
```
b,_ := toml.Marshal(export.Config)
payload := base64.StdEncoding.EncodeToString(b)
```

[conftoml]: (https://github.com/mainflux/export/blob/master/configs/config.toml)
[docker-compose]: (https://github.com/mainflux/mainflux/docker/docker-compose.yml)
[env]: (https://github.com/mainflux/export#environmet-variables)
[agent]: (https://github.com/mainflux/agent)