#### Elasticsearch version upgrade from 5.x to 6.x
Upgrading Elastic search from 5.x to 6.x (link used: 1)https://www.elastic.co/guide/en/cloud/current/ec-upgrading-v6.html 2)https://www.ibm.com/docs/en/configurepricequote/9.5.0?topic=configuring-upgrading-elasticsearch-version-171
Steps need to follow:

Step1: we'll be doing full cluster restart method to upgrade this version because zero time upgrade doesn't support this version.
The process to perform an upgrade with a full cluster restart is as follows:

1)Disable shard allocation:
When you shut down a node, the allocation process will immediately try to replicate the shards that were on that node to other nodes in the cluster, causing a lot of wasted inputs. This can be avoided by disabling allocation before shutting down a node. Copy the below curl command to disable shard allocation

curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d' { "persistent": { "cluster.routing.allocation.enable": "none" } } '
2)Perform a synced flush:
Shard recovery will be much faster if you stop indexing and issue a synced-flush request.A synced flush request is a “best effort” operation. It will fail if there are any pending indexing operations, but it is safe to reissue the request multiple times if necessary.

curl -X POST "localhost:9200/_flush/synced?pretty"
3)Shutdown and upgrade all nodes:
Stop all Elasticsearch services on all nodes in the cluster. Each node can be upgraded following the same procedure described in Stop and upgrade a single node(https://www.elastic.co/guide/en/elasticsearch/reference/2.1/rolling-upgrades.html#upgrade-node).

Upgrade one node, as follows:
a) Copy <INSTALL_DIR>/elasticsearch-1.7.1 to a new directory. This directory will work as the new Elasticsearch installation directory. b)Re-apply the configurations in the old Elasticsearch installation’s config directory to the new Elasticsearch installation’s config directory. This includes the changes to elasticsearch.yml and logging.yml. You need to re-generate the index mappings and copy them to the config directory. c)If the data directory is set differently for the old Elasticsearch installation and the new Elasticsearch installation (that is, the path.data setting in elasticsearch.yml), then move data files from the old version’s data directory to the new version’s data directory. It is better to maintain a data directory outside of the Elasticsearch installation and create a symbolic link within the Elasticsearch installation directory that points to the above data directory. This provides a simple solution for moving from one version of Elasticsearch to a newer version by deleting the symbolic link from the old version’s directory and creating it from a new version.

4)Start the cluster
If you have dedicated master nodes — nodes with node.master set to true(the default) and node.data set to false —  then it is a good idea to start them first. Wait for them to form a cluster and to elect a master before proceeding with the data nodes. You can check progress by looking at the logs.

As soon as the minimum number of master-eligible nodes have discovered each other, they will form a cluster and elect a master. From that point on, the _cat/health and _cat/nodes APIs can be used to monitor nodes joining the cluster:

curl -X GET "localhost:9200/_cat/health?pretty" curl -X GET "localhost:9200/_cat/nodes?pretty"
5)Wait for Yellow
As soon as each node has joined the cluster, it will start to recover any primary shards that are stored locally. Initially, the _cat/health request will report a status of red, meaning that not all primary shards have been allocated.

Once each node has recovered its local shards, the status will become yellow, meaning all primary shards have been recovered, but not all replica shards are allocated. This is to be expected because allocation is still disabled.

6) Renable allocation
Delaying the allocation of replicas until all nodes have joined the cluster allows the master to allocate replicas to nodes which already have local shard copies. At this point, with all the nodes in the cluster, it is safe to reenable shard allocation:

curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d' { "persistent": { "cluster.routing.allocation.enable": "all" } } ' The cluster will now start allocating replica shards to all data nodes. At this point it is safe to resume indexing and searching, but your cluster will recover more quickly if you can delay indexing and searching until all shards have recovered. You can monitor progress with the _cat/health and _cat/recovery APIs:
curl -X GET "localhost:9200/_cat/health?pretty" curl -X GET "localhost:9200/_cat/recovery?pretty"
Once the status column in the _cat/health output has reached green, all primary and replica shards have been successfully allocated.



#### Reindexing of existing documents: link used - https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex
Copies documents from a source to a destination.

The source can be any existing index, alias, or data stream. The destination must differ from the source. For example, you cannot reindex a data stream into itself.

curl -X POST "localhost:9200/_reindex?pretty" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "your-index-file-name"
  },
  "dest": {
    "index": "new-index-file-name"
  }
}
'

Prerequisties: 
If the Elasticsearch security features are enabled, you must have the following security privileges:

The read index privilege for the source data stream, index, or alias.
The write index privilege for the destination data stream, index, or index alias.
To automatically create a data stream or index with an reindex API request, you must have the auto_configure, create_index, or manage index privilege for the destination data stream, index, or alias.
If reindexing from a remote cluster, the source.remote.user must have the monitor cluster privilege and the read index privilege for the source data stream, index, or alias.
If reindexing from a remote cluster, you must explicitly allow the remote host in the reindex.remote.whitelist setting of elasticsearch.yml. See Reindex from remote.
Automatic data stream creation requires a matching index template with data stream enabled. See Set up a data stream.
