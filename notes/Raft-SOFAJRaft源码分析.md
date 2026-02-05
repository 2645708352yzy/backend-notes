

[[raft]]

## 源码分析一

<img src="./Raft-SOFAJRaft%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.assets/%E4%B8%8B%E8%BD%BD.png" alt="下载" style="zoom: 33%;" />

> 整个系统围绕着Node进行。涵盖了日志管理、元数据存储、快照、状态机、日志复制等模块。Node和Node之间通过RPC进行通信。
> 系统日志模块和状态机的任务都是通过Disruptor异步去执行。

- FSMCaller主要就是将日志同步到状态机。
- LogManager，顾名思义，就是管理日志。
- MetaStorage用来存储节点的元数据信息。
- SnapshotExecutor就是快照方面的实现。

### 启动流程和节点变化源码分析

```java
// 这里让 raft RPC 和业务 RPC 使用同一个 RPC server, 通常也可以分开
final RpcServer rpcServer = RaftRpcServerFactory.createRaftRpcServer(serverId.getEndpoint());
// 注册业务处理器
CounterService counterService = new CounterServiceImpl(this);
rpcServer.registerProcessor(new GetValueRequestProcessor(counterService));
rpcServer.registerProcessor(new IncrementAndGetRequestProcessor(counterService));
// 初始化状态机
this.fsm = new CounterStateMachine();
// 设置状态机到启动参数
nodeOptions.setFsm(this.fsm);
// 设置存储路径
// 日志, 必须
nodeOptions.setLogUri(dataPath + File.separator + "log");
// 元信息, 必须
nodeOptions.setRaftMetaUri(dataPath + File.separator + "raft\_meta");
// snapshot, 可选, 一般都推荐
nodeOptions.setSnapshotUri(dataPath + File.separator + "snapshot");
// 初始化 raft group 服务框架
this.raftGroupService = new RaftGroupService(groupId, serverId, nodeOptions, rpcServer);
// 启动
this.node = this.raftGroupService.start();
```

1.首先通过工厂模式创建一个RpcServer。RaftRpcServerFactory#addRaftRequestProcessors 主要就是注册一些时间处理器。根据不同的协议处理。这个序列化协议使用的google的protobuf。每当有不同的请求过来都会调用不同的处理器方法。

2.然后创建一个一个业务类。因为这个例子比较简单，其实就是维护一个分布式自增键。所以只有两个操作，一个就是增加value大小，一个是读取该值。对应的Processor也就是调用了CounterService的这两个操作。当然这不是我们关注的重点。

3.创建业务状态机。raft提供了一个StateMachine接口，奈何他的方法太多，业务方有时候没有必要去实现。所以他提供了一个适配器。这个也就是适配器模式。是可以去学习的。

>StateMachine有很多方法。比如节点状态变化的回调。其中核心的还是onApply方法。参数是一个Iterator，可以看出是可以批量apply的。这个方法，业务方一般要去主动实现。

![下载](./Raft-SOFAJRaft%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.assets/%E4%B8%8B%E8%BD%BD-1763272149059-2.png)

4.设置NodeOptions并初始化RaftGroupService ，最后调用strat启动服务。

#### RaftGroupService#start方法

1.该节点的信息校验
2.将该节点添加到NodeManager 中。这是一个单例的实现。用于存储该进程RPC地址信息。和raft的group 信息。
3.通过工厂创建raft node并init。init rpc server。

#### NodeImpl#init方法

1.获取JRaftServiceFactory，这个工厂主要用于创建各种存储实现类。这里通过SPI机制暴露给也业务方实现。当然raft有个默认的实现。**DefaultJRaftServiceFactory**

> 具体SPI实现后面会介绍一下。其实就是JRaftServiceLoader 实现。

2.初始化配置信息。校验，ip准确，并且一个ip:port只能创建初始化一次。初始化定时线程池。初始化各种计时器。

3.初始化配置管理器，主要就是集群配置

4.初始化请求的disruptor队列

