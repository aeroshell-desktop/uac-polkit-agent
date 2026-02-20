# UAC Polkit Agent 

UAC Polkit Agent is an alternative frontend designed to look like the User Account Control dialog on Windows Vista and 7. It's designed to coexist alongside [polkit-kde-agent-1](https://invent.kde.org/plasma/polkit-kde-agent-1), which can be replaced session-wide by setting the `USE_UAC_AGENT` environment variable to `1`. If the environment variable isn't defined, the regular Plasma agent is activated instead. This is done by creating an appropriate systemd drop-in file which handles the override logic. This is intended to be used with the AeroShell-based sessions, while the regular Plasma agent is used in the Plasma session.

## Standalone usage 

In case `polkit-kde-agent-1` isn't installed on the system, the user can use the `uac-polkit-agent` systemd service:

```bash
$ systemctl --user start uac-polkit-agent.service
```

## Window rule

To disable caption buttons other than the close button for the dialog, a KWin rule is needed. To add the KWin rule for the specific user, simply run the `add_rule.sh` script:

```bash
$ chmod +x add_rule.sh
$ ./add_rule.sh
```
