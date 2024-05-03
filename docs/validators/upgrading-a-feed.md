---
sidebar_position: 3
---

# Upgrading a Validator

:::warning
Please be aware that the latest helm chart has been renamed from `feed` to `validator`. Please use the `upgrade.sh` script to upgrade your validator to the latest version. This version embeds `musig` into the `ghost` pod. The upgrader script will clean up the generated `values.yaml` file and remove the unecessary musig values.
:::

![Dynamic YAML Badge](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fchronicleprotocol.github.io%2Fcharts%2Findex.yaml&query=%24.entries.validator%5B0%5D.version&label=Validator%20ChartVersion&color=green)
<div class="artifacthub-widget" data-url="https://artifacthub.io/packages/helm/validator/validator" data-theme="light" data-header="true" data-stars="true" data-responsive="true"><blockquote><p lang="en" dir="ltr"><b>validator</b>: A Helm chart for deploying Chronicle Validator on Kubernetes</p>&mdash; Open in <a href="https://artifacthub.io/packages/helm/validator/validator">Artifact Hub</a></blockquote></div><script async src="https://artifacthub.io/artifacthub-widget.js"></script>

<br/>

In order to upgrade a validator to the latest version, we will need to run a couple helm commands.

## Upgrade Helper Script

To simplify the upgrade process, we have created a helper script that will upgrade your validator to the latest version. 

This script will attempt to run `helm upgrade <feedname> -n <feedname> chronicle/validator` on your feed release, with any updated input variables.

:::caution
Please use the correct `FEED_NAME`, which should be the same as your helm release name, if deployed using the `install.sh` script previously
:::


```
ssh <SERVER_IP>
su - <FEED_USERNAME>
export FEED_NAME=my-feed
wget -N https://raw.githubusercontent.com/chronicleprotocol/scripts/main/feeds/k3s-install/upgrade.sh
chmod a+x upgrade.sh
./upgrade.sh
```

:::tip You can set the expected variables in the `.env` file, or export them as environment variables. If the script fails to find any of these values, it will prompt you for them when running the script.
:::

---

## Manual process TL;DR

```
ssh <SERVER_IP>
su - <FEED_USERNAME>
export FEED_NAME=my-feed
helm repo update
helm upgrade $FEED_NAME -n $FEED_NAME -f $HOME/$FEED_NAME/generated-values.yaml chronicle/validator
```

:::caution
Please use the correct `FEED_USERNAME`, and `FEED_NAME` according to your installation
:::

### Helm Upgrade Process

First, ssh on to your server:

```
ssh <SERVER_IP>
```

Once you're on the host, make sure your you switch to the \`user\` that the feeds were installed with

Eg; assuming your user name is `feed-user` :

```
$ su - feed-user
feed-user@my_feed_host:~$ 
```


Verify that your user can view the k3s cluster resources (eg below assuming the feed is named is `my-feed`:

```
export FEED_NAME=my-feed
```

```
helm list -n $FEED_NAME
NAME     NAMESPACE    	REVISION	UPDATED                                	STATUS  	CHART     	APP VERSION
my-feed	 my-feed	10      	2023-10-12 08:22:16.725518367 +0000 UTC	deployed	feed-0.2.9	2.0.0  
```

If `kubectl/helm` commands fail, please ensure you have `$KUBECONFIG` set correctly. Take a look [here](quickstart#kubectl--helm-commands-fail) for more detail

### Update helm repo

We need to fetch the latest helm chart from the registry:

```
helm repo update
```

### Run  helm upgrade

Now we can update to the latest version.

The latest chart version is:

![Dynamic YAML Badge](https://img.shields.io/badge/dynamic/yaml?url=https%3A%2F%2Fchronicleprotocol.github.io%2Fcharts%2Findex.yaml&query=%24.entries.validator%5B0%5D.version&label=Latest%20Chart&color=green)

using this version, we can upgrade our validator:

:::danger
Please ensure you pin the helm release to the lastest semver ChartVersion of the feed chart. eg `0.3.1`
The charts released are production ready, and tested thoroughly
:::

```
helm upgrade $FEED_NAME -n $FEED_NAME -f $FEED_NAME/generated-values.yaml chronicle/validator --version 0.3.1
```

You should see output like this:

```
Release "my-feed" has been upgraded. Happy Helming!
NAME: my-feed
LAST DEPLOYED: Thu Oct 12 08:21:03 2023
NAMESPACE: my-feed
STATUS: deployed
REVISION: 9
NOTES:
1. Get the application URL by running these commands:
  kubectl get pods -n my-feed
```

#### Verify the helm release and version

Verify the chart version has changed and matches what the latest feed version:

```
helm list -n $FEED_NAME
NAME     	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART          	APP VERSION
validator	validator	3       	2024-04-30 18:49:58.843309576 +0000 UTC	deployed	validator-0.3.1	0.36.0   
```

#### Verify the new pods are running:

```
kubectl get pods -n $FEED_NAME
```

#### Double check the pod logs:

```
kubectl logs -n $FEED_NAME deployments/ghost
```

```
kubectl logs -n $FEED_NAME deployments/tor-proxy
```
and you're done!

:::warning
If you encounter any issues please refer to the [Trouble Shooting](https://docs.chroniclelabs.org/validators/quickstart#trouble-shooting) steps
:::