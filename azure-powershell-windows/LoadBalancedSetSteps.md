Originally documented here: https://gist.github.com/JogoShugh/ada9223272432ab6aea8

* Deleted port 2113 from Node 1 in Azure
* Added a new port of 2113 on Node 1, making it a Balanced Set named ESClust
* For nodes 2 and 3, added this same balanced set port, resulting in all three nodes have port 2113
* Because of the original script, at this point, Node 1 has 2113, and Node 2 has 2113 AND 2213, and Node 3 has port 2113 and 2313
* Now, Only on node 1:

`New-NetFirewallRule -Name Allow_Direct_EventStore_Ext_In -DisplayName "Allow inbound External Event Store direct traffic" -Protocol TCP -Direction Inbound -Action Allow -LocalPort 2013`

> Note: It possibly needs ,2013 as below, but I cannot remember:
`New-NetFirewallRule -Name Allow_Direct_EventStore_Ext_In -DisplayName "Allow inbound External Event Store  >direct traffic" -Protocol TCP -Direction Inbound -Action Allow -LocalPort 2013,2013`

* Update the YAML config in `C:\EventStore\EventStore-Config.yaml` for each node as follows. Specifically speaking, we need RunProjections: ALL, we are changing ExtHttpPort to be the SAME for all nodes, and then we add two HttpPrefixes values, one specific to the local machine, and the other with the clustered, load-balanced address.

## Node 1:

```yaml
# EventStore configuration file. Created at: 2015-03-18 16:31:25Z
Db: F:\Data\eventstore
Log: D:\Logs\eventstore
RunProjections: ALL
IntIp: 10.0.0.4
ExtIp: 10.0.0.4
IntTcpPort: 1112
IntHttpPort: 2112
ExtTcpPort: 1113
ExtHttpPort: 2113
HttpPrefixes:
  - http://10.0.0.4:2113/
  - http://cs-eventstore-cluster.cloudapp.net:2113/
DiscoverViaDns: false
GossipSeed: ['10.0.0.5:2112','10.0.0.6:2112']
ClusterSize: 3
```

## Node 2:

```yaml
# EventStore configuration file. Created at: 2015-03-18 16:31:00Z
Db: F:\Data\eventstore
Log: D:\Logs\eventstore
RunProjections: ALL
IntIp: 10.0.0.5
ExtIp: 10.0.0.5
IntTcpPort: 1112
IntHttpPort: 2112
ExtTcpPort: 1213
ExtHttpPort: 2113
HttpPrefixes:
  - http://10.0.0.5:2113/
  - http://cs-eventstore-cluster.cloudapp.net:2113/
DiscoverViaDns: false
GossipSeed: ['10.0.0.4:2112','10.0.0.6:2112']
ClusterSize: 3
```

## Node 3:

```yaml
# EventStore configuration file. Created at: 2015-03-18 16:31:29Z
Db: F:\Data\eventstore
Log: D:\Logs\eventstore
RunProjections: ALL
IntIp: 10.0.0.6
ExtIp: 10.0.0.6
IntTcpPort: 1112
IntHttpPort: 2112
ExtTcpPort: 1313
ExtHttpPort: 2113
HttpPrefixes:
  - http://10.0.0.6:2113/
  - http://cs-eventstore-cluster.cloudapp.net:2113/
DiscoverViaDns: false
GossipSeed: ['10.0.0.4:2112','10.0.0.5:2112']
ClusterSize: 3
```

* Add port mapping on each node as follows:

## Node 1:

`netsh interface portproxy add v4tov4 listenport=2013 connectport=2113 connectaddress=10.0.0.4`

## Node 2:

`netsh interface portproxy add v4tov4 listenport=2213 connectport=2113 connectaddress=10.0.0.5`

## Node 3:

`netsh interface portproxy add v4tov4 listenport=2313 connectport=2113 connectaddress=10.0.0.6`
