# Ping Central local lab

This guide is to provide you a quick way to deploy minikube and then apply the PingDevops Helm chart to the minikube instance, then attach Ping Central to it.

## Prereq's
homebrew: https://brew.sh/

helm: `brew install helm`

minikube: `brew install minikube`

pingdevops: https://devops.pingidentity.com/get-started/pingDevopsUtil/

Have your PingDevops License, and have it applied via `pingdevops config` 
If you find the instructions for setting up pingdevops a little lacking, I wrote a guide here https://gitlab.corp.pingidentity.com/gso-labs/pingdevops

Depending how much ram your macbook has, It can limit the number of products you deploy at once. You can do things like deploy only 1 pod in a stateful set, or try and lower the ram or cpu requested by these using the values file. Reach out in the #devops-support-swarm slack if you have questions or are stuck with this.

optional - Gasmask setting hostnames on the fly on your macbook. You could just modify the /etc/hosts files.

## How to use the tool?

To use this tool you just need to clone the repo to your macbook where ever you keep your git projects. I keep mine in `~/projects` so the documentation will be based of this.

Clone this repo to your mac.
```bash
cd ~/projects
git clone <ssh-url>
cd ping-devops-minikube
```

Run Make init to set up Helm and Minikube. 
```bash
make init
```

This repo included a reconfigured ping-devops-values.yaml. If you need to make changes in the ping-devops-values.yaml file, if you need to make changes, have a look at the default values file from the helm repo at this link 

https://github.com/pingidentity/helm-charts/blob/master/charts/ping-devops/values.yaml

After this is done you can use make to apply the changes to the minikube cluster

```
make apply
```

Run `make ip` to get the Ip address of the Kubernetes cluster, then open gasmask add the following in a new file

```bash
127.0.0.1		localhost
255.255.255.255	broadcasthost
::1				localhost
fe80::1%lo0		localhost
{IP from make ip}		  pingfederate-admin.pingcentral.example.com
{IP from make ip}			pingfederate-engine.pingcentral.example.com
{IP from make ip}			pingdirectory.pingcentral.example.com
{IP from make ip}			pingaccess-engine.pingcentral.example.com
{IP from make ip}			pingaccess-admin.pingcentral.example.com
```

When the apply is completed you should be able to run `kubectl` or use `k9s` to explore the cluster and ensure the products pods are deployed and starting.

## How do I deploy Ping Central?

To deploy PingCentral you will need to run it within docker-compose, as there are no helm charts or example kubernetes deployment for the product yet. To get Ping Central to commutate with the kubernetes deployments of the ping products, we need to get the external IP address of the cluster.

Start a terminal just for running Ping Central

Then run:

```bash
make ip
```

Open the docker-compose.yaml file included in this project. Look for `extra_hosts` in the file

```yaml
    extra_hosts:
      - "pingfederate-admin.pingcentral.example.com:192.168.64.20"
      - "pingfederate-engine.pingcentral.example.com:192.168.64.20"
      - "pingdirectory.pingcentral.example.com:192.168.64.20"
      - "pingaccess-engine.pingcentral.example.com:192.168.64.20"
      - "pingaccess-admin.pingcentral.example.com:192.168.64.20"
```

Replace the IP address on each line with the output from `make ip`

```bash
docker-compose up
```

Wait for docker-compose to stand up ping central and the required SQL server. Once these are up you can navigate to https://localhost:9022/pass/login 

From the Ping Central console login with the default password for first boot.

Once logged in you can set up the PingFederate Admin console with the URL `pingfederate-admin.pingcentral.example.com` However you will need to change the portnumber from `9999` to `443` This is due to the ingress controller provided by Kubernetes.

You can continue down the page and add the Ping Access Admin Console with the URL `pingaccess-admin.pingcentral.example.com` Once again because of the ingress controller, we will have the change the port to `443` from `9999`.

Save the changes and PingCentral should be able configured enough for any Labs.


## What if I need to make changes to the values file and redeploy?

All you need to do is make the required changes to the `ping-devops-values.yaml` file, then run `make redeploy` and it will delete the old helm release and redeploy a new release.

## How to Cleanup?

To stop PingCentral, just hit `ctrl+c` on the terminal that is running the `docker-compose` command, that will terminate the containers.

To cleanup the kubernetes cluster, just run `make cleanup` and it will delete the helm chart from the cluster, and then delete the minikube cluster. This way your mac is not running the cluster 24/7 and you can save your ram for 20 more tabs in Chrome.
