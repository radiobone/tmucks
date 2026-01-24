# tmucks

A tmux config manager for the reckless.

Save, switch, and manage multiple tmux configurations from the command line or an interactive TUI.

## Installation

### From source

```
git clone https://github.com/yourusername/tmucks
cd tmucks
cargo build --release
cp target/release/tmucks ~/.local/bin/
```

### From crates.io

```
cargo install tmucks
```

## Usage

### Interactive mode

Run without arguments to launch the TUI:

```
tmucks
```

Keybindings:
- `j/k` or arrow keys: navigate
- `Enter`: apply selected config
- `s`: save current config
- `u`: update selected config
- `d`: delete selected config
- `q`: quit

### CLI commands

```
tmucks list              # list saved configs
tmucks apply <name>      # apply config to ~/.tmux.conf
tmucks save <name>       # save current ~/.tmux.conf
tmucks update <name>     # overwrite saved config with current
tmucks delete <name>     # delete saved config
```

The `.conf` extension is optional. `tmucks apply dev` and `tmucks apply dev.conf` are equivalent.

## How it works

Configs are stored in `~/.config/tmucks/`. When you apply a config, it copies the file to `~/.tmux.conf` and runs `tmux source-file ~/.tmux.conf` to reload your session.

## License

GPL-3.0
