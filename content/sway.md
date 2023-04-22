---
title: "sway"
date: 2023-04-06T22:54:01+02:00
---

The article is a modified version of [i3](/i3) as sway is a drop-in replacement of i3.

## Installation

```bash
emerge -av gui-wm/sway gui-apps/swaylock gui-apps/swaybg x11-misc/py3status gui-apps/foot gui-apps/wofi x11-misc/gammastep # on Gentoo
```

On Gentoo consider the following USE flags:
```
gui-apps/swaybg gdk-pixbuf # Support image types other than PNG
gui-wm/sway tray # for tray icons
gui-wm/sway X # for Xwayland support
```

## Configuration

First place the main config file
~/.config/sway/config
```
# i3 config file

# default modifier
set $mod Mod4
# second modifier
set $mod2 Mod1

input "type:keyboard" {
  xkb_layout de
  xkb_variant nodeadkeys
}

# font for window titles. ISO 10646 = Unicode
font pango:Inconsolata Medium 12

# Use Mouse+$mod to drag floating windows to their wanted position
floating_modifier $mod

# see https://github.com/swaywm/sway/issues/5732
exec systemctl --user import-environment

##########
# coloring				border	backgr	text indicator
client.focused			#333333 #333333 #aaaaaa #333333
client.focused_inactive	#111111 #111111 #aaaaaa #111111
client.unfocused		#111111 #111111 #999999 #111111
client.urgent			#aa2222 #aa2222 #000000 #aa2222

# black background
#exec swaybg -o '*' -c "#000000"

# gammastep
exec gammastep -l 50.8:6.1 -t 5700:3500

# start a terminal
bindsym $mod+Return exec foot

# kill focused window
bindsym $mod+Shift+q kill

##########
# program & action menu

set $wofiopts --insensitive --gtk-dark
bindsym $mod+d exec wofi --show run $wofiopts
bindsym $mod+a exec ~/.config/sway/actions/`ls ~/.config/sway/actions/ | wofi --show dmenu $wofiopts`

bindsym XF86AudioMute exec pactl set-sink-mute @DEFAULT_SINK@ toggle
bindsym XF86AudioLowerVolume exec pactl set-sink-volume @DEFAULT_SINK@ -2%
bindsym XF86AudioRaiseVolume exec pactl set-sink-volume @DEFAULT_SINK@ +2%

##########
# change focus
bindsym $mod+j focus left
bindsym $mod+k focus down
bindsym $mod+l focus up
bindsym $mod+odiaeresis focus right

# alternatively, you can use the cursor keys:
bindsym $mod+Left focus left
bindsym $mod+Down focus down
bindsym $mod+Up focus up
bindsym $mod+Right focus right

##########
# move focused window
bindsym $mod+Shift+J move left
bindsym $mod+Shift+K move down
bindsym $mod+Shift+L move up
bindsym $mod+Shift+Odiaeresis move right

# alternatively, you can use the cursor keys:
bindsym $mod+Shift+Left move left
bindsym $mod+Shift+Down move down
bindsym $mod+Shift+Up move up
bindsym $mod+Shift+Right move right

##########
# split in horizontal orientation
bindsym $mod+h split h

# split in vertical orientation
bindsym $mod+v split v

# enter fullscreen mode for the focused container
bindsym $mod+f fullscreen

# change container layout (stacked, tabbed, toggle split)
bindsym $mod+s layout stacking
bindsym $mod+w layout tabbed
bindsym $mod+e layout toggle split

# toggle tiling / floating
bindsym $mod+Shift+space floating toggle

# change focus between tiling / floating windows
bindsym $mod+space focus mode_toggle

# focus the parent container
#bindsym $mod+a focus parent

# focus the child container
#bindcode $mod+d focus child

##########
# workspaces

# switch to last used workspace
workspace_auto_back_and_forth yes

bindsym $mod+Tab workspace back_and_forth
bindsym $mod2+Tab workspace next

bindsym $mod+1 workspace 1
bindsym $mod+2 workspace 2
bindsym $mod+3 workspace 3
bindsym $mod+4 workspace 4
bindsym $mod+5 workspace 5
bindsym $mod+6 workspace 6
bindsym $mod+7 workspace 7
bindsym $mod+8 workspace 8
bindsym $mod+9 workspace 9
bindsym $mod+0 workspace 10

# move focused container to workspace
bindsym $mod+Shift+1 move container to workspace 1
bindsym $mod+Shift+2 move container to workspace 2
bindsym $mod+Shift+3 move container to workspace 3
bindsym $mod+Shift+4 move container to workspace 4
bindsym $mod+Shift+5 move container to workspace 5
bindsym $mod+Shift+6 move container to workspace 6
bindsym $mod+Shift+7 move container to workspace 7
bindsym $mod+Shift+8 move container to workspace 8
bindsym $mod+Shift+9 move container to workspace 9
bindsym $mod+Shift+0 move container to workspace 10

# resize window (you can also use the mouse for that)
mode "resize"
{

	bindsym j resize shrink left 10 px or 10 ppt
	bindsym Shift+J resize grow   left 10 px or 10 ppt

	bindsym k resize shrink down 10 px or 10 ppt
	bindsym Shift+K resize grow   down 10 px or 10 ppt

	bindsym l resize shrink up 10 px or 10 ppt
	bindsym Shift+L resize grow   up 10 px or 10 ppt

	bindsym odiaeresis resize shrink right 10 px or 10 ppt
	bindsym Shift+Odiaeresis resize grow   right 10 px or 10 ppt

	# same bindings, but for the arrow keys
	bindsym Left resize shrink left 10 px or 10 ppt
	bindsym Shift+Left resize grow   left 10 px or 10 ppt

	bindsym Down resize shrink down 10 px or 10 ppt
	bindsym Shift+Down resize grow   down 10 px or 10 ppt

	bindsym Up resize shrink up 10 px or 10 ppt
	bindsym Shift+Up resize grow   up 10 px or 10 ppt

	bindsym Right resize shrink right 10 px or 10 ppt
	bindsym Shift+Right resize grow   right 10 px or 10 ppt

	# back to normal: Enter or Escape
	bindsym Return mode "default"
	bindsym Escape mode "default"
}

bindsym $mod+r mode "resize"

##########
# status bar
bar 
{
	# official py3status
	status_command py3status -c ~/.config/sway/i3status.conf -l ~/.config/sway/i3status.log

    font pango:Inconsolata Medium 12

	colors
	{
		background #000000
		statusline #aaaaaa
		
		# class				border	backgr	text
		focused_workspace	#999999	#999999	#000000
		active_workspace	#aaaaaa #aaaaaa #111111
		inactive_workspace	#222222	#222222	#999999
		urgent_workspace	#aa2222	#aa2222	#000000
	}
}

#xwayland disable

# see https://github.com/emersion/xdg-desktop-portal-wlr#running
exec dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP=sway

# start pulseaudio if you're login manager doesn't
#exec pulseaudio --daemonize
```

