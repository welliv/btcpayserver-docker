# Agent quick reference

Use this while following `SKILL.md`. Do not paste fragment names at the user unless they ask.

## Trigger

Install / deploy / set up BTCPay Server on a VPS or dedicated server.

## Official installer only

From the `btcpayserver-docker` checkout, as root:

```bash
. ./btcpay-setup.sh -i
```

The script must be sourced. It installs Docker, generates compose, registers systemd, and starts BTCPay.

## Answer mapping

| Question | User answer | Export |
|---|---|---|
| Domain | `btcpay.example.com` | `BTCPAY_HOST`, `BTCPAYGEN_REVERSEPROXY=nginx`, `LETSENCRYPT_EMAIL` |
| Domain | none | omit `BTCPAY_HOST` (Tor still works; it is recommended by default) |
| Network | real payments | `NBITCOIN_NETWORK=mainnet` |
| Network | testing | `NBITCOIN_NETWORK=testnet` |
| Disk | 100 GB | `BTCPAYGEN_ADDITIONAL_FRAGMENTS=opt-save-storage` |
| Disk | 50 GB (default) | `BTCPAYGEN_ADDITIONAL_FRAGMENTS=opt-save-storage-s` |
| Disk | 25 GB | `BTCPAYGEN_ADDITIONAL_FRAGMENTS=opt-save-storage-xs` |
| Disk | 5 GB | `BTCPAYGEN_ADDITIONAL_FRAGMENTS=opt-save-storage-xxs` (no Lightning) |
| Lightning | yes | `BTCPAYGEN_LIGHTNING=clightning` (or `lnd` if they ask) |
| Lightning | no | `BTCPAYGEN_LIGHTNING=none` |

Always:

```bash
export BTCPAYGEN_CRYPTO1="btc"
export BTCPAY_ENABLE_SSH=true
```

Tor (`opt-add-tor`) is a recommended fragment on `btcpayserver.yml`. Do not add it to `BTCPAYGEN_ADDITIONAL_FRAGMENTS`. Do not set `BTCPAY_LOCAL_TOR`.

## Verify

```bash
docker ps
cat /var/lib/docker/volumes/generated_tor_servicesdir/_data/BTCPayServer/hostname
bitcoin-cli.sh getblockchaininfo
```

First registration on the site is the admin account.

## Manage

```bash
btcpay-restart.sh
btcpay-up.sh
btcpay-down.sh
btcpay-update.sh
changedomain.sh new.example.com
```
