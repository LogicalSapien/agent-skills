# agent-skills

A collection of [OpenClaw](https://openclaw.ai/) / [ClawHub](https://clawhub.ai/) agent skills.

## Skills

| Skill | Description | Proxy |
|---|---|---|
| [ctrader-commander](clawhub/ctrader-commander/) | Place and manage cTrader orders, fetch live quotes and OHLC candles, manage positions, and query account info. Requires [ctrader-openapi-proxy](https://github.com/LogicalSapien/ctrader-openapi-proxy). | [ctrader-openapi-proxy](https://github.com/LogicalSapien/ctrader-openapi-proxy) |

## Install a skill

**From ClawHub:**
```bash
npx clawhub@latest install <skill-name>
```

**Directly from this repo:**
```bash
git clone https://github.com/LogicalSapien/agent-skills
cp -r agent-skills/clawhub/<skill-name> ~/.openclaw/skills/<skill-name>
```

Then start a new OpenClaw session to pick it up.

## Publish a skill update

```bash
npm i -g clawhub
clawhub login
clawhub publish <skill-folder> --slug <skill-name> --name "<Display Name>" --version 1.0.1 --tags latest
```
