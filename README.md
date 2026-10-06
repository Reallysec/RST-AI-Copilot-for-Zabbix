<p align="center">
  <img src=".github/assets/product-mark.svg" width="96" height="96" alt="RST AI Copilot for Zabbix">
</p>

<h1 align="center">RST AI Copilot for Zabbix</h1>

<p align="center">
  <b>Ask your network. Find the root cause.</b><br>
  Self-hosted AI copilot for Zabbix that collapses alert storms into a single root cause, answers network questions in plain language, and proposes the fix —<br>
  read-only by default, every change confirmed, air-gap ready.
</p>

<p align="center">
  <a href="https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases"><img src="https://img.shields.io/github/v/release/reallysec/RST-Zabbix-AI-Copilot?label=release&color=D40000" alt="Latest release"></a>
  <img src="https://img.shields.io/badge/free-Community_Edition-1BA9F5" alt="Free Community Edition">
  <img src="https://img.shields.io/badge/Zabbix-6.0_%7C_7.0-D40000" alt="Zabbix 6.0 and 7.0">
  <img src="https://img.shields.io/badge/deploy-Docker-2496ED?logo=docker&logoColor=white" alt="Deploy with Docker">
  <a href="https://reallysec.com/en/docs/zabbix-ai-copilot"><img src="https://img.shields.io/badge/docs-reallysec.com-343741" alt="Documentation"></a>
</p>

<p align="center">
  <b>English</b> · <a href="README.zh-CN.md">简体中文</a> · <a href="https://reallysec.com/en/docs/zabbix-ai-copilot">Docs</a> · <a href="https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases">Download</a> · <a href="https://github.com/reallysec/RST-Zabbix-AI-Copilot/issues">Report an issue</a>
</p>

## Why RST AI Copilot for Zabbix

- **Works with the Zabbix you have.** One Docker gateway next to your existing Zabbix 6.0 or 7.0 server. No new data store, no agents; it talks to the Zabbix JSON-RPC API with a token you control.
- **Read-only first.** Queries go through a read-only API whitelist. Anything that changes Zabbix (acknowledging a problem, creating a trigger) is an explicit operator action, and generated triggers are created disabled. A host-group whitelist bounds what the model may see.
- **Your data stays in your network.** Field masking runs before anything reaches the model. Point it at Volcengine Ark, any OpenAI-compatible endpoint, or a local vLLM / Ollama for fully offline operation.
- **Every step is accountable.** Each login, query, model call and settings change is an audit event you can search.

## Quick start

You need a Linux host with Docker Engine 24+ and Compose v2, network access to the Zabbix API (`…/api_jsonrpc.php`) with an API token, an OpenAI-compatible LLM endpoint, and a hostname or IP address for the operators.

Install with one command:

```bash
curl -fsSL https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases/latest/download/install.sh | sudo bash
```

The script downloads the latest bundle, checks it against the signed release manifest, unpacks it into `/opt/rst-zabbix-ai-copilot` and runs `deploy.sh`. There is one bundle for every edition: without a licence it runs as the free Community Edition, and activating a licence on the **License** page unlocks Professional or Enterprise in place. On an offline host, run it elsewhere with `--download-only` and carry the bundle over.

Or download the bundle from [Releases](https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases) yourself:

```bash
sha256sum -c RST-Zabbix-AI-Copilot-<version>.tar.gz.sha256
tar xzf RST-Zabbix-AI-Copilot-<version>.tar.gz
cd RST-Zabbix-AI-Copilot-<version> && ./deploy.sh
```

`deploy.sh` loads the images, generates secrets and the host fingerprint, asks for the LLM and Zabbix endpoints, and starts the stack behind Caddy TLS. The gateway is never exposed directly: Caddy is the only service that listens on the host. Open `https://<hostname-or-IP>/v2/` when it reports healthy.

By default the certificate comes from Caddy's internal CA, so the browser warns until you trust its root certificate. To use your own certificate, edit the `tls` line in `Caddyfile` (see the installation guide in the bundle, `docs/INSTALL.md`).

