> [!IMPORTANT]
> **This fork has moved on.** It grew into [Locus](https://github.com/cwilliams5/omarchy-locus), a night light that follows the sun, with its own name and history. This repository stays only to carry two pull requests to the original Sundown ([#1](https://github.com/WilliamJohnDavisIII/omarchy-sundown/pull/1) and [#2](https://github.com/WilliamJohnDavisIII/omarchy-sundown/pull/2)).

# Sundown

A night light for [Omarchy](https://omarchy.org) that follows the sun. It warms the screen at sunset and clears it at sunrise, with a temperature slider and a custom schedule if you'd rather pick the hours.

It's built on `hyprsunset`, which Omarchy already ships. Sun times are calculated on your computer, so it makes no network requests unless you turn on location detection.

## Install

```bash
omarchy plugin add https://github.com/cwilliams5/omarchy-sundown.git --enable
omarchy restart shell
```

Omarchy's own night light icon stays in the indicators widget. To leave only Sundown's, take `NightLight` out of that widget's `items`.

## Use

| Do this | And |
|---|---|
| Left-click the icon | Open the panel |
| Right-click the icon | Flip the switch |
| Scroll on the icon | Change the temperature |

In the panel, Left and Right change the temperature, Enter flips the switch, and `m` changes the mode.

The modes are:

- **Sun:** on from sunset to sunrise. **Start** moves the beginning up to 3 hours before sunset or 2 hours after it, in 15-minute steps.
- **Custom:** on between two times you choose
- **Manual:** stays how you set it

With **Gradual** on, which is the default, Sun and Custom modes warm the screen over the hour after they turn on, and clear it over the 30 minutes before they turn off. **Warm up over** and **Clear over** change those lengths, from at once to 3 hours. Turn Gradual off to change at the scheduled time instead. Flipping the switch always takes about 2 seconds.

A fade moves in steps of about 3 mireds, under what the eye can tell apart, so an hour from 6500K to 3000K is about 60 steps. Sundown looks at `hyprsunset` when something is due (a scheduled change, the next fade step, a switch time in `hyprsunset.conf`) and otherwise once a minute, so a change made elsewhere is noticed within a minute. It always looks before it sends anything, so it never overwrites a change it hasn't seen yet.

Flipping the switch during a schedule holds until the next scheduled change, even across restarts. Moving the slider while the night light is off previews that temperature for 10 seconds.

`hyprsunset` stops at 1000K, and on some screens that is more orange than red. Below 1000K, Sundown leaves `hyprsunset` there and adds a Hyprland screen shader that takes away more green and blue, until 500K shows only red. A screen shader you already use, like a grayscale one, keeps working underneath. The shader goes away on a config reload and comes back within a minute.

`omarchy toggle nightlight` still works. It uses Omarchy's fixed 4000K, and that stays until the next scheduled change, even during a gradual change.

## Pausing for apps

Some apps need true colour, like games and photo or video editors. Sundown can keep the screen neutral while one of them is in front, and warm it again when you leave it. The **Pause** section of the panel has:

- **For fullscreen apps:** any app in real fullscreen (Super+F). A maximized window doesn't count.
- **The app in front**, with a button to pause whenever it's in front, or to stop. The apps you've added are listed under it, each with a button to take it off.

A name ending in `*` matches every app whose class starts with the rest, so `steam_app_*` covers every game started from Steam. Add one like that with the `pauseapp` command below.

While an app has it paused, the switch still shows what the schedule says, and the icon's tooltip names the app. Sundown hears about focus and fullscreen changes from Hyprland, so pausing adds no checking of its own.

## Location

Sun mode uses the first of these it finds:

1. Coordinates you enter with the pencil button in the panel
2. Your detected location, if the crosshairs button is on
3. Omarchy's weather location, if it has coordinates and not only a name
4. The main city of your system time zone, from `/usr/share/zoneinfo/zone1970.tab`

The time zone is only a rough guide, so sunset can be off by a while if you live far from that city. If none of these is available, the switch stays how you set it until you enter a location.

Location detection is off by default. It asks `geo.omarchy.org` for the city of your IP address, as Omarchy's weather widget does, when the shell starts, every few hours, and after a long sleep. The last answer is kept so it works offline, and is deleted when you turn this off. A VPN will make it report the VPN's location.

Coordinates you enter, and the detected location, are saved in `~/.local/state/omarchy/settings/heliotrope.json`, a name kept from when the plugin was called Heliotrope. Sundown keeps that file readable only by your account.

## Commands

```bash
ID=io.github.williamjohndavisiii.heliotrope.control

omarchy-shell $ID status
omarchy-shell $ID on
omarchy-shell $ID off
omarchy-shell $ID toggle
omarchy-shell $ID set 3000              # Save a temperature, from 500 to 5500
omarchy-shell $ID schedule sun          # Or manual, or custom
omarchy-shell $ID hours 21:00 07:00     # Custom on and off times
omarchy-shell $ID location "52.52, 13.40"
omarchy-shell $ID location ""           # Back to the automatic location
omarchy-shell $ID ramp off              # Or on: change gradually
omarchy-shell $ID autolocation on       # Or off: detect location from your IP
omarchy-shell $ID offset 30             # Sun mode starts 30 min before sunset; negative is after
omarchy-shell $ID fade 60 30            # Warm-up and clear-down in minutes, 0 to 180
omarchy-shell $ID pausefullscreen on    # Or off
omarchy-shell $ID pauseapp gimp on      # Or off; steam_app_* matches every Steam game
omarchy-shell $ID pausefront            # Pause for the app in front, or stop
```

## Show it with Omarchy's indicators (optional)

To have the icon appear among Omarchy's indicators instead of on its own:

1. Set `showIcon` to `false` on the Sundown entry in `~/.config/omarchy/shell.json`.
2. Add this to the `omarchy.indicators` `items`, using your own home folder:

```
../../../../../../../home/YOU/.config/omarchy/plugins/io.github.williamjohndavisiii.heliotrope/Indicator
```

The path is relative to Omarchy's indicators folder, because Omarchy has no supported way to load another plugin's indicator yet.

## Remove

```bash
omarchy plugin remove io.github.williamjohndavisiii.heliotrope
hyprctl hyprsunset identity
rm ~/.local/state/omarchy/settings/heliotrope.json
```

The second line clears any tint left on the screen, and the third deletes your settings.

## Requirements

Omarchy's Quickshell-based shell and `hyprsunset`, both of which come with Omarchy.

## Tested on

One MacBook Air (M1, 2020) running Asahi, where the tint survived a suspend and resume. This fork's changes: an Intel laptop on Omarchy 4.5.0 with Hyprland 0.56.2.

Omarchy's stock `~/.config/hypr/hyprsunset.conf` clears the tint at 07:00. Sundown reads that file, looks just after each of its switch times, and puts the tint back if the schedule still has it on, within a few seconds (a minute if `hyprsunset` switches late, as it can after a sleep). To avoid it altogether, remove the profile from the file.

Not verified yet: suspend and resume on other hardware. [omacom/omarchy#16052](https://github.com/omacom/omarchy/issues/16052) reports the tint disappearing on an AMD Framework laptop, and Sundown doesn't reapply it.

## Tests

```bash
TZ=America/New_York node tests/model.test.js
```

## Credits

Sundown is by [William John Davis III](https://github.com/WilliamJohnDavisIII).

This fork borrows ideas, not code, from [Dusk](https://github.com/ReidenXerx/omarchy-dusk) by ReidenXerx (MIT):

- starting before sunset
- a fade length you choose
- stepping aside for fullscreen apps and apps you name, with a button for the app in front
- spacing fade steps by what the eye can tell apart (a few mireds) and looking about once a minute
- reading `hyprsunset.conf`'s own switch times so they aren't taken for you, including one that lands late after a sleep

## License

MIT
