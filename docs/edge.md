## Edge 
Edge services:

![Edge][edge]




* [Agent][agent]
* [Export][export]

Start the Mainflux:

```bash
docker-compose -f docker/docker-compose.yml up
```

Start the Boostrap service:

```bash
docker-compose -f docker/addons/bootstrap/docker-compose.yml up
```

Start the Provision service

```bash
docker-compose -f docker/addons/provision/docker-compose.yml up
```

Create user:
```bash
mainflux-cli -m http://localhost:8180 users create test@email.com 12345678
```

```bash
curl -s -S  -X POST  http://localhost:8190/mapping -H "Authorization: $TOK" -H 'Content-Type: application/json'   -d '{"name":"testing",  "external_id" : "54:FG:66:DC:43", "external_key":"223334fw2" }' | jq
```
```json
{
  "things": [
    {
      "id": "88529fb2-6c1e-4b60-b9ab-73b5d89f7404",
      "name": "thing",
      "key": "3529c1bb-7211-4d40-9cd8-b05833196093",
      "metadata": {
        "external_id": "54:FG:66:DC:43"
      }
    }
  ],
  "channels": [
    {
      "id": "1aa3f736-0bd3-44b5-a917-a72cc743f633",
      "name": "control-channel",
      "metadata": {
        "type": "control"
      }
    },
    {
      "id": "e2adcfa6-96b2-425d-8cd4-ff8cb9c056ce",
      "name": "data-channel",
      "metadata": {
        "type": "data"
      }
    }
  ],
  "whitelisted": {
    "88529fb2-6c1e-4b60-b9ab-73b5d89f7404": true
  }
}
```
## Agent

Start the [NATS][nats] and [Agent][agent] service:
```
bash
gnatsd
MF_AGENT_BOOTSTRAP_ID=54:FG:66:DC:43 \
MF_AGENT_BOOTSTRAP_KEY="223334fw2" \
MF_AGENT_BOOTSTRAP_URL=http://localhost:8202/things/bootstrap \
build/mainflux-agent
{"level":"info","message":"Requesting config for 54:FG:66:DC:43 from http://localhost:8202/things/bootstrap","ts":"2020-05-07T15:50:58.041145096Z"}
{"level":"info","message":"Getting config for 54:FG:66:DC:43 from http://localhost:8202/things/bootstrap succeeded","ts":"2020-05-07T15:50:58.120779415Z"}
{"level":"info","message":"Saving export config file /configs/export/config.toml","ts":"2020-05-07T15:50:58.121602229Z"}
{"level":"warn","message":"Failed to save export config file Error writing config file: open /configs/export/config.toml: no such file or directory","ts":"2020-05-07T15:50:58.121752142Z"}
{"level":"info","message":"Client agent-88529fb2-6c1e-4b60-b9ab-73b5d89f7404 connected","ts":"2020-05-07T15:50:58.128500603Z"}
{"level":"info","message":"Agent service started, exposed port 9003","ts":"2020-05-07T15:50:58.128531057Z"}
```

## Export

```
git clone https://github.com/mainflux/export
make
```
Edit the `configs/config.toml` setting 
- `username` to thing from the results of provision request
- `password` to key from the results of provision request
- `mqtt_topic` in routes set to `channels/<channel_data_id>/messages` from results of provision
- `nats_topic` put whatever you need, export will subscribe to `export.<nats_topic>` and forward messages to MQTT
  
```toml
[exp]
  cache_pass = ""
  cache_url = ""
  log_level = "debug"
  nats = "localhost:4222"
  port = "8170"

[mqtt]
  ca_path = ""
  cert_path = ""
  host = "tcp://localhost:1883"
  mtls = false
  password = "3529c1bb-7211-4d40-9cd8-b05833196093"
  priv_key_path = ""
  qos = 0
  retain = false
  skip_tls_ver = false
  username = "88529fb2-6c1e-4b60-b9ab-73b5d89f7404"

[[routes]]
  mqtt_topic = "channels/1aa3f736-0bd3-44b5-a917-a72cc743f633/messages"
  nats_topic = "test.>"
  workers = 10
```


```bash
cd build
./mainflux-export
2020/05/07 17:36:57 Configuration loaded from file ../configs/config.toml
{"level":"info","message":"Export service started, exposed port :8170","ts":"2020-05-07T15:36:57.528398548Z"}
{"level":"debug","message":"Client export-88529fb2-6c1e-4b60-b9ab-73b5d89f7404 connected","ts":"2020-05-07T15:36:57.528405818Z"}
```

### Testing Export
```bash
git clone https://github.com/nats-io/nats.go
cd github.com/nats-io/nats.go/
```


[export]: (https://github.com/mainflux/docs/blob/master/docs/export.md)
[agent]: (https://github.com/mainflux/docs/blob/master/docs/agent.md)
[edge]: (https://raw.githubusercontent.com/mainflux/docs/master/docs/img/edge/edge.png)
[agconf]:(https://github.com/mainflux/mainflux/blob/master/docker/addons/provision/configs/config.toml#L2)
[nats]: (https://github.com/nats-io/nats.go)