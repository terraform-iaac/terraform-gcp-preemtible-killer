##GKE Preemtible Killer
Source: https://github.com/estafette/estafette-gke-preemptible-killer.git

Source version: 1.2.5
### Introduction
When creating a cluster, all the node are created at the same time and should be deleted after 24h of activity. To prevent large disruption, the estafette-gke-preemptible-killer can be used to kill instances during a random period of time between 12 and 24h. It makes use of the node annotation to store the time to kill value.

### Wokrflow
At a given interval, the application get the list of preemptible nodes and check weither *the node should be deleted or not. If the annotation doesn't exist, a time to kill value is added to the node annotation with a random range between 12h and 24h based on the node creation time stamp. When the time to kill time is passed, the Kubernetes node is marked as unschedulable, drained and the instance deleted on GCloud.

### Known limitations

* Pods in selected nodes are deleted, not evicted.
* Currently deletion time is based on node creation time, so if you deploy this tool when your instances have over 12h then you may experience a lot of nodes getting deleted at once.
* Selecting node pool is not supported yet, the code is processing ALL preemptible nodes attached to the cluster, and there is no way to limit it even via taints nor annotations
* This tool increases the chances to have many small disruptions instead of one major disruption.
* This tool does not guarantee that major disruption is avoided - GCP can trigger large disruption because the way preemptible instances are managed. Ensure your have PDB and enough of replicas, so for better safety just use non-preemptible nodes in different zones. You may also be interested in estafette-gke-node-pool-shifter.



### Software Requirements
Name | Description
--- | --- |
Terraform | >= 0.14.9
Google | >= 3.19.0
Kubernetes provider | >= 1.11.1
Helm | >= 1.1.1

## Usage

 ```shell script
module "preemtible-killer" {
  source = "git::*"
  whitelist_hours = ["09:00 - 12:00, 13:00 - 18:00"]
  blacklist_hours = ["07:00 - 19:00"]
  drain_timeout = "100"
  interval_checks = "60"
}
```
 ```shell script
module "preemtible-killer" {
  source = "git::*"
  whitelist_hours = ["09:00 - 12:00, 13:00 - 18:00"]
}
```

#####Before apply, please enable `Cloud Resource Manager API`

## Inputs
Name | Description | Type | Default | Example | Required
--- | --- | --- | --- |--- |--- 
whitelist_hours | List of UTC time intervals in which deletion is allowed and preferred | `list(string)` | `[]` | `["09:00 - 12:00, 13:00 - 18:00"]` | no
blacklist_hours | List of UTC time intervals in which deletion is NOT allowed | `list(string)` | `[]` | `["07:00 - 19:00"]` | no
drain_timeout | Max time in second to wait before deleting a node | `string` | `300` | n/a | no
interval_checks | Time in second to wait between each node check | `string` | `600` | n/a | no
additional_set | Add additional set for helm | `list(string)` | `[]` | n/a | no
