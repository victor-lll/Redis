# Redis

## Redis安装

1. 环境配置准备

   ```bash
   $ yum -y install gcc wget net-tools
   $ systemctl stop firewalld
   $ setenforce 0
   ```

2. 下载需要的安装包

   ```bash
   $ wget http://download.redis.io/releases/redis-5.0.3.tar.gz
   ```

3. 解压安装包

   ```bash
   $ tar xzf redis-5.0.3.tar.gz
   ```

4. 跳转到redis的压缩目录

   ```bash
   $ cd redis-5.0.3
   ```

5. 编译

   ```bash
   $ make
   ```

6. 编译的二进制文件在`src` 目录中可用 ,运行Redis

   ```bash
   $ src/redis-server
   ```

7. 查看redis的进程和端口是否监听

   ```bash
   $ netstat -anpt | grep redis
   $ netstat -anpt | grep 6379
   tcp        0      0 0.0.0.0:6379            0.0.0.0:*                    LISTEN         2290/src/redis-serv 
   tcp6       0      0 :::6379                 :::*                        LISTEN         2290/src/redis-serv 
   $ ps -aux | grep redis
   root       2290  0.1  0.2 144012  2424 ?        Sl   17:39   0:22        src/redis-     server *:6379
   root       6561  0.0  0.0 112708   980 pts/1    R+   21:58   0:00        grep --color=auto redis
   ```

## Redis 配置详解

### NETWORK (网络)

1. network相关配置

   ```bash
   1.1 监听地址
    # 绑定redis服务器网卡IP，默认为127.0.0.1,即本地回环地址，可以监听一个或多个IP地址。
    # 网络中的服务是通过ip+进程来进行区分的，当一个服务器拥有两个ip时，自然就在网络中拥有两个人身份，如内网，外网，当你指向让redis在一个网络上监听时，就可以用此配置
    $ bind 127.0.0.1 
    $ bind 192.168.1.100
    
   1.2 保护模式
    # 保护模式，默认是开启状态。
    # 只允许本地客户端连接， 可以设置密码或添加bind来连接
    $ protected-mode yes
    
   1.3 监听端口
    # 默认的端口为6379
    # 如果端口为0，则redis不会再TCP socket上进行监听（不在TCP socket上进行监听，不代表没法连接，只是无法使用网络连接而已）
    $ port 6379
    
   1.4 TCP监听的最大容纳数量
    # 此参数确定了TCP连接中已完成队列（完成三次握手之后）的长度，当然此值必须不大于linux系统定义   的/proc/sys/net/core/somaxconn值，如果超过somaxconn值，则linux系统会将其截断，默认为511。
    # 此值需要参考linux系统的两个内核参数：
    # 1. /proc/sys/net/core/somaxconn 服务端所能接受最大客户端数量，即完成连接上限。
    # 2. /proc/sys/net/ipv4/tcp_max_syn_backlog 服务端所能接受SYN同步包的最大客户端数量，即半 连接上限。
    # 所以需要提高somaxconn与tcp_max_syn_backlog的值来达到预期的效果。
    $ tcp-backlog 511
    
   1.5 socket连接
    # 指定redis监听的unix socket路径，默认不启用
    # unixsocketper指定文件的权限
    $ unixsocket /tmp/redis.sock
    $ unixsocketperm 700
    
   1.6 连接超时时间
    # 客户端和服务端的连接超时时间,默认值为0,(0表示永不关闭)。
    $ timeout 0
    
   1.7 连接保持时间
    # 单位是秒，表示将周期性的使用SO_KEEPALIVE检测客户端是否还处于健康状态，避免服务器一直阻塞，官方给出的建议值是300s，如果设置为0，则不会周期性的检测
    $ tcp-keepalive 300
   ```

### GENRAL (通用)

