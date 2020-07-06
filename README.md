참고
https://docs.ncloud.com/ko/database/database-10-3.html
https://www.howtoforge.com/tutorial/deploying-mongodb-sharded-cluster-on-centos-7/
https://kslee7746.tistory.com/entry/MongoDB-Sharding%EC%83%A4%EB%94%A9

# MongoDB sharded cluster setup

---

1. ### config server replicaset 설정

##### 각 인스턴스에 아래의 명령어로 config server 실행

mongod --configsvr --replSet replconfig --dbpath /var/lib/mongodb/configsvr -port 20001 --wiredTigerCacheSizeGB 2.0 --bind_ip 192.168.0.101 --fork --logpath /var/log/mongodb/configsvr.log
mongod --configsvr --replSet replconfig --dbpath /var/lib/mongodb/configsvr -port 20001 --wiredTigerCacheSizeGB 2.0 --bind_ip 192.168.0.102 --fork --logpath /var/log/mongodb/configsvr.log
mongod --configsvr --replSet replconfig --dbpath /var/lib/mongodb/configsvr -port 20001 --wiredTigerCacheSizeGB 2.0 --bind_ip 192.168.0.103 --fork --logpath /var/log/mongodb/configsvr.log

##### 레플리카 셋 설정

(config server로 설정한 것 중 한곳에 접속만하여 설정하면 된다)
mongo --host 192.168.0.101 --port 20001

```
rs.initiate(
  {
    _id: "replconfig",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.0.101:20001" },
      { _id : 1, host : "192.168.0.102:20001" },
      { _id : 2, host : "192.168.0.103:20001" }
    ]
  }
)
```

2. ### shard replicaset 설정

mongod --shardsvr --replSet shard1repl --dbpath /var/lib/mongodb/shard1 -port 20002 --wiredTigerCacheSizeGB 4.0 --bind_ip 192.168.0.101 --fork --logpath /var/log/mongodb/shard1.log
mongod --shardsvr --replSet shard1repl --dbpath /var/lib/mongodb/shard1 -port 20002 --wiredTigerCacheSizeGB 4.0 --bind_ip 192.168.0.102 --fork --logpath /var/log/mongodb/shard1.log
mongod --shardsvr --replSet shard1repl --dbpath /var/lib/mongodb/shard1 -port 20002 --wiredTigerCacheSizeGB 4.0 --bind_ip 192.168.0.103 --fork --logpath /var/log/mongodb/shard1.log

(shard replicaset 설정한 것 중 한곳에 접속만하여 설정하면 된다)
mongo --host 192.168.0.101 --port 20002

```
rs.initiate(
  {
    _id: "shard1repl",
    members: [
      { _id : 0, host : "192.168.0.101:20002" },
      { _id : 1, host : "192.168.0.102:20002" },
      { _id : 2, host : "192.168.0.103:20002" }
    ]
  }
)
```

3. ### mongos 설정

config server가 돌아가는 곳에 모두 query router 인 mongos를 동작시킨다.
여러 mongos를 각 디비 서버에 가지는것이 고가용성과 확장성에 좋으며 어플리케이션과 라우터사이의 네트워크 지연을 줄일 수 있다.

mongos --configdb replconfig/192.168.0.101:20001,192.168.0.102:20001,192.168.0.103:20001 --fork --logpath /var/log/mongodb/queryrouter.log
mongos --configdb replconfig/192.168.0.101:20001,192.168.0.102:20001,192.168.0.103:20001 --fork --logpath /var/log/mongodb/queryrouter.log
mongos --configdb replconfig/192.168.0.101:20001,192.168.0.102:20001,192.168.0.103:20001 --fork --logpath /var/log/mongodb/queryrouter.log

4. ### shard를 mongos에 등록

sh.addShard( "shard1repl/192.168.0.101:20002")
sh.addShard( "shard1repl/192.168.0.102:20002")
sh.addShard( "shard1repl/192.168.0.103:20002")

---

## Error case

- Failed to unlink socket file /tmp/mongodb-20002.sock Operation not permitted

  - sudo rm -rf /tmp/mongodb-20002.sock 로 해당파일 삭제 후 재실행

- NonExistentPath: Data directory /var/lib/mongodb/configsvr not found. Create the missing directory or specify another path using (1) the --dbpath command line option, or (2) by adding the 'storage.dbPath' option in the configuration file., terminating
  - 파일및 경로 생성 후 권한부여 sudo chown -R $USER:$USER path
