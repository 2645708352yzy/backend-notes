[[Raft]]

## 总述

Raft 通过选举一个领导人，然后给予他全部的管理复制日志的责任来实现一致性。领导人从客户端接收日志条目（log entries），把日志条目复制到其他服务器上，并告诉其他的服务器什么时候可以安全地将日志条目应用到他们的状态机中。拥有一个领导人大大简化了对复制日志的管理。例如，领导人可以决定新的日志条目需要放在日志中的什么位置而不需要和其他服务器商议，并且数据都从领导人流向其他服务器。一个领导人可能会发生故障，或者和其他服务器失去连接，在这种情况下一个新的领导人会被选举出来。

通过领导人的方式，Raft 将一致性问题分解成了三个相对独立的子问题，这些问题会在接下来的子章节中进行讨论：

- **领导选举**：当现存的领导人发生故障的时候, 一个新的领导人需要被选举出来
- **日志复制**：领导人必须从客户端接收日志条目（log entries）然后复制到集群中的其他节点，并强制要求其他节点的日志和自己保持一致。
- **安全性**：如果有任何的服务器节点已经应用了一个确定的日志条目到它的状态机中，那么其他服务器节点不能在同一个日志索引位置应用一个不同的指令。


<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/image-20251122162409186.png" alt="image-20251122162409186" style="zoom: 33%;" />

<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/image-20251122162449321.png" alt="image-20251122162449321" style="zoom: 33%;" />

<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/image-20251122162511230.png" alt="image-20251122162511230" style="zoom:33%;" />

<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/image-20251122163238567.png" alt="image-20251122163238567" style="zoom:33%;" />


## Raft基础

一个 Raft 集群包含若干个服务器节点；5 个服务器节点是一个典型的例子，这允许整个系统容忍 2 个节点失效。在任何时刻，每一个服务器节点都处于这三个状态之一：领导人、跟随者或者候选人。在通常情况下，系统中只有一个领导人并且其他的节点全部都是跟随者。跟随者都是被动的：他们不会发送任何请求，只是简单的响应来自领导人或者候选人的请求。领导人处理所有的客户端请求（如果一个客户端和跟随者联系，那么跟随者会把请求重定向给领导人）。第三种状态，候选人，是用来在 选举新领导人时使用。

![img](./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/v2-537082d871c75e59f6b7556b48cee932_1440w.jpg)

Raft 算法把时间轴划分为不同任期 Term。每个任期 Term 都有自己的编号 TermId，该编号全局唯一且单调递增。如下图，每个任务的开始都** Leader Election 领导选举**。如果选举成功，则进入维持任务 Term 阶段，此时 leader 负责接收客户端请求并，负责复制日志。Leader 和所有 follower 都保持通信，如果 follower 发现通信超时，TermId 递增并发起新的选举。如果选举成功，则进入新的任期。如果选举失败，TermId 递增，然后重新发起选举直到成功。

![img](./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/v2-4b0b8661f85d76f662d742ba5b2abcb9_1440w.jpg)

## 领导人选举

集群中每个节点只能处于 Leader、Follower 和 Candidate 三种状态的一种:

**follower 从节点**：

- 节点默认是 follower；
- 如果**刚刚开始 ** 或 **和 leader 通信超时**，follower 会发起选举，变成 candidate，然后去竞选 leader；
- 如果收到其他 candidate 的竞选投票请求，按照**先来先得** & **每个任期只能投票一次** 的投票原则投票;

**candidate 候选者**：

- follower 发起选举后就变为 candidate，会向其他节点拉选票。candidate 的票会投给自己，所以不会向其他节点投票
- 如果获得超过**半数**的投票，candidate 变成 leader，然后马上和其他节点通信，表明自己的 leader 的地位；
- 如果选举超时，重新发起选举；
- 如果遇到更高任期 Term 的 leader 的通信请求，转化为 follower；

**leader 主节点**：

- 成为 leader 节点后，此时可以接受客户端的数据请求，负责日志同步；
- 如果遇到更高任期 Term 的 candidate 的通信请求，这说明 candidate 正在竞选 leader，此时之前任期的 leader 转化为 follower，且完成投票；
- 如果遇到更高任期 Term 的 leader 的通信请求，这说明已经选举成功新的 leader，此时之前任期的 leader 转化为 follower；

选举结果有三种情况：

**获取超过半数投票，赢得选举**：

