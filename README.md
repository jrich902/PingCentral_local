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

After the make init is competed, it will have downloaded the latest version of the values.yaml from the PingDevops Helm Repo. It will save the file as `ping-devops-values.yaml` You are now able to open this file with your editor of choice and make the changes required. Please note that the default values file does NOT deploy any of the products, You will have to modify this to have it deploy the products you require.

In the ping-devops-values.yaml file, look for lines of code _like_ the following for each one of our products you would like to deploy.
```yaml
#############################################################
# pingfederate-admin values
#############################################################
pingfederate-admin:
  enabled: false
  name: pingfederate-admin
  image:
    name: pingfederate
```
change the value of `enabled` to `true` on each of the products you need to deploy with the helm chart. Then save the file. 

After this is done you can use make to apply the changes to the minikube cluster
```
make apply
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
