# The Grimoire on a server

The Grimoire can run its demons on a Linux server of yours, with the app on your Mac or
PC as a window onto it. The demons keep working while your computer sleeps, and every
computer you pair sees the same pacts and terminals. The setup steps are in the
[README](../README.md#three-ways-to-run-it), ways 2 and 3. This page is the reference.

## How it fits together

- **The engine** is the Grimoire itself, running on the server as a service of your user
  (`grimoire-engine`), its own window hidden on a virtual display. It starts your Claude
  Code sessions there, with your Claude Code login on that server.
- **A window** is the Grimoire on your computer with **THE HOST** set to **WINDOW ONTO**
  your server. It starts nothing of its own: it shows the engine's pacts and terminals
  and types into them. While the server is out of reach it shows its last view and takes
  no input.
- **The connection** is Tailscale only. The engine listens on the server's Tailscale
  address, port 47564, and nowhere else. It lets in a computer only when Tailscale says
  that computer belongs to the same Tailscale user as the server and carries no tag, and
  only once that computer is paired.

## The server script

`grimoire-server.sh` is the only thing you run on the server. Run it as the user the
engine should run as, never as root.

| | |
|---|---|
| `bash grimoire-server.sh install` | Installs the latest release and starts it. Run it again to update. |
| `bash grimoire-server.sh status` | Whether the engine runs, where it listens, which computers are paired. |
| `bash grimoire-server.sh pair <computer>` | The same as `grimoire-pair <computer>`. |
| `bash grimoire-server.sh uninstall` | Stops and removes the engine. Your pacts and settings stay unless you add `--purge`. |

Options: `--port <n>` runs the engine on another port (then type the server as
`name:<n>` in THE HOST); `--from <file> --sums <file>` installs a downloaded
`Grimoire-linux-x64.tar.gz` checked against the release's `SHA256SUMS.txt`;
`--no-packages` installs no system packages and lists the ones missing.

What it puts where, all in your home folder:

| | |
|---|---|
| `~/.local/opt/grimoire` | the app |
| `~/.config/systemd/user/grimoire-engine.service` | the service |
| `~/.local/bin/grimoire-pair` | pairing computers |
| `~/.config/Grimoire` | the engine's pacts, settings, paired computers and logs (`logs/main.log`) |

It asks for `sudo` three times: to install the system packages Electron needs on a server
(Xvfb and its libraries), to make Chromium's sandbox helper root's (Electron's sandbox
requires it on Linux), and to turn on "linger" for your user, so the service starts at
boot and runs with nobody logged in.

## Pairing

1. On the server: `grimoire-pair <computer>`, with the computer's Tailscale name as
   `tailscale status` lists it. It prints a code that works once, on that computer only,
   for ten minutes.
2. On that computer: the figure at the top of the left column, Settings, THE HOST. Choose
   WINDOW ONTO YOUR SERVER, type your server under SERVER and press SET, then the code
   under PAIRING CODE, PAIR and START AGAIN NOW. A window that is not paired yet also
   asks for the code on its own screen.

`grimoire-pair --list` shows the paired computers. `grimoire-pair --forget <computer>`
cuts one off within seconds; `--forget-all` cuts off all of them. Removing a computer
from your Tailscale network cuts it off too.

## Typing from two computers

One computer types at a time. The other takes over after 30 seconds of quiet, or at once
with **TAKE IT HERE** at the foot of the left column, which also says where the keyboard
is now. Files you paste or drop into a chat or a terminal are copied to the server.

## Back to one computer

THE HOST at the foot of the left column, or in Settings, switches between the server and
**THIS COMPUTER** in one click; the Grimoire starts again in the one you chose. The demons
on the server keep running.

## When something is wrong

| What you see | What to do |
|---|---|
| `CANNOT BE FOUND: IS TAILSCALE ON?` | Tailscale is off on your computer, or the name under SERVER is not the server's. Try its `100.x` address from `tailscale ip -4` on the server. |
| `IS OUT OF REACH` or `DID NOT ANSWER` | On the server: `bash grimoire-server.sh status`. If the engine is down: `journalctl --user -u grimoire-engine -n 50`. |
| `THE SERVER DOES NOT KNOW THIS COMPUTER AS ONE OF YOUR DEVICES` | Your computer and the server are signed in to Tailscale as different users, or the server carries a tag. |
| `THE SERVER REFUSED THE CODE` | The code ran out or was made for another computer's name: run `grimoire-pair` again with the name `tailscale status` shows. |
| The first tab says Claude Code is not installed | Install it on the server and sign in once (README, way 2, step 3), then open a new tab. |

## Not covered yet

- Linux as the computer in front of you: the Linux build can be a window too (it keeps its
  pairing in a key file in `~/.config/grimoire-seal`, since a Linux desktop may have no
  keyring), but the downloads for a desktop are Windows and macOS.
- ARM servers: the Linux build is x64 only.
- Reaching the engine without Tailscale: not offered, on purpose.
