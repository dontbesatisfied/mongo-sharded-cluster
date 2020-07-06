1. # config server 레플리카 셋 설정

##### 각 인스턴스에 아래의 명령어로 서버 실행

sudo mongod --config ./mongod-configsvr1.conf
sudo mongod --config ./mongod-configsvr2.conf
sudo mongod --config ./mongod-configsvr3.conf

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
      { _id : 1, host : "192.168.0.102:20002" },
      { _id : 1, host : "192.168.0.103:20003" }
    ]
  }
)
```
