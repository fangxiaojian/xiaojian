微服务里面

1. 服务通信
2. 服务注册 （zk / nacos）
3. 服务熔断

# Zookeeper 的一致性

Sequential Consistency 顺序一致性

> 并发编程 -> (有序性) 

<!-- 并发编程 (原子性,有序性,可见性) -->

 顺序一致性的特点通过[队列, zxid]实现 -> 2pc 的弱一致性有矛盾?



顺序一致性 -> 满足在时间轴上数据的一致性

# Leader 选举

1. myid (sid) 
2. epoch (时钟周期) 
3. 选举状态 
   1. LOOKING -> 竞选状态 
   2. FOLLOWING -> 已选举
   3. LEADING
   4. OBSERVING -> 观察者节点,不参与投票,不参与选举



## 选举过程

- epoch
  1. leader选举的变化
  2. 每一轮选举 epoch 都会发生变化

### 选举的比较过程:

1. 先去比较 epoch, 判断是否是同一轮选举
2. epoch 相同 比较 zxid, zxid 最大的作为 leader
3. epoch,zxid 相同 比较 myid, myid 最大的最为 leader



### 每个节点的选举流程

1. 选自己当领导
2. 广播投票信息 (vote[myid/zxid/epoch])
3. Looking
4. 获取投票信息
   - Looking
     - 投票通过
       - 清空投票信息
       - 广播投票信息
       - ......
     - 投票不通过
       - 更新投票信息
       - 广播投票信息
       - 记录投票信息
       - 判断投票是否完成
         - 否:跳转至 步骤3
   - Leading / Following
     - 判断是否是同一轮投票(比较 epoch)
       - 是
         - 记录投票信息
         - 判断投票是否完成
       - 否
         - 记录已完成投票
         - 判断投票是否完成

### 源码

>  入口：
>
> ```java
> QuorumPeerMain
>     main
> ```

#### zk 网络通信

1. 选举(多个zk节点的数据通信)
2. 客户端请求(应用程序要连接到zkserver)
3. 数据同步(leader 和 follower 之间的数据同步)

#### 功能层面

zk 数据存储, 持久化

数据同步

watch机制



#### leader选举源码

>  QuorumPeer.java -> run() 

```java
makeLEStrategy().lookForLeader()
```

对比票据,判断谁是Leader

> FastLeaderElection.java -> lookForLeader()

