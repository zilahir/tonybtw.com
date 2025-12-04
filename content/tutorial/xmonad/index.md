---
title: "The Last Honest X11 Window Manager: Xmonad"
author: ["Tony", "btw"]
description: "This is a quick and painless guide on how to install and configure XMonad, written in Haskell. An honest X11 window manager written in an honest functional programming language."
date: 2025-11-26
draft: false
image: "/img/xmonad.png"
showTableOfContents: true
---

> In an era of dishonesty, there is one X11 window manager that continues to flourish.
> The final frontier of honesty.
> The last pillar of hope in an unsettling ecosystem where windows are no longer managed by human users… but by committees, protocols, and corporations.
>
> For nearly twenty years, this window manager has been battle-tested.
> It has shined through the smoke of AUR DDoS attacks…
> endless attempts to force Wayland adoption by Red Hat and Canonical…
> and a relentless wave of dishonest “modern UX improvements” designed to undermine your ability to simply move a window one pixel to the left.
>
> At the end of the day, that’s all an X11 window manager was ever supposed to be.
>
> And XMonad is—
> as YouTux might put it—
> the last honest window manager.


## Intro {#intro}

What's up guys, my name is Tony, and today I'm gonna give you a quick and painless guide on installing and configuring Xmonad.

Let's jump into the installation.


## Install Dependencies for Xmonad {#install-dependencies-for-xmonad}

Today, we're going to be using NixOS for the installation and configuration, but Xmonad is available on virtually every package manager in existence, (at least all the honest ones.) If you are using a legacy distro such as Arch, Gentoo, or Debian, a link will be provided below the subscribe button for a written guide to accompany this video.

For nixos, we just need to do 3 things.

In our configuration.nix:

```nix
services = {
  picom.enable = true;
  displayManager = {
    ly.enable = true;
  };
  xserver = {
    enable = true;
    autoRepeatDelay = 200;
    autoRepeatInterval = 35;
    windowManager = {
      xmonad = {
        enable = true;
        enableContribAndExtras = true;
        extraPackages = hpkgs: [
          hpkgs.xmonad
          hpkgs.xmonad-extras
          hpkgs.xmonad-contrib
        ];
      };
    };
    displayManager.sessionCommands = ''
      xwallpaper --zoom ~/walls/wall1.png
    '';
  };
};
```

And then in our home.nix:

```nix
home.packages = with pkgs; [
  haskell-language-server
  xmobar
];
```


### Requirements for Xmonad {#requirements-for-xmonad}

If you're on Arch, btw, here are the core dependencies:

-   xorg-server
-   xorg-xinit
-   xmonad
-   xmonad-contrib
-   ghc (Glasgow Haskell Compiler)
-   xmobar
-   dmenu (or rofi if you prefer)
-   picom (for compositing)


### Extra stuff for my setup today: {#extra-stuff-for-my-setup-today}

