# SSV Stack 
DISCLAIMER: SSV stack is an example repository how you can run SSV node with monitoring stack. It is not recommended to use it in production.

It includes Prometheus, which scrapes the SSV node and Grafana which has dashboard for the SSV node.

## How to run

### Prerequisites
* `docker`
* `docker compose` version >= 2.0
* `ssv-node` version >= `v2.5.0` (required from the Gloas fork on)

### Clone the repository

```bash
git clone https://github.com/ssvlabs/ssv-stack.git
cd ssv-stack
```
### Adjust `env` file
```bash
cp ssv.example.env ssv.env
```
Edit the `ssv.env` file and adjust the settings to your needs.
The minimum you need to change is:
* `BEACON_NODE_ADDR` - the address of the beacon node
* `ETH_1_ADDR` - the address of the eth1 node
* `NETWORK` - the network you are running on (`mainnet`, `holesky`, etc)

Check the `ssv.example.env` file for details for all the others settings you can adjust.

### Alerting rules
You can adjust the alerting rules in the `./prometheus/alert-rules.yml` file.

For this to work, you can configure alertmanager with your telegram bot token and chat id. You can do this by editing the `./alertmanager/alertmanager.yml` file. Put the raw token in the `alertmanager/telegram_bot_token.txt`.

### SSV node private key
#### Generate New Key
This example generates a new ssv node private key which is encrypted by a newly generated random password. Both files will be in the `./ssv-node-data` directory. 
#### Use Existing Key
If you want to use your own private key, you can go to the `./ssv-node-data`folder and put in your private_key and password files such as: `./ssv-node-data/private_key` and `./ssv-node-data/password`. You might need to create the `./ssv-node-data` directory if it does not exist.

### Run the stack
Run in foreground
```bash
docker compose up
```
Run in background
```bash
docker compose up -d
```

### Glamsterdam (Gloas) upgrade
Before the Gloas fork on your network, upgrade your Execution and Beacon clients to releases that support Glamsterdam. Upgrade all Beacon nodes first, then pull the latest SSV node image and restart the node, so it reads the new fork schedule:
```bash
docker compose pull ssv-node
docker compose up -d --force-recreate ssv-node
```
If the fork date changes, upgrade your Beacon nodes again and recreate the SSV node the same way. Otherwise it shuts down near the old fork epoch.

See [Configuring ePBS (Gloas)](https://docs.ssv.network/operators/operator-node/setup-sidecars/configuring-epbs) for the fork schedule and details.

### Builders configuration (optional)
The Gloas (ePBS) `Builders` block, used to connect to specific builders or relays, is available only in YAML. To use it:
* Create `./ssv-node-data/builders.yaml` with the block, for example:
```yaml
Builders:
  Entries:
    - URL: "https://builder.example.com"
```
* Add `SHARE_CONFIG=/data/builders.yaml` to `ssv.env`, or uncomment it if your `ssv.env` already contains it
* Recreate the node so it loads the file, also after any later change to `builders.yaml`:
```bash
docker compose up -d --force-recreate ssv-node
```
If `builders.yaml` is missing or the `Builders` block is invalid, the node won't start. Check `docker compose logs ssv-node` after recreating it.

The `Builders` block must be identical on all operators of every cluster that shares a validator. See [Direct builder connections](https://docs.ssv.network/operators/operator-node/setup-sidecars/configuring-epbs#direct-builder-connections-optional) for details, and [Using relays after Gloas](https://docs.ssv.network/operators/operator-node/setup-sidecars/configuring-epbs#using-relays-after-gloas) to connect to relays.

### Run with DKG node
You can also run the stack with DKG, simplifying the setup process.
First, you need to edit the config in `./dkg-data/operator.yaml`.
The minimum you need to change is:
* `operatorID` - the ID of your operator
* `ethEndpointURL` - HTTP Address of your Execution node endpoint

Also, you'll need to expose port `3030` on your firewall for the DKG endpoint to be available.

Make sure you're in `ssv-stack` directory, and then run the command:
```bash
docker compose --profile dkg up
```
Or run in background
```bash
docker compose --profile dkg up -d
```

## Access Grafana
Open your browser and go to [http://localhost:3000](http://localhost:3000) and login with `admin`/`admin`. You can change the password later.

In the `Dashboards` section you should be able to find the `SSV Operational` dashboard. Unless the node is registered with the SSV network and has validators, some of the dashboard's columns might be empty.

**WARNING**: Grafana and Prometheus is running only on the `localhost` by default. You should change the password and/or restrict access to it. If you expose Grafana and Prometheus to the internet, you should be extra careful and make sure they are secured.

## Benchmarking tool
The Stack comes with a benchmarking tool which is [ssv-pulse](https://github.com/ssvlabs/ssv-pulse).
It helps with troubleshooting and the results can be interpreted with our [Troubleshooting page](https://docs.ssv.network/operators/operator-node/maintenance/troubleshooting/#checklists-for-troubleshooting).
To run the tool:
* Edit `docker-compose.yaml`, at the very end of the file
* Change `--consensus-addr` to your Consensus HTTP and `--execution-addr` to Execution HTTP
* If running on Hoodi - uncomment `--network=hoodi`
* Run `docker compose run ssv-pulse`
* The tool will be running for 60 minutes and produce a table with results