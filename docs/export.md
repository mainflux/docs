Bridging between two mainflux instances can be done with [export](https://github.com/mainflux/export) service is build to bridge between two Mainflux instances. 
For example we can run Mainflux on the gateway and we want to export the data to cloud instance of Mainflux as well. Or we can just export any data from local message bus on the gateway to the cloud.
Export service is listening to the NATS and forwards payload to the specified MQTT channel.

## Run Export service

Easiest way to run export service is 

```
    git clone github.com/mainflux/export
    cd github.com/mainflux/export   
    make
    cd build
    ./mainflux-export

```

service will pickup config file  `github.com/mainflux/export/configs/config.toml`

```

[exp]
  log_level = "debug"
  nats = "nats://localhost:4222"
  port = "8170"

[mqtt]
  ca_path = "ca.crt"
  cert_path = "thing.crt"
  username= "53a795db-e46c-4d75-8ddc-6d2975337a4f"
  password = "e2032af3-7323-47d6-8e8d-abe9eb0a0a8f"
  host = "tcp://142.93.118.47:18831"
  mtls = false
  priv_key_path = "thing.key"
  qos = 0
  retain = false
  skip_tls_ver = false

```

to configure the `config.toml` so that it can connect to your Mainflux cloud instance you need to 
provide 

- `username` - Mainflux thing id
- `password` - Mainflux thing key
- `channel` -  should be in format channels/<channel_id>/messages where channel_id is Mainflux channel assigned to thing
- `host` - MQTT host `tcp://host.name:1883` for plain or `tcps://host.name:8883` for mtls

you can start export service in docker as well
```docker-compose -f docker/docker-compose.yml up```

this requires that you have previously brought up Mainflux instance with docker-compose as it depends on docker network created from `docker/docker-compose.yml`

If you are running MTLS on your Mainflux cloud instance you will need to setup export service for MTLS communication. You need to set `mtls=true` and you will need to provide `thing.crt` and `thing.key` as well as `ca.crt` that you produced for provisioned thing [MTLS Authentication](./authentication.md)