```java
this.applyDisruptor.handleEventsWith(new LogEntryAndClosureHandler());

this.applyDisruptor.setDefaultExceptionHandler(new LogExceptionHandler<Object>(getClass().getSimpleName()));
this.applyQueue = this.applyDisruptor.start();
```

这里可以看出来，jraft中大部分都是通过注册回调去执行的。利用了java8函数编程。我们也可以借鉴。LogExceptionHandler其实就是在disruptor异常回调中回调我们这注册的方法reportError。我们在编码中也可以学习LogExceptionHandler的方式，这样好处就是我们可以对于不同的逻辑注册不同的回调。并且易于理解。

> LogEntryAndClosureHandler主要逻辑就是将请求预处理。然后提交到LogMannager的队列。并且注册了回调。后面会详细解释。

5.创建初始化FSMCallerImpl。

6.初始化三连，日志存储类，元数据存储类，FSMCaller。

7.初始化BallotBox。初始化SnapshotStorage

8.初始化rpcservice，主要就是raft内部通信，比如投票请求，复制日志请求。这些主要是在ReplicatorGroupImpl 实现，依赖rpcservice。

9.初始化ReadOnlyService ，用于处理读请求。

基本上初始化工作就这么多。

初始化结束之后，当前节点为follower状态。按照论文逻辑（5.2），**follower会执行选举超时逻辑。也就是electionTimer 超时**。会调用handleElectionTimeout 方法。

```java
if (this.state != State.STATE\_FOLLOWER) {
    return;
}
if (isCurrentLeaderValid()) {
    return;
}
resetLeaderId(PeerId.emptyPeer(), new Status(RaftError.ERAFTTIMEDOUT, "Lost connection from leader %s.",
    this.leaderId));
// Judge whether to launch a election.
if (!allowLaunchElection()) {
    return;
}
doUnlock = false;
```

1.如果检测心跳超时（isCurrentLeaderValid），那么他会调用resetLeaderId，这个方法逻辑很简单。如果当前leaderId不为空，调用**fsmCaller#onStopFollowin**，如果为空调用**fsmCaller#onStartFollowing**，然后清空leaderId。

2.判断当前节点是否启动选举。这里引入了优先级的概念。每个节点可以有一个优先级，越大的越优先参与选举，本地保存了一个targetPriority。

```java
this.serverId.getPriority() >= this.targetPriority;
```

**如果下届领导人在下次选举之前都没有选出，targetPriority会衰减。**
3.如果有资格执行preVote，其实就是请求选举前先询问。避免节点分区产生的没有必要的选举。这个方法首先判断节点是否存在配置中，或者节点是否正在执行快照。如果是，直接返回。因为在执行快照的时候，节点的配置可能有已经失效。

```java
if (oldTerm != this.currTerm) {
    LOG.warn("Node {} raise term {} when get lastLogId.", getNodeId(), this.currTerm);
    return;
}
this.prevVoteCtx.init(this.conf.getConf(), this.conf.isStable() ? null : this.conf.getOldConf());
for (final PeerId peer : this.conf.listPeers()) {
    if (peer.equals(this.serverId)) {
        continue;
    }
    if (!this.rpcService.connect(peer.getEndpoint())) {
        LOG.warn("Node {} channel init failed, address={}.", getNodeId(), peer.getEndpoint());
        continue;
    }
    final OnPreVoteRpcDone done = new OnPreVoteRpcDone(peer, this.currTerm);
    done.request = RequestVoteRequest.newBuilder() //
        .setPreVote(true) // it's a pre-vote request.
        .setGroupId(this.groupId) //
        .setServerId(this.serverId.toString()) //
        .setPeerId(peer.toString()) //
        .setTerm(this.currTerm + 1) // next term
        .setLastLogIndex(lastLogId.getIndex()) //
        .setLastLogTerm(lastLogId.getTerm()) //
        .build();
    this.rpcService.preVote(peer.getEndpoint(), done.request, done);
}
this.prevVoteCtx.grant(this.serverId);
if (this.prevVoteCtx.isGranted()) {
    doUnlock = false;
    electSelf();
```

#### preVote方法：

（1）遍历配置的节点列表，先connect（其实就是ping）

