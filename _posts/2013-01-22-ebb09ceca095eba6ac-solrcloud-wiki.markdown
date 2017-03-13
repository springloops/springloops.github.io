---
author: springloops
comments: true
date: 2013-01-22 03:50:22+00:00
layout: post
link: https://springloops.wordpress.com/2013/01/22/%eb%b0%9c%ec%a0%95%eb%a6%ac-solrcloud-wiki/
slug: '%eb%b0%9c%ec%a0%95%eb%a6%ac-solrcloud-wiki'
title: '[solrcloud] Solr Wiki - NewSolrColudDesign 정리'
wordpress_id: 83
categories:
- Open Source/Apache Lucene/Solr
tags:
- 태그를 입력해 주세요.
- solr
- solr4.0
- solrcloud
---

SolrCloud 를 구성해보기 위해 Wiki 페이지의 New SolrCloud Design 페이지를 발정리 해봅니다.

[http://wiki.apache.org/solr/NewSolrCloudDesign](http://wiki.apache.org/solr/NewSolrCloudDesign)

  


**어휘**

  * **Cluster** : 단일 solr 노드의 집합체. 전체 Cluster 는 단일 스키마와 solrconfig 를 가져야 한다.
  * **Node** : Solr를 실행하는 JVM (하나의 물리적 서버에 다수의 Node 설치 가능)
  * **Partition** : 전체 Document 의 하위 집합. Partition은 모든 Document 들을 하나의 인덱스에 포함할수 있는 방식으로 생성된다.? -> shard(**sharding)**
  * **Shard** : Partition 은 복제 요소에 (replication factor) 지정된 여러 노드에 저장되어야 한다. 모든 노드는 집합적 조각을 형성한다. 노드는 여러 shard의 부분이 될수 있다?  -> **replica(replication)**

(partition 과 shard 의 의미에 대해 혼란이 있어 조금 더 찾아 봤고 조금 아래 따로 정리 해본다)

  * Leader : 각 Shard는 리더로 식별된 하나의 노드를 갖는다. 모든 문서를 위한 쓰기는 파티션에 귀속되고 리더를 통해 라우팅되어진다.
  * Replication Factor : 클러스터에의해 관리되는 문서의 최소 복제계수.
  * Transaction Log : 각 노드에 의해 관리되는 쓰기 명령에 대한 추가만 가능한(append-only) 로그.
  * Partition version : 각 리더의 각 조각과 증가되는 각 쓰기 명령 그리고 동료에게 전송된 데이터의 관리 카운터.
  * Cluster Lock : range -> partition 또는 partition -> node 매핑을 변경하기 위해 획득해야하는 글로벌 락

  


**원칙**

  * 어느 명령이든 클러스터의 어떤 노드에서든 실행할수 있다.
  * 단일 실패점(single-point failure)이 없다.
  * 클러스터는 탄력(scalel up/scale down)있어야 한다.
  * 기록이 손실되지 않아야 한다. 내구성이 보장.
  * 쓰기(write)의 순서는 보장되어야 한다.
  * 다른 클라이언트에서 같은 요청이 다른 복제 노드로 같은 시간에 요청이 되어도 항상 하나의 결과로 응답해야한다.
  * 클러스터의 구성은 중앙관리 되고, 클러스터의 어떤 노드를 통해서든 업데이트 할 수 있다.
  * 노드의 로컬에 동일 포트에 다른 구성이 없고(노드의 포트당 1개의 구성만 가능), 노드 마다 index/logs 저장 위치가 필요 하다.
  * 읽기에 대해 자동 장애 조치
  * 쓰기에 대해 자동 장애 조치
  * 노드의 장애 이벤트 발생시, 자동으로 복제 계수를 인수한다. ? (hint hand off 기능?)

  


**Zookeeper**

  


Zookeeper 클러스터는 다음과 같이 사용된다.

  * Solr클러스터를 위한 중앙 설정 저장소.
  * 분산된 동기화(=Global Lock)가 필요한 명령에 대한 코디네이터.
  * Solr클러스터 토폴로지에 대한 시스템의 기록(=클러스터 멤버십 서비스 ex: 서버가 다운 되었을 경우 해당 노드를 제거, 요청이 가지 않도록 처리)

  


**Partitioning**

클러스터는 고정된 max_hash_value 'N'으로 구성된다. (상당히 큰 값 예 1000)

각 문서의 해시는 다음과 같이 계산된다.

hash = hash_function(doc.getKey()) % N

해시 값의 범위는 파티션에 할당되고,  Zookeeper에 저장된다.

예를 들어 다음과 같이 파티션에 매핑 범위를 가지고 있을수 있다.

range  : partition

---------- ---------

0 - 99  : 1

100 - 199 : 2

200 - 299 : 3

  


해시는 문서의 인덱스된 필드에 불변값(immutable)으로 추가 된다.

또한 해시는 인덱스 분리에 사용할 수 있다.

  


해시 함수는 플러그다. 해시 함수는 문서를 받아 문서에 대한 일관된 정수 해시 값을 반환 합니다.

시스템은 기본 해시 함수를 제공하고, 기본 해시 함수는 필수의 불변한 필드 (기본 unique_key 필드)로 해시 값을 계산한다.

  


### Using full hash range

Alternatively, there need not be any max_hash_value - the full 32 bits of the hash can be used since each shard will have a range of hash values anyway. Avoiding a configurable max_hash_value makes things easier on clients wanting related hash values next to each other. For example, in an email search application, one could construct a hashcode as follows:
    
    <span style="font-size:11pt;font-family:Arial;color:rgb(0,0,0);">(hash(user_id)<<24) | (hash(message_id)>>>8)</span>

By deriving the top 8 bits of the hashcode from the user_id, it guarantees that any users emails are in the same 256th portion of the cluster. At search time, this information can be used to only query that portion of the cluster.

We probably also want to view the hash space as a ring (as is done with consistent hashing) in order to express ranges that wrap (cross from the maximum value to the minimum value).

  


shard 이름

  


해시 코드로 파티셔닝 할때, 별도의 shard 해시 범위의 매핑을 관리 하는 것 보다 shard 이름이 해시 범위가 될 수 있다.

(에: shard "1-1000" 은 docs 의 1과 1000 사이 해시코드를 포함한다)

  


현재 solr-core 이름에 대한 관습은 collection 이름과 맞추는 것이다.

(*위 파티셔닝 문단에서 potition의 값을 1,2,3 이 아닌 range 처럼 그대로 이름으로 쓰라는 말인듯...)

같은 collection에 대해 여러 개의 코어가 있는 경우 우리는 여전히 좋은 명명 체꼐가 필요하다.

  


여기 까지 읽어 본 바로..

위 어휘 단에서 설명하는 partition은 일반적 분산시스템에서의 shard(sharding) 을 말하는 것으로 보인다.

  


추가 정보 : [http://www.ngdata.com/site/blog/57-ng.html](http://www.ngdata.com/site/blog/57-ng.html) (partition으로 find..)

  


예는 반대로 얘기 하는데... ㅡㅡ;;

해시 함수로 저장될 문서의 위치를 결정 하는것.. 이것은 shard (sharding) 가 맞다.

  


**Shard 할당**

  


node -> partition 매핑은 오직 Zookeeper 의 Cluster Lock 을 획득한 노드만 변경할 수 있다.

(거듭 말하지만 여기서 Cluster Lock 이란 주키퍼의 Global Lock 기능구현을 말하는 듯. 그렇겠지?)

그래서 노드는

  1. Cluster Lock을 획득하려고 시도하고,
  2. Lock을 획득하기 위해 기다리고,
  3. Lock을 획득하면 매핑을 구독할수 있고, 파티션을 식별한다.

노드 Shard 할당

  


새 노드를 찾으려 하는 노드는 먼저 클러스터 잠금을 획득해야 한다.

모든 리더의 첫 번째 샤드에 대해 식별된다. 모든 사용가능한 노드에서, 샤드들 중 최소한의 노드가 선택된다.

(먼말인지 모르곘다;;;;; 원본 붙임. 누가 답글로 좀....)

### Node to a shard assignment

The node which is trying to find a new node should acquire the cluster lock first. First of all the leader is identified for the shard. Out of the all the available nodes, the node with the least number of shards is selected. If there is a tie, the node which is a leader to the least number of shard is chosen. If there is a tie, a random node is chosen.

  


  


**Boot strapping**

  


Cluster startup

  


노드는 Zookeeper host, port를 가리키며 시작된다.

Solr 클러스터의 첫번째 노드는 Solr 클러스터의 구성 프로퍼티와 schema, config 파일과 시작된다.

첫번째 노드는 Zookeeper 와 bootstrap 클러스터에 Solr 클러스터의 구성을 업로드 한다.

클러스터는 "bootstrap" 상태로 간주된다.

"bootstrap" 상태안에서, node -> partition 매핑은 계산되지 않고, 클러스터는 클러스터 어드민 명령어(clusteradmin) 이외에 어떤 읽기/쓰기 요청도 받아들이지 않는다.

  


클러스터안에 노드들의 초기 설정이 시작되어진 이후, 관리자로 부터 clusteradmin 명령어를 실행한다.

이 명령어는 정수의 "partitions" 인자를 허용하고, 다음 단계로 수행된다.

  1. Cluster Lock 획득
  2. partitions의 "Partitions" 숫자를 할당
  3. 각 파티션을 위한 노드들을 획득
  4. Zookeeper의 node -> partition 매핑을 업데이트한다.
  5. Cluster Lock 해제.
  6. 모든 노드의  node -> partition 매핑을 Zookeeper로 부터 강제로 업데이트 하도록 알림.

Node Startup

  


노드는 시작시, 자신이 기존 Shard의 일부인 경우, Zookeeper를 확인한다.

  


Zookeeper에 노드의 기록이 없거나, 노드가 shard의 일부가 아닐경우, 다음 New Node 단계를 수행하고,

Zookeeper에 노드의 기록이 있음으로, 기존 shard의 일부인 경우, 다음 Node Restart 단계를 수행한다.

  


New Node

  


새로운 노드는 클러스터의 일부가 아니고 클러스터의 용량을 늘리기 위해 추가 되었습니다.

만약 "auto_add_new_nodes" 클러스터 속성이 false 이면, 새로운 노드는 스스로 Zookeeper에 "idle" 상태로 등록하고 다른 노드가 참여하도록 요청할 때 까지 기다린다.

그렇지 않으면 "auto_add_new_nodes" 가 true 이면 다음 을 수행한다.

  


  1. Cluster Lock 획득
  2. 적절한 소스 (node, partition) 튜플은 선택된다:
    1. 검색 노드의 "replication_factor" 값보다 작은 파티션을 찾아 검색되어진 사용가능한 파티션 리스트 동점일경우 마지막 선택된 노드. 여러개 일 경우 랜덤으로 선택
    2. 만약 모든 파티션에 충분한 복제품이 있는 경우, If all partitions have enough replicas, the nodes are scanned to find one which has most number of partitions. In case of tie, of all the partitions in such nodes, the one which has the most number of documents is chosen. In case of tie, a random partition on a random node is chosen.
    3. f moving the chosen (node, partition) tuple to the current node will decrease the maximum number of partition:node ratio of the cluster, the chosen tuple is returned.Otherwise, no (node, partition) is chosen and the algorithm terminates
    4. The node -> partition mapping is updated in [ZooKeeper](http://wiki.apache.org/solr/ZooKeeper)
  3. Zookeeper 안의 노드의 상태는 "recovery" 상태로 업데이트 된다.
  4. Cluster Lock 해제.
  5. "recovery" 는 선택된 파티션의 리더로 다시 시작된다.
  6. 복구가 완료 되면, Cluster Lock을 다시 획득
  7. 소스(node, patition)은 Zookeeper의 node -> patition 맵에서 삭제 된다.
  8. Cluster Lock 해제.
  9. 소스 노드는 Zookeeper로 부터 node -> patition 맵을 강제 리프레시 명령한다.
  10. 다시 step 1

  


Node Restart

  


A node restart can mean one of the following things:

  * The JVM crashed and was manually or automatically restarted
  * The node was in a temporary network partition and either could not reach [ZooKeeper](http://wiki.apache.org/solr/ZooKeeper) (and was supposed to be dead) or could not receive updates from the leader for a period of time. A node restart ine node failure.

  * Lifecycle of a Write Operation this scenario signifies the removal of the network partition.
  * A hardware failure or maintenance window caused the removal of the node from the cluster and the node has been started again to rejoin the cluster.

The node reads the list of partitions for which it is a member and for each partition, starts a recovery process from each partition’s leader respectively. Then, the node follows the steps in the New Node section without checking for the auto_add_new_nodes property. This ensures that the cluster recovers from the imbalance created by th

Writes are performed by clients using the standard Solr update formats. A write operation can be sent to any node in the cluster. The node uses the hash_function , and the Range-Partition mapping to identify the partition where the doc belongs to. A zookeeper lookup is performed to identify the leader of the shard and the operation is forwarded there. A SolrJ enhancement may enable it to send the write directly to the leader

The leader assigns the operation a Partition Version and writes the operation to its transaction log and forwards the document + version + hash to other nodes belonging to the shard. The nodes write the document + hash to the index and record the operation in the transaction log. The leader responds with an ‘OK’ if at least min_writes number of nodes respond with ‘OK’. The min_writes in the cluster properties can be overridden by specifying it in the write request.

The cloud mode would not offer any explicit commit/rollback operations. The commits are managed by auto-commits at intervals (commit_within) by the leader and triggers a commit on all members on the shard. The latest version available to a node is recorded with the commit point.

  


  


## Transaction Log

  * A transaction log records all operations performed on an Index between two commits
  * Each commit starts a new transaction log because a commit guarantees durability of operations performed before it
  * The sync can be tunable e.g. flush vs fsync by default can protect against JVM crashes but not against power failure and can be much faster

## Recovery

A recovery can be triggered during:

  * Bootstrap
  * Partition splits
  * Cluster re-balancing

The node starts by setting its status as ‘recovering’. During this phase, the node will not receive any read requests but it will receive all new write requests which shall be written to a separate transaction log. The node looks up the version of index it has and queries the ‘leader’ for the latest version of the partition. The leader responds with the set of operations to be performed before the node can be in sync with the rest of the nodes in the shard.

This may involve copying the index first and replaying the transaction log depending on where the node is w.r.t the state of the art. If an index copy is required, the index files are replicated first to the local index and then the transaction logs are replayed. The replay of transaction log is nothing but a stream of regular write requests. During this time, the node may have accumulated new writes, which should then be played back on the index. The moment the node catches up with the latest commit point, it marks itself as “ready”. At this point, read requests can be handled by the node.

### Handling Node Failures

There may be temporary network partitions between some nodes or between a node and [ZooKeeper](http://wiki.apache.org/solr/ZooKeeper). The cluster should wait for some time before re-balancing data.

#### Leader failure

If node fails and if it is a leader of any of the shards, the other members will initiate a leader election process. Writes to this partition are not accepted until the new leader is elected. Then it follows the steps in non-leader failure

#### Non-Leader failure

The leader would wait for the min_reaction_time before identifying a new node to be a part of the shard. The leader acquires the Cluster Lock and uses the node-shard assignment algorithm to identify a node as the new member of the shard. The node -> partition mapping is updated in [ZooKeeper](http://wiki.apache.org/solr/ZooKeeper) and the cluster lock is released. The new node is then instructed to force reload the node -> partition mapping from[ZooKeeper](http://wiki.apache.org/solr/ZooKeeper).

## Splitting partitions

A partition can be split either by an explicit cluster admin command or automatically by splitting strategies provided by Solr. An explicit split command may give specify target partition(s) for split.

Assume the partition ‘X’ with hash range 100 - 199 is identified to be split into X (100 - 149) and a new partition Y (150 - 199). The leader of X records the split action in [ZooKeeper](http://wiki.apache.org/solr/ZooKeeper) with the new desired range values of X as well as Y. No nodes are notified of this split action or the existence of the new partition.

  1. The leader of X, acquires the Cluster Lock and identifies nodes which can be assigned to partition Y (algorithm TBD) and informs them of the new partition and updates the partition -> node mapping. The leader of X waits for the nodes to respond and once it determines that the new partition is ready to accept commands, it proceeds as follows:

  2. The leader of X suspends all commits until the split is complete.
  3. The leader of X opens an [IndexReader](http://wiki.apache.org/solr/IndexReader) on the latest commit point (say version V) and instructs its peers to do the same.

  4. The leader of X starts streaming the transaction log after version V for the hash range 150 - 199 to the leader of Y.
  5. The leader of Y records the requests sent in #2 in its transaction log only i.e. it is not played on the index.
  6. The leader of X initiates an index split on the [IndexReader](http://wiki.apache.org/solr/IndexReader) opened in step #2.

  7. The index created in #5 is sent to the leader of Y and is installed.
  8. The leader of Y instructs its peers to start recovery process. At the same time, it starts playing its transaction log on the index.
    1. Once all peers of partition Y have reached at least version V:
    2. The leader of Y asks the leader of X to prepare a [FilteredIndexReader](http://wiki.apache.org/solr/FilteredIndexReader) on top of the reader created in step #2 which will have documents belonging to hash range 100 - 149 only.

    3. Once the leader of X acknowledges the completion of request in #8a, the leader of Y acquires the Cluster Lock and modifies the range -> partition mapping to start receiving regular search/write requests from the whole cluster.

    4. The leader of Y asks leader of X to start using the [FilteredIndexReader](http://wiki.apache.org/solr/FilteredIndexReader) created in #8a for search requests.

    5. The leader of Y asks leader of X to force refresh the range -> partition mapping from [ZooKeeper](http://wiki.apache.org/solr/ZooKeeper). At this point, it is guaranteed that the transaction log streaming which started in #3 will be stopped.

  9. The leader of X will delete all documents with hash values not belonging to its partitions, commits and re-opens the searcher on the latest commit point.
  10. At this point, the split is considered complete and leader of X resumes commits according to the commit_within parameters.

Notes:

  * The partition being split does not honor commit_within parameter until the split operation completes
  * Any distributed search operation performed starting at the time of #8b and till the end of #8c can return inconsistent results i.e. the number of search results may be wrong.

## Cluster Re-balancing

The cluster can be rebalanced by an explicit cluster admin command.

TBD

## Monitoring

TBD

## Configuration

### solr_cluster.properties

This are the set of properties which are outside of the regular Solr configuration and is applicable across all nodes in the cluster:

  * **replication_factor** : The number of replicas of a doc maintained by the cluster

  * **min_writes** : Minimum no:of successful writes before the write operation is signaled as successful . This may me overridden on a per write basis

  * **commit_within** : This is the max time within which write operation is visible in a search

  * **hash_function** : The implementation which computes the hash of a given doc

  * **max_hash_value** : The maximum value that a hash_function can output. Theoretically, this is also the maximum number of partitions the cluster can ever have

  * **min_reaction_time** : The time before any reallocation/splitting is done after a node comes up or goes down (in secs)

  * **min_replica_for_reaction** : If the number of replica nodes go below this threshold the splitting is triggered even if the min_reaction_time is not met

  * **auto_add_new_nodes** : A Boolean flag. If true, new nodes are automatically used as read replicas to existing partitions, otherwise, new nodes sit idle until the cluster needs them.

## Cluster Admin Commands

All cluster admin commands run on all nodes at a given path (say /cluster_admin). All nodes are capable of accepting the same commands and the behavior would be same. These are the public commands which a user can use to manage a cluster:

  * **init_cluster** : (params : partition) This command is issued after the initial set of nodes are started. Till this command is issued, the cluster would not accept any read/write commands

  * **split_partition** : (params : partitionoptional). The partition is split into two halves. If the partition parameter is not supplied, the partition with the largest number of documents is identified as the candidate.

  * **add_idle_nodes** : This can be used if auto_add_new_nodes=false. This command triggers the addition of all ‘idle’ nodes to the cluster.

  * **move_partition** : (params : partition, from, to). Move the given partition from a given node from to another node

  * **command_status** :(params : completion_idoptional) . All the above commands are asynchronous and returns with a completion_id . This command can be used to know the status of a particular running command or all the current running commands

  * **status** : (params : partitionoptional) Shows the list of partitions and for each partition, the following info is provided

    * leader node
    * nodes list
    * doc count
    * average reads/sec
    * average writes/sec
    * average time/read
    * average time/write

## Migrating from Solr to SolrCloud

A few features may be redundant or not supported when we move to cloud such as. We should continue to support the non cloud version which supports all the existing features

  * Replication. This feature is not required anymore
  * [CoreAdmin](http://wiki.apache.org/solr/CoreAdmin) commands. Explicit manipulation of cores will not be allowed. Though cores may exist internally and they meay be managed implicitly

  * Multiple schema support ? Should we just remove it from ver 1.0 for simplicity?
  * solr.xml . Is there a need at all for this in the cloud mode?

## Alternative to a Cluster Lock

It may be simpler to have a coordinator node (we avoid the term master since that is associated with traditional index replication) that is established via leader election. Following a pattern of treating the zookeeper state as the "truth" and having nodes react to changes in that state allow for more future flexibility (such as allowing an external management tool directly change the zookeeper state to control the cluster). Having a coordinator (as opposed to grabbing a lock every time) can be more scalable too. A hybrid model where a cluster lock is used only in certain circumstances can also make sense.

## Single Node Simplest Use Case

We should be able to easily start up a single node and start indexing documents. At a later point in time, we should be able to start up a second node and have it join the cluster.

  1. start up a single node, upload it's configuration (the first time) to zookeeper, and create+assign the node to shard1.
    * in the absence of other information when the config is created, a single shard system is assumed
  2. index some documents
  3. start up another node and pass it a parameter that says "if you are not already assigned, assign yourself to any shard that has the lowest number of replicas and start recovery process"
    * avoid replicating a shard on the same host if possible
    * after this point, one should be able to kill the node and start it up again and have it resume the same role (since it should see itself in zookeeper)

  

