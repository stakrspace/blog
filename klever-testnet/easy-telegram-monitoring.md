The best way to monitor the node health status would be Grafana+Prometheus+AlertBot, will cover it in the next post.
Until then, to monitor the node fast and easy, you could just use the simple way to send the status to your TG:

- place the alerting script in your home dir, the tgalarm.sh would be like

```
#!/bin/bash

telegram_bot_token="<your tg token>"
telegram_chat_id="<your chat id>"

Title="$1"
Message="$2"

curl -s \
 --data parse_mode=HTML \
--data chat_id=${telegram_chat_id} \
 --data text="<b>${Title}</b>%0A${Message}" \
 --request POST https://api.telegram.org/bot${telegram_bot_token}/sendMessage
 ```

- to check that the node process is up and synced, just add to your crontab the approporiate timing for the process

``` */10 * * * * cd && docker logs --tail 5 klever-node |grep "Status: synchronized" || ./tgalarm.sh $HOSTNAME klever_not_synchronized ```