（2）发送RequestVoteRequest请求。这里是带回调的请求。jraft的AbstractClientService有一个线程池用来专门执行回调的。具体逻辑在invokeWithDone中体现。

> 我们理一下这里的回调逻辑：（**handlePreVoteResponse方法**）
> 如果请求前后term不一致或者当前节点不是follower，返回。
> 如果目标term大于当前term。说明leader失效，那么需要调用stepDown。这个方法后面介绍。

判断是被请求节点否给当前节点投票，如果是调用grant方法执行投票逻辑。其实就是更新Ballot。

```java
private final Ballot voteCtx = new Ballot();
private final Ballot prevVoteCtx = new Ballot();
```

这里说一下这两个Ballot，raft只认同多半节点的意见。所以Ballot主要就是用来统计的。每次有节点响应同意的时候，都会更新。更新之后会调用他的isGranted方法判断。是否有多半节点已经同意。

> 所以每个节点返回之后都会执行这个操作。这也就是为什么preVote最后也会调用this.prevVoteCtx.grant和this.prevVoteCtx.isGranted方法。因为他要给自己预投票。

**上面我们只是看到了RequestVoteRequest的调用。我们来看看节点收到该请求如何处理。**
其实在rpc server启动的时候，已经将处理请求的processor注册上去了（RequestVoteRequestProcessor）。
**对应handle流程：**
（1）判断是否存活。没有存活，不给预投票
（2）判断当前leader是否有效，如果有效直接不给预投票
（3）如果当前term大于候选者term，并且当前节点为leader，检查replicator。不给投票因为当前节点成为leader的时候，replicator可能没有启动。导致候选者收不到心跳。才发起的预投票。
（4）term=候选者term+1，检查replicator。这个时候如果候选者日志新，会给预投票。
（5）判断日志新旧。如果新。投票，否则不预投票。
最后会返回RequestVoteResponse。

```java
return RequestVoteResponse.newBuilder() //
    .setTerm(this.currTerm) //
    .setGranted(granted) //
    .build();
```

#### electSelf方法：

如果有多半节点同意，会调用该方法。

（1）electionTimer.stop()，停止electionTimer，避免再次触发。

（2）resetleader节点。也就是停止follower。会调用fsmCaller.onStopFollowing

（3）更新当前状态为候选人，增加term。启动vote计时器。

（4）遍历所有节点发送vote请求。
后续逻辑和prevote一致，只不过pre通过了之后调用electSelf。而vote通过则成为leader。
在其他节点处理vote请求的时候，先会判断term是否大于请求的，如果大于，直接不给投票。如果小于，调用stepDown.

```java
this.metaStorage.setTermAndVotedFor(this.currTerm, this.serverId);
this.voteCtx.grant(this.serverId);
if (this.voteCtx.isGranted()) {
    becomeLeader();
}
```

#### becomeLeader方法

1.关闭vote定时器。

2.开启follower的日志复制，可能会add失败，因为如果节点不同的话。所以这就是为什么上面leader收到prevote请求之后需要check

```java
for (final PeerId peer : this.conf.listPeers()) {
    if (peer.equals(this.serverId)) {
        continue;
    }
    LOG.debug("Node {} add a replicator, term={}, peer={}.", getNodeId(), this.currTerm, peer);
    if (!this.replicatorGroup.addReplicator(peer)) {
        LOG.error("Fail to add a replicator, peer={}.", peer);
    }
}
```

3.开启learner的日志复制。

4.更新PendingIndex。因为raft指出，在新的任期不能提交上个任期没有提交的日志。根据配置更新选举优先值。

5.启动stepDownTimer定时器。

**简单说下stepDownTimer这个定时任务，它主要是监控集群健康状况**

1.检查之前首先会checkReplicator group所有节点，这个方法上面说过，其实就是判断是否开启当前节点的复制，如果没有则尝试开启。

2。调用checkDeadNodes0。这里是根据最后一次节点响应的时间戳和当前时间判断这个节点是否存活。。或者说是leader和此节点是否能进行正常网络通信。如果多半节点存活返回true。

