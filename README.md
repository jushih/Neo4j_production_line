## Dataset
I will be modeling production line data in a Neo4j database. Production line data is suited to a graph database because it is highly connected and has the advantage of performance and flexibility over a traditional RDMS. 
The dataset was obtained from the Kaggle website and provided by Bosh, a company that manufactures high end appliances. I use a subsample of the data put together by Hitesh of the Portland Data Science Group and deploy it to a Neo4j graph database. 

The dataset contains:

* **id** - unique identifier for parts
* **L columns** - represents multiple manufacturing lines (M), stations (S), and additional features (F). 
* **response** - 1 is a faulty part, 0 is a non-faulty part

From here on, when I refer to a "station", I am referring to the station at its most granular feature level. 

## Data Modeling and Preprocessing

A graph database is made up of nodes and relationships. I remodel the Bosch dataset into a graph database schema where each node will represent a station in the production line, and a part moving through the station will be represented by a relationship. I will record the values of each part as it moves from station to station by saving it as a relationship property. 

For ETL of the data, I will take advantage of the LOAD CSV feature in Neo4j. I restructure the data into CSVs where each row represents the node or relationship that will be created.
My final data is parsed into two CSVs:

**nodes.csv** - Each row is a station in the production line. There are 968 unique stations.

![image1](/images/nodes.png)

**relationships.csv** - Each row is the transition of a part moving from one station to another, along with its value as it exits the station, the station where the part began in the production line (line_start) and ended in the production line (line_end), and whether it was a faulty part or not (response).

![image2](/images/relationships.png)

## Neo4j

Next I load the data from the CSV files into the graph database. I download the community version Neo4j and create a new database. 


## References
https://www.kaggle.com/c/bosch-production-line-performance
