## CLI

Mainflux CLI makes it easy to manage users, things, channels and messages.

CLI can be downloaded as separate asset from [project realeses](https://github.com/mainflux/mainflux/releases) or it can be built with `GNU Make` tool:

The [GOPATH](https://golang.org/doc/gopath_code) should be set and `go get` the `mainflux` code to build the cli successfully.

Get the mainflux code

```bash
go get github.com/mainflux/mainflux
```

Build the mainflux-cli

```bash
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

```bash
docker run -it --rm mainflux/cli -m http://<IP_SERVER> [command]
```

You can execute each command with `-h` flag for more information about that command, e.g.

```bash
mainflux-cli channels -h
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
```bash
mainflux-cli version
```

### Users management
#### Create User
```bash
mainflux-cli users create <user_email> <user_password>
```

#### Login User
```bash
mainflux-cli users token <user_email> <user_password>
```

#### Retrieve User
```bash
mainflux-cli users get <user_auth_token>
```

#### Update User Metadata
```bash
mainflux-cli users update '{"key1":"value1", "key2":"value2"}' <user_auth_token>
```

#### Update User Password
```bash
mainflux-cli users password <old_password> <password> <user_auth_token>
```

### System Provisioning
#### Create Thing
```bash
mainflux-cli things create '{"name":"myThing"}' <user_auth_token>
```

#### Create Thing with metadata
```bash
mainflux-cli things create '{"name":"myThing", "metadata": {\"key1\":\"value1\"}}' <user_auth_token>
```

#### Bulk Provision Things

```bash
mainflux-cli provision things <file> <user_auth_token>
```

* `file` - A CSV or JSON file containing things (must have extension `.csv` or `.json`)
* `user_auth_token` - A valid user auth token for the current system

#### Update Thing
```bash
mainflux-cli things update '{"id":"<thing_id>", "name":"myNewName"}' <user_auth_token>
```

#### Remove Thing
```bash
mainflux-cli things delete <thing_id> <user_auth_token>
```

#### Retrieve a subset list of provisioned Things
```bash
mainflux-cli things get all --offset=1 --limit=5 <user_auth_token>
```

#### Retrieve Thing By ID
```bash
mainflux-cli things get <thing_id> <user_auth_token>
```

#### Create Channel
```bash
mainflux-cli channels create '{"name":"myChannel"}' <user_auth_token>
```

#### Bulk Provision Channels
```bash
mainflux-cli provision channels <file> <user_auth_token>
```

* `file` - A CSV or JSON file containing channels (must have extension `.csv` or `.json`)
* `user_auth_token` - A valid user auth token for the current system

#### Update Channel
```bash
mainflux-cli channels update '{"id":"<channel_id>","name":"myNewName"}' <user_auth_token>
```

#### Remove Channel
```bash
mainflux-cli channels delete <channel_id> <user_auth_token>
```

#### Retrieve a subset list of provisioned Channels
```bash
mainflux-cli channels get all --offset=1 --limit=5 <user_auth_token>
```

#### Retrieve Channel By ID
```bash
mainflux-cli channels get <channel_id> <user_auth_token>
```

### Access control
#### Connect Thing to Channel
```bash
mainflux-cli things connect <thing_id> <channel_id> <user_auth_token>
```

#### Bulk Connect Things to Channels
```bash
mainflux-cli provision connect <file> <user_auth_token>
```

* `file` - A CSV or JSON file containing thing and channel ids (must have extension `.csv` or `.json`)
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
```bash
mainflux-cli things disconnect <thing_id> <channel_id> <user_auth_token>
```

#### Retrieve a subset list of Channels connected to Thing
```bash
mainflux-cli things connections <thing_id> <user_auth_token>
```

#### Retrieve a subset list of Things connected to Channel
```bash
mainflux-cli channels connections <channel_id> <user_auth_token>
```


### Messaging
#### Send a message over HTTP
```bash
mainflux-cli messages send <channel_id> '[{"bn":"Dev1","n":"temp","v":20}, {"n":"hum","v":40}, {"bn":"Dev2", "n":"temp","v":20}, {"n":"hum","v":40}]' <thing_auth_token>
```

#### Read messages over HTTP
```bash
mainflux-cli messages read <channel_id> <thing_auth_token>
```

### Bootstrap

#### Add configuration
```bash
mainflux-cli bootstrap add '{"external_id": "myExtID", "external_key": "myExtKey", "name": "myName", "content": "myContent"}' <user_auth_token>
```

#### View configuration
```bash
mainflux-cli bootstrap view <thing_id> <user_auth_token>
```

#### Update configuration
```bash
mainflux-cli bootstrap update '{"MFThing":"<thing_id>", "name": "newName", "content": "newContent"}' <user_auth_token>
```

#### Remove configuration
```bash
mainflux-cli bootstrap remove <thing_id> <user_auth_token>
```

#### Bootstrap configuration
```bash
mainflux-cli bootstrap bootstrap <external_id> <external_key>
```
