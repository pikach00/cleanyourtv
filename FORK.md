# Weather under the clock, through a `headline` column

This branch is [cleanyourtv](https://github.com/Gathin23/cleanyourtv) with one optional column more in the
launcher's [game_day contract](docs/modules.md), `headline`, and the Weather add-on using it to put the
weather under Home's clock, over the wallpaper: the city and the sky ("LVIV · CLEAR"), then now and today's
high and low ("2°   ↑5°  ↓-1°").

![Weather line under the clock, Philips QM163E](https://raw.githubusercontent.com/pikach00/cleanyourtv/pr-assets/weather-headline/weather-under-clock.jpg)
*(city name blurred)*

It lives here instead of upstream because the launcher half adds about 470 B to the release APK, more than
the core launcher's size gate leaves room for
([#3649](https://github.com/Gathin23/cleanyourtv/pull/3649)).

## What it changes

- **Launcher** (`GameDay.java`, `MainActivity.java`): a game_day row that carries `headline` shows that text
  as it is in place of the countdown and "Live now", and while such a row is on Home the launcher asks the
  provider again every 10 minutes. It queries with a null projection and reads columns by name, so a provider
  without the column reads as before. No new view, string or permission; the launcher still has no network
  access.
- **Weather add-on** (`addons/weather`): `HomeLineProvider`, a read-only provider
  (`io.github.sumyabazar1.weather.home`) exported without a permission, as the game_day contract asks. It
  holds only what Home shows: the city name and the reading.

**Network:** the launcher has none. The Weather add-on already had `INTERNET` and uses it for
api.open-meteo.com only: it fetches when Home asks and the last reading is older than 30 minutes (5 after a
failure), never in the background. A reading older than three hours is not shown.

Nothing changes on Home until a theme or `config.xml` sets `game_day` to the add-on.

## Build and install

JDK 17 and Android SDK platform 34 with build-tools 34.0.0, as upstream:

```sh
git clone -b weather-headline https://github.com/pikach00/cleanyourtv.git
cd cleanyourtv/launcher
TV_HOST=192.168.1.50 ./build-and-install.sh
```

Then the Weather add-on, as in [its README](addons/weather/README.md#build-and-install), and choose a city in
the app. Last, point the launcher at it: `"game_day": "io.github.sumyabazar1.weather"` in your theme's
`config.json`, or `game_day` in `launcher/app/src/main/res/values/config.xml` before building.

**Tested on:** Philips QM163E (QM16XE_U, MediaTek MT5890), Android TV 8.0.0, with the nature-calm theme: the
line showed after the city was chosen and refreshed when Home came back, with the same panel and dock layout
as without it. This TV's clock ran two hours behind after a night, which is how the add-on learnt to treat a
time ahead of the clock as old.

The branch is based on upstream `main` as of 9 November 2026.
