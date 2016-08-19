# **Introduction**

ClusterFlux is a distributed InfluxDB node management and distribution library built entirely in Go programming language. It enables InfluxDB to be operated in a distributed clustered environment. ClusterFlux registers\/deregisters InfluxDB nodes, syncs metadata across InfluxDB nodes, and enables InfluxDB nodes to perform peer to peer read\/write of data (shards).

Using the [Clusterflux Service](https://github.com/apigee/influxdb/tree/cflux-v1.0.0-beta3/services/clusterflux) built inside InfluxDB, InfluxDB communicates with the [Clusterflux Application](https://revision.aeip.apigee.net/monitoring/clusterflux) through http APIs. Clusterflux stores InfluxDB metadata as well as node info in a distributed key-value store called [etcd](https://github.com/coreos/etcd).

## **Setup**

ClusterFlux requires [etcd version 2](https://github.com/coreos/etcd/releases/tag/v2.3.7) for its setup. You can install etcd by following instructions on CoreOS etcd repository [on Github](https://github.com/coreos/etcd/blob/master/Documentation/v2/README.md#getting-started). Current implementation supports etcd version 2. Once you have etcd cluster up and running you can verify the installation by running: `curl -L http://127.0.0.1:2379/version`.

If you are running etcd on server other than localhost, you can set the environment variable by running: `export CLUSTERFLUX_ETCD_ENDPOINTS=http://<etcd-endpoint>:port`

Next you need to generate ClusterFlux binaries from the [ClusterFlux repository](https://revision.aeip.apigee.net/monitoring/clusterflux/tree/master). You will need golang and might have to install dependencies. You can follow the steps below to generate the binaries:

1. Clone the [repository](https://revision.aeip.apigee.net/monitoring/clusterflux/tree/master) in `$GOPATH/src/revision.aeip.apigee.net/monitoring/clusterflux`
2. Run `glide install`
3. Run `go install $(glide nv)`
4. Run `clusterflux`

ClusterFlux looks for the following set of environment variables which can be provided through a TOML file named config.toml and placed in `/etc/clusterflux/` or `$HOME/.clusterflux` or the current working directory from which it is being executed from. The configurables variables are listed below:

1. `CLUSTERFLUX_ETCD_ENDPOINTS`: used to specify the etcd etcd-endpoint
2. `CLUSTERFLUX_ETCD_TTL`: used to specify the time to keep a node marked alive without getting any heartbeat from that node

The InfluxDB binaries can be generated from [Apigee's Github repository](https://github.com/apigee/influxdb/tree/cflux-v1.0.0-beta3) by following the steps below (You should also have git and mercurial installed):

1. Clone the [repository](https://github.com/apigee/influxdb/tree/cflux-v1.0.0-beta3) in `$GOPATH/src/github.com/influxdata/influxdb`
2. InfluxDB uses [gdm](https://github.com/sparrc/gdm) to manage dependencies. Install it by running the following:

    ```
    go get github.com/sparrc/gdm
    ```

3. Get the dependencies for the project by execute the following commands. The changes for ClusterFlux application are currently on `cflux-v1.0.0-beta3` branch, and as such you'd need to checkout the branch before proceeding further.

  ```
  cd $GOPATH/src/github.com/influxdata/influxdb
  git checkout --track -b cflux-v1.0.0-beta3 origin/cflux-v1.0.0-beta3
  gdm restore
  ```

4. To then build and install the binaries, run the following command.

  ```
  go clean ./...
  go install ./...
  ```

  The InfluxDB binary (called `influxd`) will be generated in the `$GOPATH/bin` path.

5. Run `influxd` to start the influxdb server.

6. You can test the cluster by hitting the URL: `http://localhost:8000/nodes`

# Run in kubernetes

For easiest setup, install Kubernetes and use the etcd.yaml file provided to start up three instances of etcd. All the yaml files are provided in the [kubernetes folder](https://revision.aeip.apigee.net/monitoring/clusterflux/tree/master/kubernetes) in the ClusterFlux repository.

1. To generate the InfluxDb image run: `GO_VER=1.6.2 ./build-docker.sh`. This will generate an image called `influxdb`.

2. To generate ClusterFlux image, run: `docker build -t clusterflux .`. This will generate an image called clusterflux which will be used by the yaml files for kubernetes setup.

The clusterflux.yaml file can be used to start ClusterFlux pods. The clusterflux-deployment.yaml and clusterflux-rc.yaml can be used to start ClusterFlux deployments and replication-controller respectively. It's recommended to provide a load balancer or a service for the ClusterFlux application; the clusterflux-service.yaml can be used to start the ClusterFlux service at port 30000. The influxdb.yaml file can be used to start InfluxDb pods. If you are not using these yaml files, make sure you open the required ports for the communication.

Clusterflux runs on port 8000, while InfluxDB needs ports 8083, 8086 and 8088 to be open.

You can create pods, service, etc by running the command: `kubectl create -f <filenmae>`