- 当 Candidate 获得超过半数的投票时，代表自己赢得了选举，且转化为 leader。此时，它会马上向其他节点发送请求，从而确认自己的 leader 地位，从而阻止新一轮的选举；
- **投票原则**：当多个 Candidate 竞选 Leader 时： 
  - 一个任期内，follower 只会**投票一次票**，且投票**先来显得**；
  - Candidate 存储的日志至少要和 follower 一样新（**安全性准则**），否则拒绝投票；

**投票未超过半数，选举失败**：

- 当 Candidate 没有获得超过半数的投票时，说明多个 Candidate 竞争投票导致过于分散，或者出现了丢包现象。此时，认为当期任期选举失败，任期 TermId+1，然后发起新一轮选举；
- 上述机制可能出现多个 Candidate 竞争投票，导致每个 Candidate 一直得不到超过半数的票，最终导致无限选举投票循环；
- **投票分散问题解决：** Raft 会给每个 Candidate 在固定时间内随机确认一个超时时间（一般为 150-300ms）。这么做可以尽量避免新的一次选举出现多个 Candidate 竞争投票的现象；

**收到其他 Leader 通信请求**：

- 如果 Candidate 收到其他声称自己是 Leader 的请求的时候，通过任期 TermId 来判断是否处理；
- 如果请求的任期 TermId 不小于 Candidate 当前任期 TermId，那么 Candidate 会承认该 Leader 的合法地位并转化为 Follower；
- 否则，拒绝这次请求，并继续保持 Candidate；

简单的说，**Leader Election 领导选举** 通过若干的投票原则，保证一次选举有且仅可能最多选出一个 leader，从而解决了==脑裂问题==。



## Log Replication 日志复制

选举 leader 成功后，整个集群就可以正常对外提供服务了。Leader 接收所有客户端请求，然后转化为 log 复制命令，发送通知其他节点完成日志复制请求。每个日志复制请求包括状态机命令 & 任期号，同时还有前一个日志的任期号和日志索引。状态机命令表示客户端请求的数据操作指令，任期号表示 leader 的当前任期。

follower 收到日志复制请求的处理流程：

1. follower 会使用前一个日志的任期号和日志索引来对比自己的数据：

   - 如果相同，接收复制请求，回复 ok；

   - 否则回拒绝复制当前日志，回复 error；


2. leader 收到拒绝复制的回复后，继续发送节点日志复制请求，不过这次会带上更前面的一个日志任期号和索引；
3. 如此循环往复，直到找到一个共同的任期号&日志索引。此时 follower 从这个索引值开始复制，最终和 leader 节点日志保持一致；
4. 日志复制过程中，Leader 会无限重试直到成功。如果超过半数的节点复制日志成功，就可以任务当前数据请求达成了共识，即日志可以 commite 提交了；

综上， **Log Replication 日志复制**有两个特点：

1. 如果在不同日志中的两个条目有着相同索引和任期号，则所存储的命令是相同的，这点是由 leader 来保证的；
2. 如果在不同日志中的两个条目有着相同索引和任期号，则它们之间所有条目完全一样，这点是由日志复制的规则来保证的；

举个==例子==，最上面表示日志索引，这个是保证唯一性。每个方块代表指定任期内的数据操作，目前来看，LogIndex 1-4 的日志已经完成同步，LogIndex 5 的正在同步，LogIndex6 还未开始同步。Raft 日志提交的过程有点类似==两阶段原子提交协议 2PC==，不过和 2PC 的最大区别是，Raft 要求超过一般节点同意即可 commited，2PC 要求所有节点同意才能 commited。

<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/v2-2f3ad84935c439bf16b0018351162173_1440w.jpg" alt="img" style="zoom: 25%;" />

**日志不一致问题**：在正常情况下，leader 和 follower 的日志复制能够保证整个集群的一致性，但是遇到 leader 崩溃的时候，leader 和 follower 日志可能出现了不一致的状态，此时 follower 相比 leader 缺少部分日志。

为了解决数据不一致性，Raft 算法规定 **follower 强制复制 leader 节日的日志**，即 follower 不一致日志都会被 leader 的日志覆盖，最终 follower 和 leader 保持一致。简单的说，从前向后寻找 follower 和 leader 第一个公共 LogIndex 的位置，然后从这个位置开始，follower 强制复制 leader 的日志。但是这么做还有其他的安全性问题，所以需要引入**Safety 安全性**规则 。

## 安全性

当前的 **Leader election 领导选举** 和 **Log replication 日志复制**并不能保证 Raft 算法的安全性，在一些特殊情况下，可能导致数据不一致，所以需要引入下面安全性规则。

