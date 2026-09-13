---
name: btcpay-install
description: Guide a non-technical user through installing BTCPay Server with the official Docker setup. Use when the user asks to install, deploy, set up, or self-host BTCPay Server, a Bitcoin payment processor, or a BTCPay node on a VPS or dedicated server.
---

# BTCPay Server install (non-technical)

Walk the user through a few plain-language questions, then install with the official `btcpay-setup.sh`. Do not invent wrapper scripts, unofficial fragments, or extra env vars.

Talk like a patient installer. One question at a time. No jargon unless the user asks. Do not mention fragment names, compose files, or container names in user-facing replies.

## Before you start

You must be on the machine that will host BTCPay (or have working root SSH to it). This skill cannot install onto a laptop the user is only chatting from.

Confirm, without dumping a checklist at them:

1. You can run commands as root (`sudo su -` on Linux).
2. The OS is Ubuntu or Debian. Other Linux may work; say so if it is not Ubuntu/Debian.
3. There is enough free disk for the storage choice (see mapping below).
4. Ports 80 and 443 are reachable from the internet if they want a normal website. Lightning also needs 9735.

If BTCPay is already installed (look for `btcpay-setup.sh` on `PATH`, `/etc/profile.d/btcpay-env.sh`, or a `btcpayserver-docker` checkout), ask whether they want to change settings or start over. Do not reinstall on top of a live server without confirmation.

If this repository is not already cloned, clone it as root:

```bash
sudo su -
mkdir -p /root/BTCPayServer
cd /root/BTCPayServer
git clone https://github.com/btcpayserver/btcpayserver-docker
cd btcpayserver-docker
```

All later commands run from that checkout, as root, with the setup script **sourced** (`. ./btcpay-setup.sh -i`), never executed as `./btcpay-setup.sh`.

## Questions (one at a time)

Ask these in order. Suggest the starred default when they are unsure.

### 1. Domain

A domain looks like `btcpay.example.com`.

- **Yes:** Ask for the hostname. Detect the public IP (`curl -4 -fsS https://ifconfig.me`) and tell them to point the domain's A record at that IP. Wait until `dig +short <host> A` (or `getent hosts`) returns the server before continuing with HTTPS. Set `BTCPAY_HOST`, `BTCPAYGEN_REVERSEPROXY=nginx`, and `LETSENCRYPT_EMAIL` (ask for an email, or leave it empty). Optionally set `REVERSEPROXY_DEFAULT_HOST="$BTCPAY_HOST"` so visiting the raw IP still reaches BTCPay.
- **No:** They can still install. Tor is on by default and works without DNS. Leave `BTCPAY_HOST` unset. Clearnet HTTPS will not work until they add a domain later with `changedomain.sh`.

### 2. Real payments or testing?

| Choice | `NBITCOIN_NETWORK` |
|---|---|
| Receive real payments ⭐ | `mainnet` |
| Test with fake bitcoin | `testnet` |

### 3. How much disk for Bitcoin?

| Choice | Fragment | Notes |
|---|---|---|
| 100 GB (~1 year of history) | `opt-save-storage` | |
| 50 GB (~6 months) ⭐ | `opt-save-storage-s` | Same default as the README |
| 25 GB (~3 months) | `opt-save-storage-xs` | Smallest size that still supports Lightning |
| 5 GB (~2 weeks) | `opt-save-storage-xxs` | Lightning is **not** supported |

If they pick 5 GB and also want Lightning, explain the limit and bump storage to 25 GB or drop Lightning. Do not combine `opt-save-storage-xxs` with a Lightning implementation.

### 4. Fast, cheap payments (Lightning)?

| Choice | `BTCPAYGEN_LIGHTNING` |
|---|---|
| Yes ⭐ | `clightning` (official README default). Use `lnd` only if they ask for LND. |
| No, on-chain only | `none` |

### 5. Confirm and install

Repeat their choices in one short paragraph. Ask them to say go. Then export the variables and source setup.

## Install command

Always include Bitcoin, nginx, and SSH access to the host (so the web UI can update the server). Tor is a **recommended** fragment (`opt-add-tor`) and is included automatically unless excluded — do not add it to `BTCPAYGEN_ADDITIONAL_FRAGMENTS`, and do not invent `BTCPAY_LOCAL_TOR` or prune fragment names like `opt-add-prune-*`.

Typical mainnet + domain + 50 GB + Lightning:

```bash
export BTCPAY_HOST="btcpay.example.com"
export NBITCOIN_NETWORK="mainnet"
export BTCPAYGEN_CRYPTO1="btc"
export BTCPAYGEN_ADDITIONAL_FRAGMENTS="opt-save-storage-s"
export BTCPAYGEN_REVERSEPROXY="nginx"
export BTCPAYGEN_LIGHTNING="clightning"
export BTCPAY_ENABLE_SSH=true
export LETSENCRYPT_EMAIL="me@example.com"
. ./btcpay-setup.sh -i
```

Without a domain, omit `BTCPAY_HOST` and `LETSENCRYPT_EMAIL`. Keep the other exports.

Setup takes several minutes. Do not interrupt it. Tell the user you are installing and will report back.

## After install

1. Check containers: `docker ps`. BTCPay, postgres, NBXplorer, bitcoind, nginx, and (usually) tor should be up. Lightning containers only if chosen.
2. Read the Tor address (works immediately):

   ```bash
   cat /var/lib/docker/volumes/generated_tor_servicesdir/_data/BTCPayServer/hostname
   ```

3. Give the user:
   - Tor link: `http://<onion>/` — works now, in a Tor Browser.
   - Domain link: `https://<BTCPAY_HOST>/` — once DNS and the certificate are ready.
4. Tell them the **first account they register becomes the server admin**. They should open the site and register before anyone else can.
5. Bitcoin sync takes 1–3 days in the background. The site is usable; on-chain payments wait for the node to catch up.

## Later changes (you run these)

Re-export the full intended environment, then `. ./btcpay-setup.sh -i` again. Blockchain data survives restarts.

| User ask | Command |
|---|---|
| Is it running? | `docker ps` |
| Restart | `btcpay-restart.sh` |
| Stop / start | `btcpay-down.sh` / `btcpay-up.sh` |
| Update | `btcpay-update.sh` |
| Bitcoin sync | `bitcoin-cli.sh getblockchaininfo` |
| Change domain | `changedomain.sh new.example.com` (they must disable 2FA first) |
| Tor address | `cat /var/lib/docker/volumes/generated_tor_servicesdir/_data/BTCPayServer/hostname` |

## Common problems (plain language)

- **Website shows 503 or certificate errors:** DNS has not caught up, or ports 80/443 are closed. Use the Tor link meanwhile.
- **Sync is slow:** Normal. Do not delete data or reinstall.
- **Lightning will not start on 5 GB pruning:** Switch to `opt-save-storage-xs` (25 GB) or disable Lightning.
- **Wrong network or settings:** Change the env vars and re-run `. ./btcpay-setup.sh -i`.

## Do not

- Do not add third-party install scripts. `btcpay-setup.sh` is the installer.
- Do not use `opt-add-prune-*` (those fragments do not exist). Use `opt-save-storage` / `-s` / `-xs` / `-xxs`.
- Do not enable `opt-expose-unsafe` or a Tor relay unless they clearly ask and you explain the risk.
- Do not print wallet seeds, RPC passwords, or macaroons into chat.
- Do not tell the user to copy-paste a wall of commands. You run them.
