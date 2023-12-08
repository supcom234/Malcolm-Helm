# Malcolm Helm Chart

The purpose of this project is to make installing Malcolm with all of its server and sensor components easily across a 
large Kubernetes cluster. Case in point, lets say you have 8 servers and 50 sensors. Some sensors will be high bandwidth 
while others may be low bandwith. This helm chart for example can either deploy a opensearch setup or point to a 
preconfigured external elasticsearch.  Furthermore, it will deploy live sensors for suricata and zeek as well as offline 
deployment for uploading pcaps offline.

## Features

- Opensearch or External Elastic
- Netbox
- PCAP capture
- Arkime PCAP Capture (Not implemented yet)
- Suricata Live
- Zeek Live
- Offline PCAP Processing
- Zeek file extraction 

## Cluster Requirements

The following requirements are assumed to be met prior to running the Installation procedures.  
Other Kuberenetes clusters may work but they have not been tested

- Kubernetes RKE2 installation v1.24.10+rke2r1
- Storage class https://github.com/rancher/local-path-provisioner
- Istio service mesh https://istio.io/latest/docs/setup/getting-started/
- TOOD add better instructions for setting up istio service mesh
- TODO add support for basic ingress and TLS
- TODO support longhorn storage class for multi node setup

When installing the localprovisioner update teh config.json run `kubectl edit configmap local-path-config -n local-path-storage`.
Change the config.json file to look like the following.

config.json: |-
  {
    "sharedFileSystemPath": "/opt/local-path-provisioner"
  }


## Vagrant Quickstart

It is required that you have 500GB of free space and to have at least 8 GB of free RAM as well as 8 extra CPU cores to run this quickstart.

1. Install virtual box version 7.0.10 r158379 or greater from https://www.virtualbox.org/wiki/Downloads
2. Install vagrant version 2.4.0 or greater from https://developer.hashicorp.com/vagrant/downloads
3. Install vagrant disk size plugin verion 0.1.3 with `vagrant plugin install vagrant-disksize`
4. Install vagrant reload version 0.0.1 with `vagrant plugin install vagrant-reload`
5. Run `cd <root of the project where the Vagrantfile is located>`
6. Run `vagrant up`
7. Wait until everything installs at the end of the install you should see the IP show up saying you can ssh in.
8. run `ssh vagrant@<ip outputed from installation>` and login using vagrant as the password.

## External Elasticsearch notes

Elasticsearch requires TLS termination in order for it to support Single Sign On (SSO) functionality.  The values file was updated to give 
the user of this helm chart the ability to copy the certificate file from a different namespace into Malcolm namespace for usage.

Furthermore, dashboards_url (IE kibana) is still expected to remain unencrypted when using Istio service mesh. 

## Installation procedures

Check the chart/values.yaml file for all the features that can be enabled disabled and tweaked prior to running the below installation commands.

1. `git clone <repo url>`
2. `cd <project dir that contains chart foler>`
3. `helm install malcolm chart/ -n malcolm`

## Accessing services with istio service mesh

1. Copy the EXERNAL-IP of the gateway using the `kubectl get svc -n istio-system` command.
2. Grab the hostname under the HOSTS column using `kubectl get virtualservice -n malcolm`
3. Update /etc/hosts file with `sudo vim /etc/hosts`(EX: append "10.1.25.70 malcolm.vp.bigbang.dev" )

Malcolm services can be accessed via the following URLs:
-----------------------------------------------------------
  - Arkime: https://yourhostname/
  - OpenSearch Dashboards: https://yourhostname/dashboards/
  - PCAP upload (web): https://yourhostname/upload/
  - PCAP upload (sftp): sftp://username@yourhostname:8022/files/
  - NetBox: https://yourhostname/netbox/
  - Extracted files: https://yourhostname/dl-extracted-files/  
  - Account management: https://yourhostname/auth/
  - Documentation: https://yourhostname/readme/
