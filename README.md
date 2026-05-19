# Activitus Bar

One of my work colleagues was complaining about the activity bar wasting too much space, so this simply puts icons on the status bar that open the views of the activity bar. You can then hide the activity bar from the View menu...

## Fork Changes

This fork adds a few status-bar layout features that are useful when the native activity bar is hidden:

- `activitusbar.leftViews` and `activitusbar.rightViews` for two independently configured button groups.
- `activitusbar.leftPriority` and `activitusbar.rightPriority` for positioning each group independently.
- `command.*` view entries continue to work for arbitrary commands and custom workbench containers.
- `activitus-gemini`, `activitus-claude`, and `activitus-codex` product icons are bundled for use in `codicon`.

This fork is not published to an extension marketplace. Install it manually from a local VSIX.

## Manual Installation

Clone the fork and build the VSIX:

```sh
git clone git@github.com:ChiragKalra/activitusbar.git
cd activitusbar
npm ci
npm run package
```

Install into Google Antigravity:

```sh
antigravity --install-extension activitusbar-0.0.49.vsix --force
```

Install into VS Code:

```sh
code --install-extension activitusbar-0.0.49.vsix --force
```

Install into an Antigravity SSH remote extension host:

```sh
scp activitusbar-0.0.49.vsix remote-host:/tmp/activitusbar-0.0.49.vsix
ssh remote-host '~/.antigravity-server/bin/*/bin/antigravity-server --extensions-dir ~/.antigravity-server/extensions --install-extension /tmp/activitusbar-0.0.49.vsix --force'
```

After installing, run `Developer: Reload Window` so the extension host reloads the patched version.

## Configuration

The buttons are configurable, using `activitusbar.views`. This is a array containing objects with names, their associated icons and optional tooltips or labels. By default, all standard views are enabled, i.e. Explorer, Search, SCM, Debug and Extensions. (*See Default Configuration below*). When overriding the default settings, default icons will be used if not specified.

The panel views Terminal, Problems, Output and Debug Console can now also be moved to the activity bar. The extension supports these using the names "terminal", "problems", "output" and "debugConsole", again with default icons. These buttons will still work if the panel views are kept in the default location, but the views will not hide again if `activitusbar.toggleSidebar` is enabled.

