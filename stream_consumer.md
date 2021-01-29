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

As you can see, If we `Export/Import` subject of type `stream`, it works fine. But when we `Export/Import` subject of type `service`, that's where it doesn't works.