```java
for (final PeerId peer : peers) {
    if (peer.equals(this.serverId)) {
        aliveCount++;
        continue;
    }
    if (checkReplicator) {
        checkReplicator(peer);
    }
    final long lastRpcSendTimestamp = this.replicatorGroup.getLastRpcSendTimestamp(peer);
    if (monotonicNowMs - lastRpcSendTimestamp <= leaderLeaseTimeoutMs) {
        aliveCount++;
        if (startLease > lastRpcSendTimestamp) {
            startLease = lastRpcSendTimestamp;
        }
        continue;
    }
    if (deadNodes != null) {
        deadNodes.addPeer(peer);
    }
}
if (aliveCount >= peers.size() / 2 + 1) {
    updateLastLeaderTimestamp(startLease);
    return true;
}
return false;
```

3.如果2返回false，调用stepDown。这里主要避免脑裂和信息不一致。

#### stepdown方法

1.如果当前节点为候选者，停止voteTimer。

2.如果正在执行leader转移或者为leader，停止stepDownTimer ，如果为leader，调用onLeaderStop

3.清空一些信息。状态设置为follwers。

### 总结

第一篇，主要是对架构的理解。主要有以下几点

- 整个节点变化依赖三个定时器推动。并且在变化之后启动下一个需要启动的定时器，并停止当前定时器。
- 加入prevote。在成为候选者之前，得先要通过prevote。避免无效的选举。
- 整个任务都是异步回调的方式，比如等待选举请求的时候。每次返回通过回到判断。不得不说这里的回调实现的却是不错。
- 上文主要是根据初始化流程走。分析了节点变化过程。从follower到候选者要经过pervote。从候选者到leader需要经过vote。从leader或者候选者到follower执行stepdown。并且也涉及一部分执行stepdown的时间点。

## 源码分析二

其实主要依赖Replicator、LogManager、LogStorage这三个实现。

- Replicator，leader发送日志和心跳的功能就是在此实现。每个leader都会有一个ReplicatorGroup，用来管理所有followers
- LogManager用于处理日志，主要就是消费复制或者apply的日志，将其写入磁盘。
- LogStorage主要就是日志的底层存储工作。给予RocksDB。
![下载 (1)](./Raft-SOFAJRaft%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.assets/%E4%B8%8B%E8%BD%BD%20(1).png)

### 日志同步架构源码分析

我们先从Replicator开始。首先何时创建：

当节点成为leader后，会启动所有follower和learner的replicator。其实是通过addReplicator方法实现的。

```java
for (final PeerId peer : this.conf.listPeers()) {
    if (peer.equals(this.serverId)) {
        continue;
    }
    LOG.debug("Node {} add a replicator, term={}, peer={}.", getNodeId(), this.currTerm, peer);
    if (!this.replicatorGroup.addReplicator(peer)) {
        LOG.error("Fail to add a replicator, peer={}.", peer);
    }
}
// Start learner's replicators
for (final PeerId peer : this.conf.listLearners()) {
    LOG.debug("Node {} add a learner replicator, term={}, peer={}.", getNodeId(), this.currTerm, peer);
    if (!this.replicatorGroup.addReplicator(peer, ReplicatorType.Learner)) {
        LOG.error("Fail to add a learner replicator, peer={}.", peer);
    }
}
```

#### addReplicator方法

1.从failureReplicators中移除需要add的peer，这个肯定是要执行的。可能存在failureReplicators不存在当前peer的case。

2.复制一份集群配置，然后调用Replicator.start 方法。

3.成功的话，将返回的ThreadId 加入到replicatorMap，失败加入到failureReplicators。

> Replicator是真正去完成工作的实现，ReplicatorGroup则是用来管理Replicator的实现类。
> 我们在编码中也可以这么做，理清楚每个点的意义，以及应该出现的地方，这么我们写出来的代码也是非常易于理解，并且鲁棒的。比如把所有Replicator共有的或者共同依赖对象可以放在ReplicatorGroup。

#### Replicator#start方法

1.初始化Replicator对象。

2.调用connect方法和对应节点建立连接。