2. genral相关配置

   ```bash
   2.1 守护进程
    # 默认情况下 redis 不是作为守护进程运行的，如果你想让它在后台运行，你就把它改成 yes。当redis作为守护进程运行的时候，它会写一个 pid 到 /var/run/redis.pid 文件里面
    $ daemonize no
    # 选择是upstart还是systemd管理redis进程。此参数和操作系统相关：
    # 1. supervised no 没有监督互动
    # 2. supervised upstart  upstart信号通过将Redis置于SIGSTOP模式发送
    # 3. supervised systemd  systemd信号通过READY=1写入NOTIFY_SOCKET环境变量
    # 4. supervised auto  通过UPSTART_JOB或NOTIFY_SOCKET环境变量来自动选择
    $ supervised no
    # 当redis以守护进程方式运行时，指定pid文件路径。redis默认会把pid写入/var/run/redis.pid。
    $ pidfile /var/run/redis_6379.pid
    
   2.2 日志相关
    # redis总共支持四个日志级别：
    # 1. debug：记录很多信息，用于开发和测试。
    # 2. varbose：少但有用的信息，不像debug会记录那么多信息。
    # 3. notice：适量的信息，常用于生产环境。
    # 4. warning：只有非常重要或者严重的信息会记录到日志。
    # 默认级别是notice。
    $ loglevel notice
    # 日志文件位置。
    # 当指定为空字符串时，为标准输出，如果redis已守护进程模式运行，那么日志将会输出到/dev/null。
    $ logfile /var/log/redis/redis.log
    # 如果想把日志记录到系统日志syslog中，就把它改成 yes。还可以通过下面的参数来达到符合的要求。
    $ syslog-enabled no
    # 指定syslog的标示符（系统日志中的身份），如果syslog-enabled为no，则这个选项无效。
    $ syslog-ident redis
    # 指定syslog设备（facility），必须是USER或者LOCAL0到LOCAL7。具体可以参考syslog服务本身的用法。
    $ syslog-facility local0
    
   2.3 数据库数量
    # 设置数据库的数目。默认的数据库是DB 0 ，可以在每个连接上使用select  <dbid> 命令选择一个不同的数据库，dbid是一个介于0到databases - 1 之间的数值
    $ databases 16
   ```

### SNAPSHOTTING (快照)

3. snapshotting相关配置

   ```bash
   3.1 快照持久化策略
    # 这个参数是redis持久化的支持，基于snapshot机制	，定期执行持久化存储，生成rdb文件。
    # 格式为：save <seconds> <changes> 
    # <seconds>和<changes>都满足时就会触发数据保存动作。
    # 以下为默认策略：
    # 900秒内如果至少有1个key的值发生变化，则保存到磁盘。
    # 300秒内如果至少有10个key的值发生变化，则保存到磁盘。
    # 60秒内如果至少有10000个key的值发生变化，则保存到磁盘。
    # 如果不想让redis自动保存数据，就把以下配置注释掉。也可以直接配置空字符来实现停用：save ""。
    $ save 900 1
    $ save 300 10
    $ save 60 10000
    
   3.2 持久化错误策略
    # 如果用户开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求。
    # 这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了。
    # 如果redis不顾这种不一致，一意孤行的继续接收写请求，就可能会引起一些灾难性的后果。
    # 如果下一次RDB持久化成功，redis会自动恢复接受写请求。
    # 如果不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话，可以关闭这个功能，
    # 以便在快照写入失败时，也能确保redis继续接受新的写请求。
    $ stop-writes-on-bgsave-error yes
    
   3.3 快照压缩
    # 对于存储到磁盘中的快照，可以设置是否进行压缩存储。
    # 如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，
    # 可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。
    $ rdbcompression yes
    
   3.4 快照校验
    # 在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，
    # 如果希望获取到最大的性能提升，可以关闭此功能。
    $ rdbchecksum yes
    
   3.5 快照名称
    # 设置快照的文件名
    $ dbfilename dump.rdb
    
   3.6 快照路径
    # 设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名。
    $ dir ./
   ```

### REPLICATION (复制)

