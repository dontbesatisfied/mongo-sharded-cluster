1. # config server 레플리카 셋 설정

##### 각 인스턴스에 아래의 명령어로 서버 실행

mongod --configsvr --replSet replconfig --dbpath /var/lib/mongodb/configsvr -port 20001 --wiredTigerCacheSizeGB 2.0 --bind_ip 192.168.0.101
mongod --configsvr --replSet replconfig --dbpath /var/lib/mongodb/configsvr -port 20001 --wiredTigerCacheSizeGB 2.0 --bind_ip 192.168.0.102
mongod --configsvr --replSet replconfig --dbpath /var/lib/mongodb/configsvr -port 20001 --wiredTigerCacheSizeGB 2.0 --bind_ip 192.168.0.103

Failed to unlink socket file /tmp/mongodb-20002.sock Operation not permitted 의 경우
sudo rm -rf /tmp/mongodb-20002.sock

NonExistentPath: Data directory /var/lib/mongodb/configsvr not found. Create the missing directory or specify another path using (1) the --dbpath command line option, or (2) by adding the 'storage.dbPath' option in the configuration file., terminating 의 경우
dbpath의 파일및 경로 생성 후 sudo chown -R $USER:$USER dbpath

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
      { _id : 1, host : "192.168.0.103:20001" }
    ]
  }
)
```
