# Druid

@@@@@Imply Pivot Install  @@@@@      
tar -xzf imply-2.2.3.tar.gz
cd imply-2.2.3

@@@@@Start@@@@@

bin/supervise -c conf/supervise/quickstart.conf

@@@@@Post the Index.json@@@@@

bin/post-index-task --file quickstart/wikiticker-index.json


@@@@@Zookeeper Install @@@@@

curl http://www.gtlib.gatech.edu/pub/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz -o zookeeper-3.4.6.tar.gz
tar -xzf zookeeper-3.4.6.tar.gz

@@@@@Start@@@@@

cd zookeeper-3.4.6
cp conf/zoo_sample.cfg conf/zoo.cfg
./bin/zkServer.sh start


@@@@@Druid Install@@@@@

curl -O http://static.druid.io/artifacts/releases/druid-0.10.0-bin.tar.gz
tar -xzf druid-0.10.0-bin.tar.gz
cd druid-0.10.0

@@@@@Start@@@@@

java `cat conf-quickstart/druid/historical/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/historical:lib/*" io.druid.cli.Main server historical
java `cat conf-quickstart/druid/broker/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/broker:lib/*" io.druid.cli.Main server broker
java `cat conf-quickstart/druid/coordinator/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/coordinator:lib/*" io.druid.cli.Main server coordinator
java `cat conf-quickstart/druid/overlord/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/overlord:lib/*" io.druid.cli.Main server overlord
java `cat conf-quickstart/druid/middleManager/jvm.config | xargs` -cp "conf-quickstart/druid/_common:conf-quickstart/druid/middleManager:lib/*" io.druid.cli.Main server middleManager


@@@@@Submit the query via curl@@@@@

curl -X POST "http://localhost:8082/druid/v2/?pretty" \ -H 'content-type: application/json' -d @query.body


@@@@@Sample Query@@@@@

{
    "queryType": "groupBy",
    "dataSource": "druidtest",
    "granularity": "all",
    "dimensions": [],
    "aggregations": [
        {"type": "count", "name": "rows"},
        {"type": "longSum", "name": "imps", "fieldName": "impressions"},
        {"type": "doubleSum", "name": "wp", "fieldName": "wp"}
    ],
    "intervals": ["2010-01-01T00:00/2020-01-01T00"]
}


@@@@@Delete the Data Source  Query@@@@@


curl -XDELETE http://localhost:8081/druid/coordinator/v1/datasources/Jmatek_June_2017


@@@@@Average Query [leapfrog]@@@@@


{
   "queryType":"groupBy",
   "dataSource":"leapfrog",
   "granularity":"all",
   "dimensions":[ ],   "aggregations":[{"name":"!T_1","type":"count","fieldName":"rows"},{"name":"!T_2","type":"doubleSum","fieldName":"Target DOS"}],"postAggregations":[{"type":"arithmetic","fn":"*","fields":[{"type":"arithmetic","fn":"/","fields":[{"type":"fieldAccess","fieldName":"!T_2"},{"type":"fieldAccess","fieldName":"!T_1"}]},{"type":"constant","value":1}],"name":"Avg DOS"}],
   "intervals":[
      "2006-01-01T00:00:00.000Z/2020-01-01T00:00:00.000Z"
   ]
}


@@@@@JavaScript Query For druid Aggregation@@@@@

{
   "queryType":"groupBy",
   "dataSource":"leapfrog",
   "granularity":"all",
   "dimensions":[],
   "aggregations":[{"name":"rows","type":"count","fieldName":"rows"},
{"name":"TargetDOS","type":"doubleSum","fieldName":"Target DOS"}],"postAggregations":[
{
  "type": "javascript",
  "name": "Target DOS Average",
  "fieldNames": ["TargetDOS", "rows"],
  "function": "function(TargetDOS, rows) { return Math.abs(TargetDOS) / rows; }"
}],   "intervals":[ "2006-01-01T00:00:00.000Z/2020-01-01T00:00:00.000Z"  ]}
