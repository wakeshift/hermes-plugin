# Contributing

This repository is generated from Wakeshift's main repository. Its root is the plugin exactly as Hermes installs it; the plugin's tests are published with the rest of Wakeshift's connectors, in [wakeshift/connectors](https://github.com/wakeshift/connectors) under `plugins/hermes`.

We read every issue. Report bugs, ask questions and suggest changes in this repository's issues.

Pull requests are welcome; we don't guarantee we'll accept every one. An accepted pull request is applied to Wakeshift's main repository, keeping you as the commit's author, and arrives here with the next publish; we close the pull request here with a link to it.

Before you open one:

- Run the plugin's tests in [wakeshift/connectors](https://github.com/wakeshift/connectors) (`plugins/hermes`; the command is in its README), and `hermes plugins validate .` here.
- Keep the change focused, and say what it fixes or adds.
- Don't include keys, tokens or other secrets, including in tests.
- Change only the code: the files this repository's export generates (`LICENSE`, `_version.py`, `.source-commit`, and the `version` in `plugin.yaml`) are updated when it's published.

Report a security problem privately to support@wakeshift.ai, not in an issue: see [SECURITY](SECURITY.md).

By contributing, you agree your contribution is licensed under the Apache License 2.0.