4. replication相关配置

   ```bash
   4.1 主从复制
    # 主从复制，使用 relicaof 来让一个 redis 实例成为另一个reids 实例的副本，默认关闭
    # 注意这个只需要在 replica 上配置
    $ replicaof <masterip> <masterport>
    
   4.2 密码校验
    # 如果master设置了验证密码的话（使用requirepass来设置），则在slave的配置中要使用masterauth来设置校验密码，否则的话，主redis会拒绝从redis的访问请求。
    $ masterauth <master-password>
    
   4.3 replica响应请求策略
    # 当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：
    # 1.如果 replica-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能 是正常数据，也可能是还没获得值的空数据。
    # 2.如果 replica-serve-stale-data 设置为 "no"，slave会回复"正在从master同步（SYNC with master in progress）"来处理各种请求，除了 INFO 和 SLAVEOF 命令。
    $ replica-serve-stale-data yes
    
   4.4 replica读写策略
    # 配置从是否为只读，开启后从则不能写入数据
    $ replica-read-only yes
    
   4.5 无硬盘复制
    # 备份同步策略：硬盘还是套接字
    # 新replica服务连接或者旧replica重新连接时不能只复制与master有差异的数据，必须做一个全部同步。需要一  个新RDB文件dump出来，然后从master传输到replica。有两种方式：
    # 1. 基于硬盘（disk-backed）：master创建一个新进程dump RDB，创建完成后由父进程（即主进程）增量传给replica。
    # 2. 基于socket（diskless）：master创建一个进程直接dump RDB到replica的socket，不经过主进程，不经过磁盘。
    # 基于硬盘的话，RDB文件创建完毕后，可以同时服务更多的replica。
    # 基于socket的话，新replica建立连接后需要排队。同步完一个在进行下一个。
    $ repl-diskless-sync no
    # 当使用diskless时，master收到第一个replica同步请求时，会等待多个replica请求，可以一起同步数据。请求间隔为repl-disk-sync-delay的配置时间。超过了repl-disk-sync-delay间隔时间，后来的replica需要排队。
    # 可以配置当收到第一个请求时，等待多个replica一起来请求之间的间隔时间。
    # 如果为0，则立即启动同步
    $ repl-diskless-sync-delay 5
    
   4.6 ping监测
    # 从redis会周期性的向主redis发出PING包，你可以通过repl_ping_slave_period指令来控制其周期，默认是10秒。
    $ repl-ping-replica-period 10
    
   4.7 同步超时时间
    # 在主从同步时，可能在这些情况下会有超时发生：
    # 1.从从站的角度，同步期间的批量传输的I/O
    # 2.从站角度认为的主站超时（数据，ping）
    # 3.主站角度认为的从站超时（REPLCONF ACK pings)
    # 确认这些值比定义的repl-ping-slave-period要大，否则每次主站和从站之间通信低速时都会被检测为超时
    $ repl-timeout 60
    
   4.8 同步延迟策略
    # 同步之后是否禁用从站上的TCP_NODELAY
    # 如果你选择yes，redis会使用较少量的TCP包和带宽向从站发送数据。但这会导致在从站增加一点数据的延时。
    # Linux内核默认配置情况下最多40毫秒的延时。
    # 如果选择no，从站的数据延时不会那么多，但备份需要的带宽相对较多。
    # 默认情况下我们将潜在因素优化，但在高负载情况下或者在主从站都跳的情况下，把它切换为yes是个好主意
    $ repl-disable-tcp-nodelay no
    
   4.9 同步缓存
    # 队列长度（backlog)是master中的一个缓冲区，在与replica断开连接期间，master会用这个缓冲区来缓存应该发给replica的数据。这样的话，当replica重新连接上之后，就不必重新全量同步数据，只需要同步这部分增量数据即可。
    $ repl-backlog-size 1mb
    # 如果master等了一段时间之后，还是无法连接到replica，那么缓冲队列中的数据将被清理掉。
    # 可以设置master要等待的时间，默认是1小时。
    # 如果设置为0，则表示永远不清理。
    $ repl-backlog-ttl 3600
    
   4.10 replica优先级
    # 编号越小，优先级越高
    # 在不止1个replica存在的部署环境下，当master宕机时，Redis Sentinel会将priority值最小的replica提升为master。
    # 若配置为0，则对应的replica永远不会自动提升为master。
    $ replica-priority 100
    
   4.11 Master接收写请求条件
    # 如果master有至少N个replica，并且ping心跳的超时不超过M秒，那么它就会接收写请求。
    # 如果N和M的条件都无法达到，那么master会回复一个错误信息。写请求也不会被处理。
    # 默认min-replicas-to-write为0，表示对replicas数量无限制，网络延迟为10s。
    # 两个配置中有一个被置为0，则这个特性将被关闭。
    # 需要至少3个replica链接，且延迟小于等于10秒时，master才能写数据。
    $ min-replicas-to-write 3
    $ min-replicas-max-lag 10
    
   4.12 replica信息
    # master能够以不同的方式列出所连接slave的地址和端口。
    # 例如，“INFO replication”部分提供此信息，除了其他工具之外，Redis Sentinel还使用该信息来发现replica实例。
    # 此信息可用的另一个地方在masterser的“ROLE”命令的输出中。
    # 通常由replica报告的列出的IP和地址,通过以下方式获得：
    # 1. IP：通过检查replica与master连接使用的套接字的对等体地址自动检测地址。
    # 2. 端口：端口在复制握手期间由replica通信，并且通常是replica正在使用列出连接的端口。
    # 然而，当使用端口转发或网络地址转换（NAT）时，replica实际上可以通过(不同的IP和端口对)来到达。 replica可以使用以下两个选项，以便向master报告一组特定的IP和端口，以便INFO和ROLE将报告这些值。
    # 如果你需要仅覆盖端口或IP地址，则没必要使用这两个选项。
    $ replica-announce-ip 5.5.5.5
    $ replica-announce-port 1234
   ```

### SECURITY (安全)

5. security相关配置

   ```bash
   5.1 连接密码
    # 设置客户端连接后进行任何其他指令前需要使用的密码。
    # 警告：因为redis速度相当快，所以在一台比较好的服务器下，一个外部用户可以在一秒钟进行150k次的密码尝试，这意味着需要指定非常非常强大的密码来防止暴力破解。
    $ requirepass foobared
    
   5.2 指令重命名
    # 比如将一些比较危险的命令改个名字，避免被误执行。比如可以把CONFIG命令改成一个很复杂的名字，这样可以避免外部的调用，同时还可以满足内部调用的需要，比如：
    # rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
    # 如果想禁用一个命令，直接把它重命名为一个空字符""即可。
    $ rename-command CONFIG ""
   ```

