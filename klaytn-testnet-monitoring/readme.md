1. install docker and docker-compose
2. check that the port 3000 is open
3. download folders and files from this folder https://github.com/papanomad535/blog/new/main/klaytn-testnet-monitoring
4. change configs if necessary, put the telegram_user_id and bot_token in alertmanager config
5. here is the link to the official Klaytn monitoring documentation https://testnet-docs.blockpi.io/guide-for-operators/testnet-1/monitoring#monitoring-hypernode-status
6. run ```docker compose up -d``` and open in browser http://your-host-ip:3000 with admin/admin default credentials
7. Many thanks to @vbloher and his Cosmos monitoring tool https://github.com/vbloher/monitoring-tool
