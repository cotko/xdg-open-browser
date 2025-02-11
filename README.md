# XDG Open Browser

A script sitting in front of `xdg-open` allowing for setting up the rules what browser should links be opened in based on domain or application/process that requested url to be opened

For any domain or application/process it's possible to
- set what browser should be used for opening links
- being asked what browser to open the link in
- specify a custom script that resolves the browser or handles the url on its own


## Instalation

This [script](xdg-open-browser) should be used instead of `xdg-open` ("real" xdg-open is then used if no special rules are found). The end result should therefore be that `which xdg-open` points to this script.

The easiest way to achieve this is to add the script a location registered in *$PATH* which appears before the location of default `xdg-open`. **Do not forget** to rename it to *xdg-open*, alternatively you can git clone the repo and create a *xdg-open* sym link to this script.

### Arch linux

Arch linux uses `/usr/loca/bin` for user's scripts which makes it a perfect location:

```bash
sudo curl -o /usr/local/bin/xdg-open \
    https://raw.githubusercontent.com/cotko/xdg-open-browser/refs/heads/master/xdg-open-browser \
    && sudo chmod +x /usr/local/bin/xdg-open

```
## Dependencies

- `nodejs`
- Additionaly script laverages `kdialog` or `zenity` for displaying dialogs (can be configured).

## Configuration

The configuration file is located at `~/.config/xdg-open-browser/config.json`. It is automatically created the first time script is run, or by running `xdg-open-browser --config` which will additionally open the config in default editor.

Confugration (includes some defaults set/detected that can/should be changed if needed):
- **xdgopenbin** location of real xdg-open binary (default is `/usr/bin/xdg-open`)
- **dialog** which dialog to use (kdialog or zenity)
- **browsers** key<>value map of 'browser display name' <> 'browser binary'
- **process** key<>value map of process/application name <> 'rule'. Key (process name / application) is case insensitive and partially matched to actual process name.
- **domain** key<>value map of domain <> 'rule'. Key (domain) is partially matched to the domain of url being opened.

### Rules

Each rule can be:
- a browser binary (e.g. 'firefox'), to indicate what browser should be used
- `ask` this will open a dialog to choose from browsers defined in *_browsers:* part of the config
- `resolve:<path to some script>` this will invoke the script with two arguments; `<process/application name> <url>`
  - if the script prints something out, it's considered a browser binary that should be executed as `<browser-bin> <url>`
  - if script does not output anything, default `xdg-open` will be invoked
  - if script returns `HANDLED` then no further processing is done as this indicates that script did its own thing.

#### Example config

```json
{
  "browsers": {
    "Brave": "brave",
    "Zen": "zen-browser",
    "Falkon": "falkon"
  },
  "xdgopenbin": "/usr/bin/xdg-open",
  "dialog": "zenity",
  "process": {
    "aws":"brave",
    "postman":"ask",
    "slack":"resolve:/home/xy/xdg-resolve-slack"
  },
  "domain": {
    "zoom.us":"brave"
  }
}
```

**Custom resolver for Slack**

For example, it's possible to decide what browser should a link be opened in when using Slack, based on selected workspace.

Slack logs a line like `info: [WORKSPACE-FOCUS] Setting reducedSignInWorkflowExperiment object in local storage. teamId: TEAM123, enterpriseId: undefined, isOn: true ` to `~/.config/Slack/logs/default/webapp-console.log` each time a worskpace is changed.

Therefore it's possible to find last such log and decide what browser to use for a specific teamId:

```fish

#!/usr/bin/env fish
# /home/xy/xdg-resolve-slack

set teamLine (grep "teamId": ~/.config/Slack/logs/default/webapp-console.log | tail -n 1)

# if teamId equals TEAM123, then open brave
if string match -qri 'TEAM123' $teamLine
  echo brave
end

# else do not print out anything, which will open a link in default browser

```