**Activate a licence** (skip for Community): open **License** as admin. Online: paste the licence key and activate; it binds to this host (needs outbound `license.reallysec.com:443`). Offline (Enterprise): copy the host fingerprint, get a licence file for it from [console.reallysec.com](https://console.reallysec.com) or sales, and import it.

**Updates**: the gateway checks this repository's Releases for a newer signed version (Settings). Downloading stages it; `sudo ./deploy/rst-update.sh` on the host installs it with a health check and automatic rollback.

## Features

Everything below is in the free Community Edition (one user, one node, no model-call limit; you bring your own model).

- **Smart query**: natural language to Zabbix API calls (problems, events, items, history, triggers), result tables and trend charts, multi-turn follow-ups.
- **Live alerts**: Zabbix problem polling or a Zabbix webhook media type, a summary per problem, acknowledge from the copilot.
- **Posture**: on-call overview of problems by severity, noisiest triggers and recent events.
- **Context for the model**: item-key dictionary, runbook knowledge base (RAG), host inventory and topology.
- **Scheduled inspections**: threshold, problem-summary and no-data checks with period-over-period deltas.
- **Privacy controls**: field masking (cloud / private / offline) and per-task reasoning levels.
- **Notifications**: Feishu, DingTalk, WeCom, Teams, Slack and email.

## Professional and Enterprise

Professional unlocks the AI engines: **alert batch triage** (collapse an alert storm to its root cause), **alert investigation** (agentic evidence gathering and fault troubleshooting), a **trigger copilot** (natural language to Zabbix trigger expressions, created disabled for review) and a **platform-ops copilot** (a check-up of the Zabbix server itself: queues, internal items, HA nodes, proxies), plus scheduled **operations reports** and more users. Enterprise adds organisation-scale integration. Upgrading is a licence import on the same install: no reinstall, data and host fingerprint are kept. A 14-day trial covers every Enterprise feature on one host.

<details>
<summary><b>Compare editions</b></summary>

| | Community (free) | Professional | Enterprise |
|---|:---:|:---:|:---:|
| Everything under [Features](#features) | ✅ | ✅ | ✅ |
| Users | 1 | per seat | as quoted |
| **Alert batch triage**: alert-storm convergence and root cause | — | ✅ | ✅ |
| **Alert investigation**: evidence gathering, timeline, affected hosts, actions | — | ✅ | ✅ |
| **Trigger copilot**: threshold / aggregate / no-data trigger expressions | — | ✅ | ✅ |
| **Platform-ops copilot**: AI read of the Zabbix server check-up | — | ✅ | ✅ |
| **Operations reports**: daily / weekly / monthly, scheduled delivery | — | ✅ | ✅ |
| Audit forwarding to an external SIEM (syslog / webhook) | — | — | ✅ |
| Multi-provider LLM failover | — | — | ✅ |
| OIDC single sign-on / enterprise identity | — | — | ✅ |
| Offline licence activation | — | — | ✅ |
| Nodes | 1 | 1 | unlimited |
| Model calls | unlimited | unlimited | unlimited |

Details and pricing: [editions](https://reallysec.com/en/docs/zabbix-ai-copilot/editions). Trials and licences: [console.reallysec.com](https://console.reallysec.com).

</details>

## Architecture and data boundary

- Ingress: the operator browser on 443 only, through Caddy. Egress: the LLM endpoint, your Zabbix API, and `license.reallysec.com` (not needed with an offline licence, Enterprise).
- Read-only API whitelist on Zabbix; the host-group whitelist bounds what the model may query.
- Field masking runs before anything reaches the model; in offline mode nothing leaves the network.
- Every login, query, model call and settings change is an audit event, forwardable to an external SIEM in the Enterprise edition.

## Supported versions

| Component | Supported |
|---|---|
| Zabbix | 6.0 LTS and 7.0 LTS (JSON-RPC API with an API token) |
| LLM endpoint | Volcengine Ark, any OpenAI-compatible API, self-hosted vLLM / Ollama |
| Host | Linux with Docker Engine 24+ and Docker Compose v2 |

## Support

- **Questions and bugs**: open an [issue](https://github.com/reallysec/RST-Zabbix-AI-Copilot/issues).
- **Security vulnerabilities**: do not open a public issue; follow the [security policy](SECURITY.md).

## Licensing

RST AI Copilot for Zabbix is proprietary software, distributed as compiled container images under the [End User License Agreement](LICENSE), also included in every bundle as `docs/EULA.md`. Third-party components and their licences are listed in `THIRD-PARTY-NOTICES.md` in the bundle. The Community Edition is free to use without a licence; Professional and Enterprise are activated online or with an offline licence file from [console.reallysec.com](https://console.reallysec.com). "RST", "Reallysec", "斯普朗克" and the product logos are trademarks. Zabbix is a registered trademark of Zabbix LLC. RST AI Copilot for Zabbix is not affiliated with or endorsed by Zabbix LLC.

© Anhui Reallysec Information Technology Ltd.