und the config file for py3status
~/.config/sway/i3status.conf
```
order += "diskdata /boot"
order += "diskdata /"
order += "diskdata /home"
order += "ethernet enp0s25" # TODO substitute ethernet device (in the complete file)
order += "wifi"
#order += "imap mail"
#order += "systemd firewall"
#order += "emerge_status"
order += "battery_level"
order += "sysdata"
order += "volume_status"
order += "clock"

diskdata "/boot" {
	disk = "/dev/sda1"
	format = " /boot {used} GB {free} GB "
	format_space = "{value:.2f}"
}

diskdata "/" {
	disk = "/dev/sda2"
	format = " / {used} GB {free} GB "
	format_space = "{value:.1f}"
}

diskdata "/home" {
	disk = "/dev/sda4"
	format = " /home {used} GB {free} GB "
	format_space = "{value:.1f}"
}

ethernet enp0s25 {
	format_down = ""
	format_up = " %ip "
}

wifi {
	bitrate_bad = 0
	bitrate_degraded = 0
	device = "wlp3s0"  # TODO substitute wlan device
	format = " {ssid} ({ip}, {signal_percent}) |"
}

#imap mail {
#	cache_timeout = 60
#	format = ' {unseen} '
#	hide_if_zero = True
#	mailbox = 'INBOX'
#	name = 'Mail'
#	on_click 1 = "exec thunderbird" # "exec urxvt -e mutt"
#	password = '' # TODO
#	port = '993'
#	security = 'ssl'
#	server = '' # TODO IMAP-Server
#	user = '' # TODO Username / Mail-Address
#}

#systemd firewall {
#	format = ' \?if=!hide {unit} '
#	hide_extension = True
#	hide_if_default = 'on'
#	unit = 'firewall.service'
#}

#emerge_status {
#	format = "[\?if=is_running  [\?if=!total=0 {current}/{total} {action} {category}/{pkg}|calculating...] ]"
#}

battery_level {
	format = " [\?if=charging Charging {percent}%|Discharging {percent}% {time_remaining}h] "
	hide_seconds = True
	hide_when_full = True
}

sysdata {
	format = " {load1} {mem_used} GB "
}

volume_status {
	cache_timeout = 1
	command = "pactl"
	format = " [\?if=is_input 😮|♪] {percentage}% "
	format_muted = " [\?if=is_input 😶|♪] {percentage}% "
	thresholds = [(0, 'good')]
}

clock {
	format = "{Europe/Berlin}"
	format_time = " %a %d.%m.%Y %T"
}
```

