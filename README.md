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



SELECT type, repo.name, created_at,actor.login,
JSON_EXTRACT(payload, '$.action') as event, 
FROM (TABLE_DATE_RANGE([githubarchive:day.], 
TIMESTAMP('2017-04-22'), 
TIMESTAMP('2017-04-22')
)) 
WHERE type = 'PushEvent'
LIMIT 100

SELECT type, repo.name, created_at,actor.login,
JSON_EXTRACT(payload, '$.action') as event, 
FROM (TABLE_DATE_RANGE([githubarchive:day.], 
TIMESTAMP('2017-04-22'), 
TIMESTAMP('2017-04-22')
)) 
WHERE type = 'IssueCommentEvent'
LIMIT 1000


LOAD CSV WITH HEADERS FROM "file:///github-events/pushevent.csv" AS line
MERGE (u:User {name:line.actor_login}) CREATE (ev:PushEvent {time:toInt(line.created_at) }) MERGE (u)-[:DO]->(ev) 
MERGE (repo:Repository {id:toInt(line.created_at)}) 
SET repo.name=line.repo_name
MERGE (owner:User {name:line.actor_login}) 
MERGE (repo)-[:OWNED_BY]->(owner)



LOAD CSV WITH HEADERS FROM "file:///github-events/pullrequestevent.csv" AS line
MERGE (u:User {name:line.actor_login}) CREATE (ev:PullRequestEvent {time:toInt(line.created_at), event:line.event }) MERGE (u)-[:DO]->(ev) 
MERGE (repo:Repository {id:toInt(line.created_at)}) 
SET repo.name=line.repo_name
MERGE (owner:User {name:line.actor_login}) 
MERGE (repo)-[:OWNED_BY]->(owner)


LOAD CSV WITH HEADERS FROM "file:///github-events/forkevent.csv" AS line
MERGE (u:User {name:line.actor_login}) CREATE (ev:ForkEvent {time:toInt(line.created_at) }) MERGE (u)-[:DO]->(ev) 
MERGE (repo:Repository {id:toInt(line.created_at)}) 
SET repo.name=line.repo_name
MERGE (owner:User {name:line.actor_login}) 
MERGE (repo)-[:OWNED_BY]->(owner)


LOAD CSV WITH HEADERS FROM "file:///github-events/watchevent.csv" AS line
MERGE (u:User {name:line.actor_login}) CREATE (ev:WatchEvent {time:toInt(line.created_at), event:line.event }) MERGE (u)-[:DO]->(ev) 
MERGE (repo:Repository {id:toInt(line.created_at)}) 
SET repo.name=line.repo_name
MERGE (owner:User {name:line.actor_login}) 
MERGE (repo)-[:OWNED_BY]->(owner)

LOAD CSV WITH HEADERS FROM "file:///github-events/issuecomment.csv" AS line
MERGE (u:User {name:line.actor_login}) CREATE (ev:IssueCommentEvent {time:toInt(line.created_at), event:line.event }) MERGE (u)-[:DO]->(ev) 
MERGE (repo:Repository {id:toInt(line.created_at)}) 
SET repo.name=line.repo_name
MERGE (owner:User {name:line.actor_login}) 
MERGE (repo)-[:OWNED_BY]->(owner)

#With most events
MATCH (u:User)-[r:DO]->()
RETURN u.name, count(r) as events
ORDER BY events DESC
LIMIT 1


#fork Events
MATCH (u:User)-[r:DO]->(f:ForkEvent)
RETURN u.name, count(r) as forkevents
ORDER BY forkevents DESC
LIMIT 1

#maximum touches on repo
MATCH (repo:Repository)-[r]->()
RETURN repo.name, count(r) as touchs
ORDER BY touchs DESC
LIMIT 1


