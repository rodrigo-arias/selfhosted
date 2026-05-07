# Mac client setup

Configures a macOS machine to receive ntfy push notifications from this stack as native banners — useful for Uptime Kuma alerts, Sonarr/Radarr events, etc.

The ntfy CLI subscribes to a topic and pipes each message into `terminal-notifier`, which renders it via macOS Notification Center. A LaunchAgent keeps the subscriber running across reboots and respawns it if it dies.

## Prerequisites

- ntfy reachable on the LAN at `<LAN_IP>:8090` (the direct host port published by the `ntfy` container; bypasses NPM, which buffers SSE).
- Homebrew.

## Install

```bash
brew install ntfy terminal-notifier
```

## Client config

Drop the following at `~/Library/Application Support/ntfy/client.yml`:

```yaml
default-host: http://<LAN_IP>:8090
default-command: /opt/homebrew/bin/terminal-notifier -title "$title" -message "$message"
subscribe:
  - topic: infra
  - topic: downloads
```

`default-command` runs once per message; `$title` and `$message` are substituted by the ntfy CLI. Use the absolute path to `terminal-notifier` — LaunchAgents run with a minimal `PATH`.

## LaunchAgent

Create `~/Library/LaunchAgents/<reverse-domain>.ntfy.plist` (e.g. `com.example.ntfy.plist`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string><reverse-domain>.ntfy</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/ntfy</string>
        <string>subscribe</string>
        <string>--from-config</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/ntfy.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/ntfy.err</string>
</dict>
</plist>
```

`--from-config` makes the CLI subscribe to every topic in `client.yml`. Use absolute paths to all binaries.

## Activate

```bash
launchctl load ~/Library/LaunchAgents/<reverse-domain>.ntfy.plist
launchctl list | grep ntfy           # should show the label with PID
```

## Test

```bash
ntfy publish --title "Test" http://<LAN_IP>:8090/infra "hello from the lab"
```

A native banner should appear within a second. If it doesn't, check `/tmp/ntfy.err` and confirm `<LAN_IP>:8090` is reachable from the Mac.
