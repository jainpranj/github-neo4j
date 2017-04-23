CYPHER QUERIES

Match(p:Packages)-[d:DEPENDS]->(n) Where p.packagename = 'django' Return n order by d.count desc limit 10


Match(p:Packages)-->(n) Where p.packagename = 'django' Return n

Match(p:Packages)-->(n) Where p.packagename = 'os' Return n

MATCH (n:Packages) RETURN n LIMIT 25

USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM "file:///github-clustering/packages_in_file_edges_py.csv" AS csvLine
MATCH (package:Packages { packagename: csvLine.package1}),(package2:Packages { packagename: csvLine.package2})
MERGE (package)-[:DEPENDS { count: csvLine.count }]->(package2)

LOAD CSV WITH HEADERS FROM "file:///github-clustering/packages_in_file_nodes.csv" AS csvLine
CREATE (p:Packages { packagename: csvLine.package, count: toInt(csvLine.count) })

CREATE INDEX ON :Packages(packagename)