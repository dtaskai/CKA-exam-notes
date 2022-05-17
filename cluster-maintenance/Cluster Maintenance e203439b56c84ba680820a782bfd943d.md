# Cluster Maintenance

### OS Upgrades

- If a Node goes down for more than 5 minutes then the Pods on it are considered dead and are terminated, if the Pods were a part of a ReplicaSet then they are recreated on another Node
    - The timeout is called the **pod-eviction-timeout** and it can be set on the kube-controller-manager
- You can drain the Nodes of their workloads with the `kubectl drain $NODE_NAME` command which gracefully terminates the Pods on that Node and recreates them on another Node
- Drained Nodes are also cordoned which means that workloads won’t be scheduled on it unless you uncordon it with the `kubectl uncordon $NODE_NAME` command
    - Of course you can cordon Nodes with the `kubectl cordon $NODE_NAME` command

Links:

- [https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

### Cluster Upgrade Process

- Since the API Server is in the “center of attention”, none of the components can be of a higher version than it. The Controller-manager and the kube-scheduler can be one version lower than the API Server and the kubelet and the kube-proxy can be two versions lower.
- Kubectl can be one version higher of lower than the API Server
- The recommended way is to upgrade one minor version at a time
- Upgrading a Cluster has 2 steps, upgrading the Master Nodes and then the Worker Nodes
    - While the Master Node is down, the API Server and the Scheduler are not available, but the workloads continue to run on the Worker Nodes
- kubeadm provides a tool to upgrade Nodes, use the `kubeadm upgrade plan` to see a rundown of versions and the available versions
    - Use the `kubeadm upgrade apply $VERSION` command to run an upgrade, but first you have to update the kubeadm tool itself
    - After upgrading a Node you have to update the kubelet yourself

Links:

- [https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

### Backup and Restore

- You can backup all the resource configurations with the `kubectl get all -A -o yaml > all-objects.yaml` command
- You can backup ETCD itself with the ETCDCTL utility
    - To be able to use the snapshot functionality, you have to use version 3 of ETCTCTL, to do this run the `export ETCDCTL_API=3` command
    - To take a snapshot run `etcdctl snapshot save $SNAPSHOT_NAME` command
    - You can view the status of the snapshot with the `etcdctl snapshot status $SNAPSHOT_NAME` command
    - To restore from a snapshot run `etcdctl snapshot restore $SNAPSHOT_NAME --data-dir $DATA_LOCATION` command, the data directory will be created and you’ll have to point ETCD to that directory by editing the ETCD configuration and setting its **—data-dir** property to that directory
    - When taking a snapshot you have to specify the `--endpoints`, `--cacert`, `--cert`, `--key` for verification purposes

Links:

- [https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)