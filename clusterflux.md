# **Introduction**

Clusterflux is a distributed InfluxDB node management and distribution library built entirely in Go. Clusterflux enables InfluxDB to operated in a distributed clustered environment. Clusterflux registers\/deregisters InfluxDB nodes, syncs metadata across InfluxDB nodes, and enables InfluxDB nodes to read\/write data peer to peer.

Using the [Clusterflux Service](https://revision.aeip.apigee.net/monitoring/clusterflux) library, InfluxDB automatically communicates with Clusterflux through an http API. Clusterflux stores InfluxDB metadata as well as node info in a distributed key-value store called [etcd](https://github.com/coreos/etcd).

## **Setup**

The Clusterflux Manager requires etcd and Docker to be set up.

For easiest setup, install Kubernetes and use the etcd.yaml file provided to start up three instances of etcd. The Clusterflux yaml files can be used to start Clusterflux pods\/deployments\/services. All environment variables can be modified inside of meta.go or through the command line.

