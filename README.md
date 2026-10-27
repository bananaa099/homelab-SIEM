# homelab-SIEM

My home lab for learning SOC work. I wanted to see logs from a real Windows machine in a SIEM, so I set this up on my PC with VirtualBox.

## What's inside

Two VMs:
- Ubuntu Server 22.04 with Splunk Enterprise (192.168.56.101)
- Windows 11 with Sysmon and the Splunk Universal Forwarder (192.168.56.102)

The forwarder sends Windows logs (Security, System, Application) and Sysmon logs to Splunk on port 9997.

To check that logs arrive I usually use:

```
index=* | stats count by host, sourcetype
```

## Dashboard

Made a simple dashboard with 3 panels: events over time, failed logons by account, and PowerShell processes from Sysmon.

## What I tested

Ran two attacks on the Windows VM and found both in Splunk:

- **Encoded PowerShell** — a command with `-enc` (Base64). Caught it in Sysmon EventID 1 by searching the CommandLine for `-enc`.
- **New admin account** — `net user` + `net localgroup administrators`. Showed up as EventID 4720 (user created) and 4732 (added to Administrators) a few seconds apart.

Write-ups are in [attack-detection-cases](https://github.com/bananaa099/attack-detection-cases), detection logic in [detection-rules](https://github.com/bananaa099/detection-rules).

## Stuff that broke

Honestly most of the time went here.

- After every reboot Ubuntu lost its 192.168.56.101 address. cloud-init rewrites the netplan file. Put the static IP in a separate file (02-server-lab.yaml.png) and now it stays.
- Splunk didn't start on boot. This version won't run as root without `--run-as-root`, so I still start it by hand. Want to fix with a systemd service.
- Changed the owner of /opt/splunk once and broke everything with permission errors. Had to reinstall Splunk.
- Port 9997 disappeared after the reinstall, added it back in inputs.conf.
- CommandLine wasn't showing in my dashboard table because Sysmon logs are XML. Pulled it out with `rex`.