```java
public Vote lookForLeader() throws InterruptedException {
        try {
            self.jmxLeaderElectionBean = new LeaderElectionBean();
            MBeanRegistry.getInstance().register(
                    self.jmxLeaderElectionBean, self.jmxLocalPeerBean);
        } catch (Exception e) {
            LOG.warn("Failed to register with JMX", e);
            self.jmxLeaderElectionBean = null;
        }
        if (self.start_fle == 0) {
           self.start_fle = Time.currentElapsedTime();
        }
        try {
            // 接收到的票据的集合
            HashMap<Long, Vote> recvset = new HashMap<Long, Vote>();

            //发出去的票据
            HashMap<Long, Vote> outofelection = new HashMap<Long, Vote>();

            int notTimeout = finalizeWait;

            synchronized(this){
                // 逻辑时钟 -> epoch
                logicalclock.incrementAndGet();
                // 更新当前节点的属性
                updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
            }

            LOG.info("New election. My id =  " + self.getId() +
                    ", proposed zxid=0x" + Long.toHexString(proposedZxid));
            sendNotifications(); // 广播自己的票据

            /*
             * Loop in which we exchange notifications until we find a leader
             */

            // 接受到了票据
            while ((self.getPeerState() == ServerState.LOOKING) &&
                    (!stop)){
                /*
                 * Remove next notification from queue, times out after 2 times
                 * the termination time
                 */
                // recvqueue 是从网络上接收到的其他机器的 Notification 信息
                Notification n = recvqueue.poll(notTimeout,
                        TimeUnit.MILLISECONDS); // 异步的

                /*
                 * Sends more notifications if haven't received enough.
                 * Otherwise processes new notification.
                 */
                //没收到信息
                if(n == null){
                    if(manager.haveDelivered()){
                        sendNotifications();
                    } else {
                        manager.connectAll(); // 重新连接集群中的所有节点
                    }

                    /*
                     * Exponential backoff
                     */
                    int tmpTimeOut = notTimeout*2;
                    notTimeout = (tmpTimeOut < maxNotificationInterval?
                            tmpTimeOut : maxNotificationInterval);
                    LOG.info("Notification time out: " + notTimeout);
                }
                // 验证 sid 和 leader 是否合法
                else if(validVoter(n.sid) && validVoter(n.leader)) { // 判断是否是一个有效票据
                    /*
                     * Only proceed if the vote comes from a replica in the
                     * voting view for a replica in the voting view.
                     */
                    switch (n.state) {
                    case LOOKING: // 第一次进入到这个 case
                        // If notification > current, replace and send messages out
                        if (n.electionEpoch > logicalclock.get()) { // 判断是不是大于 逻辑时钟
                            logicalclock.set(n.electionEpoch);
                            recvset.clear(); // 清空票据
                            // 收到票据之后,当前的server要听谁的
                            // 可能是听 server1 的, 也可能是听 server2 的
                            if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                    getInitId(), getInitLastLoggedZxid(), getPeerEpoch())) {
                                // 把自己的票据更新成对方的票据, 那么下一次 发送的票据就是新的票据
                                updateProposal(n.leader, n.zxid, n.peerEpoch);
                            } else {
                                // 收到的票据小于当前的节点的票据, 那么下一次发送票据,仍然发送自己的
                                updateProposal(getInitId(),
                                        getInitLastLoggedZxid(),
                                        getPeerEpoch());
                            }
                            // 继续发送通知
                            sendNotifications();
                        } else if (n.electionEpoch < logicalclock.get()) { // 说明当前的数据已经过期了
                            if(LOG.isDebugEnabled()){
                                LOG.debug("Notification election epoch is smaller than logicalclock. n.electionEpoch = 0x"
                                        + Long.toHexString(n.electionEpoch)
                                        + ", logicalclock=0x" + Long.toHexString(logicalclock.get()));
                            }
                            break;
                        } else if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                proposedLeader, proposedZxid, proposedEpoch)) {
                            updateProposal(n.leader, n.zxid, n.peerEpoch);
                            sendNotifications();
                        }

                        if(LOG.isDebugEnabled()){
                            LOG.debug("Adding vote: from=" + n.sid +
                                    ", proposed leader=" + n.leader +
                                    ", proposed zxid=0x" + Long.toHexString(n.zxid) +
                                    ", proposed election epoch=0x" + Long.toHexString(n.electionEpoch));
                        }

                        // 存储对方的票据
                        recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));

                        // 决断时刻 (当前节点的更新后的 vote 信息, 和 recvset 集合中的票据进行归纳)
                        // 半数的比较
                        if (termPredicate(recvset,
                                new Vote(proposedLeader, proposedZxid,
                                        logicalclock.get(), proposedEpoch))) {

                            // Verify if there is any change in the proposed leader
                            while((n = recvqueue.poll(finalizeWait,
                                    TimeUnit.MILLISECONDS)) != null){
                                if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                                        proposedLeader, proposedZxid, proposedEpoch)){
                                    recvqueue.put(n);
                                    break;
                                }
                            }

                            /*
                             * This predicate is true once we don't read any new
                             * relevant message from the reception queue
                             */
                            if (n == null) {
                                self.setPeerState((proposedLeader == self.getId()) ?
                                        ServerState.LEADING: learningState());

                                Vote endVote = new Vote(proposedLeader,
                                                        proposedZxid,
                                                        logicalclock.get(),
                                                        proposedEpoch);
                                leaveInstance(endVote);
                                return endVote;
                            }
                        }
                        break;
                    case OBSERVING:
                        LOG.debug("Notification from observer: " + n.sid);
                        break;
                    case FOLLOWING:
                    case LEADING:
                        /*
                         * Consider all notifications from the same epoch
                         * together.
                         */
                        if(n.electionEpoch == logicalclock.get()){
                            recvset.put(n.sid, new Vote(n.leader,
                                                          n.zxid,
                                                          n.electionEpoch,
                                                          n.peerEpoch));
                           
                            if(ooePredicate(recvset, outofelection, n)) {
                                self.setPeerState((n.leader == self.getId()) ?
                                        ServerState.LEADING: learningState());

                                Vote endVote = new Vote(n.leader, 
                                        n.zxid, 
                                        n.electionEpoch, 
                                        n.peerEpoch);
                                leaveInstance(endVote);
                                return endVote;
                            }
                        }

                        /*
                         * Before joining an established ensemble, verify
                         * a majority is following the same leader.
                         */
                        outofelection.put(n.sid, new Vote(n.version,
                                                            n.leader,
                                                            n.zxid,
                                                            n.electionEpoch,
                                                            n.peerEpoch,
                                                            n.state));
           
                        if(ooePredicate(outofelection, outofelection, n)) {
                            synchronized(this){
                                logicalclock.set(n.electionEpoch);
                                self.setPeerState((n.leader == self.getId()) ?
                                        ServerState.LEADING: learningState());
                            }
                            Vote endVote = new Vote(n.leader,
                                                    n.zxid,
                                                    n.electionEpoch,
                                                    n.peerEpoch);
                            leaveInstance(endVote);
                            return endVote;
                        }
                        break;
                    default:
                        LOG.warn("Notification state unrecognized: {} (n.state), {} (n.sid)",
                                n.state, n.sid);
                        break;
                    }
                } else {
                    if (!validVoter(n.leader)) {
                        LOG.warn("Ignoring notification for non-cluster member sid {} from sid {}", n.leader, n.sid);
                    }
                    if (!validVoter(n.sid)) {
                        LOG.warn("Ignoring notification for sid {} from non-quorum member sid {}", n.leader, n.sid);
                    }
                }
            }
            return null;
        } finally {
            try {
                if(self.jmxLeaderElectionBean != null){
                    MBeanRegistry.getInstance().unregister(
                            self.jmxLeaderElectionBean);
                }
            } catch (Exception e) {
                LOG.warn("Failed to unregister with JMX", e);
            }
            self.jmxLeaderElectionBean = null;
            LOG.debug("Number of connection processing threads: {}",
                    manager.getConnectionThreadCount());
        }
    }
```



选举玩之后, 要做什么?

1. 把 leader 和 follower 的角色设置好
2. follower 是不是要去连接 leader (事务)