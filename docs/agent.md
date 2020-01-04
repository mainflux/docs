Agent is application that is used to manage gateways that are connected to Mainflux. It provides a way to send commands to gateway and receive response via mqtt.
There are two type of channels used for **Agent** `data` and `control`.
Over the `control` we are sending commands and receiving response from commands.
Data collected from sensors connected to gateway are being sent over `data` channel.
Agent is able to configure itself provided that [bootstrap server](./bootstrap.md) is running, it will retrieve configuration from bootstrap server provided few arguments - `external_id` and `external_key` see [bootstraping](./bootstrap.md/#bootstraping).

## Run Agent

Before running agent we need to provision a thing and DATA and CONTROL channel. Thing that will be used as gateway representation and make bootstrap configuration.
If using Mainflux UI this is done automatically when adding gateway through UI.


To create a bootstrap config you will need to provision a thing and channels.
You can do it with 
```
docker run -it mainflux/cli -m http://mainflux-host provision test
```

if you run this command against your locally deployed Mainflux you will have to replace `mainflux-host` with docker network IP
```
sudo ifconfig -a
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500                                                                                                                 
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255                                                                                                
        inet6 fe80::42:91ff:fe1f:c31f  prefixlen 64  scopeid 0x20<link>                                                                                               
        ether 02:42:91:1f:c3:1f  txqueuelen 0  (Ethernet)                                                                                                             
        RX packets 13  bytes 3202 (3.2 KB)                                                                                                                            
        RX errors 0  dropped 0  overruns 0  frame 0                                                                                                                   
        TX packets 26  bytes 5671 (5.6 KB)                                                                                                                            
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0 

```

Take a not on inet and then run
```
docker run -it mainflux/cli -m http://172.17.0.1 provision test
{
  "email": "focused_bassi@email.com",
  "password": "123"
}


"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzU1NjEwNDAsImlhdCI6MTU3NTUyNTA0MCwiaXNzIjoibWFpbmZsdXgiLCJzdWIiOiJmb2N1c2VkX2Jhc3NpQGVtYWlsLmNvbSJ9.mW2Lbuscqu7K7e2QIaeXc5WsN4SU13RJioXrAsGBr3I"


[
  {
    "id": "2a94f899-be92-4023-910b-1670f6cedf77",
    "key": "d476eca9-9ad2-4046-8c7b-4219ea0c180a",
    "name": "d0"
  },
  {
    "id": "5f9d61de-90fa-45ae-9790-17b3cf38bad1",
    "key": "16719d37-8a8f-4d4c-a705-9de4aa7c39d4",
    "name": "d1"
  }
]


[
  {
    "id": "0af460d5-59c0-4140-81a3-23fdb6a6007f",
    "name": "c0"
  },
  {
    "id": "37acadcf-6b89-40f9-9fd9-9d350a32b40f",
    "name": "c1"
  }
]

```
This will create two things and two channels assigned to those two things display it along with authorization token.
Take a note on first thing and on channels and use it in the next commands as thing_id and in channels for data and control channels

Then use this token to create bootstrap configuration
```
curl -s -S -i -X POST -H "Authorization: TOKEN" -H "Content-Type: application/json" http://localhost:8202/things/configs -d '{ "external_id":"44:e1:2d:e6:cf:03","thing_id": "THING_ID","external_key":"edged","name":"edged","channels":[ "CONTROL_CH", "DATA_CH" ], "content" :"{\"log_level\":\"debug\", \"http_port\":\"9000\", \"mqtt_url\":\"tcp://localhost:18831\",\"edgex_url\":\"http://localhost:48090/api/v1/\" }"}'

```


- `external_id` is usually MAC address 
- `external_key` is key that will be provided to agent process
- `thing_id` is mainflux thing id 
- `channels` is 2-element array where first channel is CONTROL and second is DATA, both channels should be assigned to thing
- `content` is used for configuring parameters of agent service 


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

### Executing commands via Agent

To see how commands are executed on remote device via **agent** subscribe first to CONTROL_CH like this

```
mosquitto_sub -u <thing_id> -P <thing_key> -t channels/CONTROL_CH/messages/res -h localhost -p 1883
```

Then send command to be executed send senml vi mqtt like this ( use different terminal than for subscribe)
```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/CONTROL_CH/messages/req -h localhost -p 1883  -m  '[{"bn":"1:", "n":"exec", "vs":"ls, -l"}]'
```

In the terminal where you subscribed you will get result of executing `ls -l` in the dir where your agent is running (`build`) so you should get result something like this
```
[{"bn":"1","n":"ls","vs":"total 14504\n-rw-r--r-- 1 mirko mirko      448 дец  5 12:03 config.toml\n-rwxrwxr-x 1 mirko mirko 14848000 дец  4 18:02 mainflux-agent\n"}]
```

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