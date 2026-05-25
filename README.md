# Dotfiles

Personal config files for syncing across machines.

## Setup

### Zed

```bash
ln -sf $(pwd)/zed/settings.json ~/.config/zed/settings.json
```

### OpenCode

```bash
ln -sf $(pwd)/opencode/opencode.json ~/.config/opencode/opencode.json
ln -sf $(pwd)/opencode/opencode.jsonc ~/.config/opencode/opencode.jsonc
ln -sf $(pwd)/opencode/agents ~/.opencode/agents
```

Set the provider API key via environment variable:

```bash
export OPENCODE_API_KEY="your-key-here"
```