###  **Election Safety 选举安全性：避免脑裂问题**

选举安全性要求一个任期 Term 内只能有一个 leader，即不能出现脑裂现象，否者 raft 的日志复制原则很可能出现数据覆盖丢失的问题。Raft 算法通过规定若干投票原则来解决这个问题：

- 一个任期内，follower 只会**投票一次票**，且先来先得；
- Candidate 存储的日志至少要和 follower 一样新；
- 只有获得超过半数投票才有机会成为 leader；

### **Leader Append-Only 日志只能由 leader 添加修改**

Raft 算法规定，所有的数据请求都要交给 leader 节点处理，要求：

1. leader 只能日志追加日志，**不能覆盖日志**；
2. 只有 leader 的日志项才能被提交，follower 不能接收写请求和提交日志；
3. 只有已经提交的日志项，才能被应用到状态机中；
4. 选举时限制新 leader 日志包含所有已提交日志项；

### **Log Matching 日志匹配特性**

这点主要是为了保证日志的唯一性，要求：

1. 如果在不同日志中的两个条目有着相同索引和任期号，则所存储的命令是相同的；
2. 如果在不同日志中的两个条目有着相同索引和任期号，则它们之间所有条目完全一样；

###  **Leader Completeness 选举完备性：leader 必须具备最新提交日志**

Raft 规定：只**有拥有最新提交日志的 follower 节点才有资格成为 leader 节点。** 具体做法：candidate 竞选投票时会携带最新提交日志，follower 会用自己的日志和 candidate 做比较。

- 如果 follower 的更加新，那么拒绝这次投票；
- 否则根据前面的投票规则处理。这样就可以保证只有最新提交节点成为 leader；

因为日志提交需要超过半数的节点同意，所以针对日志同步落后的 follower（还未同步完全部日志，导致落后于其他节点）在竞选 leader 的时候，肯定拿不到超过半数的票，也只有那些完成同步的才有可能获取超过半数的票成为 leader。

日志更新判断方式是比较日志项的 term 和 index：

- 如果 TermId 不同，选择 TermId 最大的；
- 如果 TermId 相同，选择 Index 最大的；

下面举个==例子==来解释为什么需要这个原则，如下图，假如集群中 follower4 在 LogIndex3 故障宕机，经过一段时间间，任期 Term3 的 leader 接收并提交了很多日志（LogIndex1-5 已经提交，LogIndex6 正在复制中）。然后 follower4 恢复正常，在没有和 leader 完成同步日志的情况下，如果 leader 突然宕机，此时开始领导选举。再假设在 Term4 follower4 当选 leader。根据日志复制的规则，其他 follower 强制复制 leader 的日志，那么已经提交却没完成同步的日志将会被强制覆盖掉，这回导致已提交日志被覆盖。

<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/v2-b82a1fbff057248ef6f203113b28a41b_1440w.jpg" alt="img" style="zoom:25%;" />

### **State Machine Safety 状态机安全性：确保当前任期日志提交**

考虑到当前的日志复制规则

- 当前 follower 节点强制复制 leader 节点；
- 假如以前 Term 日志复制超过半数节点，在面对当前任期日志的节点比较中，很明显当前任期节点更新，有资格成为 leader；

上述两条就可能出现已有任期日志被覆盖的情况，这意味着已复制超过半数的以前任期日志被强制覆盖了，和前面提到的日志安全性矛盾。

所以，Raft 对日志提交有额外安全机制：leader 只能提交当前任期 Term 的日志，旧任期 Term（以前的数据）只能通过当前任期 Term 的数据提交来间接完成提交。简单的说，日志提交有两个条件需要满足：

1. **当前任期**；
2. **复制结点超过半数**；

下面举个例子来解释为什么需要这个原则，如下图： 

<img src="./Raft-%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0.assets/v2-7d59b070a40226665fdfec7aad4135c7_1440w.jpg" alt="img" style="zoom:25%;" />



1. 任期 Term2：
   - follower1 是 leader，此时 LogIndex3 已经复制到 follower2，且正在给 follower3 复制，此时 follower 突然宕机；
2. 任期 Term3：

   - leader 选举。follower5 发起投票，可以得到自己、follower3、follower4 的票（3/5），最终成为 leader；

   - 在任期 Term3 内，提交接收客户请求并提交 LogIndex3-5，但是暂时未复制到其他节点，然后宕机；

