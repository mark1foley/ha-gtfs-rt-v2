# Home Assistant GTFS Realtime (rt)

This project contains a new sensor that provides real-time departure data for
local transit systems that provide gtfs feeds.

It is based on the excellent work that has been done previously by @zacs and @phardy.  Originally inspired by a desire to make the existing code work with realtime data for trains and buses provided by Translink in Queensland, Australia (who have unique route ids for each route/calendar combination) this version also contains a number of other improvements.

## Installation (HACS) - Recommended
0. Have [HACS](https://hacs.xyz/) installed, this will allow you to easily update
1. Add `https://github.com/mark1foley/ha-gtfs-rt-v2` as a [custom repository](https://hacs.xyz/docs/faq/custom_repositories/) as Type: Integration
2. Click install under "GTFS Realtime", then restart your instance for the installation to take effect.

## Installation (Manual)
1. Download this repository as a ZIP (green button, top right) and unzip the archive
2. Copy `/custom_components/gtfs_rt` to your `<config_dir>/custom_components/` directory
   * You will need to create the `custom_components` folder if it does not exist
   * On Hassio the final location will be `/config/custom_components/gtfs_rt`
   * On Hassbian the final location will be `/home/homeassistant/.homeassistant/custom_components/gtfs_rt`

## Configuration

Add the following to your `configuration.yaml` file:

```yaml
# Example entry for Queensland, Australia

sensor:
  - platform: gtfs_rt
    trip_update_url: 'https://gtfsrt.api.translink.com.au/api/realtime/SEQ/TripUpdates'
    vehicle_position_url: 'https://gtfsrt.api.translink.com.au/api/realtime/SEQ/VehiclePositions'
    route_delimiter: '-'
    departures:
    - name: Ferny Grove Train
      route: BNFG
      stopid: 600196
      icon: mdi:train
      service_type: Train
    - name: Uni Qld Ferry
      route: NHAM
      stopid: 319665
      icon: mdi:ferry
      service_type: Ferry
    - name: 1 0 7 Bus
      route: 107
      stopid: 4843
      icon: mdi:bus
      service_type: Bus
```

```yaml
# Example entry for Long Island Rail Road, New York

sensor:
  - platform: gtfs_rt
    trip_update_url: 'https://api-endpoint.mta.info/Dataservice/mtagtfsfeeds/lirr%2Fgtfs-lirr'
    vehicle_position_url: 'https://api-endpoint.mta.info/Dataservice/mtagtfsfeeds/lirr%2Fgtfs-lirr'
    x_api_key: <insert your API key here - see https://new.mta.info/developers>
    departures:
    - name: Bellmore Station to Penn Station
      route: '1'
      stopid: '16'
      directionid: '1'
      icon: mdi:train
      service_type: 'train'
```

Configuration variables:

- **trip_update_url** (*Required*): Provides route etas. See the **Finding Feeds** section at the bottom of the page for more details on how to find these
- **vehicle_position_url** (*Optional*): Provides live position tracking on the home assistant map
- **api_key_header** (*Optional*): If provided, this key will be included in the header, along with the api_key value. In not provided, the default of "Authorization" will be used (only if and api_key value is also given)
- **api_key** (*Optional*): If provided, this key will be sent with API requests in an "Authorization" (or whatever value is specified in api_header_key) header.
- **x_api_key** (*Optional*): If provided, this key will be sent with API requests in an "x_api_key" header.  Being maintained for legacy support only as it could also be specified in the api_header_key value
- **route_delimiter** (*Optional*): If provided, the text in the feed's route id before the delimiter is used as the route id.  Useful if the provider incorporates calendar ids into their route ids.1
- **departures** (*Required*): A list of routes and departure locations to watch
- **name** (*Required*): The name of the sensor in HA.  When displaying on the map card HA generates the name using the first letters of the first 3 words.  So, 1<space>0<space>7<space>Bus shows as "107" on the map.  Different labels can be defined when displaying the sensor on an entiry card etc.
- **route** (*Required*): The name of the gtfs route (if route_delimiter is used, the text before the delimiter)
- **stopid** (*Required*): The stopid for the location you want etas for
- **directionid** (*Optional*): Supports the direction_id from the GTFS feed trips.txt file, which indicates the direction of travel.  Use when the stops are direction neutral. **Caution:** Although added to the GTFS specification thein 2015, the direction_id field is still classified as *experimental*.  So there may be variations in implementation between providers or its use may be subject to change.
- **icon** (*Optional*): The icon used in HA for the sensor (default is mdi:bus if non supplied)
- **service_type** (*Optional*): The name used when created the "Next <service type>" attribute for the sensor in HA.  For example, Next Bus, Next Ferry etc etc (default is "Next Service" if non supplied)

## Screenshot

![screenshot](GTFS-RT-V2.JPG)

## Finding Feeds

[The Mobility Database](https://database.mobilitydata.org/) is a good source for realtime
gtfs feeds. Though the database is currently lacking a web interface, you can filter the [csv](https://bit.ly/catalogs-csv) on gtfs-rt in the data_type column and then further by location in order to find gtfs-rt feeds near you.

GTFS providers should also publish a zip file containing static data, including route and stop information. The same CSV file should contain a link to the static ZIP file. For example [Translink SEQ ZIP](https://gtfsrt.api.translink.com.au/GTFS/SEQ_GTFS.zip).  The route and stop IDs you need to configure the realtime feed in Home Assistant configuration are provided in this file.

## Troubleshooting

As it can be time-consuming performing troubleshooting in Home Assistant, a test.py script is provided that is almost identical code but can be run in any python3 environment.  It uses an input yaml file that is in the same format as the configuration file used in Home Assistant, making it quick and easy to test many different GTFS-RT providers, routes and stop configurations (see test_translink.yaml for an example).  The output can optionally be redirected to a text file.

Usage:
```
test.py -f <yaml file> -d INFO|DEBUG { -l <outfile log file> }
```

## Reporting an Issue

1. Setup your logger to print debug messages for this component using:
```yaml
logger:
  default: info
  logs:
    custom_components.gtfs_rt: debug
```
2. Restart Home Assistant
3. Verify you're still having the issue
4. File an issue in this Github Repository containing your HA log (Developer section > Info > Load Full Home Assistant Log)
   * You can paste your log file at pastebin https://pastebin.com/ and submit a link.
   * Please include details about your setup (Pi, NUC, etc, docker?, HASSOS?)
   * The log file can also be found at `/<config_dir>/home-assistant.log`