3.创建ThreadId，其实是一个对Replicator对象的不可重入锁。只有获取到锁的情况Replicator才可用。其实就是维护Replicator的竞态条件。全局锁。我理解不可重入的原因就是同一线程不同操作的时候需要保证Replicator的安全。

4.执行监听回调，业务方可以实现对应监听器。

5.打开超时心跳，因为这个心跳是可动态调整的，所以并没有直接使用定时器。每次通过定时任务启动。这个方法会在每次心跳返回的时候再次调用。

```java
final long dueTime = startMs + this.options.getDynamicHeartBeatTimeoutMs();
try {
    this.heartbeatTimer = this.timerManager.schedule(() -> onTimeout(this.id), dueTime - Utils.nowMs(),
        TimeUnit.MILLISECONDS);
} catch (final Exception e) {
    LOG.error("Fail to schedule heartbeat timer", e);
    onTimeout(this.id);
}
```

onTimeout触发之后会发送一条心跳信息。如果这个心跳没有返回。那么leader不会一直发送心跳。

6.发送一条探测消息（sendEmptyEntries方法）。
到这里我们基本了解了Replicator的启动流程，其实就是初始化，启动心跳，发送探针消息。其实就是用来询问当前follower的复制偏移。

#### sendEmptyEntries方法

这个方法如果为false，代表是探测消息，如果是true代表是心跳。

![下载 (2)](./Raft-SOFAJRaft%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.assets/%E4%B8%8B%E8%BD%BD%20(2).png)

什么时候发送探测消息？

- 节点刚成为leader（start）
- 发送日志超时的时候，会发送探测消息。
- 如果响应超时，如果jraft打开pipeline，会有一个pendingResponses阈值。如果响应队列数据大于这个值会调用该方法，并不会在响应超时的时候，无限loop。
- 收到无效的日志请求。
- 发送日志请求不成功

##### 探测消息的发送逻辑

```java
this.statInfo.runningState = RunningState.APPENDING_ENTRIES;
this.statInfo.firstLogIndex = this.nextIndex;
this.statInfo.lastLogIndex = this.nextIndex - 1;
this.appendEntriesCounter++;
this.state = State.Probe;
final int stateVersion = this.version;
final int seq = getAndIncrementReqSeq();
```

其实就是用来探测各种异常情况，或者探测当前节点的nextIndex。

##### 发送心跳的逻辑

这里注意其实jraft会有两种心跳。一种就是在读leader的时候会用到。后面会说。另一种就是保活心跳

```java
if (isHeartbeat) {
    // Sending a heartbeat request
    this.heartbeatCounter++;
    RpcResponseClosure<AppendEntriesResponse> heartbeatDone;
    // Prefer passed-in closure.
    if (heartBeatClosure != null) {
        heartbeatDone = heartBeatClosure;
    } else {
        heartbeatDone = new RpcResponseClosureAdapter<AppendEntriesResponse>() {
            @Override
            public void run(final Status status) {
                onHeartbeatReturned(Replicator.this.id, status, request, getResponse(), monotonicSendTimeMs);
            }
        };
    }
    this.heartbeatInFly = this.rpcService.appendEntries(this.options.getPeerId().getEndpoint(), request,
        this.options.getElectionTimeoutMs() / 2, heartbeatDone);
}
```

**其实follower处理心跳的逻辑比较简单**。

1.根据请求的任期，进行对应的操作（更新leader或通知leader跟高的任期）

2.如果日志对不上，会返回false。并本地最后一条日志索引号。

3.如果都ok返回成功。

**心跳消息的回调。RpcResponseClosureAdapter的run逻辑**

1.如果不ok，也就是请求失败，启动下一轮心跳定时器。调用监听回调方法通知失败。更新Replicator的状态为probe

2.如果当前节点term失效，执行stepDown，也就是心跳响应，返回了更高的term

3.如果日志不匹配。发送探测请求。启动下一轮心跳定时器。

4.更新lastRpcSendTimestamp。

> 这里可以看出来，如果当前leader失效，就不会再启动心跳定时器。

上面分析了心跳和探针消息。如何处理探针消息？

