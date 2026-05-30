# Clawdbox

Running LLM and OpenClaw (relatively) safely in a Vagrant box.

## Security model

The isolation boundary here is **the VM itself** — agents run on a disposable
VirtualBox guest instead of your host machine. Understand what that does and
does not protect before trusting it:

- **Agents run directly as the `vagrant` user, not in containers.** There is no
  per-agent sandbox inside the box. Anything an agent can do, every other agent
  in the box can do too.
- **The `vagrant` user is in the `docker` group**, which is effectively root on
  the guest. A compromised or misbehaving agent can trivially escalate to root
  inside the VM. (Docker is installed but not currently used by anything in this
  repo — it is pure attack surface today.)
- **Synced folders cross the boundary.** `./agent-workspace` and
  `./openclaw-workspace` are mounted into the VM, so agents read and write those
  host directories directly. Keep anything you don't want touched out of them.
- **Install scripts are unpinned `curl | bash`.** The provisioner and the
  OpenClaw setup fetch and execute remote scripts with no version pinning or
  checksum verification. You are trusting those upstreams on every provision.

In short: treat the VM as a blast radius, not a jail. It protects your host from
casual mistakes; it does not contain a determined or compromised agent.

## Getting Started

### Tailscale

Tailscale is to ensure this box is only directly accessible to other devices within the same Tailscale VPN.

1. Generate a [Tailscale auth key](https://tailscale.com/kb/1085/auth-keys).
2. In your [DNS page](https://login.tailscale.com/admin/dns), in "Global nameservers", enable "Override DNS servers", and add a Global DNS such as `1.1.1.1` (Cloudflare Public DNS).
3. Run `cp .env.example .env`, and put in your auth key in `.env`.

### Vagrant

```sh
vagrant up
vagrant ssh
```

### OpenClaw

```sh
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install and configure OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
```
