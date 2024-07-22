# Hedera Kubernetes Deployment


## Introduction

The Hedera Kubernetes Deployment project allows developers to set up their own local network in a Kubernetes Cluster with 3 worker nodes, each hedera node will be deployed on a worker node. The single local node network is composed of one mirror node and one consensus node. The multi local node network is composed of one mirror node and three consensus nodes.

## Requirements

## Setup (single node)

1. Created Statefulsets, Services, Persistent Volumes, Configmaps and Secrets. 

### Start the network

1. To run a single node make the following changes:

- In file `/hedera-local-node/01_network_node_statefulset.yml` under `spec:template:spec:nodeName` mention the name of the node worker-1 (worker node on which you want to run the pod). 

- Make sure that the following pods run on same worker node: `network-node-0`, `haveged`, `record-streams-uploader`, `account-balances-uploader` and `record-sidecar-uploader`. This can be taken care by mentioning the name of the worker node under `spec:template:spec:nodeName` in files `/hedera-local-node/01_network_node_statefulset.yml`, `/hedera-local-node/02_haveged_statefulset.yml`, `/hedera-local-node/03_record_streams_uploader_statefulset.yml`, `/hedera-local-node/05_account_balances_uploader_statefulset.yml` and `/hedera-local-node/06_record_sidecar_uploader_statefulset.yml`. 

- In configmap `network-node-config-txt-volume` (line 32 in `/hedera-local-node/01_network_node_config.yml`) in update the IP address with the IP address of the worker-1 of the kubernetes cluster (on which `network-node-0` is supposed to run).

2. Init containers in the statefulsets takes care of the sequence in which the pods should get initialized and run. Create a directory `/hedera-local-node/volumes` on all the worker nodes and create volumes (`grafana-data`, `minio-data`, `mirror-node-postgres`, `prometheus-data`) inside the directory on the worker nodes. 

3. Create directory `/hedera-local-node/compose-network/mirror-node` on all the worker nodes of the kubernetes cluster and copy `addressBook.bin`. This file can be found at `/hedera-local-node/compose-network/mirror-node/addressBook.bin`.

4. Run the following commands in the terminal:
``` sh
cd /hedera-local-node
kubectl apply -f .
kubectl get pods
``` 
5. Run the following commands in the terminal to stop Hedera multi node network:
``` sh
kubectl delete statefulsets --all
kubectl delete pvc --all
kubectl delete pv --all
kubectl delete svc --all
kubectl delete configmaps --all
```

## Setup (Multinode deployment)

1. Hedera multinode kubernetes setup has 1 mirror node and 3 consensus nodes. 

2. The readiness of 3 nodes is in the following sequence: node-0, node-1, node-2.

3. Each of the network nodes should use the host network, this done by setting `hostNetwork: true` under `spec.template.spec`. This has been taken care for all the 3 network-nodes.

4. Originally the manifest files are coded to support single node network. To run a multi node network make the following changes:

- In file `/hedera-local-node/01_network_node_statefulset.yml` under `spec:template:spec:nodeName` mention the name of the worker-1 (node on which you want to run the pod). Follow the same for `network-node-1` (/hedera-local-node/network_node_1/01_multi_network_node_1_statefulset.yml) and `network-node-2` (/hedera-local-node/network_node_2/01_multi_network_node_2_statefulset.yml) by pinning them worker-2 and worker-3 respectively. 

- Network node-0 statefulset `/hedera-local-node/01_network_node_statefulset.yml` should have the health check (`spec:template:spec:containers:readinessProbe:exec:command`, line 51) set to `"Now current platform statue = ACTIVE|OBSERVING|CHECKING"` instead of `"Now current platform status = ACTIVE|Hedera - Hederanode#0 is ACTIVE"`.

- Make sure that the following pods run on second worker node: `network-node-1`, `haveged-1`, `record-streams-uploader-1`, `account-balances-uploader-1` and `record-sidecar-uploader-1`. Lkewise pods `network-node-2`, `haveged-2`, `record-streams-uploader-2`, `account-balances-uploader-2` and `record-sidecar-uploader-2` should run on third worker node. This can be taken care by mentioning the name of the worker node under `spec:template:spec:nodeName` in files (`network_node_1/01_multi_network_node_1_statefulset.yml`, `network_node_1/02_multi_haveged_2_statefulset.yml`, `network_node_1/03_multi_record_streams_uploader_1_statefulset.yml`, `network_node_1/05_multi_account_balances_uploader_1_statefulset.yml`, `network_node_1/06_multi_record_sidecar_uploader_1_statefulset.yml`, `network_node_2/01_multi_network_node_2_statefulset.yml`, `network_node_2/02_multi_haveged_2_statefulset.yml`, `network_node_2/03_multi_record_streams_uploader_2_statefulset.yml`, `network_node_2/05_multi_account_balances_uploader_2_statefulset.yml`, `network_node_2/06_multi_record_sidecar_uploader_2_statefulset.yml`). 

- In file `/hedera-local-node/01_network_node_statefulset.yml`, point configmap `config-txt-volume` to use configmap `network-node-config-txt-multi-volume`. To do that, replace `network-node-config-txt-volume` with `network-node-config-txt-multi-volume` on line 130 in `01_network_node_statefulset.yml`. Do the same for `network_node_1/01_multi_network_node_1_statefulset.yml` and `network_node_2/01_multi_network_node_2_statefulset.yml`.

- In configmap `network-node-config-txt-multi-volume` (in `/hedera-local-node/multi_network_node/01_multi_network_node_config.yml`) update the IP address with the IP address of the worker-1, worker-2, worker-3 of the kubernetes cluster (on which network-node-0, network-node-1 and network-node-2 are supposed to run).

- In file `/hedera-local-node/09_importer_statefulset.yml`, point volumeClaimTemplate `addressbook-volume` to use Persistent Volume `addressbook-volume-multi-pv`. To do that, replace `addressbook-volume-pv` with `addressbook-volume-multi-pv` on line 55 in `09_importer_statefulset.yml`. 

5. Init containers in the statefulsets takes care of the sequence in which the pods should get initialized and run. Create a directory `/hedera-local-node/volumes` on all the worker nodes and create volumes (`grafana-data`, `minio-data`, `mirror-node-postgres`, `prometheus-data`) inside the directory on the worker nodes. . 

6. Create directory `/hedera-local-node/compose-network/mirror-node` and copy `modified_addressBook.multinode.bin` on all the worker nodes. This file can be found at `/hedera-local-node/compose-network/mirror-node/modified_addressBook.multinode.bin`

7. Run the following commands in the terminal to setup Hedera multi node network:
``` sh
cd /hedera-local-node
kubectl apply -f multi_network_node/.
kubectl apply -f . & kubectl apply -f network_node_1/. & kubectl apply -f network_node_2/. 
kubectl get pods
``` 
8. Run the following commands in the terminal to stop Hedera multi node network:
``` sh
kubectl delete statefulsets --all
kubectl delete pvc --all
kubectl delete pv --all
kubectl delete svc --all
kubectl delete configmaps --all
```