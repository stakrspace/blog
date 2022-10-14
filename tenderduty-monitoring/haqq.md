# Haqq monitoring with TenderDuty

[Tenderduty](https://github.com/blockpane/tenderduty) is a monitoring tool for Cosmos chains. 

If you don't want to run prometheus+node_exporter+grafana then it would be good choice.


Install Docker 

```curl -fsSL https://get.docker.com | sh```

Create folder

```mkdir tenderduty && cd tenderduty```

Get config

```docker run --rm ghcr.io/blockpane/tenderduty:latest -example-config >config.yml```

Edit *config.yml* and add chains, notification methods, etc.

```docker run -d --name tenderduty -p "8888:8888" -p "28686:28686" --restart unless-stopped -v $(pwd)/config.yml:/var/lib/tenderduty/config.yml ghcr.io/blockpane/tenderduty:latest```

Check logs

```docker logs -f --tail 20 tenderduty```

The dashboard will be public at `http://YOUR_SERVER_IP:8888`

Many thanks to @CrazySerGo