-   alacritty (terminal emulator)
-   dmenu/rofi (application launcher)
-   feh or xwallpaper (for wallpapers, we're using xwallpaper in our nix config)
-   firefox (web browser)
-   ttf-jetbrains-mono-nerd (font)
-   scrot or maim (for screenshots)
-   picom (compositor for shadows and transparency)

So let's install these with pacman -Sy

```sh
sudo pacman -Sy xorg-server xorg-xinit xmonad xmonad-contrib ghc xmobar dmenu alacritty xwallpaper firefox ttf-jetbrains-mono-nerd scrot picom
```

Alright, let's configure xmonad and get it up and running.


## Configure Xmonad {#configure-xmonad}

After running nixos-rebuild switch (or installing via pacman if you're on Arch), we need to create our xmonad.hs config file.

Let's create the directory and config:

```sh
mkdir -p ~/.config/xmonad
vim ~/.config/xmonad/xmonad.hs
```


### Starting from Zero {#starting-from-zero}

Let's start with the absolute minimum xmonad configuration. This is a complete, working config that does nothing more than launch xmonad with default settings:

```haskell
import XMonad

main = xmonad def
```

That's it. Three lines. This will give you a tiling window manager with default keybindings. Alt+Shift+Enter opens a terminal, Alt+p for dmenu, etc.

Let's compile and test it:

```sh
xmonad --recompile
```

Now let's build it up piece by piece.


### Adding Custom Terminal and Mod Key {#adding-custom-terminal-and-mod-key}

Most people want to use the Super key (Windows key) instead of Alt, and specify their preferred terminal:

```haskell
import XMonad

main = xmonad def
    { modMask = mod4Mask      -- Use Super instead of Alt
    , terminal = "alacritty"  -- Use alacritty as terminal
    }
```


### Adding Basic Keybindings {#adding-basic-keybindings}

Let's add some custom keybindings using EZConfig for a more readable syntax:

```haskell
import XMonad
import XMonad.Util.EZConfig (additionalKeysP)

myKeys =
    [ ("M-<Return>", spawn "alacritty")
    , ("M-d", spawn "dmenu_run")
    , ("M-q", kill)
    ]

main = xmonad $ def
    { modMask = mod4Mask
    , terminal = "alacritty"
    }
    `additionalKeysP` myKeys
```


### Adding Colors and Borders {#adding-colors-and-borders}

Let's add some visual customization:

```haskell
import XMonad
import XMonad.Util.EZConfig (additionalKeysP)

myKeys =
    [ ("M-<Return>", spawn "alacritty")
    , ("M-d", spawn "dmenu_run")
    , ("M-q", kill)
    ]

main = xmonad $ def
    { modMask = mod4Mask
    , terminal = "alacritty"
    , borderWidth = 2
    , normalBorderColor = "#444b6a"
    , focusedBorderColor = "#ad8ee6"
    }
    `additionalKeysP` myKeys
```


### Adding Gaps and Spacing {#adding-gaps-and-spacing}

Now let's add some breathing room with gaps between windows:

```haskell
import XMonad
import XMonad.Util.EZConfig (additionalKeysP)
import XMonad.Layout.Spacing

myLayoutHook = spacingWithEdge 3 $ layoutHook def

myKeys =
    [ ("M-<Return>", spawn "alacritty")
    , ("M-d", spawn "dmenu_run")
    , ("M-q", kill)
    ]

main = xmonad $ def
    { modMask = mod4Mask
    , terminal = "alacritty"
    , borderWidth = 2
    , normalBorderColor = "#444b6a"
    , focusedBorderColor = "#ad8ee6"
    , layoutHook = myLayoutHook
    }
    `additionalKeysP` myKeys
```


### Adding XMobar Status Bar {#adding-xmobar-status-bar}

Now let's integrate xmobar to show workspaces and window information:

```haskell
import XMonad
import XMonad.Util.EZConfig (additionalKeysP)
import XMonad.Layout.Spacing
import XMonad.Hooks.DynamicLog
import XMonad.Hooks.StatusBar
import XMonad.Hooks.StatusBar.PP
import XMonad.Hooks.ManageDocks

myLayoutHook = avoidStruts $ spacingWithEdge 3 $ layoutHook def

myXmobarPP :: PP
myXmobarPP = def
    { ppCurrent = xmobarColor "#0db9d7" ""
    , ppHidden = xmobarColor "#a9b1d6" ""
    , ppHiddenNoWindows = xmobarColor "#444b6a" ""
    }

myStatusBar = statusBarProp "xmobar" (pure myXmobarPP)

myKeys =
    [ ("M-<Return>", spawn "alacritty")
    , ("M-d", spawn "dmenu_run")
    , ("M-q", kill)
    , ("M-S-r", spawn "xmonad --recompile && xmonad --restart")
    ]

main = xmonad $ withEasySB myStatusBar defToggleStrutsKey $ def
    { modMask = mod4Mask
    , terminal = "alacritty"
    , borderWidth = 2
    , normalBorderColor = "#444b6a"
    , focusedBorderColor = "#ad8ee6"
    , layoutHook = myLayoutHook
    , manageHook = manageDocks
    }
    `additionalKeysP` myKeys
```

Now we have a pretty solid foundation! But what does a full-featured config look like?


### My Full Xmonad Configuration {#my-full-xmonad-configuration}

Here's my actual daily driver xmonad configuration with TokyoNight colors, custom layouts, workspace rules, gap controls, and all my keybindings:

```haskell
import Data.Map qualified as M
import XMonad
import XMonad.Hooks.DynamicLog
import XMonad.Hooks.EwmhDesktops
import XMonad.Hooks.ManageDocks
import XMonad.Hooks.StatusBar
import XMonad.Hooks.StatusBar.PP
import XMonad.Hooks.InsertPosition
import XMonad.Layout.NoBorders
import XMonad.Layout.ResizableTile
import XMonad.Layout.Spacing
import XMonad.Layout.Spiral
import XMonad.Layout.Renamed
import XMonad.StackSet qualified as W
import XMonad.Util.EZConfig (additionalKeysP)
import XMonad.Util.Loggers
import XMonad.Util.SpawnOnce

-- TokyoNight Colors
colorBg = "#1a1b26" -- background
colorFg = "#a9b1d6" -- foreground
colorBlk = "#32344a" -- black
colorRed = "#f7768e" -- red
colorGrn = "#9ece6a" -- green
colorYlw = "#e0af68" -- yellow
colorBlu = "#7aa2f7" -- blue
colorMag = "#ad8ee6" -- magenta
colorCyn = "#0db9d7" -- cyan
colorBrBlk = "#444b6a" -- bright black

-- Appearance
myBorderWidth = 2

myNormalBorderColor = colorBrBlk

myFocusedBorderColor = colorMag

-- Gaps (matching dwm: 3px all around)
mySpacing = spacingWithEdge 3

-- Workspaces
myWorkspaces = ["1", "2", "3", "4", "5", "6", "7", "8", "9"]

-- Mod key (Super/Windows key)
myModMask = mod4Mask

-- Terminal
myTerminal = "st"

-- Layouts
myLayoutHook =
  avoidStruts $
        renamed [Replace "Tall"] (mySpacing tall)
        ||| renamed [Replace "Wide"] (mySpacing (Mirror tall))
        ||| renamed [Replace "Full"] (mySpacing Full)
        ||| renamed [Replace "Spiral"] (mySpacing (spiral (6 / 7)))
  where
    tall = ResizableTall 1 (3 / 100) (11 / 20) []

-- Window rules (matching dwm config)
myManageHook =
  composeAll
    [ className =? "Gimp" --> doFloat
    , className =? "Brave-browser" --> doShift "2"
    , className =? "firefox" --> doShift "3"
    , className =? "Slack" --> doShift "4"
    , className =? "kdenlive" --> doShift "8"
    ]
    <+> insertPosition Below Newer

-- Key bindings (matching dwm as closely as possible)
myKeys =
  -- Launch applications
  [ ("M-<Return>", spawn myTerminal)
  , ("M-d", spawn "rofi -show drun -theme ~/.config/rofi/config.rasi")
  , ("M-r", spawn "dmenu_run")
  , ("M-l", spawn "slock")
  , ("C-<Print>", spawn "maim -s | xclip -selection clipboard -t image/png")
  , -- Window management
    ("M-q", kill)
  , ("M-j", windows W.focusDown)
  , ("M-k", windows W.focusUp)
  , ("M-<Tab>", windows W.focusDown)
  , -- Master area
    ("M-h", sendMessage Expand)
  , ("M-g", sendMessage Shrink)
  , ("M-i", sendMessage (IncMasterN 1))
  , ("M-p", sendMessage (IncMasterN (-1)))
  , -- Layout switching
    ("M-t", sendMessage $ JumpToLayout "Tall")
  , ("M-f", sendMessage $ JumpToLayout "Full")
  , ("M-c", sendMessage $ JumpToLayout "Spiral")
  , ("M-S-<Return>", sendMessage NextLayout)
  , ("M-n", sendMessage NextLayout)
  , -- Floating
    ("M-S-<Space>", withFocused toggleFloat)
  , -- Gaps (z to increase, x to decrease, a to toggle)
    ("M-z", incWindowSpacing 3)
  , ("M-x", decWindowSpacing 3)
  , ("M-a", toggleWindowSpacingEnabled >> toggleScreenSpacingEnabled)
  , ("M-S-a", setWindowSpacing (Border 3 3 3 3) >> setScreenSpacing (Border 3 3 3 3))
  , -- Quit/Restart
    ("M-S-r", spawn "xmonad --recompile && xmonad --restart")
  , -- Keychords for tag navigation (Mod+Space then number)
    ("M-<Space> 1", windows $ W.greedyView "1")
  , ("M-<Space> 2", windows $ W.greedyView "2")
  , ("M-<Space> 3", windows $ W.greedyView "3")
  , ("M-<Space> 4", windows $ W.greedyView "4")
  , ("M-<Space> 5", windows $ W.greedyView "5")
  , ("M-<Space> 6", windows $ W.greedyView "6")
  , ("M-<Space> 7", windows $ W.greedyView "7")
  , ("M-<Space> 8", windows $ W.greedyView "8")
  , ("M-<Space> 9", windows $ W.greedyView "9")
  , ("M-<Space> f", spawn "firefox")
  , -- Volume controls
    ("<XF86AudioRaiseVolume>", spawn "pactl set-sink-volume @DEFAULT_SINK@ +3%")
  , ("<XF86AudioLowerVolume>", spawn "pactl set-sink-volume @DEFAULT_SINK@ -3%")
  , ("<XF86AudioMute>", spawn "pactl set-sink-mute @DEFAULT_SINK@ toggle")
  ]
    ++
    -- Standard TAGKEYS behavior (Mod+# to view, Mod+Shift+# to move)
    [ (mask ++ "M-" ++ [key], windows $ action tag)
    | (tag, key) <- zip myWorkspaces "123456789"
    , (action, mask) <- [(W.greedyView, ""), (W.shift, "S-")]
    ]

-- Helper function for toggling float
toggleFloat w =
  windows
    ( \s ->
        if M.member w (W.floating s)
          then W.sink w s
          else W.float w (W.RationalRect 0.15 0.15 0.7 0.7) s
    )

-- XMobar PP (Pretty Printer) configuration
myXmobarPP :: PP
myXmobarPP =
  def
    { ppSep = xmobarColor colorBrBlk "" " │ "
    , ppTitleSanitize = xmobarStrip
    , ppCurrent = xmobarColor colorCyn ""
    , ppHidden = xmobarColor colorFg ""
    , ppHiddenNoWindows = xmobarColor colorBrBlk ""
    , ppUrgent = xmobarColor colorRed colorYlw
    , ppOrder = \[ws, l, _, wins] -> [ws, l, wins]
    , ppExtras = [logTitles formatFocused formatUnfocused]
    }
  where
    formatFocused = wrap (xmobarColor colorCyn "" "[") (xmobarColor colorCyn "" "]") . xmobarColor colorFg "" . ppWindow
    formatUnfocused = wrap (xmobarColor colorBrBlk "" "[") (xmobarColor colorBrBlk "" "]") . xmobarColor colorBrBlk "" . ppWindow
    ppWindow :: String -> String
    ppWindow = xmobarRaw . (\w -> if null w then "untitled" else w) . shorten 30

-- Main configuration
myConfig =
  def
    { modMask = myModMask
    , terminal = myTerminal
    , workspaces = myWorkspaces
    , borderWidth = myBorderWidth
    , normalBorderColor = myNormalBorderColor
    , focusedBorderColor = myFocusedBorderColor
    , layoutHook = myLayoutHook
    , manageHook = myManageHook <+> manageDocks
    , startupHook = spawnOnce "xsetroot -cursor_name left_ptr"
    }
    `additionalKeysP` myKeys

-- XMobar status bar configuration
myStatusBar = statusBarProp "xmobar ~/.config/xmobar/xmobarrc" (pure myXmobarPP)

main :: IO ()
main = xmonad . ewmhFullscreen . ewmh . withEasySB myStatusBar defToggleStrutsKey $ myConfig
```

This config includes TokyoNight colors, custom layouts (Tall, Wide, Full, Spiral), gap controls, workspace rules for automatically moving apps to specific workspaces, and tons of keybindings.

Now let's compile and test it:

```sh
xmonad --recompile
```

If you're on NixOS, you can just rebuild and then either log out and select xmonad from your display manager, or if you want to test it immediately:

```sh
nixos-rebuild switch
startx
```

Alright, we're in xmonad now. As you can see, we have a clean slate with xmobar at the top. Super minimal, super honest.


## Xmobar Configuration {#xmobar-configuration}

We already have xmobar launching from our xmonad config, but let's customize it. Let's create an xmobar config:

```sh
mkdir -p ~/.config/xmobar
vim ~/.config/xmobar/xmobarrc
```


### Starting with a Minimal Xmobar Config {#starting-with-a-minimal-xmobar-config}

Here's the absolute minimal xmobar configuration:

```haskell
Config {
     font     = "xft:monospace-10"
   , bgColor  = "#000000"
   , fgColor  = "#ffffff"
   , position = Top
   , commands = [ Run XMonadLog ]
   , template = "%XMonadLog%"
   }
```

This will show your workspaces and focused window. Nothing fancy, but it works.


### Adding System Information {#adding-system-information}

Let's add some system monitors like CPU, memory, and the date:

```haskell
Config {
     font     = "xft:monospace-10"
   , bgColor  = "#000000"
   , fgColor  = "#ffffff"
   , position = Top
   , sepChar  = "%"
   , alignSep = "}{"
   , template = "%XMonadLog% }{ CPU: %cpu% | MEM: %memory% | %date%"
   , commands =
        [ Run XMonadLog
        , Run Cpu ["-t", "<total>%"] 10
        , Run Memory ["-t", "<usedratio>%"] 10
        , Run Date "%a %b %_d %H:%M" "date" 10
        ]
   }
```

Now we have workspace info on the left, and system stats on the right.


### Adding Colors and Better Formatting {#adding-colors-and-better-formatting}

Let's make it prettier with some color coding:

```haskell
Config {
     font     = "xft:JetBrainsMono Nerd Font-12"
   , bgColor  = "#1a1b26"
   , fgColor  = "#a9b1d6"
   , position = TopSize C 100 30
   , sepChar  = "%"
   , alignSep = "}{"
   , template = " %XMonadLog% }{ <fc=#7aa2f7>CPU:</fc> %cpu% | <fc=#7aa2f7>MEM:</fc> %memory% | <fc=#ad8ee6>%date%</fc> "
   , commands =
        [ Run XMonadLog
        , Run Cpu
            [ "-t", "<total>%"
            , "-L", "30"
            , "-H", "70"
            , "-l", "#9ece6a"
            , "-n", "#e0af68"
            , "-h", "#f7768e"
            ] 10
        , Run Memory
            [ "-t", "<usedratio>%"
            , "-L", "30"
            , "-H", "70"
            , "-l", "#9ece6a"
            , "-n", "#e0af68"
            , "-h", "#f7768e"
            ] 10
        , Run Date "<fc=#ad8ee6>%a %b %_d %H:%M</fc>" "date" 10
        ]
   }
```

Now we've got TokyoNight colors, with green for low usage, yellow for medium, and red for high.


### My Full Xmobar Configuration {#my-full-xmobar-configuration}

Here's my actual daily driver xmobar config with borders, Nerd Font icons, battery monitoring, and full TokyoNight theming:

```haskell
-- TokyoNight XMobar Config
Config {
   -- Appearance
     font            = "JetBrainsMono Nerd Font Mono Bold 16"
   , additionalFonts = [ "JetBrainsMono Nerd Font Mono 18" ]
   , bgColor         = "#1a1b26"
   , fgColor         = "#a9b1d6"
   , position        = TopSize C 100 35
   , border          = BottomB
   , borderColor     = "#444b6a"
   , borderWidth     = 2

   -- Layout
   , sepChar         = "%"
   , alignSep        = "}{"
   , template        = " %XMonadLog% }{ %cpu% <fc=#444b6a>│</fc> %memory% <fc=#444b6a>│</fc> %battery% <fc=#444b6a>│</fc> %date% "

   -- Plugins
   , commands        =
        [ Run XMonadLog
        , Run Cpu
            [ "-t", "<fc=#7aa2f7> CPU:</fc> <total>%"
            , "-L", "30"
            , "-H", "70"
            , "-l", "#9ece6a"
            , "-n", "#e0af68"
            , "-h", "#f7768e"
            ] 10
        , Run Memory
            [ "-t", "<fc=#7aa2f7>󰍛 MEM:</fc> <usedratio>%"
            , "-L", "30"
            , "-H", "70"
            , "-l", "#9ece6a"
            , "-n", "#e0af68"
            , "-h", "#f7768e"
            ] 10
        , Run Battery
            [ "-t", "<fc=#7aa2f7>󱐋 BAT:</fc> <acstatus>"
            , "-L", "20"
            , "-H", "80"
            , "-l", "#f7768e"
            , "-n", "#e0af68"
            , "-h", "#9ece6a"
            , "--"
            , "-o", "<left>% (<timeleft>)"
            , "-O", "<fc=#e0af68>Charging</fc> <left>%"
            , "-i", "<fc=#9ece6a>Charged</fc>"
            ] 50
        , Run Date "<fc=#ad8ee6> %a %b %_d %H:%M</fc>" "date" 10
        ]
   }
```

This config has Nerd Font icons, a bottom border, battery monitoring, and uses the full TokyoNight color palette with dynamic colors based on resource usage.

Now if we reload xmonad with Mod+Shift+r, we should see our customized xmobar. Sweet.


## Wallpaper {#wallpaper}

Good news - we already set up the wallpaper in our configuration.nix! The line we added earlier:

```nix
displayManager.sessionCommands = ''
  xwallpaper --zoom ~/walls/wall1.png
'';
```

This will automatically set your wallpaper on startup. But we still need to grab a wallpaper. Let's open firefox with super+d, type firefox, and head over to wallhaven.cc to pick one out.

Let's grab this one and put it in ~/walls/wall1.png:

```sh
mkdir -p ~/walls
# download your wallpaper and move it here
mv ~/Downloads/wallpaper.png ~/walls/wall1.png
```

If you want to test it without reloading X, you can run:

```sh
xwallpaper --zoom ~/walls/wall1.png
```

And boom, there's our wallpaper.


## Screenshot Script {#screenshot-script}

For screenshots, I'm using maim with xclip to copy directly to clipboard. In the xmonad config above, I already have it bound to Control+Print:

```haskell
, ("C-<Print>", spawn "maim -s | xclip -selection clipboard -t image/png")
```

This lets you select an area with your mouse, and it copies the screenshot directly to your clipboard. Super convenient for pasting into Discord, Slack, or wherever.

If you want to save screenshots to a file instead, you can create a script:

```sh
mkdir -p ~/.local/bin
vim ~/.local/bin/screenshot
```

Add this:

```sh
#!/bin/sh
# Screenshot script using maim
maim -s ~/Pictures/screenshots/$(date +%Y-%m-%d_%H-%M-%S).png
```

Make it executable:

```sh
chmod +x ~/.local/bin/screenshot
```

And you can add another keybind in your xmonad.hs if you want both options.


## Customization and Tweaks {#customization-and-tweaks}

So at this point, the world is really your oyster. Xmonad is written in Haskell, so if you know Haskell, you can literally do anything you want with this window manager. That's the beauty of it - your window manager IS your config file.

In my config, I've already set up a bunch of stuff:

The TokyoNight color scheme with that nice magenta focused border, 3 pixel gaps all around (matching my dwm setup), and I'm using st as my terminal because it's fast and minimal.

For layouts, I've got ResizableTall, Mirror ResizableTall, Full, and Spiral. You can jump between them with:

-   Super+t for tiled
-   Super+f or Super+m for fullscreen
-   Super+c for spiral
-   Super+Shift+Enter to cycle through all layouts

For gaps, I have some really nice keybinds:

-   Super+a toggles gaps on/off
-   Super+z increases gap size
-   Super+x decreases gap size
-   Super+Shift+a resets gaps back to 3 pixels

Window rules are set up so different apps automatically go to specific workspaces:

-   Browsers (Chrome, Brave) go to workspace 2
-   Firefox goes to workspace 3
-   Slack goes to 4
-   Discord goes to 5
-   Kdenlive goes to 8

Here are my essential keybinds:

| Keybind              | Action                                |
|----------------------|---------------------------------------|
| Super+Enter          | Launch terminal (st)                  |
| Super+d              | Launch rofi application launcher      |
| Super+r              | Launch dmenu                          |
| Super+l              | Lock screen with slock                |
| Super+q              | Close focused window                  |
| Super+j              | Focus next window                     |
| Super+k              | Focus previous window                 |
| Super+Tab            | Focus next window                     |
| Super+h              | Expand master area                    |
| Super+g              | Shrink master area                    |
| Super+i              | Increase number of windows in master  |
| Super+p              | Decrease number of windows in master  |
| Super+t              | Switch to Tall layout                 |
| Super+f              | Switch to Full layout                 |
| Super+c              | Switch to Spiral layout               |
| Super+Shift+Enter    | Cycle to next layout                  |
| Super+n              | Cycle to next layout                  |
| Super+Shift+Space    | Toggle floating for focused window    |
| Super+z              | Increase gap size                     |
| Super+x              | Decrease gap size                     |
| Super+a              | Toggle gaps on/off                    |
| Super+Shift+a        | Reset gaps to default (3px)           |
| Super+Shift+r        | Recompile and restart xmonad          |
| Super+Space [1-9]    | Keychord: Jump to workspace 1-9       |
| Super+Space f        | Keychord: Launch Firefox              |
| Super+[1-9]          | Switch to workspace 1-9               |
| Super+Shift+[1-9]    | Move window to workspace 1-9          |
| Ctrl+Print           | Screenshot (select area to clipboard) |
| XF86AudioRaiseVolume | Increase volume by 3%                 |
| XF86AudioLowerVolume | Decrease volume by 3%                 |
| XF86AudioMute        | Toggle mute                           |

I've put together the full xmonad config above with all my customizations, the TokyoNight colors, and all my keybinds. If you want even more customization ideas, check out the xmonad documentation - it's honestly one of the best-documented window managers out there.


## Final Thoughts {#final-thoughts}

You're now ready to use XMonad as an honest X11 window manager.

Thanks so much for checking out this tutorial. If you got value from it, and you want to find more tutorials like this, check out
my youtube channel here: [YouTube](https://youtube.com/@tony-btw), or my website here: [tony,btw](https://www.tonybtw.com)

You can support me here: [kofi](https://ko-fi.com/tonybtw)