我们接下来直接分析服务端如何处理AppendEntriesRequest请求。

#### AppendEntriesRequestProcessor#processRequest0

```java
if (node.getRaftOptions().isReplicatorPipeline()) {
    final String groupId = request.getGroupId();
    final String peerId = request.getPeerId();
    final int reqSequence = getAndIncrementSequence(groupId, peerId, done.getRpcCtx().getConnection());
    final Message response = service.handleAppendEntriesRequest(request, new SequenceRpcRequestClosure(done,
        reqSequence, groupId, peerId));
    if (response != null) {
        sendSequenceResponse(groupId, peerId, reqSequence, done.getRpcCtx(), response);
    }
    return null;
} else {
    return service.handleAppendEntriesRequest(request, done);
}
```

如果开启Pipeline，则会走前面if逻辑，对于Pipeline优化后面说。这里说一下，处理日志的是一个单线程的过程。因为没有io。通过队列异步解藕。

#### handleAppendEntriesRequest方法

这个方法，用于处理日志请求，心跳请求和探测请求。
因为不同的请求，都有一部分逻辑是公用的，也就是检测当前leader的合法性。

1.如果不可用，直接返回无效

2.如果请求中term小于当前term，说明leader失效。

3.检查更新当前节点的term。更新lastLeaderTimestamp

4.如果正在安装快照，那么拒绝接受该请求，返回busy。

5.如果last日志索引不一致。返回false，并返回当前lastLogIndex。

6.如果是心跳请求，或者探测请求，更新LastCommitIndex。返回相应信息。

探测请求其实就是所有follower的日志都必须跟随当前leader，如果超前，那么会多次探测，直到和leader一致。

```java
this.ballotBox.setLastCommittedIndex(
Math.min(request.getCommittedIndex(), prevLogIndex));
```

7.如果是日志消息，根据请求解析日志，这里每条日志都有一个EntryMeta，用于记录对应日志的元数据信息。解析完成后，如需校验则进行校验。

8.调用logManager的appendEntries 方法添加日志。并且注册了回调FollowerStableClosure 。

其实logManager成功append之后，回调会响应leader。后面会分析logManager的append。

我们再来看一下是如何处理响应消息的。

#### onRpcReturned方法

这个方法主要就是发送探针日志、发送正常日志、发送安装快照之后的回调。
其实首先就是对消息的预处理，比如pipeline的实现。利用了优先队列存储已响应的请求。保证消息的有序。只会按照发送顺序处理响应。如果顺序错乱则会无视所有已发送的请求。

1.如果消息版本不正确，则忽略。

2.构造响应，加入到优先队列，如果阻塞的响应过多。

3.迭代队列，如果seq不匹配，直接返回。因为raft是需要顺序处理所有响应的。

如果消息错乱那么会重置当前发送状态

```java
void resetInflights() {
    this.version++;
    this.inflights.clear();
    this.pendingResponses.clear();
    final int rs = Math.max(this.reqSeq, this.requiredNextSeq);
    this.reqSeq = this.requiredNextSeq = rs;
    releaseReader();
}
```

并在一定时间后重新发送探测消息。

4.如果有正常的消息处理，根据类型执行对应的操作。我们这里主要看处理日志消息的方法

#### onAppendEntriesReturned方法

这个方法才是真正处理消息的回调方法。上面的方法只是实现了消息的预处理。预处理失败就不会执行这个方法

1.如果请求不OK，则会阻塞一段时间后再测探测，而不是一直执行失败的请求。这个是物理链路问题

2.如果success为false，说明失败，清空发送请求的队列。更新nextIndex ，发送探测消息。这个是raft状态问题导致的失败。

3.如果成功，如果是探测消息，那么会更新r.state。
如果是日志消息成功，那么会调用commitAt方法更新提交偏移。
更新r.nextIndex。

