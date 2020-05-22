# Test spec

## Tools
- [MZBench](https://github.com/mzbench/mzbench)
- [vmq_mzbench](https://github.com/vernemq/vmq_mzbench)
- [mzb_api_ec2_plugin](https://github.com/mzbench/mzbench/blob/master/doc/cloud_plugins.md#amazon-ec2)

### Setting up MZBench

MZbench is open-source tool for that can generate large traffic and measure performance of the application. MZBench is distributed, cloud-aware benchmarking tool that can seamlessly scale to millions of requests. It's originally developed by [satori-com](https://github.com/satori-com/mzbench) but we will use [mzbench](https://github.com/mzbench/mzbench) fork because it can run with newest Erlang releases and the original MzBench repository is not maintained anymore.

We will describe installing MZBench server on Ubuntu 18.04 (this can be on your PC or some external cloud server, like droplet on Digital Ocean)

Install latest OTP/Erlang (it's version 22.3 for me)
```
sudo apt update
sudo apt install erlang
```

For running this tool you will also need libz-dev package:
```
sudo apt-get update
sudo apt-get install libz-dev
```

and pip:
```
sudo apt install python-pip
```

Clone mzbench tool and install the requirements:
```
git clone https://github.com/mzbench/mzbench
cd mzbench
sudo pip install -r requirements.txt
```

This should be enough for installing MZBench, and you can now start MZBench server with this CLI command:
```
./bin/mzbench start_server
```

The [MZBench CLI](https://github.com/mzbench/mzbench/blob/master/doc/cli.md) lets you control the server and benchmarks from the command line.

Another way of using MZBench is over [Dashboard](https://github.com/mzbench/mzbench/blob/master/doc/dashboard.md). After starting server you should check dashboard on `http://localhost:4800`. 

Note that if you are installing MZBench on external server (i.e. Digital Ocean droplet), that you'll be able to reach MZBench dashboard on your server's IP address:4800, if you previously:
- change default value for `network_interface` from `127.0.0.1` to `0.0.0.0` in configuration file. Default configuration file location is `~/.config/mzbench/server.config`, create it from sample configuration file `~/.config/mzbench/server.config.example`
- open port `4800` with `ufw allow 4800`

MZBench can run your test scenarios on many nodes, simultaneously. For now, you are able to run tests locally, so your nodes will be virtual nodes on machine where MZBench server is installed (your PC or DO droplet). You can try one of our [MQTT scenarios](https://github.com/mainflux/benchmark/tree/master/mzbench) that uses [vmq_mzbench](https://github.com/vernemq/vmq_mzbench) worker. Copy-paste scenario in MZBench dashboard, click button _Environmental variables_ -> _Add from script_ and add appropriate values. Because it's running locally, you should try with smaller values, for example for fan-in scenario use 100 publishers on 2 nodes.
Try this before moving forward in setting up Amazon EC2 plugin.

### Setting up Amazon EC2 plugin

For larger-scale tests we will set up MZBench to run each node as one of Amazon EC2 instance with built-in plugin [mzb_api_ec2_plugin](https://github.com/mzbench/mzbench/blob/master/doc/cloud_plugins.md#amazon-ec2).

This is basic architecture when running MZBench:

![MZBench Architecture Running](https://github.com/mzbench/mzbench/raw/master/doc/images/scheme_2.png)

Every node that runs your scenarios will be one of Amazon EC2 instance; plus one more additional node — the director node. The director doesn't run scenarios, it collects the metrics from the other nodes and runs [post and pre hooks](https://github.com/mzbench/mzbench/blob/master/scenarios/spec.md#pre_hook-and-post_hook). So, if you want to run jobs on 10 nodes, actually 11 EC2 instances will be created.
All instances will be automatically terminated when the test finishes.

We will use one of ready-to-use Amazon Machine Images (AMI) with all necessary dependencies. We will choose AMI with OTP 22, because that is the version we have on MZBench server. So, we will search for `MZBench-erl22` AMI and find one with id `ami-03a169923be706764` available in `us-west-1b` zone.
If you have chosen this AMI, everything you do from now must be in us-west-1 zone.
We must have IAM user with `AmazonEC2FullAccess` and `IAMFullAccess` permissions policies, and his `access_key_id` and `secret_access_key` goes to configuration file.
In EC2 dashboard, you must create new security group `MZbench_cluster` where you will add inbound rules to open ssh and TCP ports 4801-4804.
Also, in EC2 dashboard go to section `key pairs`, click `Actions` -> `Import key pair` and upload public key you have on your MZBench server in `~/.ssh/id_rsa.pub` (if you need to create new, run `ssh-keygen` and follow instructions). Give it a name on EC2 dashboard, put that name (`key_name`) and path (`keyfile`) in configuration file.


```
[
{mzbench_api, [
{network_interface,"0.0.0.0"},
{keyfile, "~/.ssh/id_rsa"},
{cloud_plugins, [
                  {local,#{module => mzb_dummycloud_plugin}},
                  {ec2, #{module => mzb_api_ec2_plugin,
                        instance_spec => [
                          {image_id, "ami-03a169923be706764"},
                          {group_set, ["MZbench_cluster"]},
                          {instance_type, "t2.micro"},
                          {availability_zone, "us-west-1b"},
                          {iam_instance_profile_name, "mzbench"},
                          {key_name, "key_pair_name"}
                        ],
                        config => [
                          {ec2_host, "ec2.us-west-1.amazonaws.com"},
                          {access_key_id, "IAM_USER_ACCESS_KEY_ID"},
                          {secret_access_key, "IAM_USER_SECRET_ACCESS_KEY"}
                        ],
                        instance_user => "ec2-user"
                  }}              
              ]
}
]}].
```
There is both `local` and `ec2` plugin in this configuration file, so you can choose to run tests on either of them.
Default path for configuration file is `~/.config/mzbench/server.config`, if it's somewhere else, server is starting with:
`$ ./bin/mzbench start_server --config <config_file>`
Note that every time you update the configuration you have to restart the server:
`$ ./bin/mzbench restart_server`

## Test scenarios

Testing environment to be determined.

### Message publishing

In this scenario, large number of requests are sent to HTTP adapter service
every second. This test checks how much time HTTP adapter needs to respond
to each request.

#### Results

TBD

### Create and get client

In this scenario, large number of requests are sent to things service to create
things and than to retrieve their data. This test checks how much time things
service needs to respond to each request.

#### Results

TBD
