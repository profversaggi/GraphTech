# Algoritms, Graph traversals
docs [here](https://docs.tigergraph.com/graph-algorithm-library)
[tiger graph open source library](https://github.com/tigergraph/ecosys/tree/master/graph_algorithms)

## Types
* Path finding, Search
* Centrality
* Community detection
* Similarity

### Path findingi \ Analysis, Search
* These algorithms help find the shortest path or evaluate the availability and quality of routes. 
* The two foundational traversal algorithms are Breadth first search and Dept first search.

| Algorithm | Purpose | Example |
| --- | --- | --- | 
| Single-Pair Shortest Path | Find shortest path between the source and target vertex| How are two individuals related? Estimate influence A has on B. |
| Single-Source shortest path | Find shortest path between the source and all other vertices in the entire graph | Find the lowest cost routing. |
| All pairs shortest path | Find shortest paths between every pair of vertices in the entire graph | |
| Minimum Spanning Tree | Given an undirected and connected graph, a minimum spanning tree is a set of edges which can connect all the vertices in the graph with the minimal sum of edge weight. |  Create a low cost tour plan for a set of destinations|
| Cycle detection | Find all loops in the graph | |

### Centrality
* These algorithms determine the importance of each vertex within a network.

| Algorithm | Purpose | Example |
| --- | --- | --- |
| PageRank | Measures the influence of each vertex on every other vertex (considers connections, their connections’ connections, and the weights). | Determine the most influential person. |
| Closeness centrality | Provides a precise measure of how "centrally located" is a vertex (based on their ‘closeness’ to all other nodes) | find the individuals who are best placed to influence the entire network most quickly. Find the best location for maximum accessibility. |
| Betweenness centrality | Measures the number of times a vertex lies on the shortest path between others. | find the individuals who influence the flow around a system. Find bottlenecks. |
| Degree centrality | Assigns an importance score based purely on the number of relationships associated with every vertex. | find most connected individuals |

### Community detection
These algorithms evaluate how a group is clustered or partitioned, as well as its tendency to strengthen or break apart.

| Algorithm | Purpose | Example |
| --- | --- | --- |
| Label propagation | This is a heuristic method for determining communities |  |
| Louvain Method | This partitions the vertices in a graph by approximately maximizing the graph's modularity score. |  |

### Similarity
These algorithms measure similarity between two vertices based on the features on the vertices or the relationships.

| Algorithm | Purpose | Example |
| --- | --- | --- |
| Cosine Similarity | This calculates the cosines similarity score based on the specified feature vector. Calculates the cosine of the angle between the two real-valued vectors. | Find people with conditions similar to me. |
| Jaccard Similarity | This calculates the Jaccard index based on the specified feature vector. Compares two binary vectors (sets). | Find people with conditions similar to me. |

Similarity is the measure of how much alike two data objects are. It is usually described as a distance with dimensions representing features of the objects. If this distance is small, there will be a high degree of similarity; if a distance is large, there will be a low degree of similarity.

Similarity on graph: Similarity between 2 vertices can be calculated by comparing
* Attributes
* Degree of relationships
* Features derived based on the degree of relationships or based on the attributes on the connected nodes (based on aggregation or presence of indicators or etc...)

Sample patient similarity:
```
USE GRAPH HealthCare

DROP QUERY findSimilarMembersCosineSimilarity

CREATE DISTRIBUTED QUERY findSimilarMembersCosineSimilarity(Vertex<Individual> member, int k) FOR GRAPH HealthCare {
    MapAccum<string, float> @feature_map;
    MapAccum<string, float> @@source_map;

    SumAccum<float> @score;
    SumAccum<float> @@norm1;

    DATETIME startDate, endDate;
    startDate = now();
    endDate = datetime_sub(now(), INTERVAL 90 DAY);

    Start = {Individual.*};

    //Build feature map
    // Age, Gender and intialize the feature map
    S = SELECT s FROM Start:s
          accum
            IF s.gender == "MALE" THEN
              s.@feature_map += ("gender" ->  0)
            ELSE
              s.@feature_map += ("gender" ->  1)
            END,
            s.@feature_map += ("age" -> str_to_int(s.age)), //TODO: calculate AGE
            s.@feature_map += ("callCount" ->  0),
            s.@feature_map += ("medClaimCount" ->  0),
            s.@feature_map += ("medClaimSpend" ->  0),
            s.@feature_map += ("rxClaimCount" ->  0),
            s.@feature_map += ("rxClaimSpend" ->  0),
            s.@feature_map += ("bhClaimCount" ->  0),
            s.@feature_map += ("bhClaimSpend" ->  0);

    // Number of calls in the last 90 days that resulted in requests
    S = SELECT s FROM Start:s -(MADE_CALL)-> PhoneCall:c
          WHERE c.callStartTime > endDate
          ACCUM
            s.@feature_map += ("callCount" ->  1);

    // Number of claims in the last 90 days
    // Amount of medical claim spend in the last 90 days
    S = SELECT s FROM Start:s -(R_SERVICES_PROVIDED_TO)-> MedicalClaim:c
          WHERE c.serviceFromDate > endDate
          ACCUM
            s.@feature_map += ("medClaimCount" ->  1),
            s.@feature_map += ("medClaimSpend" ->  c.allowedAmount);

    // Number of prescriptions in the last 90 days
    // Amount of Rx spend in the last 90 days
    S = SELECT s FROM Start:s -(R_PRESCRIBED_TO)-> RxClaim:c
          WHERE c.serviceDate > endDate
          ACCUM
            s.@feature_map += ("rxClaimCount" ->  1),
            s.@feature_map += ("rxClaimSpend" ->  c.totalPaidAmount);

    // Number of OBH claims in the last 90 days
    // Amount of OBH claims in the last 90 days
    S = SELECT s FROM Start:s -(R_BH_SERVICES_PROVIDED_TO)-> BehavioralClaim:c
          WHERE c.serviceFromDate > endDate
          ACCUM
            s.@feature_map += ("bhClaimCount" ->  1),
            s.@feature_map += ("bhClaimSpend" ->  c.allowedAmount);

    Source = select s from Source:s
              post-accum
                @@source_map += s.@feature_map,
                foreach (key, value) in s.@feature_map do
                  @@norm1 += value * value
                end;
    print Source;

    Start = select s from Start:s
              where s != member
              post-accum
                float norm2 = 0,
                float numerator = 0,
                foreach (feature, value) in s.@feature_map do
                  numerator = numerator + @@source_map.get(feature) * value,
                  norm2 = norm2 + value * value
                end,
                if @@norm1 > 0 and norm2 > 0 then
                  s.@score = numerator/(sqrt(@@norm1)* sqrt(norm2))
                  //,log(true,"[redrain]#1", s.name, numerator, @@norm1, norm2, numerator/(sqrt(@@norm1)* sqrt(norm2)))
                end
                order by s.@score DESC
                limit k;

    print Start;
}

INSTALL QUERY findSimilarMembersCosineSimilarity
```
