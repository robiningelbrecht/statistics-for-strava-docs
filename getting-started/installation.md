---
icon: hammer
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Installation

{% hint style="info" %}
To run this application, you'll need [Docker](https://docs.docker.com/engine/install/) with [docker-compose](https://docs.docker.com/compose/install/).
{% endhint %}

Start off by showing some ❤️ and give this repo a star. Then from your command line:

```bash
# Create a new directory
> mkdir statistics-for-strava
> cd statistics-for-strava

# Create docker-compose.yml and copy the example contents into it
> touch docker-compose.yml
> nano docker-compose.yml

# Create .env and copy the example contents into it. Configure as you like
> touch .env
> nano .env
```

#### docker-compose.yml

```yaml
services:
  app:
    image: robiningelbrecht/strava-statistics:latest
    container_name: statistics-for-strava
    restart: unless-stopped
    volumes:
      - ./build:/var/www/build
      - ./storage/database:/var/www/storage/database
      - ./storage/files:/var/www/storage/files
      - ./storage/gear-maintenance:/var/www/storage/gear-maintenance
    env_file: ./.env
    ports:
      - 8080:8080
```

#### .env

{% hint style="warning" %}
Every time you change the .env file, you need to restart your container for the changes to take effect.
{% endhint %}

```bash
# ⚠ ️Every time you change the .env file, you need to restart your container for the changes to take effect.

# The URL on which the app will be hosted. This URL will be used in the manifest file. 
# This will allow you to install the web app as a native app on your device.
MANIFEST_APP_URL=http://localhost:8080/
# The client id of your Strava app.
STRAVA_CLIENT_ID=YOUR_CLIENT_ID
# The client secret of your Strava app.
STRAVA_CLIENT_SECRET=YOUR_CLIENT_SECRET
# You will need to obtain this token the first time you launch the app. 
# Leave this unchanged for now until the app tells you otherwise.
# Do not use the refresh token displayed on your Strava API settings page, it will not work.
STRAVA_REFRESH_TOKEN=YOUR_REFRESH_TOKEN_OBTAINED_AFTER_AUTH_FLOW
# Strava API has rate limits (https://github.com/robiningelbrecht/statistics-for-strava/wiki),
# to make sure we don't hit the rate limit, we want to cap the number of new activities processed
# per import. Considering there's a 1000 request per day limit and importing one new activity can
# take up to 3 API calls, 250 should be a safe number.
NUMBER_OF_NEW_ACTIVITIES_TO_PROCESS_PER_IMPORT=250
# The schedule to periodically run the import and HTML builds. Leave empty to disable periodic imports.
# The default schedule runs once a day at 04:05. If you do not know what cron expressions are, please leave this unchanged
# Make sure you don't run the imports too much to avoid hitting the Strava API rate limit. Once a day should be enough.
IMPORT_AND_BUILD_SCHEDULE="5 4 * * *"
# Set the timezone used for the schedule
# Valid timezones can found under TZ Identifier column here: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
TZ=Etc/GMT

# Allowed options: en_US, fr_FR, nl_BE, de_DE, pt_BR, pt_PT or zh_CN
LOCALE=en_US
# Allowed options: metric or imperial
UNIT_SYSTEM=metric
# Time format to use when rendering the app
# Allowed formats: 24 or 12 (includes AM and PM)
TIME_FORMAT=24
# Date format to use when rendering the app
# Allowed formats: DAY-MONTH-YEAR or MONTH-DAY-YEAR
DATE_FORMAT=DAY-MONTH-YEAR
# Sport types to import. Leave empty to import all sport types
# With this list you can also decide the order the sport types will be rendered in.
# A full list of allowed options is available on https://github.com/robiningelbrecht/statistics-for-strava/wiki/Supported-sport-types/
SPORT_TYPES_TO_IMPORT='[]'
# Activity visibilities to import. Leave empty to import all visibilities
# This list can be combined with SPORT_TYPES_TO_IMPORT.
# Allowed values: ACTIVITY_VISIBILITIES_TO_IMPORT='["everyone", "followers_only", "only_me"]', 
ACTIVITY_VISIBILITIES_TO_IMPORT='[]'
# Optional, the date (YYYY-MM-DD) from which you want to start importing activities. 
# Any activity recorded before this date, will not be imported.
# This can be used if you want to skip the import of older activities. Leave empty to disable.
SKIP_ACTIVITIES_RECORDED_BEFORE=''
# An array of activity ids to skip during import. 
# This allows you to skip specific activities during import.
# ACTIVITIES_TO_SKIP_DURING_IMPORT='["123456789", "987654321"]'
ACTIVITIES_TO_SKIP_DURING_IMPORT='[]'
# Your birthday. Needed to calculate heart rate zones.
ATHLETE_BIRTHDAY=YYYY-MM-DD
# History of weight (in kg or pounds, depending on UNIT_SYSTEM). Needed to calculate relative w/kg.
# Check https://github.com/robiningelbrecht/statistics-for-strava/wiki for more info.
ATHLETE_WEIGHT_HISTORY='{
    "YYYY-MM-DD": 100,
    "YYYY-MM-DD": 200
}'
# The formula used to calculate your max heart rate. The default is Fox (220 - age).
# Allowed values: arena, astrand, fox, gellish, nes, tanaka (https://pmc.ncbi.nlm.nih.gov/articles/PMC7523886/table/t2-ijes-13-7-1242/)
# Or you can set a fixed number for any given date range.  
MAX_HEART_RATE_FORMULA='fox'
# MAX_HEART_RATE_FORMULA='{
#    "2020-01-01": 198,
#    "2025-01-10": 193
# }'
# Optional, history of FTP. Needed to calculate activity stress level.
# Check https://github.com/robiningelbrecht/statistics-for-strava/wiki for more info. Example:
# FTP_HISTORY='{
#    "2024-10-03": 198,
#    "2025-01-10": 220
#}'
FTP_HISTORY='[]'
# Optional, a link to your profile picture. Will be used to display in the nav bar and link to your Strava profile.
# Leave empty to disable this feature.
PROFILE_PICTURE_URL=''
# Optional, your Zwift level (1 - 100). Will be used to render your Zwift badge. Leave empty to disable this feature
ZWIFT_LEVEL=
# Optional, your Zwift racing score (0 - 1000). Will be used to add to your Zwift badge if ZWIFT_LEVEL is filled out.
ZWIFT_RACING_SCORE=
# Optional, full URL with ntfy topic included. This topic will be used to notify you when a new HTML build has run.
# Leave empty to disable notifications.
NTFY_URL=''

# The UID and GID to create/own files managed by statistics-for-strava
# May only be necessary on Linux hosts, see File Permissions in Wiki
#PUID=
#PGID=
```

### Obtaining a Strava refresh token

The first time you launch the app, you will need to obtain a `Strava refresh token`. The app needs this token to be able to access your data and import it into your local database.

Navigate to [http://localhost:8080/](http://localhost:8080/). You should see this page—just follow the steps to complete the setup.

<figure><img src="https://github.com/robiningelbrecht/statistics-for-strava/raw/master/public/assets/images/readme/strava-oauth.png" alt=""><figcaption></figcaption></figure>
