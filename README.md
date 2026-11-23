# coinhive-stratum-mining-proxy

Browser miners built with CoinHive.js need a WebSocket endpoint to submit work to a Stratum mining pool. This Python proof of concept connects those clients to a Monero pool instead of CoinHive's backend.

**Legacy software:** CoinHive shut down in 2019. These instructions describe the original integration; they require a compatible copy of CoinHive.js and a compatible pool. Do not assume compatibility with current Monero mining software.

## Features

- Forward CoinHive.js mining traffic to a configured Stratum TCP host and port.
- Pass a Monero address to the pool, with an optional worker name when the pool supports it.
- Accept secure WebSocket connections with a supplied TLS key and certificate.
- Report connected clients, uptime, and total hashes through the `/stats` endpoint.
- Protect the statistics endpoint with an optional password.

## Installation

From the project source directory, install the Python dependencies:

```sh
cd coinhive-stratum-mining-proxy
python -m pip install -v -r requirements.txt
```

The original setup requires Python development headers, a C compiler, OpenSSL development headers, and libffi development headers. Some environments also require musl development headers. Package names vary by operating system.

The donor does not specify a supported Python version. Its dependencies may need a legacy environment; current Python compatibility is not established.

## Usage

Start the proxy with your pool's Stratum TCP hostname and port:

```sh
python coinhive-stratum-mining-proxy.py <stratum tcp host> <stratum tcp port>
```

The original project used this example:

```sh
python coinhive-stratum-mining-proxy.py xmr-eu1.nanopool.org 14444
```

That is a historical endpoint, not a verified current service. Replace it with the address and port supplied by your compatible pool.

### Connect a browser miner

Load a compatible copy of CoinHive.js before this code. The original hosted library URL is no longer a dependable installation source.

```html
<script>
CoinHive.CONFIG.WEBSOCKET_SHARDS = [["ws://localhost:8892/proxy"]];

var miner = new CoinHive.Anonymous('YOUR_MONERO_ADDRESS');
miner.start();
</script>
```

`localhost` refers to the machine running the browser. For remote clients, replace it with your proxy's reachable hostname.

If the pool supports worker names, replace the miner constructor with:

```javascript
var miner = new CoinHive.User('YOUR_MONERO_ADDRESS', 'YOUR_WORKER_NAME');
```

Only start mining after the visitor has explicitly agreed. Provide a visible way to stop it, such as a control that calls `miner.stop()`.

### Enable secure WebSockets

For local testing, generate a self-signed certificate:

```sh
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 100 -nodes
```

Start the proxy with the certificate and key:

```sh
python coinhive-stratum-mining-proxy.py <stratum tcp host> <stratum tcp port> --ssl=key.pem:cert.pem
```

If an intermediate certificate is needed, use `--ssl=key.pem:cert.pem:chain.pem`. Change the browser endpoint to `wss://YOUR_PROXY_HOST:8892/proxy` and use a certificate trusted by the browser. HTTPS pages should use secure WebSockets.

### Read statistics

Set a statistics password when starting the proxy:

```sh
python coinhive-stratum-mining-proxy.py <stratum tcp host> <stratum tcp port> --password=YOUR_PASS
```

Query the endpoint:

```sh
curl 'http://localhost:8892/stats?password=YOUR_PASS'
```

Example response:

```json
{"total_hashes": 0, "uptime": 15.53, "clients": 1}
```

No statistics password is configured by default. The password option protects statistics; it is not documented as access control for mining connections.

<!-- seorepo:network:start -->
## More tools from our network

- [**octo-cli**](https://github.com/boostlyapi/octo-cli) — CLI на Go для публикации данных из PostgreSQL, MSSQL и MySQL через бессерверные веб-сервисы в OpenFaaS или Knative. Конфигурация сервиса задаётся в YAML.
- [**tool**](https://github.com/boostlyapi/tool) — CLI на Go для Apache Kafka: чтение и отправка сообщений в JSON, просмотр топиков, управление смещениями consumer groups и базовое администрирование.
- [**docker-ikev2-vpn-server**](https://github.com/Cabral-it/docker-ikev2-vpn-server) — Docker-based IKEv2 VPN server with shared-secret authentication and .mobileconfig generation for iOS and macOS, based on gaomd/docker-ikev2-vpn-server.
- [**akrobateo**](https://github.com/Dyooooou/akrobateo) — A Go Kubernetes operator that exposes LoadBalancer services through node hostPorts using DaemonSets and synchronizes their load balancer addresses.
- [**node-hnsn**](https://github.com/ghb13/node-hnsn) — A TypeScript node for decentralized document timestamping with Bitcoin and IPFS, with an HTTP API for signed work claims and file uploads.
- [**wg-access-server**](https://github.com/EngHirsi/wg-access-server) — A WireGuard VPN server with a web UI for user authentication, one-click device registration, QR-code setup, and configurable network isolation.
<!-- seorepo:network:end -->

## License

MIT. This project is based on [x25/coinhive-stratum-mining-proxy](https://github.com/x25/coinhive-stratum-mining-proxy). Retain the original copyright and MIT permission notice in copies or substantial portions of the software.

This project is not endorsed by or affiliated with CoinHive.
