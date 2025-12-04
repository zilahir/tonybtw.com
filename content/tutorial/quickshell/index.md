---
title: "Quickshell Tutorial - Build Your Own Bar"
author: ["Tony", "btw"]
description: "This is a quick and painless tutorial on how to build a custom status bar using Quickshell, a powerful Qt/QML based shell framework for Wayland."
date: 2025-12-03
draft: false
image: "/img/quickshell.png"
showTableOfContents: true
---

## Intro {#intro}

What's up guys, my name is Tony, and today I'm gonna give you a quick and painless introduction to Quickshell.

Quickshell is a full shell framework built on Qt/QML. You can build pretty much any desktop widget you can imagine with it - bars, dashboards, wallpaper managers, screen lock widgets, and more.

We're going to do a basic overview of how to create a bar using Quickshell today. In future tutorials I'll cover creating a wallpaper manager, building a dashboard overlay, and making a screen lock widget. But for now, let's start with the fundamentals by building a functional status bar step by step.


## Install Quickshell {#install-quickshell}

Alright so I'm on NixOS today, but this is going to work on Arch, Gentoo, and other distributions. I'll leave install instructions for all of those below.

[Quickshell Documentation](https://quickshell.org/docs/v0.2.1/types/)


#### NixOS {#nixos}

```nix
{
  environment.systemPackages = with pkgs; [
    quickshell
  ];
}
```


#### Arch Linux {#arch-linux}

```sh
yay -S quickshell-git
```


#### Gentoo {#gentoo}

For Gentoo, you'll need to compile from source:

```sh
git clone https://github.com/outfoxxed/quickshell
cd quickshell
# Follow the build instructions in their README
```


## Running Examples {#running-examples}

You can run any of these examples with:

```sh
qs -p ~/.config/testshell/01-hello.qml
```

Just swap out the filename as we go through each one.


## 01 - Hello World {#01-hello-world}

Quickshell has excellent documentation, and we're going to be following that today. Let's start with the absolute basics - just getting something on screen.

```qml
import Quickshell
import QtQuick

FloatingWindow {
    visible: true
    width: 200
    height: 100

    Text {
        anchors.centerIn: parent
        text: "Hello, Quickshell!"
        color: "#0db9d7"
        font.pixelSize: 18
    }
}
```

So what's going on here?

Every Quickshell config starts with imports. We're pulling in `Quickshell` for the core stuff and `QtQuick` for basic UI elements like `Text` and `Rectangle`.

We're using `FloatingWindow` as our root element. This is just a regular floating window - it doesn't dock to any edges or reserve any screen space. We're setting it to 200x100 pixels and making it visible.

The `Text` element is pretty self-explanatory. The `anchors.centerIn: parent` bit is QML's layout system - it just centers the text inside its parent container.


## 02 - Empty Bar {#02-empty-bar}

Alright, let's turn this into an actual bar that docks to the top of your screen.

```qml
import Quickshell
import Quickshell.Wayland
import QtQuick

PanelWindow {
    anchors.top: true
    anchors.left: true
    anchors.right: true
    implicitHeight: 30
    color: "#1a1b26"

    Text {
        anchors.centerIn: parent
        text: "My First Bar"
        color: "#a9b1d6"
        font.pixelSize: 14
    }
}
```

The big change here is we're using `PanelWindow` as our root element instead of `FloatingWindow`. This is a Wayland-specific thing (hence the new import), and it lets us dock the window to screen edges.

Setting `anchors.top`, `anchors.left`, and `anchors.right` to `true` tells it to stick to the top edge and span the full width. The `implicitHeight: 30` gives us a 30 pixel tall bar.

Unlike a floating window, a `PanelWindow` actually reserves space - your other windows won't overlap with it.


## 03 - Workspaces {#03-workspaces}

So we are on Hyprland today, lets add quickshell to our Hyprland config.

Now let's add some actual functionality - workspace indicators that show which workspace you're on and let you click to switch.

```qml
import Quickshell
import Quickshell.Wayland
import Quickshell.Hyprland
import QtQuick
import QtQuick.Layouts

PanelWindow {
    anchors.top: true
    anchors.left: true
    anchors.right: true
    implicitHeight: 30
    color: "#1a1b26"

    RowLayout {
        anchors.fill: parent
        anchors.margins: 8

        Repeater {
            model: 9

            Text {
                property var ws: Hyprland.workspaces.values.find(w => w.id === index + 1)
                property bool isActive: Hyprland.focusedWorkspace?.id === (index + 1)
                text: index + 1
                color: isActive ? "#0db9d7" : (ws ? "#7aa2f7" : "#444b6a")
                font { pixelSize: 14; bold: true }

                MouseArea {
                    anchors.fill: parent
                    onClicked: Hyprland.dispatch("workspace " + (index + 1))
                }
            }
        }

        Item { Layout.fillWidth: true }
    }
}
```

Okay, there's a lot more going on here. Let me break it down.

We're importing `Quickshell.Hyprland` which gives us access to Hyprland's IPC.

`QtQuick.Layouts` gives us `RowLayout`, which arranges its children horizontally. Way easier than manually positioning everything.

The `Repeater` is super useful - it takes a model (in this case, just the number 9) and creates that many copies of whatever's inside it. Each copy gets an `index` variable (0-8).

For each workspace number, we're looking up the actual workspace from the window manager with `Hyprland.workspaces.values.find()`. This gives us live data - when workspaces change, the bar updates automatically. We also check if it's the active workspace using `Hyprland.focusedWorkspace`.

The color logic is straightforward: cyan if it's the active workspace, blue if it exists but isn't active, and muted gray if there's no windows on that workspace.

The `MouseArea` makes the whole thing clickable, and `Hyprland.dispatch()` sends commands to Hyprland. So clicking on "3" runs `workspace 3`.

That `Item { Layout.fillWidth: true }` at the end is just a spacer - it pushes everything to the left.


## 04 - System Stats {#04-system-stats}

Now let's add some system stats. This is where things get interesting because we need to run shell commands and parse their output.


### Theme Properties {#theme-properties}

Instead of hardcoding colors everywhere, we define them once on the PanelWindow. Now you can reference `root.colBg` anywhere in your config, and if you want to change your color scheme, you only have to do it in one place.

```qml
PanelWindow {
    id: root
    property color colBg: "#1a1b26"
    property color colFg: "#a9b1d6"
    property color colMuted: "#444b6a"
    property color colCyan: "#0db9d7"
    property color colBlue: "#7aa2f7"
    property color colYellow: "#e0af68"
    property string fontFamily: "JetBrainsMono Nerd Font"
    property int fontSize: 14
}
```


### Running Shell Commands {#running-shell-commands}

This is how you run external commands and capture their output. We import `Quickshell.Io` which gives us the `Process` type.

```qml
Process {
    id: cpuProc
    command: ["sh", "-c", "head -1 /proc/stat"]
    stdout: SplitParser {
        onRead: data => {
            if (!data) return
            var p = data.trim().split(/\s+/)
            var idle = parseInt(p[4]) + parseInt(p[5])
            var total = p.slice(1, 8).reduce((a, b) => a + parseInt(b), 0)
            if (lastCpuTotal > 0) {
                cpuUsage = Math.round(100 * (1 - (idle - lastCpuIdle) / (total - lastCpuTotal)))
            }
            lastCpuTotal = total
            lastCpuIdle = idle
        }
    }
    Component.onCompleted: running = true
}
```

The `command` is an array - first element is the program, rest are arguments. The `SplitParser` attached to `stdout` calls `onRead` for each line of output. Setting `running = true` triggers the process to run.


### Timers {#timers}

To update the CPU usage periodically, we use a `Timer`. This one fires every 2 seconds and re-runs the CPU process.

```qml
Timer {
    interval: 2000        // Every 2 seconds
    running: true         // Start immediately
    repeat: true          // Keep going forever
    onTriggered: cpuProc.running = true
}
```


## 05 - Adding Widgets {#05-adding-widgets}

Let's expand our bar with a clock and memory usage. This builds on the same patterns - more `Process` calls and `Timer` elements.


### Memory Widget {#memory-widget}

```qml
// Add to your system data properties
property int memUsage: 0

// Memory process
Process {
    id: memProc
    command: ["sh", "-c", "free | grep Mem"]
    stdout: SplitParser {
        onRead: data => {
            if (!data) return
            var parts = data.trim().split(/\s+/)
            var total = parseInt(parts[1]) || 1
            var used = parseInt(parts[2]) || 0
            memUsage = Math.round(100 * used / total)
        }
    }
    Component.onCompleted: running = true
}

// Update your timer to run both processes
Timer {
    interval: 2000
    running: true
    repeat: true
    onTriggered: {
        cpuProc.running = true
        memProc.running = true
    }
}
```


### Clock {#clock}

The clock is just a `Text` element with its own timer. Every second it updates the text with the current time.

```qml
Text {
    id: clock
    text: Qt.formatDateTime(new Date(), "ddd, MMM dd - HH:mm")

    Timer {
        interval: 1000
        running: true
        repeat: true
        onTriggered: clock.text = Qt.formatDateTime(new Date(), "ddd, MMM dd - HH:mm")
    }
}
```


### Adding Dividers {#adding-dividers}

To visually separate widgets, you can use simple `Rectangle` elements:

```qml
Rectangle { width: 1; height: 16; color: root.colMuted }
```


## Complete Bar Example {#complete-bar-example}

Here's the final bar with workspaces, CPU, memory, and a clock all together:

```qml
import Quickshell
import Quickshell.Wayland
import Quickshell.Io
import Quickshell.Hyprland
import QtQuick
import QtQuick.Layouts

PanelWindow {
    id: root

    // Theme
    property color colBg: "#1a1b26"
    property color colFg: "#a9b1d6"
    property color colMuted: "#444b6a"
    property color colCyan: "#0db9d7"
    property color colBlue: "#7aa2f7"
    property color colYellow: "#e0af68"
    property string fontFamily: "JetBrainsMono Nerd Font"
    property int fontSize: 14

    // System data
    property int cpuUsage: 0
    property int memUsage: 0
    property var lastCpuIdle: 0
    property var lastCpuTotal: 0

    // Processes and timers here...

    anchors.top: true
    anchors.left: true
    anchors.right: true
    implicitHeight: 30
    color: root.colBg

    RowLayout {
        anchors.fill: parent
        anchors.margins: 8
        spacing: 8

        // Workspaces
        Repeater {
            model: 9
            Text {
                property var ws: Hyprland.workspaces.values.find(w => w.id === index + 1)
                property bool isActive: Hyprland.focusedWorkspace?.id === (index + 1)
                text: index + 1
                color: isActive ? root.colCyan : (ws ? root.colBlue : root.colMuted)
                font { family: root.fontFamily; pixelSize: root.fontSize; bold: true }
                MouseArea {
                    anchors.fill: parent
                    onClicked: Hyprland.dispatch("workspace " + (index + 1))
                }
            }
        }

        Item { Layout.fillWidth: true }

        // CPU
        Text {
            text: "CPU: " + cpuUsage + "%"
            color: root.colYellow
            font { family: root.fontFamily; pixelSize: root.fontSize; bold: true }
        }

        Rectangle { width: 1; height: 16; color: root.colMuted }

        // Memory
        Text {
            text: "Mem: " + memUsage + "%"
            color: root.colCyan
            font { family: root.fontFamily; pixelSize: root.fontSize; bold: true }
        }

        Rectangle { width: 1; height: 16; color: root.colMuted }

        // Clock
        Text {
            id: clock
            color: root.colBlue
            font { family: root.fontFamily; pixelSize: root.fontSize; bold: true }
            text: Qt.formatDateTime(new Date(), "ddd, MMM dd - HH:mm")
            Timer {
                interval: 1000
                running: true
                repeat: true
                onTriggered: clock.text = Qt.formatDateTime(new Date(), "ddd, MMM dd - HH:mm")
            }
        }
    }
}
```


## Key Concepts Summary {#key-concepts-summary}

| Concept        | Description                             |
|----------------|-----------------------------------------|
| FloatingWindow | A regular floating window, doesn't dock |
| PanelWindow    | Docks to screen edges, reserves space   |
| RowLayout      | Arranges children horizontally          |
| Repeater       | Creates multiple copies of a component  |
| Process        | Runs shell commands and captures output |
| Timer          | Triggers actions at intervals           |
| MouseArea      | Makes elements clickable                |
| anchors        | QML's layout system for positioning     |
| property       | Declare custom variables on components  |


## Next Steps {#next-steps}

That's the core of it. Quickshell can do way more than just bars though - you can build:

-   Wallpaper managers with smooth transitions
-   Dashboard overlays with system stats
-   Screen lock widgets
-   Notification centers
-   Application launchers

Check out the Quickshell documentation for more advanced features and the full API reference.


## Final Thoughts {#final-thoughts}

Thanks so much for checking out this tutorial. If you got value from it, and you want to find more tutorials like this, check out
my youtube channel here: [YouTube](https://youtube.com/@tony-btw), or my website here: [tony,btw](https://www.tonybtw.com)

You can support me here: [kofi](https://ko-fi.com/tonybtw)