We placed a bunch of scripts in ~/.config/sway/actions/. They can be run by Meta+A.

~/.config/sway/actions/hibernate

```
#!/bin/sh
~/.config/sway/actions/lock &
systemctl hibernate
```

~/.config/sway/actions/lock

```
#!/bin/sh
swaylock --color "#000000" --indicator-caps-lock
```

~/.config/sway/actions/poweroff

```
#!/bin/sh
systemctl poweroff
```

~/.config/sway/actions/quit

```
#!/bin/sh
sway exit
```

~/.config/sway/actions/reboot

```
#!/bin/sh
systemctl reboot
```

~/.config/sway/actions/reload

```
#!/bin/sh
swaymsg "reload"
```

Remember to make these scripts executable

```bash
chmod 755 ~/.config/sway/actions/*
```

Finally configure the foot terminal emulator in ~/.config/foot/foot.ini

```
# -*- conf -*-

# shell=$SHELL (if set, otherwise user's default shell from /etc/passwd)
# term=rxvt-unicode-256color
# login-shell=no

# app-id=foot
# title=foot
# locked-title=no

font=Inconsolata:size=12
# font-bold=<bold variant of regular font>
# font-italic=<italic variant of regular font>
# font-bold-italic=<bold+italic variant of regular font>
# line-height=<font metrics>
# letter-spacing=0
# horizontal-letter-offset=0
# vertical-letter-offset=0
# underline-offset=<font metrics>
# box-drawings-uses-font-glyphs=no
dpi-aware=no

# initial-window-size-pixels=700x500  # Or,
# initial-window-size-chars=<COLSxROWS>
# initial-window-mode=windowed
# pad=2x2                             # optionally append 'center'
# resize-delay-ms=100

# notify=notify-send -a ${app-id} -i ${app-id} ${title} ${body}

# bold-text-in-bright=no
# word-delimiters=,│`|:"'()[]{}<>
# selection-target=primary
# workers=<number of logical CPUs>

[environment]
# name=value

[bell]
# urgent=no
# notify=no
# command=
# command-focused=no

[scrollback]
# lines=1000
# multiplier=3.0
# indicator-position=relative
# indicator-format=

[url]
# launch=xdg-open ${url}
# label-letters=sadfjklewcmpgh
# osc8-underline=url-mode
# protocols=http, https, ftp, ftps, file, gemini, gopher
# uri-characters=abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_.,~:;/?#@!$&%*+="'()[]

[cursor]
# style=block
# color=<inverse foreground/background>
# blink=no
# beam-thickness=1.5
# underline-thickness=<font underline thickness>

[mouse]
# hide-when-typing=no
# alternate-scroll-mode=yes

[colors]
# alpha=1.0
# foreground=dcdccc
# background=111111

## Normal/regular colors (color palette 0-7)
regular0=222222  # black
regular1=e60e0e  # red
regular2=579906  # green
regular3=e3ac10  # yellow
regular4=6ca0a3  # blue
regular5=dc8cc3  # magenta
regular6=93e0e3  # cyan
regular7=dcdccc  # white

