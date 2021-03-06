## Modeling production line data in Neo4j 

In this project, I will deploy a Neo4j graph database and populate it with data from the Bosch manufacturing line. Pictured below is a sample from my final database. It shows the paths of two parts (23 and 76) as they move through stations in the production line. Manufacturing and supply chain data is suited to graph databases because it is highly connected and has the advantage of performance and flexibility over a traditional RDMS. 

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/2parts.png" width="800">

## Dataset
The dataset was obtained from the Kaggle website and provided by Bosch, a company that manufactures high end appliances. I use a subsample of the data put together by Hitesh Basantani of the Portland Data Science Group.

The dataset contains:

* **id** - unique identifier for parts
* **L columns** - represents multiple manufacturing lines (L), stations (S), and additional features (F). In each column is the value of the part measured when it leaves the station.
* **response** - 1 is a faulty part, 0 is a non-faulty part

From here on, when I refer to a "station", I am referring to the station at its most granular feature (F) level. 

## Data Modeling and Preprocessing

A graph database is made up of nodes and relationships. I remodel the Bosch dataset into a graph database schema where each node will represent a station in the production line, and a part moving through the station will be represented by a relationship. I will record the values of each part as it moves from station to station by saving it as a relationship property. 

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/data_schema.png" width="700">

For ETL of the data, I will take advantage of the LOAD CSV feature in Neo4j. I restructure the Bosch data into CSVs where each row represents the node or relationship that will be created. I also find the starting stations and end stations of each part. The preprocessing steps can be found in the parse_transitions iPython notebook.

My final dataset is parsed into two CSVs:

**nodes.csv** - Each row is a station in the production line. There are 968 unique stations:

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/nodes.png" width="250">

**relationships.csv** - Each row is the transition of a part moving from one station to another, along with its value as it exits the station, the station where the part began in the production line (line_start) and ended in the production line (line_end), and whether it was a faulty part or not (response):

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/relationships.png" width="600">

## Loading to Neo4j

Next is loading the data from the CSV files into the graph database. I download the community version Neo4j and create a new database. 

To allow importing from CSV, I change the settings in the neo4j.conf file and put the CSVs in the import folder where they are read from.
```
dbms.security.allow_csv_import_from_file_urls=true

dbms.directories.import=import #path to csv directory
```

Before the first load, I check the header mapping to ensure the fields are correctly formatted.
```
LOAD CSV WITH HEADERS FROM "file:///nodes.csv" AS line WITH line
RETURN line
LIMIT 5; 
```

I then create the station nodes. 
```
USING PERIODIC COMMIT
LOAD CSV WITH HEADERS FROM "file:///nodes.csv" AS row
CREATE (:Station {stationId: row.stationId, line:row.line, station: row.station, feature: row.feature});
```

I add a constraint to ensure station nodes are unique. This also indexes the station and allows for faster querying, which will be important for scaling.
```
CREATE CONSTRAINT ON (s:Station) ASSERT s.stationId IS UNIQUE;
```

Once the nodes are created, I run a Cypher query to look at the nodes in the database. Cypher is the query language of Neo4j. This simple query pulls up 10 stations.
```
MATCH(n)
RETURN n
LIMIT 10;
```

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/neo4j_nodes.png" width="800">

Hovering over each node will show its properties. For the highlighted station feature F1145, we can see that it belongs to Line 1 and Station 24. Its full ID is L1_S24_F1145.

Now that the nodes are created, it's time to add relationships. I use Cypher to "match" the nodes to the station that each part is entering and exiting. Then I draw a relationship between those nodes and fill it with the properties of that part during its transition.

```
USING PERIODIC COMMIT
LOAD CSV WITH HEADERS FROM "file:///relationships" AS row
MATCH (en:Station {stationId: row.enter})
MATCH (ex:Station {stationId: row.exit})
MERGE (en)-[r:TO {lineStart:row.line_start,lineEnd:row.line_end,value:row.value,partID: row.part_id,response:row.response}]->(ex)
```

I can run a Cypher query to look at the path of one part through the manufacturing line. 

```
MATCH (n)-[r]-(m)
WHERE r.partID = '71'
return n,r, m
```

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/part71.png" width="800">

Part 71 moves through 190 stations. I toggle the relationships to display the part's value. I can zoom in on the image to look at a singular relationship. As the part moves from station F314 to F318, its value is -0.219. The part was not a defective part (response = 0). The part entered the manufacturing line at station F0, and finished at station F3950.

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/part71_zoom.png" width="800">

Here is another sample of the data that shows multiple parts moving between stations. The relationships were toggled to display the part's response. Due to the large number of parts, only a subset of the relationships connected to the nodes are shown. The highlighted relationship is a faulty part that was produced by the station. 

<img src="https://github.com/jushih/Neo4j_production_line/blob/master/images/faulty_part.png" width="800">


## References
https://www.kaggle.com/c/bosch-production-line-performance