```java
if (entriesSize > 0) {
    if (r.options.getReplicatorType().isFollower()) {
        // Only commit index when the response is from follower.
        r.options.getBallotBox().commitAt(r.nextIndex, r.nextIndex + entriesSize - 1, r.options.getPeerId());
    }
} else {
    // The request is probe request, change the state into Replicate.
    r.state = State.Replicate;
}
r.nextIndex += entriesSize;
r.hasSucceeded = true;
r.notifyOnCaughtUp(RaftError.SUCCESS.getNumber(), false);
// dummy_id is unlock in _send_entries
if (r.timeoutNowIndex > 0 && r.timeoutNowIndex < r.nextIndex) {
    r.sendTimeoutNow(false, false);
}
```

#### commitAt方法。

```java
    final long startAt = Math.max(this.pendingIndex, firstLogIndex);
    Ballot.PosHint hint = new Ballot.PosHint();
    for (long logIndex = startAt; logIndex <= lastLogIndex; logIndex++) {
        final Ballot bl = this.pendingMetaQueue.get((int) (logIndex - this.pendingIndex));
        hint = bl.grant(peer, hint);
        if (bl.isGranted()) {
            lastCommittedIndex = logIndex;
        }
    }
    if (lastCommittedIndex == 0) {
        return true;
    }

    this.pendingMetaQueue.removeFromFirst((int) (lastCommittedIndex - this.pendingIndex) + 1);
    this.pendingIndex = lastCommittedIndex + 1;
    this.lastCommittedIndex = lastCommittedIndex;
} finally {
    this.stampedLock.unlockWrite(stamp);
}
this.waiter.onCommitted(lastCommittedIndex);
```

核心代码如上，其中pendingIndex为上一次阻塞的偏移。他为lastCommittedIndex + 1。没有真正commit的ballot都会在pendingMetaQueue中存在，每次响应成功都会调用bl.grant方法。最后根据bl.isGranted结果断定是否更新
lastCommittedIndex。

最后调用this.waiter.onCommitted执行状态机提交操作。

#### LogManager#appendEntries

这个方法对于follower来说主要是处理leader同步过来的日志，其中done则是成功后响应leader。

对于leader，则是用来接受客户端的Task请求。这个done则是成功后，更新提交。

```java
public void appendEntries(final List<LogEntry> entries, 
final StableClosure done) 
```

如果是配置日志，增加到configManager ，将日志增加到对应的disruptor 队列。

```java
while (true) {
    if (tryOfferEvent(done, translator)) {
        break;
    } else {
        retryTimes++;
        if (retryTimes > APPEND_LOG_RETRY_TIMES) {
            reportError(RaftError.EBUSY.getNumber(), "LogManager is busy, disk queue overload.");
            return;
        }
        ThreadHelper.onSpinWait();
    }
}
doUnlock = false;
if (!wakeupAllWaiter(this.writeLock)) {
    notifyLastLogIndexListeners();
}
```

这里有个wakeupAllWaiter方法，如果当前节点状态为leader，这个方法其实就是通知leader继续发送日志。

> 因为上面说到过，如果探测消息成功，会去调用send发送日志给follower，但是如果此时没有日志，那么就会注册一个waiter。当有日志产生的时候，则会触发waiter去发送日志。这里就是出发的过程。

#### disruptor 的回调方法StableClosureEventHandler。

这个方法其实会调用AppendBatcher的flush方法，当然他还有一些其他的逻辑，我们不是很关注。

Flush其实就是将日志持久化到磁盘，然后回调对应日志的回调，follower的回调就是响应leader。
底层存储依赖rocksdb。

到这里，日志复制已经基本上已经全部分析完毕。

## 源码分析三

对于状态机，我们关注问题如下：

- 何时会将日志同步到状态机？
- 对于节点变化，状态机会做什么？
- 状态机为了业务解藕做了怎么样封装？
- 对于读取操作：主要就是如何做读取优化操作？

### 状态机源码分析

主要就是leader和follower将日志应用到状态机的过程。当然leader和follower应用的时机不一样，但是过程都是一样的。

我们先来看leader。leader再增加日志的时候，会有一个回调，如果成功会执行这个回调方法（具体时机为将日志添加到本地磁盘后，也就是AppendBatcher 的flush方法）。

**这个回调回执行ballotBox的commitAt方法。**

