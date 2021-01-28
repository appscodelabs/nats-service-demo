# nats-service-demo

## Accounts

- My configuration for NATS Server export/import
    - confs/server.conf is the server configuration


    - confs/X-account.jwt is the export account, here included `exports` preview
        ```
        "exports": [
            {
                "name": "Events",
                "subject": "Events",
                "type": "stream"
            },
            {
                "name": "Notifications",
                "subject": "Notifications",
                "type": "service",
                "response_type": "Stream"
            }
        ]
        ```
    - confs/Admin-account.jwt is the import account, here included `imports` preview
        ```
        "imports": [
            {
                "name": "Events",
                "subject": "Events",
                "account": "AAMBZABDGEJKZR3TJYH7MG44LYRO5OOM2XNRCGKJ25DPHD6CL6KUSJBK",
                "local_subject": "user.x.Events",
                "type": "stream"
            },
            {
                "name": "Notifications",
                "subject": "Notifications",
                "account": "AAMBZABDGEJKZR3TJYH7MG44LYRO5OOM2XNRCGKJ25DPHD6CL6KUSJBK",
                "local_subject": "user.x.Notifications",
                "type": "service"
            }
        ]
        ```


## Run NATS Server
I build the `nats-server` binary from the latest `master` branch.

```shell
$ cd $HOME/go/src/github.com/nats-io/nats-server
$ go install -v ./...

$ nats-server -c confs/server.conf
```

## Publish/Subscribe to Channel

### pub/sub to `stream` type channel

```shell
$ $HOME/go/src/github.com/appscodelabs/nats-service-demo

$ nats sub user.x.Events --creds confs/admin.creds
13:57:20 Subscribing on user.x.Events
[#1] Received on "user.x.Events"
Hello there
```

```shell
$ nats pub Events "Hello there" --creds confs/x.creds
13:57:46 Published 11 bytes to "Events"
```
In `stream` type, I am receiving all the messages
### pub/sub to `service` type channel

```shell
$ $HOME/go/src/github.com/appscodelabs/nats-service-demo

$ nats sub Notifications --creds confs/x.creds
14:05:58 Subscribing on Notifications
```

```shell
$ nats pub user.x.Notifications "Hello Notifications" --creds confs/admin.creds
14:07:31 Published 19 bytes to "user.x.Notifications"

```
But in `service` type, I am not receiving any message.

I also tried subscribing to `">"` from the `Export` account but got nothing.

```shell
$ nats sub ">" --creds confs/x.creds
13:59:38 Subscribing on >
```
Although in `stream` type I receivied messages.

```shell
$ nats sub ">" --creds confs/admin.creds
14:00:09 Subscribing on >
[#1] Received on "user.x.Events"
Hello there
```
