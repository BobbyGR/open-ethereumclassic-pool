# Open Ethereum Classic Pool
Ethereum Classic pool software based on [sammy007's open-ethereum-pool](https://github.com/sammy007/open-ethereum-pool)

# Prerequisites
* Building from source
  * `sudo apt-get install build-essential`
* [Git](http://git-scm.com/)
  * `sudo apt-get install git`
* [Nginx](https://nginx.org)
  * `sudo apt-get install nginx`
* [Golang](https://golang.org/doc/install)
  * `tar -C /usr/local -xzf go*.linux*tar.gz`
  * bash
    * `echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc`
  * zsh
    * `echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.zshrc`
* [Redis](https://redis.io/)
  * `sudo apt-get install redis-server`
* [NVM/Node.js](https://github.com/creationix/nvm)
  * Install NVM, then:
  * `nvm install lts/carbon && nvm alias default lts/carbon`
* [Bower](http://bower.io/)
  * `npm install -g bower`
* [Geth](https://github.com/ethereumproject/go-ethereum/releases)
  * `tar xfz geth-classic-linux-v*.tar.gz`

# Important
**CHAIN MUST BE FULLY SYNCED BEFORE STARTING POOL**

**WALLET MUST BE UNLOCKED FOR PAYMENTS**
```bash
# Attach to running geth instance & unlock
./geth attach
geth> personal.unlockAccount('0x1413bE04f3767fB606D5B50091b2E92E3241eD22', null, 0)
# First argument is the wallet address
# Second argument is the password to unlock wallet.  LEAVE IT NULL
# Third argument specifies how long to leave unlocked. LEAVE AT 0 (forever)
```

# Installation
## Frontend
From within the root directory of this repo
```bash
cd www
nano config/environemnt.js
#  Change:
#    ApiUrl
#    HttpHost
#    HttpPort
#    StratumHost
#    StratumPort
#    PoolFee
#    PayoutThreshold
#    BlockTime (only for non ETC/ETH networks with different blocktimes)
npm install
bower install
./build.sh
# Configure Nginx to serve /api from /path/to/open-ethereumclassic-pool/www/dist/
```

## Backend
```
git clone git@github.com:mikeyb/open-ethereumclassic-pool.git
cd open-ethereumclassic-pool
make
```
### Configurations
It is highly recommended to run each service individually. Create configs for each service, disabling or removing service entries from each config
* `api.json`
```json
{
    "threads": 2,
    "coin": "etc",
    "name": "main",
    "api": {
        "enabled": true,
        "purgeOnly": false,
        "purgeInterval": "10m",
        "listen": "0.0.0.0:8080",
        "statsCollectInterval": "5s",
        "hashrateWindow": "30m",
        "hashrateLargeWindow": "3h",
        "luckWindow": [64, 128, 256],
        "payments": 100,
        "blocks": 100
    },
    "upstreamCheckInterval": "5s",
    "upstream": [
        {
            "name": "main",
            "url": "http://localhost:8545",
            "timeout": "10s"
        }
    ],
    "redis": {
        "endpoint": "localhost:6379",
        "poolSize": 10,
        "database": 0,
        "password": "SomeS3cured!Passw0rd"
    }
}
```
* `proxy.json`
```json
{
	"threads": 2,
	"coin": "etc",
	"name": "main",
	"proxy": {
		"enabled": true,
		"listen": "0.0.0.0:8888",
		"limitHeadersSize": 1024,
		"limitBodySize": 256,
		"behindReverseProxy": true,
		"blockRefreshInterval": "120ms",
		"stateUpdateInterval": "3s",
		"difficulty": 2000000000,
		"hashrateExpiration": "3h",
		"healthCheck": true,
		"maxFails": 100,
		"stratum": {
			"enabled": true,
			"listen": "0.0.0.0:8008",
			"timeout": "20s",
			"maxConn": 8192
		},
		"policy": {
			"workers": 8,
			"resetInterval": "60m",
			"refreshInterval": "1m",
			"banning": {
				"enabled": false,
				"ipset": "blacklist",
				"timeout": 1800,
				"invalidPercent": 30,
				"checkThreshold": 30,
				"malformedLimit": 5
			},
			"limits": {
				"enabled": false,
				"limit": 30,
				"grace": "5m",
				"limitJump": 10
			}
		}
	},
	"upstreamCheckInterval": "5s",
	"upstream": [
		{
			"name": "main",
			"url": "http://localhost:8545",
			"timeout": "10s"
		}
	],
	"redis": {
		"endpoint": "localhost:6379",
		"poolSize": 10,
		"database": 0,
		"password": "SomeS3cured!Passw0rd"
	}
}
```
* `unlocker.json`
```json
{
	"threads": 2,
	"coin": "etc",
	"name": "main",
	"upstreamCheckInterval": "5s",
	"upstream": [
		{
			"name": "main",
			"url": "http://localhost:8545",
			"timeout": "10s"
		}
	],
	"redis": {
		"endpoint": "localhost:6379",
		"poolSize": 10,
		"database": 0,
		"password": "SomeS3cured!Passw0rd"
	},
	"unlocker": {
		"enabled": true,
		"poolFee": 1.0,
		"poolFeeAddress": "0x1413bE04f3767fB606D5B50091b2E92E3241eD22",
		"depth": 120,
		"immatureDepth": 20,
		"keepTxFees": false,
		"interval": "10m",
		"daemon": "http://localhost:8545",
		"timeout": "10s"
	}
}
```
* `payouts.json`
```json
{
	"threads": 2,
	"coin": "etc",
	"name": "main",
	"upstreamCheckInterval": "5s",
	"upstream": [
		{
			"name": "main",
			"url": "http://localhost:8545",
			"timeout": "10s"
		}
	],
	"redis": {
		"endpoint": "localhost:6379",
		"poolSize": 10,
		"database": 0,
		"password": "SomeS3cured!Passw0rd"
	},
	"payouts": {
		"enabled": true,
		"requirePeers": 10,
		"interval": "30m",
		"daemon": "http://localhost:8545",
		"timeout": "10s",
		"address": "0x1413bE04f3767fB606D5B50091b2E92E3241eD22",
		"gas": "21000",
		"gasPrice": "50000000000",
		"autoGas": true,
		"threshold": 500000000,
		"bgsave": false
	}
}
```
## Running Backend - Production
```bash
# Use something like tmux or screen to run each process in its own shell
./build/bin/open-ethereum-pool api.json
./build/bin/open-ethereum-pool proxy.json
./build/bin/open-ethereum-pool unlocker.json
./build/bin/open-ethereum-pool payouts.json
```

# DEVELOPMENT ONLY
## Building
```
ember build
ember build --environment production
```
## Running - Development
```
ember server
# Visit your app at [http://localhost:4200](http://localhost:4200).
```
## Running Tests
```
ember test
ember test --server
```
## Code Generators
Make use of the many generators for code, try `ember help generate` for more details