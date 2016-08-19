# Introduction

This file contains design decisions that are difficult to explain through code.

## Cluster Read
Do a Get before Watcher. In ClusterFlux Application we do an extra Get before starting the watcher for long polling on updates.

Reason: The Watcher can be set to watch after some index, but the influxDB node’s index might be too old for etcd to remember. Etcd keeps track of only a limited amount of history across all nodes. On the other hand if we use ‘start from the current index’, we might be waiting for the next update even if there’s an update upstream; if influxDB is version 2 and remote cluster is at version 4 and if we start watching from current version (=4) we won’t update influxDB unless a version 5 comes in. So the best approach would be to do a Get and check for version mismatch. If there’s a version mismatch, return with the update, if not, start the watcher.

## Extra post to remote on starting service

Currently we do an extra Post to ClusterFlux when Open() method of ClusterFlux Client is called. This is done to create a key in Etcd. If this is not done, the ClusterFlux sync service (service that polls for updates) starts throwing key not found; if this is the first node coming up in a new cluster.