```java
@Override
public void run(final Status status) {
    if (status.isOk()) {
        NodeImpl.this.ballotBox.commitAt(this.firstLogIndex, this.firstLogIndex + this.nEntries - 1,
            NodeImpl.this.serverId);
    } else {
        LOG.error("Node {} append [{}, {}] failed, status={}.", getNodeId(), this.firstLogIndex,
            this.firstLogIndex + this.nEntries - 1, status);
    }
}
```

#### commitAt方法

这里说一下，ballotBox主要记录了日志提交的状态。每个节点提交日志成功后都会调用这个方法。

上面只说了leader成功，如果follower提交成功，则会以响应的形式告诉leader。在onAppendEntriesReturned 中也会调用该方法。如下图。

![下载 (4)](./Raft-SOFAJRaft%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.assets/%E4%B8%8B%E8%BD%BD%20(4)-1763275231401-8.png)

```java
final long startAt = Math.max(this.pendingIndex, firstLogIndex);
Ballot.PosHint hint = new Ballot.PosHint();
for (long logIndex = startAt; logIndex <= lastLogIndex; logIndex++) {
    final Ballot bl = this.pendingMetaQueue.get((int) (logIndex - this.pendingIndex));
    hint = bl.grant(peer, hint);
    if (bl.isGranted()) {
        lastCommittedIndex = logIndex;
    }
}
if (lastCommittedIndex == 0) {
    return true;
}
this.pendingMetaQueue.removeFromFirst((int) (lastCommittedIndex - this.pendingIndex) + 1);
LOG.debug("Committed log fromIndex={}, toIndex={}.", this.pendingIndex, lastCommittedIndex);
this.pendingIndex = lastCommittedIndex + 1;
this.lastCommittedIndex = lastCommittedIndex;
...
this.waiter.onCommitted(lastCommittedIndex);
```

- pendingIndex：当前已经ok的日志索引+1（何为ok，就是大多数节点都持久化的）
- firstLogIndex：本次提交成功日志的起始值。
- lastLogIndex：本次提交成功日志的终止值。

> 为什么这里startAt为max，因为这个有很大的可能pendingIndex比firstLogIndex大，原因是这个节点响应比较慢。在他响应之前Ballot的isGranted已经返回true了。

这样的话，我们能理解这个方法其实就是用来维护this.lastCommittedIndex这个成员变量。最后他会调用this.waiter.onCommitted方法。

#### onCommitted方法

其实这个方法就是commit到状态机的入口。
当然这个方法也会在两处被调用。一处是被leader，一处是被follower调用。

```java
@Override
public boolean onCommitted(final long committedIndex) {
    return enqueueTask((task, sequence) -> {
        task.type = TaskType.COMMITTED;
        task.committedIndex = committedIndex;
    });
}
```

这个方法逻辑比较简单，就是创建一个commit时间丢到FSMCallerImpl 的队列。
我们顺藤摸瓜看看follower何时调用这个onCommitted方法。

#### FollowerStableClosure#run

1.在follower增加日志成功之后，有执行**FollowerStableClosure** 回调，上篇文章说过，他就是用来响应leader。当然在响应之前回执行node.ballotBox.setLastCommittedIndex方法。其实这个方法最后会调用onCommitted方法。

2.在follower处理心跳或者探针消息的时候。也会调用**setLastCommittedIndex**方法。ok，到这里我们已经了解了。follower和leader何时会给FSMCallerImpl 的队列提交commit事件。接下来我们只需要关注如何处理事件了。

```java
if (entriesCount == 0) {
    // heartbeat
    final AppendEntriesResponse.Builder respBuilder = AppendEntriesResponse.newBuilder() //
        .setSuccess(true) //
        .setTerm(this.currTerm) //
        .setLastLogIndex(this.logManager.getLastLogIndex());
    doUnlock = false;
    this.writeLock.unlock();
    // see the comments at FollowerStableClosure#run()
    this.ballotBox.setLastCommittedIndex(Math.min(request.getCommittedIndex(), prevLogIndex));
    return respBuilder.build();
}
```

## SOFAJRaft Snapshot 源码简析
