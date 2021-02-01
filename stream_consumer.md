# Trying out `Jetstream` dedicated features (`stream`, `consumer`)

### All the account configurations, accounts (including export/import), credential infos are stored in `confs` folder.

## Run Jetstream server

```shell
$ cd $HOME/go/src/github.com/appscodelabs/nats-service-demo

$ nats-server -c confs/server.conf &

$ # Create strem and consumer using account admin

$ cat confs/stream.json
{
    "name": "ReceivedEvents",
    "subjects": [
      "user.*.Events"
    ],
    "retention": "workqueue",
    "max_consumers": -1,
    "max_msgs": -1,
    "max_bytes": -1,
    "max_age": 31536000000000000,
    "max_msg_size": -1,
    "storage": "file",
    "discard": "old",
    "num_replicas": 1,
    "duplicate_window": 3600000000000
  }

$ nats str add ReceivedEvents --config confs/stream.json --creds confs/admin.creds

$ cat confs/consumer.json
{
    "durable_name": "ProcessEvents",
    "deliver_policy": "all",
    "ack_policy": "explicit",
    "ack_wait": 30000000000,
    "max_deliver": -1,
    "filter_subject": "user.*.Events",
    "replay_policy": "instant"
  }


$ nats con add ReceivedEvents --config confs/consumer.json --creds confs/admin.creds
```

## Publish message by user `X`

```shell
$ nats pub Events "Hello there" --creds confs/x.creds

$ nats pub Events "Hello there 1" --creds confs/x.creds

$ nats con next ReceivedEvents ProcessEvents --creds confs/admin.creds
[15:30:22] subj: Events / tries: 1 / cons seq: 1 / str seq: 1 / pending: 1

Hello there

Acknowledged message

$ nats con next ReceivedEvents ProcessEvents --creds confs/admin.creds
[15:30:25] subj: Events / tries: 1 / cons seq: 2 / str seq: 2 / pending: 0

Hello there 1

Acknowledged message
```

---

## Jetstream for exort/import type `service`

```shell
$ # create stream/consumer for service type
$ cat confs/srv_stream.json 
{
    "name": "Notifications",
    "subjects": [
      "Notifications"
    ],
    "retention": "workqueue",
    "max_consumers": -1,
    "max_msgs": -1,
    "max_bytes": -1,
    "max_age": 31536000000000000,
    "max_msg_size": -1,
    "storage": "file",
    "discard": "old",
    "num_replicas": 1,
    "duplicate_window": 3600000000000
  }
$ nats str add Notifications --config confs/srv_stream.json --creds confs/x.creds 

$ cat confs/srv_consumer.json 
{
    "durable_name": "ProcessNotifications",
    "deliver_policy": "all",
    "ack_policy": "explicit",
    "ack_wait": 30000000000,
    "max_deliver": -1,
    "filter_subject": "Notifications",
    "replay_policy": "instant"
  }
$ nats con add Notifications --config confs/srv_consumer.json --creds confs/x.creds
```
### Publish message to `user.x.Notifications` by Admin
```shell
$ nats pub user.x.Notifications "Hello Notifications" --creds confs/admin.creds
$ nats pub user.x.Notifications "Hello Notifications 2" --creds confs/admin.creds
```

### Get next message by user x
```shell
$ nats con next Notifications ProcessNotifications --creds confs/x.creds
--- subject: Notifications

Headers:

  Nats-Request-Info: {"acc":"AAVOHCQ2R2QICRH427VEOWEXM3A5M4LG6OPFARZYHK2T77ZQCCKFUBFQ","rtt":1250574}

Data:


Hello Notifications

Acknowledged message

$ nats con next Notifications ProcessNotifications --creds confs/x.creds
--- subject: Notifications

Headers:

  Nats-Request-Info: {"acc":"AAVOHCQ2R2QICRH427VEOWEXM3A5M4LG6OPFARZYHK2T77ZQCCKFUBFQ","rtt":941554}

Data:


Hello Notifications 2

Acknowledged message
```