### CLIENTS (客服端)

6. clients相关配置

   ```bash
   6.1 客户端最大连接数
    # 设置同一时间最大客户端连接数，默认为10000。
    # Redis可以同时打开的客户端连接数为redis进程可以打开的最大文件描述符数-32（redis server自身会使用一些）。
    # 如果设置maxclients 0，表示不作限制。
    # 当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息。
    $ maxclients 10000
   ```

### MEMORY MANAGEMENT (内存管理)

7. memory management相关配置

   ```bash
   7.1 内存限制及清除策略
    # 指定redis最大内存限制，redis在启动时会把数据加载到内存中，达到最大内存后，redis会按照清除策略尝试清除已到期的key。
    # 如果redis无法根据移除规则来移除内存中的数据，或者我们设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。但是对于无内存申请的指令，仍然会正常响应，比如GET等。
    # 注意：redis新的vm机制，会把key存放内存，value会存放在swap分区。
    # 注意：如果redis有slave的话，maxmemory最好设置为master内存的一半及以下，因为master需要启动新进程去dump rdb文件。
    $ maxmemory <bytes>
    
    # 当内存达到最大值的时候redis会选择清除一些数据，清除规则有：
    # 1. volatile-lru：利用LRU算法移除设置过过期时间的key。
    # 2. allkeys-lru：利用LRU算法移除任何key。
    # 3. volatile-random：随机地删除过期set中的key。
    # 4. allkeys-random：随机删除任意一个key。
    # 5. volatile-ttl：移除那些TTL值最小的key，即那些最近才过期的key。
    # 6. noeviction：不删除任何key，写操作会报错。
    # 注意：对于上面的策略，如果没有合适的key可以移除，当写的时候redis会返回一个错误。
    # 默认值为：noeviction
    $ maxmemory-policy noeviction
    
    # LRU 和 minimal TTL 算法都不是精准的算法，但是相对精确的算法(为了节省内存)。
    # 所以可以对它进行优化，以获得速度或准确性。默认情况下，Redis会检查5个键，并选择最近使用较少的键， 您可以使用下面的配置指令更改样本大小。
    # 默认值为5个样本通常情况下能够达到很好的效果。10个样本精确度更高，但是花费更多的CPU。3个样本是非常快的，但不是很准确。
    $ maxmemory-samples 5
    
    # 从Redis 5开始，默认情况下，副本将忽略其maxmemory设置
    # Memory，在主程序命中之前永远不会遇到真正的内存不足情况。
    $ replica-ignore-maxmemory yes
   ```

### LAZY FREEING (释放)

8. lazy freeing相关配置

   ```bash
   8.1 延迟释放功能
    # 针对redis内存使用达到maxmeory，并设置有淘汰策略时；在被动淘汰键时，是否采用lazy free机制；
    # 因为此场景开启lazy free, 可能使用淘汰键的内存释放不及时，导致redis内存超用，超过maxmemory的限制。此场景使用时，请结合业务测试。
    $ lazyfree-lazy-eviction no
    
    # 针对设置有TTL的键，达到过期后，被redis清理删除时是否采用lazy free机制;
    # 此场景建议开启，因TTL本身是自适应调整的速度。
    $ lazyfree-lazy-expire no
    
    # 针对有些指令在处理已存在的键时，会带有一个隐式的DEL键的操作。如rename命令，当目标键已存在,redis会先删除目标键，如果这些目标键是一个big key,那就会引入阻塞删除的性能问题。 此参数设置就是解决这类问题，建议可开启。
    $ lazyfree-lazy-server-del no
    
    # 针对replica进行全量数据同步，replica在加载master的RDB文件前，会运行flushall来清理自己的数据场景，
    # 参数设置决定是否采用异常flush机制。如果内存变动不大，建议可开启。可减少全量同步耗时，从而减少主库因输出缓冲区爆涨引起的内存使用增长。
    $ replica-lazy-flush no
   ```

### APPEND ONLY MODE (AOF)

