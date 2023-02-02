# Signalbot Example

An example bot for Signal that uses the [signalbot](https://github.com/filipre/signalbot) Python package.

## Getting Started

Please check out the example code to get an idea on how to develop your own bot or commands. Also see https://github.com/bbernhard/signal-cli-rest-api#getting-started to learn more about [signal-cli-rest-api](https://github.com/bbernhard/signal-cli-rest-api) and [signal-cli](https://github.com/AsamK/signal-cli). A good first step is to make the example bot work:

1. Run signal-cli-rest-api in `normal` mode first.
```bash
docker run -p 8080:8080 \
    -v $(PWD)/signal-cli-config:/home/.local/share/signal-cli \
    -e 'MODE=normal' bbernhard/signal-cli-rest-api
```

2. Open http://127.0.0.1:8080/v1/qrcodelink?device_name=local to link your account with the signal-cli-rest-api server

3. In your Signal app, open settings and scan the QR code. The server can now receive and send messages. The access key will be stored in `$(PWD)/signal-cli-config`.

4. Restart the server in `json-rpc` mode.
```bash
docker run -p 8080:8080 \
    -v $(PWD)/signal-cli-config:/home/.local/share/signal-cli \
    -e 'MODE=json-rpc' bbernhard/signal-cli-rest-api
```

5. The logs should show something like this. You can also confirm that the server is running in the correct mode by visiting http://127.0.0.1:8080/v1/about.
```
...
time="2022-03-07T13:02:22Z" level=info msg="Found number +491234567890 and added it to jsonrpc2.yml"
...
time="2022-03-07T13:02:24Z" level=info msg="Started Signal Messenger REST API"
```

6. The bot needs to listen to a group. Use the following snippet to get a group's `id` and `internal_id`:
```bash
curl -X GET 'http://127.0.0.1:8080/v1/groups/+49123456789' | python -m json.tool
```

7. Install `signalbot` and start `bot.py`. You need to pass following environment variables to make the example run:
- `SIGNAL_SERVICE`: Address of the signal service without protocol, e.g. `127.0.0.1:8080`
- `PHONE_NUMBER`: Phone number of the bot, e.g. `+49123456789`
- `GROUP_ID`: Group that the bot should listen to. Prefixed with `group.`
- `GROUP_INTERNAL_ID`: Group's `internal_id`

```bash
export SIGNAL_SERVICE="127.0.0.1"
export PHONE_NUMBER="+49123456789"
export GROUP_ID="group.qwerqwerqwerqwerqwerqwerqweqwer=="
export GROUP_INTERNAL_ID="asdfasdfasdfasdf="

# If you use pip
pip install signalbot
python bot.py

# If you use poetry
poetry install
poetry run python bot.py
```

8. The logs should indicate that one "producer" and three "consumers" have started. The producer checks for new messages sent to the linked account using a web socket connection. It creates a task for every registered command and the consumers work off the tasks. In case you are working with many blocking function calls, you may need to adjust the number of consumers such that the bot stays reactive.
```
INFO:root:[Bot] Producer #1 started
INFO:root:[Bot] Consumer #1 started
INFO:root:[Bot] Consumer #2 started
INFO:root:[Bot] Consumer #3 started
```

9. Send the message `ping` (case sensitive) to the group that the bot is listening to. The bot (i.e. the linked account) should respond with a `pong`. Confirm that the bot received a raw message, that the consumer worked on the message and that a new message has been sent.
```
INFO:root:[Raw Message] {"envelope":{"source":"+49123456789","sourceNumber":"+49123456789","sourceUuid":"fghjkl-asdf-asdf-asdf-dfghjkl","sourceName":"René","sourceDevice":3,"timestamp":1646000000000,"syncMessage":{"sentMessage":{"destination":null,"destinationNumber":null,"destinationUuid":null,"timestamp":1646000000000,"message":"pong","expiresInSeconds":0,"viewOnce":false,"groupInfo":{"groupId":"asdasdfweasdfsdfcvbnmfghjkl=","type":"DELIVER"}}}},"account":"+49123456789","subscription":0}
INFO:root:[Bot] Consumer #2 got new job in 0.00046 seconds
INFO:root:[Bot] Consumer #2 got new job in 0.00079 seconds
INFO:root:[Bot] Consumer #2 got new job in 0.00093 seconds
INFO:root:[Bot] Consumer #2 got new job in 0.00106 seconds
INFO:root:[Bot] New message 1646000000000 sent:
pong
```

## Example Commands

*Todo*