The colour of the active and inactive buttons can also be specified using `activitusbar.activeColour` and `activitusbar.inactiveColour`. The configuration accepts either theme colour names (e.g. `editor.foreground`, see <https://code.visualstudio.com/api/references/theme-color> for the full list), standard HTML/CSS colour names or hex colour codes (e.g. `#ff0000`). If the colours are not specified in the configuration, the current status bar foreground (`statusBar.foreground`) and the inactive activity bar icon (`activityBar.inactiveForeground`) are used.

If required, the position of the icons can be adjusted by changing the value of `activitusbar.priority` and `activitusbar.alignment`. The defaults are `99999` and `Left` which should place them at the far left or the status bar. Depending on what other extensions are installed, you may need to experiment to find a value which suits. For example, to move everything to the far right, try `Right` and `-99999`.

To configure two independent groups, use `activitusbar.leftViews` and `activitusbar.rightViews`. When either split setting is configured, `activitusbar.views` is ignored for layout. The view object format is the same as `activitusbar.views`. Use `activitusbar.leftPriority` and `activitusbar.rightPriority` to position each group independently.

Example Antigravity configuration with left-side workbench controls and right-side agent controls:

```json
"workbench.activityBar.location": "hidden",
"workbench.auxiliaryActivityBar.location": "default",
"activitusbar.toggleSidebar": false,
"activitusbar.rightPriority": -99999,
"activitusbar.leftViews": [
    {
        "name": "explorer",
        "codicon": "explorer-view-icon"
    },
    {
        "name": "scm",
        "codicon": "source-control-view-icon"
    },
    {
        "name": "search",
        "codicon": "search-view-icon"
    },
    {
        "name": "debug",
        "codicon": "run-view-icon"
    },
    {
        "name": "command.workbench.view.remote",
        "codicon": "remote-explorer",
        "tooltip": "Remote SSH"
    }
],
"activitusbar.rightViews": [
    {
        "name": "command.antigravity.agentViewContainerId",
        "codicon": "activitus-gemini",
        "tooltip": "Gemini"
    },
    {
        "name": "command.workbench.view.extension.claude-sidebar-secondary",
        "codicon": "activitus-claude",
        "tooltip": "Claude"
    },
    {
        "name": "command.workbench.view.extension.codexSecondaryViewContainer",
        "codicon": "activitus-codex",
        "tooltip": "Codex"
    },
    {
        "name": "extensions",
        "codicon": "extensions-view-icon"
    },
    {
        "name": "terminal",
        "codicon": "terminal-view-icon"
    },
    {
        "name": "command.workbench.view.extension.claude-sessions-sidebar",
        "codicon": "activitus-claude",
        "tooltip": "Claude sessions"
    }
]
```

This build also contributes `activitus-gemini`, `activitus-claude`, and `activitus-codex` product icons for use in `codicon`.

Now that custom view containers are available, the configuration has been extended to support this. To add a button for a custom view, you'll need to find the name of the view name. This is normally shown in the *Feature Contributions* tab of the extension page in the extensions view.

Once you have the name of the view, choose an icon (see known issues below) from the list of [codicons](https://microsoft.github.io/vscode-codicons/dist/codicon.html) and add an entry to `activitusbar.views` with the format **"{ "name": "*&lt;view name&gt;*", "codicon": "*&lt;icon-name&gt;*"}**, e.g.

```json
{
    "name": "todo-tree-view",
    "codicon": "list-tree",
}
```

By default, clicking a button again will toggle the sidebar so that it is not visible. If you want to disable this behaviour, set `activitusbar.toggleSidebar` to false.

Normally configuration in your workspace settings overwrites your user settings. If you want to have a base set of buttons in your user settings and then have additional buttons on a per workspace basis, set `activitusbar.combineWorkspaceSettings` to `true`. This will then combine the settings together for each vscode window.

One last option is `activitusbar.searchViewInPanel`. If you move the search view to the panel (using the context menu) then set this to true to allow the panel to be toggled instead of the sidebar. *Note: Don't set this to true with the search view in the normal position!*

### Separators

You can add spaces between buttons by including an empty object (`{}`) in the view list. If you want a spacer with an icon, just include an icon without a view name, e.g.

```json
{
    "codicon": "kebab-vertical"
}
```

### Task Buttons

Buttons can also be configured to start tasks. Just use "task.*task-label*". For example, this:

```json
{
    "name": "task.build",
    "codicon": "tools",
    "tooltip": "Build project"
}
```

will create a button with the tools icons which starts the "build" task when the button is clicked.

### Command Buttons

Buttons can also be configured to run arbitrary commands. Use "command.*command-name*". For example,

```json
{
    "name": "command.workbench.action.reloadWindow",
    "codicon": "refresh"
}
```

will create a button which reloads the window.

### Settings Button

You can also add a button which opens the settings GUI using

```json
{
    "name": "settings",
    "codicon": "gear"
}
```

*Note: Buttons can all have optional tooltips and labels specified. Labels are added to the right of the icon.*

### Default Configuration

```json
"activitusbar.views": [
    {
      "name": "explorer",
      "codicon": "explorer-view-icon"
    },
    {
      "name": "search",
      "codicon": "search-view-icon"
    },
    {
      "name": "scm",
      "codicon": "source-control-view-icon"
    },
    {
      "name": "debug",
      "codicon": "run-view-icon"
    },
    {
      "name": "extensions",
      "codicon": "extensions-view-icon"
    }
  ]
```

### Source code counter

You can also show a counter which is displayed next to the SCM view button. This attempts to emulate the badge that is shown in the activity bar. This can be enabled by settting:

```json
"activitusbar.showSourceControlCounter": true
```

*Note: Some users have reported that this interferes with VSCode dialogs*.

### Keybindings

You may want to override the default keybindings in order to use them with activitusbar. This will also allow the highlighted icon to stay in sync if you use keyboard shortcuts to change view.

*Note: You may also have to disable the default keyboard bindings.*

```json
{
  "command": "activitusbar.showExplorerView",
  "key": "ctrl+shift+E",
  "mac": "shift+cmd+E"
},
{
  "command": "activitusbar.showSearchViewWithSelection",
  "key": "ctrl+shift+F",
  "mac": "shift+cmd+F"
},
{
  "command": "activitusbar.showReplaceViewWithSelection",
  "key": "ctrl+shift+H",
  "mac": "shift+cmd+H"
},
{
  "command": "activitusbar.showScmView",
  "key": "ctrl+shift+G",
  "mac": "shift+cmd+G"
},
{
  "command": "activitusbar.showDebugView",
  "key": "ctrl+shift+D",
  "mac": "shift+cmd+D"
},
{
  "command": "activitusbar.showExtensionsView",
  "key": "ctrl+shift+X",
  "mac": "shift+cmd+X"
}
```

## Known Issues

Because there is no way to know when a view has been selected, the currently active view will not stay in sync if the normal activity bar is used, or the view is changed using the **Open View...** from the **View** menu.

If **Find in Files** is selected from the Edit Menu, again, the currently active view will become out of sync.

## Installing

You can install the latest version of the extension via the Visual Studio Marketplace [here](https://marketplace.visualstudio.com/items?itemName=Gruntfuggly.activitusbar).

Alternatively, open Visual Studio code, press `Ctrl+P` or `Cmd+P` and type:

```sh
> ext install activitusbar
```

### Source Code

The source code is available on GitHub [here](https://github.com/Gruntfuggly/activitusbar).