9. append only mode相关配置

   ```bash
   9.1 AOF持久化功能
    # 默认情况下，redis会异步的将数据持久化到磁盘。这种模式在大部分应用程序中已被验证是很有效的，但是在一些问题发生时，比如断电，则这种机制可能会导致数分钟的写请求丢失。
    # 所以redis提供了另外一种更加高效的数据库备份及灾难恢复方式，AOF(Append Only File)。
    # 开启append only模式之后，redis会把所接收到的每一次写操作请求都追加到appendonly.aof文件中，
    # 但是这样会造成appendonly.aof文件过大，所以redis还支持了BGREWRITEAOF指令，对appendonly.aof进行重新整理。
    # 默认是不开启的。
    # 注意：如果需要，可以同时开启AOF模式和RDB（快照）模式。这种情况下，redis重建数据集时会优先使用appendonly.aof而忽略dump.rdb。
    $ appendonly no
    
    # 设置AOF文件名称，默认为appendonly.aof。
    $ appendfilename "appendonly.aof"
    
   9.2 AOF持久化策略
    # redis支持三种同步AOF文件的策略：
    # 1. always：每一次写操作都会调用一次fsync，这时数据是最安全的。当然，由于每次都会执行fsync，所以其性能也会受到影响。这种模式下，redis会相对较慢，但数据最安全。
    # 2. everysec：redis会默认每隔一秒进行一次fsync调用，将缓冲区的数据写入磁盘。但是当这一次的fsync调用时长超过1秒钟时，redis会采取延迟fsync的策略，再等一秒钟。也就是在两秒后在进行fsync，这一次的fsync不管会执行多长时间都会进行。这时候由于在fsync时文件描述符会阻塞，所以当前的写操作就会阻塞。所以，在绝大多数情况下，redis会每隔一秒进行一次fsync。在最坏的情况下，两秒钟会进行一次fsync操作。这一操作在大多数数据库系统中被称为group commit，就是组合多次写操作的数据，一次性将日志写到磁盘。这种模式是性能和安全的折中。
    # 3. no：redis不会主动调用fsync去将AOF日志内容同步到磁盘，所以这一切就完全依赖于操作系统的调试了。对于大多数linux系统，是每30秒进行一次fsync，将缓冲区中的数据写到磁盘上。这种模式下，redis的性能会最快。
    # 默认为，everysec。
    $ appendfsync everysec
    
   9.3 AOF重写策略
    # 当aof增长到一定规模时，redis会隐式调用BGREWRITEAOF来重写log文件，以缩减文件体积。
    # 设置为no时在后台aof文件rewrite期间调用fsync，默认为no，表示要调用fsync。
    # 设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入。（Linux的默认fsync策略是30秒。可能丢失30秒数据。）
    # redis在后台写RDB文件或重写AOF文件期间会存在大量磁盘IO，此时在某些linux系统中，调用fsync可能会阻塞。
    # 如果对延迟要求很高的应用，这个字段可以设置为yes，否则还是设置为no，这样对持久化特性来说这是更安全的选择。
    $ no-appendfsync-on-rewrite no
    
    # 1. auto-aof-rewrite-percentage：指定redis重写aof文件的条件，默认为100，表示与上次rewrite的aof文件大小相比，当前aof文件增长量超过上次aof文件大小的100%时，就会触发backgroud rewrite。
    # 若auto-aof-rewrite-percentage配置为0，则会禁用自动rewrite功能。
    # 2. auto-aof-rewrite-min-size：指定触发rewrite的aof文件大小。若aof文件小于该值，即时当前文件的增量比例达到auto-aof-rewrite-percentage的配置值，也不会触发自动rewrite。
    # 这两个配置同时满足时，才会触发rewrite。
    $ auto-aof-rewrite-percentage 100
    $ auto-aof-rewrite-min-size 64mb
    
   9.4 AOF文件修复
    # AOF文件可能在尾部是不完整的（尤其是mount ext4文件系统时没有加上data=ordered选项）。那redis重启时load进内存的时候就可能出现问题。
    # 配置aof-load-truncated为yes时，redis会自动发布一个log给客户端然后load。
    # 如果配置为no时，用户必须手动redis-check-aof修复aof文件才可以。
    $ aof-load-truncated yes
    
   9.5 AOF重写和恢复
    # 在重写AOF文件时，Redis能够在AOF文件，用于更快的重写和恢复；
    # 加载Redis时，Redis识别出AOF文件以“REDIS”开头string并加载前缀的RDB文件，并继续加载AOF尾巴
    $ aof-use-rdb-preamble yes
   ```

### LUA SCRIPTING (LUA脚本)

10. lua scripting相关配置

    ```bash
      10.1 脚本执行时间限制
    # Redis以毫秒为单位限定lua脚本的最大执行时间。
    # 当lua脚本在超出最大允许执行时间之后，redis会记录这个脚本到日志中，并且会向这个请求返回error的错误。
    # 仅当SCRIPT KILL和SHUTDOWN NOSAVE命令可用的时候，一个运行时间超过最大限定时间的脚本才会被继续执行。
    # SCRIPT KILL用来停止一个没有调用写入命令的脚本。当用户不想等待脚本的自然中止又在进行写操作的时候，采用SHUTDOWN NOSAVE是解决这个问题的唯一方法。可以立即停掉这个脚本。
    # 以下配置为一个lua脚本最长的执行时间为5000毫秒（5秒），如果为0或者负数，则表示无限执行时间。
    $ lua-time-limit 5000
    ```

### REDIS CLUSTER (集群)

