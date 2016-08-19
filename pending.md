# Introduction

This file contains list of pending features that needs to be implemented.

1. Test cases for cluster meta watcher. Currently it's only testing read behavior. Would be good to have more granular tests. Most of the code is not covered through tests. This should be of highest priority.

2. Handle all errors. Some of the errors are not handled properly.

3. Benchmark Protobuf (serializing slice of protobuf vs serilizing protobuf for a slice (eg. point vs points proto))

4. All configurations should be from TOML (eg. Bind Address). Most of our configuration is through environment variables.

5. Look at the \_internal database replication (remote/local?).

6. Handle scenarios where node goes down. Move data from existing replicas to the replaced node.

7. Create StatMap for ClusterFlux and log ClusterFlux metrics (remote write, meta update success/failure) to \_internal database.

8. Kubernetes (with petset)

9. Upgrade etcd2 to etcd3.

10. UI for admins to monitor alive nodes, etc.

11. Store last 10 data versions of metadata. store recent versions as /cluster/history/<version> to etcd, say last 10 versions for a cluster				

12. URL QueryUnescape in ClusterFlux application. Cluster names can have some special characters which might not work with our API unless escaped. We should escape all names in the request and response.

13. Ability to stop cluster read thread.	There's a done channel on which the service listens to. We can implement a method that users can call to put a message on this channel to cancel the read thread. This is a low priority as the maximum life of long poll is anyways 5 minutes.

14. Fix the following bug (occurred during preparing for final presentation):

  ```
  [clusterflux] 2016/08/17 03:44:00 Polling for updates from Clusterflux.
  panic: runtime error: invalid memory address or nil pointer dereference
  [signal 0xb code=0x1 addr=0x58 pc=0x68b888]

  goroutine 43 [running]:
  panic(0xcb8560, 0xc820010080)
  	/usr/local/go/src/runtime/panic.go:481 +0x3e6
  github.com/influxdata/influxdb/gossip.(*TSDBStore).WriteToShard(0xc8201826c0, 0x1, 0xc82042ff80, 0x6, 0x8, 0x0, 0x0)
  	/gopath/src/github.com/influxdata/influxdb/gossip/tsdbstore.go:85 +0x88
  github.com/influxdata/influxdb/coordinator.(*PointsWriter).writeToShard(0xc8200725a0, 0xc82034ba20, 0xc82011bbd0, 0x9, 0xddfc28, 0x7, 0xc82042ff80, 0x6, 0x8, 0x0, ...)
  	/gopath/src/github.com/influxdata/influxdb/coordinator/points_writer.go:323 +0x2c1
  github.com/influxdata/influxdb/coordinator.(*PointsWriter).WritePoints.func1(0xc8202a47e0, 0xc8200725a0, 0xc82034ba20, 0xc82011bbd0, 0x9, 0xddfc28, 0x7, 0xc82042ff80, 0x6, 0x8)
  	/gopath/src/github.com/influxdata/influxdb/coordinator/points_writer.go:278 +0x8d
  created by github.com/influxdata/influxdb/coordinator.(*PointsWriter).WritePoints
  	/gopath/src/github.com/influxdata/influxdb/coordinator/points_writer.go:279 +0x3e6
    ```
14. bindAddress is actually advertised address; peer address of the node. Not the address at which it listens on. The address it listens on is currently hard coded to :8888.
