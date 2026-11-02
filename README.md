# Proton VPN TUI

An unofficial, keyboard-first terminal interface for Proton VPN. Search countries,
cities and servers, combine feature filters, save favorites, inspect connection
status, and connect without opening the desktop GUI.

**Early alpha. Live VPN control currently supports Linux only.** macOS and Windows
backends are planned. This project is independent of Proton AG.

![Proton VPN TUI with simulated server data](docs/images/overview.png)

## Install

Install [uv](https://docs.astral.sh/uv/getting-started/installation/) and Python 3.11
or newer. The package is distributed from this Git repository; it is not on PyPI.

```sh
uv tool install git+https://github.com/paulbrav/proton-vpn-tui.git
proton-vpn-tui --demo
```

For a one-off preview:

```sh
uvx --from git+https://github.com/paulbrav/proton-vpn-tui.git proton-vpn-tui --demo
```

`proton-terminal` remains a command alias for existing users. Check the installed
version with `proton-vpn-tui --version`.

For a local checkout:

```sh
git clone https://github.com/paulbrav/proton-vpn-tui.git
cd proton-vpn-tui
uv run --no-dev proton-vpn-tui --demo
uv tool install .
```

### Use your Proton account on Linux

Install the [official Proton VPN Linux CLI](https://protonvpn.com/support/linux-cli)
through Proton's distribution instructions. A working desktop D-Bus session,
keyring, NetworkManager and Proton's system dependencies are required. uv installs
the TUI; it does not install those system packages.

Close the Proton VPN GUI, then:

```sh
protonvpn signin YOUR_USERNAME
proton-vpn-tui --doctor
proton-vpn-tui
```

`--doctor` checks the installed CLI version. It does not verify authentication,
networking or leak protection. The private read adapter targets CLI **1.0.x** and
has been checked against **1.0.5**. Upstream changes can require adapter updates.

The defaults are `/usr/bin/protonvpn` and `/usr/bin/python3`. Use `--protonvpn` and
`--system-python` to override them. System Python must own the installed Proton
packages; the isolated uv interpreter does not need those packages.

## Controls

| Key | Action |
| --- | --- |
| `/` | Search country, city, server ID or feature; all words must match |
| `Esc` | Return to the server list or close a dialog |
| Arrows, `j` / `k` | Move selection |
| `Enter` | Connect to the selected server |
| `c` | Quick connect; Proton chooses globally, ignoring browser filters |
| `d` | Disconnect |
| `f` | Favorite or unfavorite the selected server |
| `F` | Show favorites only |
| `p` | Feature checkboxes |
| `s` | Toggle location/load ordering; favorites stay first |
| `[` / `]` | Previous/next page of matching servers |
| `r` | Refresh status and server data |
| `o` | VPN settings |
| `ctrl+l` | Sign in through the official CLI in your terminal |
| `?` | Help |
| `q`, `ctrl+c` | Quit the TUI; no disconnect request is sent |

Use Tab to move between controls. Ordinary shortcut letters remain text while
searching. Features combine with **AND**: P2P + IPv6 requires both. Space toggles
checkboxes; Apply commits the filter, Clear + Apply removes it, and Esc cancels.

Search covers the full catalog. Only 200 matching rows are rendered per page to
keep broad queries responsive. Load is Proton's reported figure, not measured
latency. The top bar shows the observed connection; the inspector describes the
highlighted candidate server.

<details>
<summary>Compact layout and feature filters</summary>

![80-column layout with simulated data](docs/images/compact.png)

![Multiple required features](docs/images/features.png)

</details>

## Compatibility and limitations

| Platform | Current scope |
| --- | --- |
| Linux desktop | Alpha VPN backend; Proton CLI 1.0.x required |
| macOS | UI/demo preview; live VPN backend planned |
| Windows | UI/demo preview; live VPN backend planned |

The preview uses generated data and needs no Proton account. Native preview CI is
separate from Linux backend tests; it does not establish VPN support on those OSes.

- The official GUI and this client cannot operate concurrently.
- Free-plan users can quick connect. Explicit server/location selection requires
  a paid plan in the official CLI.
- A failed status observation shows **UNKNOWN**. Command success alone is not
  treated as evidence of a connection.
- A fresh catalog request has an eight-second deadline. If it fails, Proton's saved
  list remains available with a **STALE** warning. Load figures and saved server
  availability may be outdated. Fresh-data refresh failures have occurred locally.
- Settings changes can require reconnection; the TUI does not reconnect
  automatically. Kill switch changes require disconnection.
- End-to-end live connect → switch server → disconnect, DNS/IPv6 leakage,
  kill-switch behavior, sleep/resume and other distributions still need release
  qualification. Automated tests use demo data and fake executables. Live
  read-only status, catalog and settings checks have been performed on Linux.

## Local data

The TUI stores only favorite server IDs. For compatibility with earlier versions,
preferences remain at `$XDG_CONFIG_HOME/proton-terminal/preferences.json` (normally
`~/.config/proton-terminal/preferences.json`). Demo favorites stay in memory.

Passwords and 2FA prompts are handled by the official CLI. This package does not
persist credentials or command transcripts. Proton's own client retains its
existing keyring, cache and logging behavior. Its read operations may refresh
session data or caches.

## Development

```sh
uv sync --locked
uv run pytest
uv run ruff check .
uv run ruff format --check .
uv build
```

See [CONTRIBUTING.md](CONTRIBUTING.md) and the
[architecture notes](docs/architecture.md). Demo SVGs can be regenerated with
`uv run python tools/capture_screenshots.py`; committed PNGs show only simulated data.

## Credits and license

Inspired by [Proton Orbit](https://github.com/mikhailfur/protonvpn-orbit-tui)'s
terminal workflow, without copying its implementation or map. Uses the installed
[official Proton VPN CLI](https://github.com/ProtonVPN/proton-vpn-cli).

Licensed under GPL-3.0-or-later; see [LICENSE](LICENSE).