11. redis cluster相关配置

    ```bash
    11.1 集群模式
     # 集群开关，默认是不开启集群模式。
     $ cluster-enabled yes
     
    11.2 集群配置文件
     # 集群配置文件的名称，每个节点都有一个集群相关的配置文件，持久化保存集群的信息。
     # 这个文件并不需要手动配置，这个配置文件有Redis生成并更新，每个Redis集群节点需要一个单独的配置文件
     # 确保与实例运行的系统中配置文件名称不冲突
     $ cluster-config-file nodes-6379.conf
     
    11.3 节点超时时间
     # 节点互连超时的阀值，集群节点超时毫秒数。
     $ cluster-node-timeout 15000
     
    11.4 故障转移策略
     # 在进行故障转移的时候，全部replica都会请求申请为master，但是有些replica可能与master断开连接一段时间了，导致数据过于陈旧，这样的replica不应该被提升为master。该参数就是用来判断replica节点与master断线的时间是否过长。
     # 判断方法是：
     # 1. replica断开连接的时间
     # 2. (node-timeout * replica-validity-factor) + repl-ping-replica-period
     # 如果节点超时时间为30s, 并且replica-validity-factor为10,即如果超过310秒replica将不会尝试进行故障转移。
     # 默认为10
     $ cluster-slave-validity-factor 10
     
     # master的replica数量大于该值，replica才能迁移到其他孤立master上。
     # 如这个参数若被设为2，那么只有当一个主节点拥有2个可工作的从节点时，它的一个从节点会尝试迁移。
     $ cluster-migration-barrier 1
     
    11.5 集群slot策略
     # 默认情况下，集群全部的slot有节点负责，集群状态才为ok，才能提供服务。
     # 设置为no，可以在slot没有全部分配的时候提供服务。
     # 不建议打开该配置，这样会造成分区的时候，小分区的master一直在接受写请求，而造成很长时间数据不一致。
     $ cluster-require-full-coverage yes
     
    11.6 replica故障转移
     # 选项在设置为yes时，防止副本尝试故障转移
     # master主控程序仍然可以执行手动故障转移,在不同的场景中很有用，特别是在多个场景中数据中心操作
     $ cluster-replica-no-failover no
    ```

### CLUSTER DOCKER/NAT support (集群对接/NAT支持)

12. cluster docker/nat support相关配置

    ```bash
    12.1 集群节点地址
     # 在部署过程中，会出现redis集群节点地址发现失败的情况，因为地址是NAT-ted或者端口被转发(常见的是Docker容器和其它容器)；
     # 为了使Redis集群在这样的环境中工作，静态配置，其中每个节点知道其公共地址是必需的。
     # 集群通知ip
     $ cluster-announce-ip
     # 集群通知端口
     $ cluster-announce-port
     # 集群通知总线端口
     $ cluster-announce-bus-port
    ```

### SLOW LOG (慢日志)

13. slow log相关配置

    ```bash
    13.1 慢日志策略
     # Redis Slow Log记录超过特定执行时间的命令。执行时间不包括I/O计算比如连接客户端、返回结果等，只是命令执行时间（线程因为执行这个命令而锁定且无法处理其他请求的阶段）。即执行比较慢的命令。
     # 可以通过两个参数设置slow log：
     # 1. 一个是慢查询的阈值，单位是毫秒。
     # 2. 另一个是slow log的长度（即记录的最大数量），相当于一个队列。
     # 默认配置记录redis执行超过10000毫秒的参数。
     # 负数则关闭slow log，0则会导致每个命令都被记录。
     $ slowlog-log-slower-than 10000
     
     # 不设置会消耗过多内存。可以通过SLOWLOG RESET回收慢日志消耗的内存。
     # 默认值为128，当慢日志超过128时，最先进入的队列的记录会被踢出。
     $ slowlog-max-len 128
    ```

### LATENCY MONITOR (延迟监控)

14. latency monitor相关配置

    ```bash
    14.1 延迟监控
     # Redis延迟监控子系统案例与操作系统收集的redis实例相关的数据不同。
     # 通过LATENCY命令，可以为用户打印出相关信息的图形和报告。
     # 这个系统只会记录运行时间和超出指定时间值的命令，如果设置为0，这个监控会被关闭。
     # 默认情况下是关闭延迟监控。因为如果没有延迟的问题大部分情况下不需要，并且收集数据的行为会对性能造成影响。
     # 延迟监控可以使用命令来打开：
     # CONFIG SET latency-monitor-threshold <milliseconds>
     $ latency-monitor-threshold 0
    ```

### EVENT NOTIFICATION (事件通知)

