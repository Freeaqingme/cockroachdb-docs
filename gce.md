---
title: Google Cloud Platform
summary: Learn how to deploy CockroachDB on Google Cloud Platform's Compute Engine.
toc: false
---

This page shows you how to manually deploy a multi-node CockroachDB cluster on Google Cloud Platform's Compute Engine.

{{site.data.alerts.callout_info}} For testing and development, you can <a href="start-a-local-cluster.html">Start a Local Cluster</a> or <a href="cloud-deployment.html">Deploy on GCE or AWS using Terraform</a>. You can also <a href="http://uptimedba.github.io/cockroach-vb-single/cockroach-vb-single/home.html">Run CockroachDB inside a VirtualBox VM</a> (community-supported).{{site.data.alerts.end}}

<div id="toc"></div>

## Requirements

This process assumes the following:

- You have SSH access to each machine. This is necessary for distributing binaries and, in the case of a secure cluster, certificates. 
- Your network configuration allows the machines to talk to each other and clients to talk to the machines.

## Recommendations

For guidance on cluster topology, clock synchronization, and file descriptor limits, see [Recommended Production Settings](recommended-production-settings.html).

Determine how you want to expose the admin UI and handle it through `--http-addr`:
- `localhost` requires SSH tunnel
- Firewall rules on port `8080` letting in your IP address

## Configure Your Network

Your nodes must allow TCP communication on port 26257 (`tcp:26257`). For help setting this up, see [Google Cloud Platform: Using Networks and Firewalls](https://cloud.google.com/compute/docs/networking).

## Create Instances

[Create as many instances](https://cloud.google.com/compute/docs/instances/create-start-instance) as you want to have nodes in your cluster. You'll need to have the IP addresses of each instance to use as the hostname for your certificates.

You'll also need to create a secure `certs` directory on each instance to store your certificate files:

~~~shell
ssh <em><u>username</u></em>@<em><u>instance IP address</u></em>
mkdir certs
~~~

## Generate Certificates

Create self-signed certificates to let your nodes communicate securely.

#### CockroachDB

1. Create the CA key pair:
	~~~
	cockroach cert create-ca --ca-cert=certs/ca.cert --ca-key=certs/ca.key
	~~~

2. Create key pairs for each node and then move them to the instance:
	~~~
	cockroach cert create-node <em><u>instance IP address</u></em> --ca-cert=certs/ca.cert --ca-key=certs/ca.key --cert=certs/node<em>N</em>.cert --key=certs/node<em>N</em>.key
	scp certs/ca.cert __username__@__IP address__:certs/
	scp certs/node<em>N</em>.cert __username__@__IP address__:certs/
	scp certs/node<em>N</em>.key __username__@__IP address__:certs/
	~~~

	~~~
	cockroach cert create-node 104.196.44.35 127.0.0.1 10.142.0.2 --ca-cert=certs/ca.cert --ca-key=certs/ca.key --cert=certs/node1.cert --key=certs/node1.key
	scp certs/ca.cert sloiselle@104.196.44.35:certs/
	scp certs/node1.cert sloiselle@104.196.44.35:certs/
	scp certs/node1.key sloiselle@104.196.44.35:certs/
	~~~



 




cockroach start --ca-cert=certs/ca.cert --cert=certs/node1.cert --key=certs/node1.key --logtostderr --host=10.142.0.2


 node1 104.196.44.35
 10.142.0.2
 
 node2 130.211.183.47

 instance-1 104.196.174.202



#### OpenSSL


## Connect to Instance

1. SSH to your instance:
	~~~
	ssh __username__@__IP address__
	~~~

2. Install CockroachDB:
	~~~
	wget https://binaries.cockroachdb.com/cockroach-{{site.data.strings.version}}.linux-amd64.tgz
	tar -xf cockroach-{{site.data.strings.version}}.linux-amd64.tgz --strip=1 cockroach-{{site.data.strings.version}}.linux-amd64/cockroach
	sudo mv cockroach /usr/bin
	~~~

3. Start the node on the machine. For your first machine, do not include the `--join` flag, but do for your subsequent nodes.
	- First node:
	~~~
	cockroach cert create-node 104.196.44.35 --ca-cert=certs/ca.cert --ca-key=certs/ca.key --cert=certs/node1.cert --key=certs/node1.key
	~~~
	- Additional nodes:
	~~~
	cockroach cert create-node 104.196.44.35 --ca-cert=certs/ca.cert --ca-key=certs/ca.key --cert=certs/node1.cert --key=certs/node1.key --join=104.196.44.35:26257
	~~~

cockroach start --store=node1 --ca-cert=certs/ca.cert --cert=certs/node1.cert --key=certs/node1.key --background
cockroach start --store=node1 --join=localhost:26257 --ca-cert=certs/ca.cert --cert=certs/node.cert --key=certs/node.key --background



cockroach cert create-node 104.196.44.35 --ca-cert=certs/ca.cert --ca-key=certs/ca.key --cert=certs/node1.cert --key=certs/node1.key
cockroach cert create-node localhost $(hostname) --ca-cert=certs/ca.cert --cert=certs/node.cert --key=certs/node.key







cockroach cert create-node localhost $(hostname) --ca-cert=certs/ca.cert --ca-key=certs/ca.key --cert=certs/node.cert --key=certs/node.key







## Deploy an Secure Cluster

1. Set up firewall rules allowing communication on port tcp:26258.

2. Create Compute Engine instances (one instance per node). We recommend:
	- Allowing HTTPS traffic
	- Including your SSH key on the server


## Install CockroachDB








26258














To deploy an insecure development or test cluster on Google Cloud Engine or AWS using [Terraform](https://www.terraform.io/), see the configuration files and instructions in the [`gce`](https://github.com/cockroachdb/cockroach/blob/master/cloud/gce) and [`aws`](https://github.com/cockroachdb/cockroach/blob/master/cloud/aws) directories of our main cockroach GitHub repository. 

More cloud deployment docs coming soon.

## See Also

- [Manual Deployment](manual-deployment.html)
- [Start a Local Cluster](start-a-local-cluster.html)