## Bright colors (color palette 8-15)
# bright0=666666   # bright black
# bright1=dca3a3   # bright red
# bright2=bfebbf   # bright green
# bright3=f0dfaf   # bright yellow
# bright4=8cd0d3   # bright blue
# bright5=fcace3   # bright magenta
# bright6=b3ffff   # bright cyan
# bright7=ffffff   # bright white

## dimmed colors (see foot.ini(5) man page)
# dim0=<not set>
# ...
# dim7=<not-set>

## The remaining 256-color palette
# 16 = <256-color palette #16>
# ...
# 255 = <256-color palette #255>

## Misc colors
# selection-foreground=<inverse foreground/background>
# selection-background=<inverse foreground/background>
# jump-labels=<regular0> <regular3>          # black-on-yellow
# scrollback-indicator=<regular0> <bright4>  # black-on-bright-blue
# search-box-no-match=<regular0> <regular1>  # black-on-red
# search-box-match=<regular0> <regular3>     # black-on-yellow
# urls=<regular3>

[csd]
# preferred=server
# size=26
# font=<primary font>
# color=<foreground color>
# hide-when-typing=no
# border-width=0
# border-color=<csd.color>
# button-width=26
# button-color=<background color>
# button-minimize-color=<regular4>
# button-maximize-color=<regular2>
# button-close-color=<regular1>

[key-bindings]
# scrollback-up-page=Shift+Page_Up
# scrollback-up-half-page=none
# scrollback-up-line=none
# scrollback-down-page=Shift+Page_Down
# scrollback-down-half-page=none
# scrollback-down-line=none
# clipboard-copy=Control+Shift+c XF86Copy
# clipboard-paste=Control+Shift+v XF86Paste
# primary-paste=Shift+Insert
# search-start=Control+Shift+r
# font-increase=Control+plus Control+equal Control+KP_Add
# font-decrease=Control+minus Control+KP_Subtract
# font-reset=Control+0 Control+KP_0
# spawn-terminal=Control+Shift+n
# minimize=none
# maximize=none
# fullscreen=none
# pipe-visible=[sh -c "xurls | fuzzel | xargs -r firefox"] none
# pipe-scrollback=[sh -c "xurls | fuzzel | xargs -r firefox"] none
# pipe-selected=[xargs -r firefox] none
# show-urls-launch=Control+Shift+u
# show-urls-copy=none
# show-urls-persistent=none
# prompt-prev=Control+Shift+z
# prompt-next=Control+Shift+x
# unicode-input=none
# noop=none

[search-bindings]
# cancel=Control+g Control+c Escape
# commit=Return
# find-prev=Control+r
# find-next=Control+s
# cursor-left=Left Control+b
# cursor-left-word=Control+Left Mod1+b
# cursor-right=Right Control+f
# cursor-right-word=Control+Right Mod1+f
# cursor-home=Home Control+a
# cursor-end=End Control+e
# delete-prev=BackSpace
# delete-prev-word=Mod1+BackSpace Control+BackSpace
# delete-next=Delete
# delete-next-word=Mod1+d Control+Delete
# extend-to-word-boundary=Control+w
# extend-to-next-whitespace=Control+Shift+w
# clipboard-paste=Control+v Control+Shift+v Control+y XF86Paste
# primary-paste=Shift+Insert
# unicode-input=none

[url-bindings]
# cancel=Control+g Control+c Control+d Escape
# toggle-url-visible=t

[text-bindings]
# \x03=Mod4+c  # Map Super+c -> Ctrl+c

[mouse-bindings]
# selection-override-modifiers=Shift
# primary-paste=BTN_MIDDLE
# select-begin=BTN_LEFT
# select-begin-block=Control+BTN_LEFT
# select-extend=BTN_RIGHT
# select-extend-character-wise=Control+BTN_RIGHT
# select-word=BTN_LEFT-2
# select-word-whitespace=Control+BTN_LEFT-2
# select-row=BTN_LEFT-3

# vim: ft=dosini
```

## Run sway

```bash
dbus-run-session sway
```