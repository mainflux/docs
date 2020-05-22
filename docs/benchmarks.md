# Test spec
## Tools
- [MZBench](https://github.com/mzbench/mzbench)
- [vmq_mzbench](https://github.com/vernemq/vmq_mzbench)
- [mzb_api_ec2_plugin](https://github.com/mzbench/mzbench/blob/master/doc/cloud_plugins.md#amazon-ec2)

### Setting up MZBench

MZbench is open-source tool for that can generate large traffic and measure performance of the application. MZBench is distributed, cloud aware benchmarking tool that can seamlessly scale to millions of requests. It's originally developped by [satori-com](https://github.com/satori-com/mzbench) but we will use [mzbench](https://github.com/mzbench/mzbench) fork because this one is still alive it can run with newest Erlang releases.

I will describe setting it up on Ubuntu 18.04 (droplet on Digital Ocean)

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

Another way of using MZBench is over [Dashboard](https://github.com/mzbench/mzbench/blob/master/doc/dashboard.md). After starting server you should check dashboard on `http://localhost:4800`. If you run MZBench server on some server, you should change default value for `network_interface` from `127.0.0.1` to `0.0.0.0` in configuration file. Configuration file location must be in `~/.config/mzbench/server.config`, create it from sample configuration file `~/.config/mzbench/server.config.example`.

MZBench can run your test scenarios on many nodes, simultaneously. For now, you are able to run tests locally, so your nodes will be virutal nodes on your server. You can try one of our [MQTT scenarios](https://github.com/mainflux/benchmark/tree/master/mzbench) that uses [vmq_mzbench](https://github.com/vernemq/vmq_mzbench) worker. Copy-paste scenario in MZBench dashboard, click button _Environmental variables_ -> _Add from script_ and add appropriate values. Because it's running localy, you should try with smaller values, for example for fan-in scenario use 100 pulishers on 2 nodes.
Try this before moving forward in setting up Amazon EC2 plugin.

### Setting up Amazon EC2 plugin

For larger scale tests we will settup MZBench to run each node as one of Amazon EC2 instance with built-in plugin [mzb_api_ec2_plugin](https://github.com/mzbench/mzbench/blob/master/doc/cloud_plugins.md#amazon-ec2).

This is basic architecture when runing MZBench:

![MZBench Architecture Running](https://github.com/mzbench/mzbench/raw/master/doc/images/scheme_2.png)

Every node that runs your scenarios will be one of Amazon EC2 instance; plus one more additional node — the director node. The director doesn't run scenarios, it collects the metrics from the other nodes and runs [post and pre hooks](https://github.com/mzbench/mzbench/blob/master/scenarios/spec.md#pre_hook-and-post_hook). So, if you want to run jobs on 10 nodes, acctualy 11 EC2 instances will be created.
All instances will be automaticly terminated when test finishes.

We will use one of ready-to-use Amazon Machine Images (AMI) with all necessary dependencies. We will choose AMI with OTP 22, because that is the version we have on MZBench server. So, we will search for `MZBench-erl22` AMI and find one with id `ami-03a169923be706764` available in `us-west-1b` zone.
If you have choosen this AMI, everything you do from now must be in us-west-1 zone.
We must have IAM user with `AmazonEC2FullAccess` and `IAMFullAccess` permissions policies, and his `access_key_id` and `secret_access_key` goes to configuration file.
In EC2 dashboard, you must create new security group `MZbench_cluster` where you will add inbound roules to open ssh and TCP ports 4801-4804.
Also in EC2 dashboard go to section `key pairs`, create and download key on MZBench server or import public key you already have from MZBench server. Give it a name, put that name (`key_name`) and path (`keyfile`) in configuration file. If you download key from EC2 dashboard, don't forget to `ssh-add` it on MZBench server.


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
With this configuration file, when you start server you can choose to run tests either from local or ec2 plugin. 