3. 任期 Term4：

   - leader 选举，follower1 发起选举，可以得到自己、follower2、follower3、follower4 的票（4/5），最终成为 leader；

   - 此时 follower1 将 LogIndex3 复制到 follower3，此时 LogIndex3 复制超过半数，接着在本地提交了 LogIndex4，然后宕机；

4. 任期 Term4：

   - leader 选举：follower5 发起选举，可以得到自己、follower2、follower3、follower4 的票（4/5），最终成为 leader；

   - 此时其他节点需要强制复制 follower5 的日志，那么 follower1、follower2、follower3 的日志被强制覆盖掉。即虽然 LogIndex3 被复制到了超过半数节点，但也有可能被覆盖掉；

如何解决这个问题呢？Raft 在日志项提交上增加了限制：只有 **当前任期** 且 **复制超过半数** 的日志才可以提交。即只有 LogIndex4 提交后，LogIndex3 才会被提交。

## 附录

<div style="font-family: Arial, sans-serif; margin: 20px;">
    <div style="text-align: center; background-color: #005cbf; color: white; padding: 10px; border-radius: 5px; font-size: 24px; font-weight: bold;">state</div>
    <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">所有服务器上的持久化状态：</div>
    <div style="color: #0066cc; font-style: italic;">（在响应RPC之前更新到稳定存储）</div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">currentTerm</span>
        <span style="display: inline-block;">当前看到的最大任期号（首次启动时初始化为0，单调递增）</span>
    </div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">votedFor</span>
        <span style="display: inline-block;">在当前任期内收到选票的候选人ID（如果没有则为null）</span>
    </div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">log[]</span>
        <span style="display: inline-block;">日志条目；每个条目包含状态机的命令，以及领导者接收该条目的任期（第一个索引是1）</span>
    </div>
    <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">所有服务器上的易失性状态：</div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">commitIndex</span>
        <span style="display: inline-block;">已知被提交的最高日志条目的索引（初始化为0，单调递增）</span>
    </div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">lastApplied</span>
        <span style="display: inline-block;">应用到状态机的最高日志条目的索引（初始化为0，单调递增）</span>
    </div>
    <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">领导者的易失性状态：</div>
    <div style="color: #0066cc; font-style: italic;">（选举后重新初始化）</div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">nextIndex[]</span>
        <span style="display: inline-block;">针对每台服务器，要发送给那台服务器的下一个日志条目的索引（初始化为领导者的最后日志索引+1）</span>
    </div>
    <div style="margin: 10px 0;">
        <span style="font-weight: bold; display: inline-block; width: 150px;">matchIndex[]</span>
        <span style="display: inline-block;">针对每台服务器，已知在其上复制的最高日志条目的索引（初始化为0，单调递增）</span>
    </div>
</div>

---

<div style="font-family: Arial, sans-serif; margin: 20px;">     <div style="text-align: center; background-color: #005cbf; color: white; padding: 10px; border-radius: 5px; font-size: 24px; font-weight: bold;">AppendEntries RPC</div>          <div style="color: #0066cc; margin: 10px 0;">         由领导者调用以复制日志条目；也用作心跳）。     </div>      <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">参数：</div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">term</span>         <span style="display: inline-block;">领导者的任期</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">leaderId</span>         <span style="display: inline-block;">以便跟随者重定向客户端</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">prevLogIndex</span>         <span style="display: inline-block;">紧邻新条目前一个日志条目的索引</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">prevLogTerm</span>         <span style="display: inline-block;">prevLogIndex 对应条目的任期</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">entries[]</span>         <span style="display: inline-block;">要存储的日志条目（心跳时为空；为提高效率可发送多个）</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">leaderCommit</span>         <span style="display: inline-block;">领导者的 commitIndex</span>     </div>      <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">返回结果：</div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">term</span>         <span style="display: inline-block;">当前任期，供领导者更新自身</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">success</span>         <span style="display: inline-block;">如果跟随者包含与 prevLogIndex 和 prevLogTerm 匹配的条目，则为 true</span>     </div>      <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">接收方实现：</div>     <div style="margin: 8px 0; text-align: left;">         <ol style="margin-left: 20px; list-style-type: decimal;">             <li>如果 term < currentTerm，返回 false</li>             <li>如果日志在 prevLogIndex 处没有条目，或其任期不匹配 prevLogTerm，返回 false</li>             <li>如果已有条目与新条目冲突（相同索引但不同任期），删除该条目及其之后的所有条目）</li>             <li>追加任何不在日志中的新条目</li>             <li>如果 leaderCommit > commitIndex，设置 commitIndex = min(leaderCommit, 最后一个新条目的索引)</li>         </ol>     </div> </div>

