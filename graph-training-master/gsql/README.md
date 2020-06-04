# GSQL Reference
## Tigergraph Platform overview
docs [here](https://docs.tigergraph.com/intro/tigergraph-platform-overview)

### Key services

* `GSE` - storage processing engine
* `GPE` - graph processing engine
* `RESTPP` - accepts RESTful requests and passes it on to GPE
* `DICT` - metadata stores that saves config and the schema catalog
* `GSQL` - interprets and executes graph processing, including DDL, DML, SELECT \ queries.
* `GraphStudio` - UI that lets the user interact with the system. Alternative to the GSQL CLI.
* `GADMIN` - admin utility
* Misc internal - Kafka, Zookeeper, Ngnix, IDS, GBAR

### What is GSQL?

* `GSQL Service` - component or a service
* `GSQL CLI` - interactive shell to run GSQL language statements
* `GSQL Language` - set of DDL, DML, etc. construts that instruct GSQL service to perform some actions.

Note: [Run a remote GSQL client](https://docs.tigergraph.com/dev/using-a-remote-gsql-client)

### Glossary

* Path - sequence of Vertices and Edges. `Patient -(has)-> Coverage -(under)-> (Product)`
* Hop - hop or an edge hop is the act of traversing an edge
 
### Graph objects

* Vertices
* Edges
* Tuples
* Graphs
* Users, Roles assignments
* Jobs - Loading, Schema change, etc. 
* Queries

### GSQL Language: DDL constructs
docs [define graph schema](https://docs.tigergraph.com/dev/gsql-ref/ddl-and-loading/defining-a-graph-schema), 
     [modify graph schema](https://docs.tigergraph.com/dev/gsql-ref/ddl-and-loading/modifying-a-graph-schema)

* `CREATE VERTEX`
* `CREATE EDGE… [DIRECTED OR UNDIRECTED] … WITH REVERSE_EDGE`
* `CREATE GRAPH`
* `SCHEMA CHANGE JOB` - ALTER, DROP objects from an existing GRAPH. CREATE and RUN schema change.
* `DROP VERTEX|EDGE|GRAPH|ALL` 

Note: Most of the operations are available via REST APIs as well.

#### Key concepts

* [Multigraphs](https://docs.tigergraph.com/intro/multigraph-overview)
  * One tigergraph instance can support multiple graphs, each with its own schema, data and security.
  * Graphs can have overlap in both schema and data. 
* Context 
  * `GLOBAL` context - vertices, edges, tuples can be created here
  * `GRAPH` context - queries, load jobs are created here
`Note: Objects in global contexts can be part of multiple graphs sometimes AKA subgraphs or Multigraphs`
* Graph security- [Roles & Privileges](https://docs.tigergraph.com/admin/admin-guide/user-access-management/user-privileges-and-authentication#roles-and-privileges)
  * Privileges - Superuser, Graph level - Admin, Designer, QueryWriter, QueryReader
  * Super user can manage users, roles and access tokens using GSQL
* [Base data types](https://docs.tigergraph.com/dev/gsql-ref/querying/data-types#overview-of-types)
  * `INT, UINT, FLOAT, DOUBLE, STRING, BOOL, DATETIME, VERTEX, EDGE, JSONOBJECT, or JSONARRAY, STRING COMPRESS`

### GSQL Language: DML constructs
docs [here](https://docs.tigergraph.com/dev/gsql-ref/ddl-and-loading)

* `LOADING JOB` - GSQL Language construct
* `UPSERT, DELETE` - GSQL Language constructs
* `UPSERT, DELETE` - [REST API construct](https://docs.tigergraph.com/dev/restpp-api/built-in-endpoints#accessing-and-modifying-the-graph-data)

### GSQL Language: Query constructs
docs [here](https://docs.tigergraph.com/dev/gsql-ref/querying)

* `SELECT` statements
* Control flow statements - `IF THEN ELSE, CASE, WHILE, FOREACH`
* [Output statements](https://docs.tigergraph.com/dev/gsql-ref/querying/output-statements-and-file-objects) - `PRINT, LOG, TO_CSV`
  * Produce JSON formatted results by default, can be converted to CSV and can be printed to a FILE
  * PRINT statement is represented as one JSON object with the "results" array. 
* [Built-in: Operators, functions](https://docs.tigergraph.com/dev/gsql-ref/querying/operators-functions-and-expressions)
  * `COUNT() approx_count(*),, sum(), min(), max(), avg, PRINT, LIMIT, ANY, ORDER BY, primary_id, outdegree(), ACCUM, HAVING, WHILE, CASE, IF THEN ELSE, FOREACH, DISTRIBUTED, COALESCE`
  * Standard mathematical, boolean, String, comparision and bitwise operators
  * funcions: datetime, UDFs 

`REST endpoints - built-in endpoints, every query gets deployed as an endpoint.`

#### Key concepts

* `ACCUM` - [Accumulators](https://docs.tigergraph.com/dev/gsql-ref/querying/accumulators) are specialized data objects which support accumulation operations.
```Accums are mutable mutex variables shared among all the graph computation threads exploring the graph within a given query. To improve performance, the graph processing engine employs multithreaded processing. Modification of accumulators is coordinated at run-time so the accumulation operator works correctly (i.e., mutually exclusively) across all threads. ACCUM is also a CLAUSE used in conjuction with a SELECT statement to trigger accumulation phase```
  * Vertex attached accumulators -  mutable state variables that are attached to each vertex in the graph for the duration of the query's lifetime. Can only be accessed in an ACCUM or POST-ACCUM clause within a SELECT block.
```Example: SumAccum<int>   @neighbors;```
  * Global accumulators - a single mutable accumulator that can be accessed or updated within a query. They can be accessed or updated via the accumulate operator += anywhere within a query, including inside a SELECT block.
```Example: SumAccum<int>   @@totalNeighbors;```
  * Accumulator types
    * Scalar (single value): `SumAccum, MinAccum, MaxAccum, ...`
    * Collection (set of values): `ListAccum, MapAccum, HeapAccum, ...`
    * Nested (Collection of Accums)

* `TUPLE` - A tuple is a user-defined data structure consisting of a fixed sequence of baseType variables.

* [`DISTRIBUTED QUERY`](https://docs.tigergraph.com/dev/gsql-ref/querying/distributed-query-mode)
  * Default execution mode: all processing happens at an execution hub (one of the nodes). Data is copied over to the hub for processing.
  * Distributed mode: all nodes representing the full copy of the graph participate in processing. Output is collected at one node. There are several restrictions around what GSQL language features are available in this mode.

#### Query Example
```
  Claims = SELECT c FROM Patient:p -(HAS)-> Claims:c
            WHERE p ==seed
            ACCUM p.@claimCount + =1
            HAVING p.@claimCount > threshold_cnt
            ORDER BY p.@claimCount DESC
            LIMIT 5;
```

### 3 steps to run a query

* `CREATE QUERY helloworld() …`
* `INSTALL QUERY helloWorld() …` - installs the query to the catalog and generates a REST endpoint.
* `RUN QUERY helloWorld()…` either via GSQL CLI or via HTTP(S) REST endpoint

Note: New interpreted mode bypassess installation. It interprets and run processing line by line. 
`INTERPRET QUERY() FOR GRAPH graph_name SYNTAX ("v2") {<query body>}`

### Misc notes

* The latest version supports multi hop patterns within a single statement. This was limited to single hop per SELECT block in earlier versions.
* There are a few [built in path finding algorithms](https://docs.tigergraph.com/dev/restpp-api/built-in-endpoints#path-finding-algorithms)

### Example: infer care team from claims

```
Graph model

Individual -(HAS_MBI)-> MBI
MedicalClaim -(SERVICE_PROVIDED_TO)-> Individual
MedicalClaim -(SERVIVE_PROVIDED_BY)-> Provider
```
```
USE GRAPH HealthCare

DROP QUERY getCareTeamForMember

CREATE QUERY getCareTeamForMember(Vertex<MBI> mbi) FOR GRAPH HealthCare
        RETURNS (MapAccum<Vertex<Provider>, INT>)
{
	TYPEDEF tuple<INT visitCount, Vertex<Provider> prv> providerVisits;
        HeapAccum<providerVisits>(5, visitCount DESC, prv DESC) @@topProvidersByVisits;
	MapAccum<Vertex<Provider>, INT> @@prvVisits;
	MapAccum<Vertex<Provider>, INT> @@returnTop;

        Start = {mbi};

        Indiv  = SELECT i from Start:m -(R_HAS_MBI:e)-> Individual:i;
	Claims = SELECT c from Indiv:i -(R_SERVICES_PROVIDED_TO:e)-> MedicalClaim:c
			WHERE c.serviceToDate BETWEEN now() AND datetime_sub(now(), INTERVAL 90 DAY);

	Prv    = SELECT p from Claims:c -(SERVICES_PROVIDED_BY:e)-> Provider:p
			WHERE e.providerType IN ("attending", "rendering")
                	POST-ACCUM @@prvVisits += (p -> 1);

        FOREACH (key, val) IN @@prvVisits DO
                @@topProvidersByVisits += providerVisits(val, key);
        END;

        PRINT @@topProvidersByVisits;

        FOREACH p IN @@topProvidersByVisits DO
                @@returnTop += (p.prv -> p.visitCount);
        END;

        PRINT @@returnTop;
}

INSTALL QUERY getCareTeamForMember

```

### Example: Infer and add edges

```
CREATE QUERY insertSharedIndividualsEdge(INT offst, INT lim, DateTime sinceDate, UINT numOfDays) FOR GRAPH HealthCare
{
        MapAccum<Vertex<Individual>, SetAccum<Vertex<Provider>>> @@providers;
        MapAccum<Vertex<Provider>, MapAccum<Vertex<Provider>, INT>> @@sharedProviders;

        INT pCtr, p1Ctr;

        DATETIME toDate;
        toDate = datetime_add(sinceDate, INTERVAL numOfDays DAY);

        Start = {Individual.*};
        PRINT Start.size();

	// Get all individuals with claims in the period of interest
        Start = SELECT s FROM Start:s -(R_SERVICES_PROVIDED_TO)-> MedicalClaim:c WHERE c.serviceToDate BETWEEN sinceDate AND toDate ORDER BY getvid(s) limit offst, lim;
        PRINT Start.size();

	// Get all providers the individual interacted with in the period of interest and roll up the data at provider level
        Start = SELECT s FROM Start:s
                        POST-ACCUM @@providers += (s -> getAllProvidersForIndividual(s, sinceDate, toDate));

	// Determine Src and Tgt for the derived edge and determine the weight
        FOREACH (key, val) IN @@providers DO
                CASE WHEN (val.size() > 1) THEN
                        pCtr = 0;
                        FOREACH p IN val DO
                                pCtr += 1; p1Ctr = 0;
                                FOREACH p1 IN val DO
                                        p1Ctr += 1;
                                        IF (p1Ctr > pCtr) THEN
                                                @@sharedProviders += (p -> (p1 -> 1));
                                        END;
                                END;
                        END;
                END;
        END;

	// INSERT the edge
        FOREACH (key, val) IN @@sharedProviders DO
                FOREACH (key1, val1) IN val DO
                        insert into SHARED_INDIVIDUALS (FROM, TO, weight) VALUES(key Provider, key1 Provider, val1);
                END;
        END;
}
```
