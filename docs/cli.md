## CLI

Mainflux CLI makes it easy to manage users, things, channels and messages.

CLI can be downloaded as separate asset from [project realeses](https://github.com/mainflux/mainflux/releases) or it can be built with `GNU Make` tool:

```
make cli
```

which will build `mainflux-cli` in `<project_root>/build` folder.

Executing `build/mainflux-cli` without any arguments will output help with all available commands and flags:

```
Usage:
  mainflux-cli [command]

Available Commands:
  channels    Channels management
  help        Help about any command
  messages    Send or read messages
  provision   Bulk create things and channels from a config file
  things      Things management
  users       Users management
  version     Mainflux system version

Flags:
  -c, --content-type string    Mainflux message content type (default "application/senml+json")
  -h, --help                   help for mainflux-cli
  -a, --http-prefix string     Mainflux http adapter prefix (default "http")
  -i, --insecure               Do not check for TLS cert
  -l, --limit uint             limit query parameter (default 100)
  -m, --mainflux-url string    Mainflux host URL (default "http://localhost")
  -o, --offset uint            offset query parameter
  -t, --things-prefix string   Mainflux things service prefix
  -u, --users-prefix string    Mainflux users service prefix

Use "mainflux-cli [command] --help" for more information about a command.
```

It is also possible to use the docker image `mainflux/cli` to execute CLI command:

```
docker run -it --rm mainflux/cli -m http://<IP_SERVER> [command]
```

You can execute each command with `-h` flag for more information about that command, e.g.

```
./mainflux-cli channels -h
```

will get you usage info:

```
Channels management: create, get, update or delete Channels and get list of Things connected to Channels

Usage:
  mainflux-cli channels [flags]
  mainflux-cli channels [command]

Available Commands:
  connections connections <channel_id> <user_auth_token>
  create      create <JSON_channel> <user_auth_token>
  delete      delete <channel_id> <user_auth_token>
  get         get <channel_id | all> <user_auth_token>
  update      update <JSON_string> <user_auth_token>

```

## Service
#### Get the version of Mainflux services
```
mainflux-cli version
```

### Users management
#### Create User
```
mainflux-cli users create <user_email> <user_password>
```

#### Login User
```
mainflux-cli users token <user_email> <user_password>
```

#### Retrieve User
```
mainflux-cli users get <user_auth_token>
```

#### Update User Metadata
```
mainflux-cli users update '{"key1":"value1", "key2":"value2"}' <user_auth_token>
```

#### Update User Password
```
mainflux-cli users password <old_password> <password> <user_auth_token>
```

### System Provisioning
#### Create Thing
```
mainflux-cli things create '{"name":"myThing"}' <user_auth_token>
```

#### Create Thing with metadata
```
mainflux-cli things create '{"name":"myThing", "metadata": {\"key1\":\"value1\"}}' <user_auth_token>
```

#### Bulk Provision Things

```bash
mainflux-cli provision things <file> <user_auth_token>
```

* `file` - A CSV or JSON file containing things
* `user_auth_token` - A valid user auth token for the current system

#### Update Thing
```
mainflux-cli things update '{"id":"<thing_id>", "name":"myNewName"}' <user_auth_token>
```

#### Remove Thing
```
mainflux-cli things delete <thing_id> <user_auth_token>
```

#### Retrieve a subset list of provisioned Things
```
mainflux-cli things get all --offset=1 --limit=5 <user_auth_token>
```

#### Retrieve Thing By ID
```
mainflux-cli things get <thing_id> <user_auth_token>
```

#### Create Channel
```
mainflux-cli channels create '{"name":"myChannel"}' <user_auth_token>
```

#### Bulk Provision Channels

```bash
mainflux-cli provision channels <file> <user_auth_token>
```

* `file` - A CSV or JSON file containing channels
* `user_auth_token` - A valid user auth token for the current system

#### Update Channel
```
mainflux-cli channels update '{"id":"<channel_id>","name":"myNewName"}' <user_auth_token>

```
#### Remove Channel
```
mainflux-cli channels delete <channel_id> <user_auth_token>
```

#### Retrieve a subset list of provisioned Channels
```
mainflux-cli channels get all --offset=1 --limit=5 <user_auth_token>
```

#### Retrieve Channel By ID
```
mainflux-cli channels get <channel_id> <user_auth_token>
```

### Access control
#### Connect Thing to Channel
```
mainflux-cli things connect <thing_id> <channel_id> <user_auth_token>
```

#### Bulk Connect Things to Channels

```bash
mainflux-cli provision connect <file> <user_auth_token>
```

* `file` - A CSV or JSON file containing thing and channel ids
* `user_auth_token` - A valid user auth token for the current system

An example CSV file might be

```csv
<thing_id>,<channel_id>
<thing_id>,<channel_id>
```

in which the first column is thing IDs and the second column is channel IDs.  A connection will be created for each thing to each channel.  This example would result in 4 connections being created.

A comparable JSON file would be

```json
{
    "thing_ids": [
        "<thing_id>",
        "<thing_id>"
    ],
    "channel_ids": [
        "<channel_id>",
        "<channel_id>"
    ]
}
```

#### Disconnect Thing from Channel
```
mainflux-cli things disconnect <thing_id> <channel_id> <user_auth_token>

```

#### Retrieve a subset list of Channels connected to Thing
```
mainflux-cli things connections <thing_id> <user_auth_token>
```

#### Retrieve a subset list of Things connected to Channel
```
mainflux-cli channels connections <channel_id> <user_auth_token>
```


### Messaging
#### Send a message over HTTP
```
mainflux-cli messages send <channel_id> '[{"bn":"Dev1","n":"temp","v":20}, {"n":"hum","v":40}, {"bn":"Dev2", "n":"temp","v":20}, {"n":"hum","v":40}]' <thing_auth_token>
```

#### Read messages over HTTP
```
mainflux-cli messages read <channel_id> <thing_auth_token>
```

### Bootstrap

#### Add configuration
```
mainflux-cli bootstrap add '{"external_id": "myExtID", "external_key": "myExtKey", "name": "myName", "content": "myContent"}' <user_auth_token>
```

#### View configuration
```
mainflux-cli bootstrap view <thing_id> <user_auth_token>
```

#### Update configuration
```
mainflux-cli bootstrap update '{"MFThing":"<thing_id>", "name": "newName", "content": "newContent"}' <user_auth_token>
```

#### Remove configuration
```
mainflux-cli bootstrap remove <thing_id> <user_auth_token>
```

#### Bootstrap configuration
```
mainflux-cli bootstrap bootstrap <external_id> <external_key>
```