---

<div style="font-family: Arial, sans-serif; margin: 20px;">     <div style="text-align: center; background-color: #005cbf; color: white; padding: 10px; border-radius: 5px; font-size: 24px; font-weight: bold;">RequestVote RPC</div>          <div style="color: #0066cc; margin: 10px 0;">         由Candidate调用以收集选票。     </div>      <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">参数：</div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">term</span>         <span style="display: inline-block;">候选人的任期</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">candidateId</span>         <span style="display: inline-block;">请求投票的候选人 ID</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">lastLogIndex</span>         <span style="display: inline-block;">候选人最后日志条目的索引</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">lastLogTerm</span>         <span style="display: inline-block;">候选人最后日志条目的任期</span>     </div>      <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">返回结果：</div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">term</span>         <span style="display: inline-block;">当前任期，供候选人更新自身</span>     </div>     <div style="margin: 8px 0;">         <span style="font-weight: bold; display: inline-block; width: 150px;">voteGranted</span>         <span style="display: inline-block;">true 表示候选人获得投票</span>     </div>      <div style="color: #d32f2f; font-weight: bold; margin-top: 15px;">接收方实现：</div>     <div style="margin: 8px 0; text-align: left;">         <ol style="margin-left: 20px; list-style-type: decimal;">             <li>如果 term < currentTerm，返回 false</li>             <li>如果 votedFor 为空或等于 candidateId，且候选人的日志至少与接收者日志一样新，则授予投票</li>         </ol>     </div> </div>

---

<div style="font-family: Arial, sans-serif; margin: 20px;">
  <div style="text-align: center; background-color: #005cbf; color: white; padding: 10px; border-radius: 5px; font-size: 24px; font-weight: bold;">server rules</div>
  <div style="color: #d32f2f; font-weight: bold; margin-top: 20px;">所有服务器：</div>
  <div style="margin: 8px 0;">
    <span>• 若 commitIndex > lastApplied：递增 lastApplied，并将 log[lastApplied] 应用于状态机（§5.3）</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若收到 RPC 请求/响应中的任期 T > currentTerm：设 currentTerm = T，并转为跟随者（§5.1）</span>
  </div>
  <div style="color: #d32f2f; font-weight: bold; margin-top: 20px;">跟随者（§5.2）：</div>
  <div style="margin: 8px 0;">
    <span>• 响应候选者和领导者的 RPC</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若选举超时且未收到当前领导者的 AppendEntries RPC，也未向其他候选者投票，则转为候选者</span>
  </div>
  <div style="color: #d32f2f; font-weight: bold; margin-top: 20px;">候选者（§5.2）：</div>
  <div style="margin: 8px 0;">
    <span>• 转为候选者时启动选举：</span>
  </div>
  <div style="margin: 8px 0 8px 20px;">
    <span>– currentTerm 加 1</span>
  </div>
  <div style="margin: 8px 0 8px 20px;">
    <span>– 为自己投票、重置选举计时器</span>
  </div>
  <div style="margin: 8px 0 8px 20px;">
    <span>– 向所有服务器发送 RequestVote RPC</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若获得多数投票，则成为领导人</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若收到新领导人的 AppendEntries RPC，则转为跟随者</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若选举超时，则开始新一轮选举</span>
  </div>
  <div style="color: #d32f2f; font-weight: bold; margin-top: 20px;">领导人：</div>
  <div style="margin: 8px 0;">
    <span>• 当选后立即向所有服务器发送空 AppendEntries（心跳），并在空闲时周期性发送以防选举超时（§5.2）</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 收到客户端命令后，追加到本地日志，待应用至状态机后再返回响应（§5.3）</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 对每个跟随者，若其 nextIndex ≤ 本地最后日志索引，则发送从 nextIndex 开始的 AppendEntries RPC</span>
  </div>
  <div style="margin: 8px 0 8px 20px;">
    <span>– 若成功：更新该跟随者的 nextIndex 和 matchIndex（§5.3）</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若因日志不一致导致 AppendEntries 失败，则递减 nextIndex 并重试（§5.3）</span>
  </div>
  <div style="margin: 8px 0;">
    <span>• 若存在 N > commitIndex，满足：log[N].term == currentTerm 且多数 matchIndex[i] ≥ N，则设 commitIndex = N（§5.3, §5.4）</span>
  </div></div>
