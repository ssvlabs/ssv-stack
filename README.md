# SSV stack 
DISCLAIMER: SVV stack is an example repository how you can run SSV node with monitoring stack. It is not recommended to use it in production.

It includes Prometheus, which scrapes the SSV node and Grafana which has dashboard for the SSV node.

## How to run

### Prerequisites
* `docker`
* `docker-compose` version >= 2.0
* `ssv-node` version >= `v2.1.0`

### Clone the repository

```bash
git clone https://github.com/ssvlabs/ssv-stack.git
cd ssv-stack
```
### Adjust `env` file
```bash
cp ssv.env.example ssv.env
```
Edit the `ssv.env` file and adjust the settings to your needs.
The minimum you need to change is:
* `BEACON_NODE_ADDR` - the address of the beacon node
* `ETH_1_ADDR` - the address of the eth1 node
* `NETWORK` - the network you are running on (`mainnet`, `holesky`, etc)

Check the `ssv.example.env` file for details for all the others settings you can adjust.

### Run the stack
run in foreground
```bash
docker-compose up
```
or run in background
```bash
docker-compose up -d
```

### Access Grafana
Open your browser and go to [http://localhost:3000](http://localhost:3000) and login with `admin`/`admin`. You can change the password later.

**WARNING**: Grafana and Prometheus is running only on the `localhost` by default. You should change the password and/or restrict access to it. If you expose Grafana and Prometheus to the internet, you should be extra careful and make sure they are secured.

## SSV node private key
This example generates a new ssv node private key which is encrypted by newly generated random password. Both files are persisted in the `./ssv-node-data` directory. If you want to use your own private key, you can replace the `./ssv-node-data/private_key` and `./ssv-node-data/password` files with your own. You might need to create dir if it does not exist.