15. event notification相关配置

    ```bash
    15.1 事件通知
     # Redis可以在key空间中采用发布/订阅模式来通知时间的发生。
     # 对于一个实例，如果键空间事件通知是启用状态，当一个客户端执行在一个存储在Database 0名为“foo”的key DEL操作时，有如下两条信息将会通过发布订阅系统产生：
     # PUBLISH __keyspace@0__:foo del
     # PUBLISH __keyevent@0__:del foo
     # 可以在下表中选择Redis要通知的事件类型，事件类型由单个字符来标识：
     # K keyspace事件，以__keyspace@__的前缀方式发布
     # E keyevent事件，以__keysevent@__的前缀方式发布
     # g 通用事件（不指定类型），像DEL，EXPIRE，RENAME，...
     # $ string命令
     # l list命令
     # s set命令
     # h hash命令
     # z 有序集合命令
     # x 过期时间（每次key过期时生成）
     # e 清除时间（当key在内存被清除时生成）
     # A g$shzxe的别称，意味着通知所有的事件
    
     # 例如：启用list和通用事件
     # notify-keyspace-events elg
    
     # 默认所有的通知被禁用，空字符串的意思是通知被禁用。
     # 注意如果不指定至少K或E之一，不会发送任何事件。
     $ notify-keyspace-events ""
    ```

### ADVANCED CONFIG (高级配置)

