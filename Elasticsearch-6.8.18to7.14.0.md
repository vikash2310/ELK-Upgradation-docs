##### Elasticsearch version upgrade from 6.x to 7.x(Using ROLLING UPGRADE METHOD)
Upgrading Elastic search from 6.x to 7.x (link used: 1)https://www.elastic.co/guide/en/cloud/current/ec-upgrading-v7.html 2)https://documentation.wazuh.com/current/upgrade-guide/legacy/upgrading-elastic-stack/from-6.8-to-7.x.html

If you have a cluster and want to upgrade to version 7.x, there are a few things you must do to prepare for the upgrade. Preparing for your upgrade ahead of time ensures that you can enjoy the new features and improved usability of Elasticsearch 7.x as quickly as possible.
Elasticsearch Service upgrades differ from your on-premise installation because all major configuration changes for Elasticsearch, Kibana, and the security features are handled for you.

When you upgrade, Elasticsearch Service automatically runs the deprecation API to retrieve information about the cluster, node, and index-level settings that need to be removed or changed. If there are any deprecation issues that would prevent the upgraded deployment from successfully performing, the upgrade fails. To resolve the deprecation issues, use the Upgrade Assistant in Kibana.

##### Steps to follow:
1)Download and install the Public Signing Key:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

2)Save the repository definition to /etc/apt/sources.list.d/elastic-7.x.list:
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

###### Upgrading Elasticsearch:
1)Disable shard allocation:

curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": "primaries"
  }
}
'

2)Stop non-essential indexing and perform a synced flush (optional):

###### curl -X POST "localhost:9200/_flush/synced"

3)Shut down a single node:

systemctl stop elasticsearch or sudo -i service elasticsearch stop

4)Upgrade the shut down node:

 ###### sudo apt-get install elasticsearch=7.14.0
 ###### systemctl restart elasticsearch or sudo -i service elasticsearch restart
 
 5)Starting with Elasticsearch 7.0, master nodes require a configuration setting with the list of the cluster master nodes. The following settings must be added in the configuration of the Elasticsearch master node (elasticsearch.yml):

discovery.seed_hosts:
  - master_node_name_or_ip_address
cluster.initial_master_nodes:
  - master_node_name_or_ip_address

6)Restart the service:

###### systemctl daemon-reload
###### systemctl restart elasticsearch

7)Start the newly-upgraded node and confirm that it joins the cluster by checking the log file or by submitting a _cat/nodes request:

###### curl -X GET "localhost:9200/_cat/nodes"

8)Reenable shard allocation:

###### curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
'

9)Before upgrading the next node, wait for the cluster to finish shard allocation:

###### curl -X GET "localhost:9200/_cat/health?v"

10)Repeat the steps for every Elasticsearch node.

