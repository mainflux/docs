Agent is service that is used to manage gateways that are connected to Mainflux in cloud. It provides a way to send commands to gateway and receive response via mqtt.
There are two types of channels used for **Agent** `data` and `control`. Over the `control` we are sending commands and receiving response from commands.
Data collected from sensors connected to gateway are being sent over `data` channel. Agent is able to configure itself provided that [bootstrap server](./bootstrap.md) is running, it will retrieve configuration from bootstrap server provided few arguments - `external_id` and `external_key` see [bootstraping](./bootstrap.md/#bootstraping).

Agent service has following features:
* Remote execution of commands
* Remote terminal, remote session to `bash` managed by `Agent`
* Heartbeat - listening to NATS topic `heartbeat.>` it can remotely provide info on running services, if services are publishing heartbeat ( like [Export](export.md))
* Proxying commands to other gateway services
* Edgex SMA - remotely making requests to EdgeX endpoints and fetching results, if EdgeX is deployed.


## Run Agent

Before running agent we need to provision a thing and DATA and CONTROL channel. Thing that will be used as gateway representation and make bootstrap configuration.
If using Mainflux UI this is done automatically when adding gateway through UI.
Gateway can be provisioned with [`provision`](provision.md) service.

When you provisioned gateway as described in [provision](provision.md) you can check results

```bash
curl -s -S -X GET http://mainflux-domain.com:8202/things/bootstrap/<external_id> -H "Authorization: <external_key>" -H 'Content-Type: application/json' |jq
```

```json
{
  "mainflux_id": "e22c383a-d2ab-47c1-89cd-903955da993d",
  "mainflux_key": "fc987711-1828-461b-aa4b-16d5b2c642fe",
  "mainflux_channels": [
    {
      "id": "fa5f9ba8-a1fc-4380-9edb-d0c23eaa24ec",
      "name": "control-channel",
      "metadata": {
        "type": "control"
      }
    },
    {
      "id": "24e5473e-3cbe-43d9-8a8b-a725ff918c0e",
      "name": "data-channel",
      "metadata": {
        "type": "data"
      }
    },
    {
      "id": "1eac45c2-0f72-4089-b255-ebd2e5732bbb",
      "name": "export-channel",
      "metadata": {
        "type": "export"
      }
    }
  ],
  "content": "{\"agent\":{\"edgex\":{\"url\":\"http://localhost:48090/api/v1/\"},\"heartbeat\":{\"interval\":\"30s\"},\"log\":{\"level\":\"debug\"},\"mqtt\":{\"mtls\":false,\"qos\":0,\"retain\":false,\"skip_tls_ver\":true,\"url\":\"tcp://mainflux-domain.com:1883\"},\"server\":{\"nats_url\":\"localhost:4222\",\"port\":\"9000\"},\"terminal\":{\"session_timeout\":\"30s\"}},\"export\":{\"exp\":{\"cache_db\":\"0\",\"cache_pass\":\"\",\"cache_url\":\"localhost:6379\",\"log_level\":\"debug\",\"nats\":\"nats://localhost:4222\",\"port\":\"8172\"},\"mqtt\":{\"ca_path\":\"ca.crt\",\"cert_path\":\"thing.crt\",\"channel\":\"\",\"host\":\"tcp://mainflux-domain.com:1883\",\"mtls\":false,\"password\":\"\",\"priv_key_path\":\"thing.key\",\"qos\":0,\"retain\":false,\"skip_tls_ver\":false,\"username\":\"\"},\"routes\":[{\"mqtt_topic\":\"\",\"nats_topic\":\"channels\",\"subtopic\":\"\",\"type\":\"mfx\",\"workers\":10},{\"mqtt_topic\":\"\",\"nats_topic\":\"export\",\"subtopic\":\"\",\"type\":\"default\",\"workers\":10}]}}"
```


- `external_id` is usually MAC address, but anything that suits applications requirements can be used
- `external_key` is key that will be provided to agent process
- `thing_id` is mainflux thing id 
- `channels` is 2-element array where first channel is CONTROL and second is DATA, both channels should be assigned to thing
- `content` is used for configuring parameters of agent and export service.


Then to start the agent service you can do it like this

```
git clone https://github.com/mainflux/agent
make
cd build

MF_AGENT_LOG_LEVEL=debug \
MF_AGENT_BOOTSTRAP_KEY=edged \
MF_AGENT_BOOTSTRAP_ID=34:e1:2d:e6:cf:03 ./mainflux-agent

{"level":"info","message":"Requesting config for 34:e1:2d:e6:cf:03 from http://localhost:8202/things/bootstrap","ts":"2019-12-05T04:47:24.98411512Z"}
{"level":"info","message":"Getting config for 34:e1:2d:e6:cf:03 from http://localhost:8202/things/bootstrap succeeded","ts":"2019-12-05T04:47:24.995465239Z"}
{"level":"info","message":"Connected to MQTT broker","ts":"2019-12-05T04:47:25.009645082Z"}
{"level":"info","message":"Agent service started, exposed port 9000","ts":"2019-12-05T04:47:25.009755345Z"}
{"level":"info","message":"Subscribed to MQTT broker","ts":"2019-12-05T04:47:25.012930443Z"}

```

 - `MF_AGENT_BOOTSTRAP_KEY` - is `external_key` in bootstrap configuration.
 - `MF_AGENT_BOOSTRAP_ID` - is `external_id` in bootstrap configuration.

### Remote execution of commands via Agent

```bash
# Set connection parameters as environment variables in shell
CH=`curl -s -S -X GET http://some-domain-name:8202/things/bootstrap/34:e1:2d:e6:cf:03 -H "Authorization: edged" -H 'Content-Type: application/json' | jq -r '.mainflux_channels[0].id'`
TH=`curl -s  -S -X GET http://some-domain-name:8202/things/bootstrap/34:e1:2d:e6:cf:03 -H "Authorization: edged" -H 'Content-Type: application/json' | jq -r .mainflux_id`
KEY=`curl -s  -S -X GET http://some-domain-name:8202/things/bootstrap/34:e1:2d:e6:cf:03 -H "Authorization: edged" -H 'Content-Type: application/json' | jq -r .mainflux_key`

# Subscribe for response
mosquitto_sub -d -u $TH -P $KEY  -t channels/$CH/messages/res/# -h some-domain-name -p 1883

# Publish command e.g `ls`
mosquitto_pub -d -u $TH -P $KEY  -t channels/$CH/messages/req -h some-domain-name -p 1883  -m '[{"bn":"1:", "n":"exec", "vs":"ls, -l"}]'


```

### Remote terminal 

This can be checked from the UI, click on the details for gateway and below the gateway parameters you will se box with prompt, if `agent` is running and it is properly connected you should be able to execute commands remotely.

### Heartbeat 
If there are services that are running on same gateway as `agent` and they are publishing heartbeat to NATS subject `heartbeat.service_name.service`
You can get the list of services by sending following mqtt message
```

# View services that are sending heartbeat
mosquitto_pub -d -u $TH -P $KEY  -t channels/$CH/messages/req -h some-domain-name -p 1883  -m '[{"bn":"1:", "n":"service", "vs":"view"}]'
```
Response can be observed on `channels/$CH/messages/res/#`

## Proxying commands

You can send commands to services running on the same edge gateway as Agent if they are subscribed on same NATS server and correct subject.

Service commands are being sent via MQTT to topic:

`channels/<control_channel_id>/messages/services/<service_name>/<subtopic>`
  
when messages is received Agent forwards them to NATS on subject:

`commands.<service_name>.<subtopic>`

Payload is up to the application and service itself.


## EdgeX 

[Edgex](https://github.com/edgexfoundry/edgex-go) control messages are sent and received over control channel. MF sends a control SenML of the following form:
```
[{"bn":"<uuid>:", "n":"control", "vs":"<cmd>, <param>, edgexsvc1, edgexsvc2, …, edgexsvcN"}}]
```
For example,
```
[{"bn":"1:", "n":"control", "vs":"operation, stop, edgex-support-notifications, edgex-core-data"}]
```
Agent, on the other hand, returns a response SenML of the following form:
```
[{"bn":"<uuid>:", "n":"<>", "v":"<RESP>"}]
```
### Remote Commands
EdgeX defines SMA commands in the following [RAML file](https://github.com/edgexfoundry/edgex-go/blob/master/api/raml/system-agent.raml)

Commands are:

* OPERATION
* CONFIG
* METRICS
* PING

**Operation**
  
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/req -h localhost -m '[{"bn":"1:", "n":"control", "vs":"edgex-operation, start, edgex-support-notifications, edgex-core-data"}]'
```
**Config**
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/req -h localhost -m '[{"bn":"1:", "n":"control", "vs":"edgex-config, edgex-support-notifications, edgex-core-data"}]'
```
**Metrics**
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/req -h localhost -m '[{"bn":"1:", "n":"control", "vs":"edgex-metrics, edgex-support-notifications, edgex-core-data"}]'
```

If you subscribe to

```
mosquitto_sub -u <thing_id> -P <thing_key> -t channels/<channel_id>/messages/#
```
You can observe commands and response from commands executed against edgex
```
[{"bn":"1:", "n":"control", "vs":"edgex-metrics, edgex-support-notifications, edgex-core-data"}]                                                                            
[{"bn":"1","n":"edgex-metrics","vs":"{\"Metrics\":{\"edgex-core-data\":{\"CpuBusyAvg\":15.568632467698606,\"Memory\":{\"Alloc\":2040136,\"Frees\":876344,\"LiveObjects\":15134,\"Mallocs\":891478,\"Sys\":73332984,\"TotalAlloc\":80657464}},\"edgex-support-notifications\":{\"CpuBusyAvg\":14.65381169745318,\"Memory\":{\"Alloc\":961784,\"Frees\":127430,\"LiveObjects\":6095,\"Mallocs\":133525,\"Sys\":72808696,\"TotalAlloc\":11665416}}}}\n"}] 
```

