# Clawdbox

Running LLM and OpenClaw (relatively) safely in a Vagrant box.

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
curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh

# Install and configure OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
```
