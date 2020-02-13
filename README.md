## How to search into schema-less JSON with DSE?

The idea is to flatten JSON keys and put keys/values in a dynamic map field through a Field Input Transformer.

If you need different analyzers, you can add several dynamic field in the `schema.xml`.

### Field input/output (FIT) transformer API

This is an example of https://docs.datastax.com/en/dse/5.1/dse-dev/datastax_enterprise/search/fieldInputOutputTransformerApi.html

```
CREATE KEYSPACE fit WITH replication = {'class': 'SimpleStrategy' , 'replication_factor': 1 };

# create table with JSON in complexvalue column
CREATE TABLE fit.jsonlist (
    id text PRIMARY KEY,
    complexvalue text
);
```

You need to download and unzip dse and update the path in maven `pom.xml` of 3 jars.
Then, you will need to change (if you want) the name of the column containing JSON in JsonListFieldInputTransformer.java

```
mvn clean install
```
copy target/fit-1.0-SNAPSHOT.jar to install-location/resources/solr/lib 

```
# on my laptop..
cp target/fit-1.0-SNAPSHOT.jar /home/florent/soft/dse-5.1.6/resources/solr/lib/
```
restart dse node
```
# create search index 
dsetool create_core fit.jsonlist schema=schemaList.xml solrconfig=configList.xml
```
```
INSERT INTO fit.jsonlist (id, complexvalue) VALUES ('test2','{"name":"Mortgage","amount":"100000","duration":"60","garantor":{"partyId1":"abc123","partyId2":"def456"}}');

select * from fit.jsonlist where solr_query='map_garantor.partyId1:abc123' ;

 id    | complexvalue                                                                                               | solr_query
-------+------------------------------------------------------------------------------------------------------------+------------
 test2 | {"name":"Mortgage","amount":"100000","duration":"60","garantor":{"partyId1":"abc123","partyId2":"def456"}} |       null


INSERT INTO fit.jsonlist (id, complexvalue) VALUES ('test','{"name":"John","age":30,"cars":["Ford","BMW","Fiat Panda"]}');

select * from fit.jsonlist where solr_query='map_cars_:ford' ;

 id   | complexvalue                                          | solr_query
------+-------------------------------------------------------+------------
 test | {"name":"John","age":30,"cars":["Ford","BMW","Fiat Panda"]} |       null



select * from fit.jsonlist where solr_query='map_cars_:"ford bmw"' ;

 id | complexvalue | solr_query
----+--------------+------------

(0 rows)    // as expected


select * from fit.jsonlist where solr_query='map_cars_:"fiat panda"' ; 

 id   | complexvalue                                                | solr_query
------+-------------------------------------------------------------+------------
 test | {"name":"John","age":30,"cars":["Ford","BMW","Fiat Panda"]} |       null
 
 ```

