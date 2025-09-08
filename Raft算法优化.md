# Raft算法优化

## Learner

Raft内部角色有两种，投票成员（Voter）和非投票成员（即Learner）

引入Learner的好处：

1. 安全的集群扩缩容：
   - 当你想给集群增加一个新节点时，如果直接作为Voter加入，它需要从Leader那里同步大量历史数据。在同步完成前，它会拖慢整个集群的提交速度（因为它无法及时响应心跳和日志）
   - 最佳实践：先将新节点以Learner身份加入，让它默默同步数据。等数据追平后，再把它提升为Voter。整个过程对集群写入性能无影响。
2. 只读节点/异地灾备：
   - 如果你只想增加一个节点来分担读请求的压力，或者在另一个数据中心建一个灾备节点，但又不希望它影响主集群的写入性能和选举（例如跨地域网络延迟高）。
   - 将这个节点设为 Learner 是完美的选择。它能保持数据同步，但不会因为网络延迟而拖慢主集群的决策。



## Pre-Vote

没有Pre-Vote时，网络分区/GC卡顿的节点会直接term++发起选举，请求到达多数派会让现任Leader发现更大任期被动下台（降级为Follower），造称不必要的领导者切换与抖动。

- 额外的RPC：RequestPreVote（不提升任期、不重置对方选举定时器）
- 候选前置流程：
  1. 候选者先广播PreVote，携带自己的lastLogTerm/lastLogIndex
  2. Follower仅在“如果现在进入选举，我可能投你”时才回PreVoteGranted（即：候选者日志不落后、本地没有看到有效Leader心跳等）
  3. 若拿到多数PreVte，节点才term++进入Candidate，发送正式RequestVote
- Follower收到PreVote时不重置计时器、不更新currentTerm，因此不会触发Leader被动下台。



