## Pre-reqs

Install skupper curl https://skupper.io/install.sh | sh

Spin up two RHDP instances of Service Interconnect - RHEL 9

Spin up one OCP 4.13 cluster

Login as cluster-admin

Login via OC CLI as cluster-admin

SSH into RHEL instances

SSH Host: ssh lab-user@bastion.xxxx.sandboxxxxx.opentlc

vi .ssh/authorized_keys

Copy public key over

add hostnames to inventory/hosts

export DEV=bastion.xxx.sandboxxxx.opentlc.com

export PROD=bastion.dj64p.sandbox1577.opentlc.com

Install requirements

ansible-galaxy collection install -r ./collections/requirements.yml

Install postgreSQL on both RHEL instances

ansible-playbook -i ./inventory/hosts postgresql.yml
 
## Quay stuff

Create repositories in phorg, public repos

e.g. fedexapp, make sure it's public


## Radius stuff

oc new-project radius-system

oc adm policy add-scc-to-user anyuid -z contour-contour-certgen -n radius-system

oc adm policy add-scc-to-user anyuid -z contour-envoy -n radius-system

oc adm policy add-scc-to-user anyuid -z contour-contour -n radius-system

oc adm policy add-scc-to-user privileged -z contour-envoy -n radius-system

rad init 

Don't install application

rad group create dev

rad env create development --group dev --namespace dev

rad group create prod

rad env create production --group prod --namespace prod

vi ~/.rad/config.yaml 

change scope to /planes/radius/local/resourceGroups/dev

rad recipe register default   --environment development --group dev --template-kind bicep  --template-path "ghcr.io/radius-project/recipes/local-dev/rediscaches:0.26"  --resource-type "Applications.Datastores/redisCaches"

## add Azure to production environment

az login


export AZ_SUB_ID=azure-subidxxxxx

vi ~/.rad/config.yaml change scope to /planes/radius/local/resourceGroups/prod

rad env update production --azure-subscription-id $AZ_SUB_ID --azure-resource-group  radius

Create a service principle 

az ad sp create-for-rbac

This command will output details of the service principal as follows.

{
  "appId": "clientxxxxxxxxx",
  "displayName": "azure-cli-xxx",
  "password": "passxxxxx",
  "tenant": "tenantxxxxxx"
}

Go to https://portal.azure.com/#@phayesredhat.onmicrosoft.com/resource/subscriptions/e228d488-653c-4479-ac39-77c6c1e96281/users

Add contributor role to sp

Use the results from this command to register azure credentials

rad credential register azure --client-id clientxxxxxxxxx  --client-secret passxxxxx  --tenant-id tenantxxxxxx

rad credential register azure --client-id xxx  --client-secret xxx  --tenant-id xxx

rad recipe register default  --environment production   --group prod --template-kind bicep  --template-path "radius.azurecr.io/recipes/azure/rediscaches"  --resource-type "Applications.Datastores/redisCaches"

// rad recipe show redis-dev --resource-type Applications.Datastores/redisCaches --group dev --environment myEnvironment

# Demo Steps

## pre-steps
Login to quay with podman

https://quay.io/user/hayesphilip/?tab=settings

az login

Start backstage

oc new-project prod-radius


# DEMO 

oc new-project dev-radius

## Create skupper networks

Do this for both dev-radius and prod-radius namespaces

 oc project dev-radius 

 skupper init --enable-console  --enable-flow-collector --console-auth openshift

 skupper service create database 5432

 skupper gateway generate-bundle ./skupper/gateway.yml ./package

scp ./package/database.tar.gz lab-user@$DEV:/home/lab-user

ssh lab-user@$DEV
### on vm

mkdir gateway

tar -xvf database.tar.gz --directory gateway

cd gateway
sudo su
sh ./launch.sh -t podman

Show skupper console

## Create App from backstage

Create app called corner

image owner phorg

app name corner

github owner deewhyweb

Show github repo

Git clone repo

follow build instructions.

Push image to repository

Show image in backstage

## Reset

Delete redis from azure

oc delete project dev-radius

oc delete project prod-radius

Delete github repository deewhyweb/corner

Delete quay.io images phorg/corner

ssh lab-user@$DEV

sudo su

podman ps

podman kill

podman container prune

ssh lab-user@$PROD

podman ps

podman kill

podman container prune

 