16. advanced config相关配置

    ``` bash
    16.1 编码策略
     # 创建空白哈希表时，程序默认使用REDIS_ENCODING_ZIPLIST编码，当以下任何一个条件被满足时，程序将编码切换为REDIS_ENCODING_HT：
     # 1. 哈希表中某个键或某个值的长度大于server.hash_max_ziplist_value（默认值为64）。
     # 2. 压缩列表中的节点数量大于server.hash_max_ziplist_entries（默认值为512）。
     # ziplist是一个解决空间问题的紧凑数据库存储结构，但是当数据超过阈值时，将采用原生的数据存储结构。
     $ hash-max-ziplist-entries 512
     $ hash-max-ziplist-value 64
     
     # 当取正值的时候，表示按照数据项个数来限定每个quicklist节点上的ziplist长度。比如，当这个参数配置成5的时候，表示每个quicklist节点的ziplist最多包含5个数据项。
     # 当取负值的时候，表示按照占用字节数来限定每个quicklist节点上的ziplist长度。这时，它只能取-1到-5这五个值，每个值含义如下：
     # 5: 每个quicklist节点上的ziplist大小不能超过64 Kb。（注：1kb => 1024 bytes）
     # 4: 每个quicklist节点上的ziplist大小不能超过32 Kb。
     # 3: 每个quicklist节点上的ziplist大小不能超过16 Kb。
     # 2: 每个quicklist节点上的ziplist大小不能超过8 Kb。（-2是Redis给出的默认值）
     # 1: 每个quicklist节点上的ziplist大小不能超过4 Kb。
     $ list-max-ziplist-size -2
     
     # 数据量小于等于set-max-intset-entries用intset，大于set-max-intset-entries用set
     $ set-max-intset-entries 512
     # 数据量小于等于zset-max-ziplist-entries用ziplist，大于zset-max-ziplist-entries用zset
     $ zset-max-ziplist-entries 128
     $ zset-max-ziplist-value 64
     
    16.2 节点压缩
     # 这个参数表示一个quicklist两端不被压缩的节点个数。
     # 注：这里的节点个数是指quicklist双向链表的节点个数，而不是指ziplist里面的数据项个数。
     # 实际上，一个quicklist节点上的ziplist，如果被压缩，就是整体被压缩的。
     # 参数list-compress-depth的取值含义如下：
     # 0: 是个特殊值，表示都不压缩。这是Redis的默认值。
     # 1: 表示quicklist两端各有1个节点不压缩，中间的节点压缩。
     # 2: 表示quicklist两端各有2个节点不压缩，中间的节点压缩。
     # 3: 表示quicklist两端各有3个节点不压缩，中间的节点压缩。
     # 依此类推...
     # 由于0是个特殊值，很容易看出quicklist的头节点和尾节点总是不被压缩的，以便于在表的两端进行快速存取。
     $ list-compress-depth 0
     
    16.3 数据结构
     # value大小小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse）。
     # 大于hll-sparse-max-bytes使用稠密的数据结构（dense），一个比16000大的value是几乎没用的。
     # 建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右。
     $ hll-sparse-max-bytes 3000
     
    16.4 流宏节点
     # 流数据结构是基数，对内部多个项进行编码的大节点树。
     # 使用此配置可以配置单个节点有多大（以字节为单位），以及在切换到新节点之前，它可能包含的项的最大数量。
     # 附加新的流条目。如果以下设置中的任何设置被设置为0，该限制被忽略
     # max实体通过将max-bytes设置为0，将max-entrys设置为所需来限制α值。
     # 流节点最大字节
     $ stream-node-max-bytes 4096
     # 流节点最大条目
     $ stream-node-max-entries 100
    16.5 内存释放
     # Redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。
     # 当使用场景中有严格的实时性需要，不能够接受redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。
     # 如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存。
     $ activerehashing yes
     
    16.6 缓存限制
     # 客户端输出缓冲区显示可以用来解决由于某些原因导致的强制断线而造成的不能读到足够的数据。
     # 一个比较常见的原因是由于发布订阅中，客户端不能够足够快速地消费发布者生成的信息。
     # 这个限制可以设置为如下的三种类型：
     # 1. normal：正常普通的客户端，包含监控客户端
     # 2. slave：主从服务器的从客户端
     # 3. pubsub：订阅了最少一个频道的客户端
     # 每一个client-output-buffer-limit格式如下：
     # client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
     # 在两种情况下服务器认为客户端不是意外临时掉线：
     # 1. 缓冲区的数据达到硬限制
     # 2. 缓冲区的数据达到软限制，同时时间超过了指定值
     # 因为一个客户离线，有可能是临时性的网络故障或者传输问题，也有可能是永久性离线或者强制性离线，此时服务器将不会保留它的缓存数据。
     # 以下的设置就是判断这一情况的。
     # 硬限制和软限制都可以通过将其设置为0来关闭掉。
     # 对于normal client，第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal  client默认取消限制，因为如果没有查询，他们是不会接收数据的。
     $ client-output-buffer-limit normal 0 0 0
     # 对于slave client和MONITER client，如果client-output-buffer一旦超过256mb，又或者超过64mb持续60秒，那么服务器就会立即断开客户端连接。
     $ client-output-buffer-limit slave 256mb 64mb 60
     # 对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接
     $ client-output-buffer-limit pubsub 32mb 8mb 60
     
    16.7 客户端查询缓冲区限制
     # 客户端查询缓冲区累积新的命令。它们仅限于固定的。
     # 默认情况下,以避免协议异步（用于instance由于客户机中的bug）将导致查询缓冲区。
     # 如果您非常特殊，可以在这里配置它。
     # 客户端查询缓冲区限制
     $ client-query-buffer-limit 1gb
     
    16.8 批量请求限制
     # 单个元素的元素字符串，通常限制为512mb。
     $ proto-max-bulk-len 512mb
     
    16.9 执行任务频率
     # redis会按照一定的频率来处理诸如关闭超时连接，清理没有被使用的过期key等等此类后台任务，并不是所有的任务都是以相同的频率来执行的，redis通过一个hz的值执行检查任务。
     # 提高该值将在redis空闲时使用更多的CPU，但同时当有多个key同时到期会使redis的反应更灵敏，以及超时可以更精准地处理。
     # 范围是1到500之间，但是值超过100通常不是一个好主意。
     # 默认情况下，hz的值被设定为10。大多数用户应该使用10这个预设值，只有在非常低的延迟情况下有必要提高最大到100。
     # 计算方法：1s / hz
     $ hz 10
     
    16.10 fsync增量同步
     # 在aof重写的时候，如果打开了aof-rewrite-incremental-fsync开关，系统会每32MB数据执行一次fsync。
     # 这对于把文件写入磁盘是有帮助的，可以减少aof大文件写入对磁盘的操作次数，避免过大的延迟峰值。
     $ aof-rewrite-incremental-fsync yes
     
    16.11 连接客户端
     # 每个后台任务调用处理太多的客户端以避免延迟峰值。
     # 默认情况下HZ值保守地设置为10，Redis提供并默认启用使用自适应HZ值的能力；当存在许多连接的客户端时，将临时引发。
     # 当启用动态HZ时，实际配置的HZ将被用作作为基线，但是配置的HZ值的倍数实际上是根据需要再次使用客户端。这样instance将使用非常少的CPU时间，而繁忙实例将响应性更强。
     $ dynamic-hz yes
     
    16.12 RDB保存增量fsync
     # 当Redis保存RDB文件时,文件将每生成32MB数据进行fsync。
     # 为了更增量地将文件提交到磁盘，并避免大延迟峰值
     $ rdb-save-incremental-fsync yes
    ```

## Redis 主从同步

在Redis复制的基础上（不包括由Redis Cluster或Redis Sentinel作为附加层提供的高可用性功能），有一个非常简单的使用和配置(主从）复制：它允许从属Redis实例准确主实例的副本。每次连接断开时，从站将自动重新连接到主站，并且*无论*主站发生什么情况，它将尝试成为它的精确副本。

该系统使用三种主要机制：

1. 当主实例和从属实例连接良好时，主设备通过向从设备发送命令流来保持从设备更新，以便复制对主设备端发生的数据集的影响，原因是：客户端写入，密钥已过期或驱逐，更改主数据集的任何其他操作。
2. 当主设备和从设备之间的链路中断时，对于网络问题或者因为主设备或从设备中检测到超时，从设备重新连接并尝试继续部分重新同步：这意味着它将尝试仅获取部件它在断开连接时错过的命令流。
3. 当无法进行部分重新同步时，从站将要求完全重新同步。这将涉及一个更复杂的过程，其中主机需要创建其所有数据的快照，将其发送到从机，然后在数据集更改时继续发送命令流