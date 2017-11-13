node-cryptonote-pool
====================

High performance Node.js (with native C addons) mining pool for CryptoNote based coins such as Electroneum, Monero, Bytecoin, QuazarCoin, HoneyPenny, etc..
Comes with lightweight example front-end script which uses the pool's AJAX API.



#### Table of Contents
* [Features](#features)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Configure Easyminer](#3-optional-configure-cryptonote-easy-miner-for-your-pool)
  * [Starting the Pool](#4-start-the-pool)
  * [Host the front-end](#5-host-the-front-end)
  * [Customizing your website](#6-customize-your-website)
  * [Upgrading](#upgrading)
* [Setting up Testnet](#setting-up-testnet)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Donations](#donations)
* [Credits](#credits)
* [License](#license)


#### Features

* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
  * Splintered transactions to deal with max transaction size
  * Minimum payment threshold before balance will be paid out
  * Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Detailed logging
* Ability to configure multiple ports - each with their own difficulty
* Variable difficulty / share limiter
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* Live stats API (using AJAX long polling with CORS)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, etc)
  * Blocks found (pending, confirmed, and orphaned)
* An easily extendable, responsive, light-weight front-end using API to display data
* Worker login validation (make sure miners are using proper wallet addresses for mining)


### Community / Support

* [CryptoNote Forum](https://forum.cryptonote.org/)
* [Bytecoin Github](https://github.com/amjuarez/bytecoin)
* [Electroneum Github](https://github.com/electroneum/electroneum)

#### Pools Using This Software

* https://asiapool.electroneum.com
* https://eupool.electroneum.com
* https://uspool.electroneum.com
* http://ucrypto.com
* http://electroneum.hashparty.io
* http://pool.electroneumcharts.com
* http://etnhash.com
* https://easyhash.io/pools/etn
* https://etn.uax.io/

Usage
===

#### Requirements
* Coin daemon [electroneum repo](https://www.github.com/electroneum/electroneum)
  * To compile the coin you'll need several packages as detailed in the repo's readme. A quick one line command is below:
  * `sudo apt install build-essential cmake pkg-config libboost-all-dev libssl-dev libunbound-dev`
* [Node.js](http://nodejs.org/) v0.10.48 ([follow these installation instructions](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))
* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install libssl-dev`
* Boost is required for the cryptonote-util module
  * For Ubuntu: `sudo apt-get install libboost-all-dev`


##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.


[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).


#### 1) Downloading & Installing


Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/electroneum/electroneum-pool.git pool
cd pool
npm update
```

#### 2) Configuration


*Warning for Cryptonote coins other than Electroneum:* this software may or may not work with any given cryptonote coin.
Be wary of altcoins that change the number of minimum coin units because you will have to reconfigure several config
values to account for those changes. Unless you're offering a bounty reward - do not open an issue asking for help
getting a coin other than electroneum working with this software.


Copy the `config_example.json` file to `config.json` then overview each options and change any to match your preferred setup.


Explanation for each field:
```javascript
{
    "coin": "electroneum",
    "symbol": "ETN",
    "logging": {
        "files": {
            "level": "info",
            "directory": "logs",
            "flushInterval": 5
        },

        "console": {
            "level": "info",
            "colors": true
        }
    },
    "poolServer": {
        "enabled": true,
        "clusterForks": "auto",
        "poolAddress": "WALLETADDRESSHERE",
        "blockRefreshInterval": 1000,
        "minerTimeout": 900,
        "ports": [
            {
                "port": 3333,
                "difficulty": 100,
                "desc": "Low end hardware"
            },
            {
                "port": 5555,
                "difficulty": 2000,
                "desc": "Mid range hardware"
            },
            {
                "port": 7777,
                "difficulty": 10000,
                "desc": "High end hardware"
            }
        ],
        "varDiff": {
            "minDiff": 2,
            "maxDiff": 100000,
            "targetTime": 100,
            "retargetTime": 30,
            "variancePercent": 30,
            "maxJump": 100
        },
        "shareTrust": {
            "enabled": true,
            "min": 10,
            "stepDown": 3,
            "threshold": 10,
            "penalty": 30
        },
        "banning": {
            "enabled": true,
            "time": 600,
            "invalidPercent": 25,
            "checkThreshold": 30
        },
        "slushMining": {
            "enabled": false,
            "weight": 300,
            "lastBlockCheckRate": 1
        }
    },
    "payments": {
        "enabled": true,
        "interval": 600,
        "maxAddresses": 10,
        "mixin": 0,
        "transferFee": 1,
        "minPayment": 10000,
        "denomination": 100
    },
    "blockUnlocker": {
        "enabled": true,
        "interval": 30,
        "depth": 20,
        "poolFee": 1.8,
        "devDonation": 0.1,
        "coreDevDonation": 0.1
    },
    "api": {
        "enabled": true,
        "hashrateWindow": 600,
        "updateInterval": 3,
        "port": 8117,
        "blocks": 30,
        "payments": 30,
        "password": "test"
    },
    "daemon": {
        "host": "127.0.0.1",
        "port": 18081
    },
    "wallet": {
        "host": "127.0.0.1",
        "port": 8082
    },
    "redis": {
        "host": "127.0.0.1",
        "port": 6379,
        "auth": null
    }
}
```

#### 3) [Optional] Configure cryptonote-easy-miner for your pool
Your miners that are Windows users can use [cryptonote-easy-miner](https://github.com/zone117x/cryptonote-easy-miner)
which will automatically generate their wallet address and stratup multiple threads of simpleminer. You can download
it and edit the `config.ini` file to point to your own pool.
Inside the `easyminer` folder, edit `config.init` to point to your pool details
```ini
pool_host=example.com
pool_port=5555
```

Rezip and upload to your server or a file host. Then change the `easyminerDownload` link in your `config.json` file to
point to your zip file.

#### 4) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.


#### 5) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Minimum units in a single coin, for Electroneum its 100. */
var coinUnits = 100;

/* Pool server host to instruct your miners to point to.  */
var poolHost = "poolhost.com";

/* IRC Server and room used for embedded KiwiIRC chat. */
var irc = "irc.freenode.net/#electroneum";

/* Contact email address. */
var email = "support@poolhost.com";

/* Market stat display params from https://www.cryptonator.com/widget - Not used for Electroneum pool */
var cryptonatorWidget = ["XMR-BTC", "XMR-USD", "XMR-EUR", "XMR-GBP"];

/* Download link to cryptonote-easy-miner for Windows users. */
var easyminerDownload = "https://github.com/zone117x/cryptonote-easy-miner/releases/";

/* Used for front-end block links. */
var blockchainExplorer = "https://blockexplorer.electroneum.com/block/";

/* Used by front-end transaction links. */
var transactionExplorer = "https://blockexplorer.electroneum.com/tx;

```

#### 6) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.


#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [New Relic](http://newrelic.com/)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever)


Donations
---------
* ETN: `etnkCFXeExRaBhbDNq66TQ3rF9XSkMreei8t1jXfT2ndgY6bnsNEUtEKbEhba3YHk2CuPCpxZFrGNcAJ9Lv22ACY7eapPgQtNb`
* XMR: `48rB9X6ygZQ4uNBSKJP3MKU9iFK87e8vejEi762eNdTPB61A9VayUnL75HHi2SER3gf5i9PXpiS2kiCwDyanvNm5S5Aq3WF`

Credits
===

* [LucasJones](//github.com/LucasJones) - Co-dev on this project; did tons of debugging for binary structures and fixing them. Pool couldn't have been made without him.
* [surfer43](//github.com/iamasupernova) - Did lots of testing during development to help figure out bugs and get them fixed
* [wallet42](http://moneropool.com) - Funded development of payment denominating and min threshold feature
* [Wolf0](https://bitcointalk.org/index.php?action=profile;u=80740) - Helped try to deobfuscate some of the daemon code for getting a bug fixed
* [Tacotime](https://bitcointalk.org/index.php?action=profile;u=19270) - helping with figuring out certain problems and lead the bounty for this project's creation

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
