# Wakeshift for Hermes Agent

[Wakeshift](https://wakeshift.ai) wakes AI agents when something happens in the apps they're connected to. This is Wakeshift's plugin for [Hermes Agent](https://hermes-agent.nousresearch.com): when a Wakeshift-enabled MCP server you've added to Hermes has news for a conversation (a price dropped, a build finished, someone replied), the plugin starts a new turn in that conversation, even while it's idle, as long as Hermes is running.

One install serves every app built on Wakeshift. Apps that use it ask you to install it the first time they need it; you can also install it ahead of time.

## Install

```bash
hermes plugins install wakeshift/hermes-plugin --enable
```

Then start a new Hermes chat, or resume one with `hermes chat --continue`. If you run Hermes's gateway, restart it. Run the commands where Hermes runs: inside its container, if it's in one (`docker exec -u hermes <container> hermes plugins install wakeshift/hermes-plugin --enable`).

Wakeshift also has its own installer, which installs the same plugin from Wakeshift's servers. Download it, read it if you like, then run it:

```bash
curl -fsSL https://app.wakeshift.ai/install/hermes.sh -o wakeshift-hermes.sh && sh wakeshift-hermes.sh
```

[Wakeshift for Hermes Agent](https://wakeshift.ai/docs/hermes) covers the installer's options and exit codes, Docker, and bundling the plugin with your own Hermes plugin.

## How it works

1. You call a tool of a Wakeshift-enabled app that subscribes your conversation to something (for example `watch_price`). Its result carries a signed Wakeshift beacon.
2. The plugin removes the beacon before the model sees the result, checks its signature against the Wakeshift servers it trusts, and connects that conversation to Wakeshift. The result gets one line: "Wakeshift: connected."
3. When the app has news for the conversation, the plugin starts a new turn in it once it's idle, with the message marked as outside information from that app, never as something you wrote.

It wakes conversations through Hermes's API server (the gateway) and `hermes chat` CLI conversations. It doesn't wake conversations on messaging platforms such as Telegram or Slack; their messages wait until you next use the app.

## Which Wakeshift it trusts

By default the plugin acts only on beacons signed by `https://app.wakeshift.ai`. To trust other Wakeshift servers instead, set the `issuers` setting to a list of `https://` URLs:

```yaml
# $HERMES_HOME/config.yaml
plugins:
  entries:
    wakeshift:
      settings:
        issuers:
          - https://app.wakeshift.ai
          - https://wakeshift.example.com
```

Setting it replaces the default, so list `https://app.wakeshift.ai` too if you still want it. Plain `http://` is accepted only for a Wakeshift on this machine (`localhost`, `127.0.0.1`, `::1`). Wakeshift's installer also records the server it came from in the plugin's `config.json`, which is trusted in addition. Changes take effect in new chats, or after a gateway restart.

## What it does on your machine

What you'd want to know before installing:

- **Network.** It connects only to the Wakeshift servers it trusts (above): it fetches their signing keys (`/oauth/jwks`), exchanges a beacon for the conversation's stream with that app, holds that stream open (server-sent events, or long polling), and confirms each message it delivered. Each request names the plugin and its version (`X-Wakeshift-Connector: wakeshift-hermes/<version> hermes`). Nothing else leaves your machine: no telemetry or usage reporting.
- **Hermes's API server.** To wake a gateway conversation, it starts a run in it on Hermes's own API server, on `127.0.0.1`, with the API server's key, read through Hermes (`agent.secret_scope`). For a CLI conversation, it queues the message as the conversation's next input.
- **Tool results.** It reads MCP tool results to find Wakeshift beacons, and changes a result only to remove a beacon and add its one-line note. A beacon counts only in an MCP server's own result, from the server first seen carrying that app's beacons. If a beacon it should act on can't be used (Wakeshift can't be reached, or refuses it), the note says the conversation isn't connected and how to retry, and Hermes's log says why.
- **Tool calls.** When the model calls a Wakeshift-enabled app's tool (an MCP tool that takes Wakeshift's `session_key`, `harness` and `conversation_id` arguments), it sets `harness` to `hermes`, so the app knows which program the conversation runs in. It changes no other argument and no other tool's calls.
- **Files.** It keeps its state in `$HERMES_HOME/wakeshift/`: each connected conversation's stream links (one per app), which include an access token (files readable only by you, in a folder only you can open), the trusted servers' signing keys, which MCP server carries which app, and which messages already started a turn.
- **Background work.** A background thread per connected conversation and app holds its stream while Hermes runs; after a gateway restart, it reconnects the gateway's conversations.
- **Approvals.** A woken run follows your Hermes approval settings. The plugin never answers an approval and runs no shell commands.
- **Updates.** It never updates itself. Update it with `hermes plugins update wakeshift`, or by running Wakeshift's installer again.

## Remove it

```bash
hermes plugins remove wakeshift
```

Then delete the plugin's state, the `wakeshift` folder in your Hermes home (`$HERMES_HOME/wakeshift`; by default `~/.hermes/wakeshift`), which holds the conversations' stream links. Apps that subscribed your conversations keep their subscriptions until you end them with the app's own tools, or they expire.

## Terms

Wakeshift's [Connector Terms](https://wakeshift.ai/connector-terms) and [Privacy Policy](https://wakeshift.ai/privacy) apply to this plugin.

## Versions

`plugin.yaml`'s `version` is Wakeshift's build version for the plugin, `YYYY.MM.DD.HHMM+<commit>` (UTC). It's a valid PEP 440 version, and it increases with each release, so a plugin that needs Wakeshift's can depend on it in its own `plugin.yaml`:

```yaml
requires_plugins:
  - id: wakeshift
    version_range: ">=2026.10.31"
```

## Contributing and security

This repository is generated from Wakeshift's main repository and never edited by hand; each commit says which commit it came from (`.source-commit`). Pull requests are welcome: see [CONTRIBUTING](CONTRIBUTING.md). Report a security problem privately to support@wakeshift.ai, not in an issue ([SECURITY](SECURITY.md)).

## License

Apache License 2.0; see [LICENSE](LICENSE) and [NOTICE](NOTICE).
