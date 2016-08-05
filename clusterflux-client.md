# Introduction

The Clusterflux Service \(`cflux.Client`\) is a wrapper on InfluxDB's `meta.Client`. The wrapper, while wraps some of the methods of `meta.Client` to implement the Clusterflux service, uses the existing implementations for all the methods; thus not introducing any new behavior to the InfluxDB itself. This wrapper service is responsible for registering the InfluxDB node in the cluster and for sending a heartbeat to the Clusterflux manager reporting it's alive status. Also, the service syncs the InfluxDB metadata with the remote Clusterflux manager. All http requests from the node to the remote manager is done with exponential backoff to avoid overwhelming the manager. A total of 5 attempts are made before reporting failure to connect.

## Node Registration and Heartbeat

When a new InfluxDB node comes up, it checks the environment variables called `CLUSTER` and `CFLUX_ENDPOINT`. It uses the `CLUSTER` variable to register to the specified cluster. If no value is specified, it registers to a cluster named `default`. The `CFLUX_ENDPOINT` environment variable specifies the address of the remote Clusterflux manager. This address could be the load balancer's \(for the Clusterflux managers\) address or address of any one of the managers. These environment variables can be set through a configuration file or through the commandline itself.

The service reads the hostname and IP addresses of the InfluxDB machine and reports this metadata to the Clusterflux manager for book keeping. In future, this service will also report the bind address of the machine for the service-manager communication. Currently the bind address is set to the remote address of the http request sent from the node. Once the Clusterflux receives the node data sent by InfluxDB node, it creates a unique key for this node. This unique key \(node ID\) is then used for the heartbeat and all future communications. A heartbeat with this node ID is sent every 5 seconds to the Clusterflux manager reporting the alive status for the node.

## Cluster Sync Service

The `cflux.Client` starts a sync service for the InfluxDB metadata syncing. It makes a http GET request to the manager with it's cluster name and current version of local metadata. The manager returns an updated metadata \(with the new version\) if the version reported in the request was older than the version on remote. If no new changes are found on remote in an interval of 5 minutes, the clusterflux manager sends a Gateway Timeout \(to be changed to 204 - No content\), a new request is made from the service.

## Updating remote metadata

On any local changes to the InfluxDB metadata, a new database metadata snapshot is pushed to the cluster. The metadata version is maintained by the local as well as the remote Clusterflux manager. If the remote metadata is higher than the local version, the local metadata is updated followed by applying the new changes on the updated metadata. A retry to update the remote is then made. The Clusterflux manager on receiving a new update, updates it's metadata and version, notifying other InfluxDB nodes to update themselves.

