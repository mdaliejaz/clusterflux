# Introduction

Following are the list of APIs that ClusterFlux application provides to the InfluxDB nodes.

## Register

`/nodes/{cluster}`

Registers an InfluxDB node with a specified cluster assigns an ID, and stores influxDB metadata and node data in etcd with a default time to live set for 15 seconds.

Automatically called from ClusterFlux Service when InfluxDB starts up.

## Deregister

`/nodes/{cluster}/{id}`

Deregisters and deletes InfluxDB node data with a specified cluster and id from etcd.

## Refresh

`/nodes/{cluster}/{id}`

Refreshes the time to live \(TTL\) for an InfluxDB node with {cluster} and {id}.

Automatically called from ClusterFlux Service every 10 seconds to tell ClusterFlux manager that the node is alive.

## List

`/nodes`

Lists all InfluxDB nodes and ClusterFlux nodes

```
{
    "cflux": [
    {
        "id": 8278,
        "ip": "192.168.64.1,10.224.16.214,",
        "hostname": "cflux-host",
        "bind-address": "10.224.16.214:8000",
        "alive": true
    }
    ],
    "influx": [
    {
        "id": 8319,
        "ip": "10.224.16.214,",
        "hostname": "influx-host",
        "bind-address": "10.224.16.214:8888",
        "alive": true
    }
    ]
}
```

## Cluster

`/nodes/{cluster}`

Lists all InfluxDB nodes in a specfied cluster.

```
[
     {
       "id": 8319,
       "ip": "10.224.16.214,",
       "hostname": "influx-host",
       "bind-address": "10.224.16.214:8888",
       "alive": true
    }
]

```

## Write

`/clusters/{cluster}/versions/{version}`

Writes InfluxDB metadata from a specified cluster with a specified version.  Uses etcd's prevIndex option. This allows a user to specify what version the metadata should be, so that old versions do not rewrite new versions. On a successful write, the version is incremented from inside the ClusterFlux Manager. If an old version tries to write over a newer version, ClusterFlux will return the most recent version of the metadata to the node requesting to update.

## Read

`/clusters/{cluster}/versions/{version}`

Reads metadata from a specified cluster and version. Read uses etcd's watcher to track changes on a specific data version. If a version is updated, the new metadata automatically gets returned.
