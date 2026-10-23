# Nova SDS011 for WeeWX

Add SDS011 particulate concentrations, `pm2_5` and `pm10_0` in micrograms per
cubic meter, to an existing WeeWX station. The service queries a serial SDS011
sensor with configurable reading/sleep cycles. Catalog version 1.1.0 also polls
a local Sensor.Community-format HTTP endpoint when `url` is configured.

Copyright 2025 Ian Millard. Distributed under [GPLv3](LICENSE). This repository
mirrors upstream; the catalog archive includes separate Python 3 and HTTP-mode
repairs. Install the versioned archive, rather than a mirror branch download.

## Install

An existing WeeWX station is required. Replace `/path/to/weewx.conf`
with its configuration file before running the commands.

The catalog release requires Python >=3.10 and WeeWX >=5.2,<6.

Before running the commands, activate the existing virtual environment that
runs your WeeWX service. Both `python` and `weectl` must come from that
environment. For an OS-packaged WeeWX installation, use compatible distribution
packages instead, or migrate WeeWX and its service to a virtual environment
first. Installing into an unrelated virtual environment will not supply the
running service with dependencies.

Install `pyserial` in the WeeWX environment, including for HTTP mode because the
module imports it:

```sh
python -m pip install 'pyserial==3.5'
```

The distribution is named `pyserial`; it provides the `serial` import.
Do not install the different package named `serial`.

Use the `weectl` from your WeeWX Python environment:

```sh
weectl extension install https://github.com/weewx-packages/weewx-NovaSDS011/releases/download/v1.1.0/weewx-NovaSDS011-1.1.0.zip --config=/path/to/weewx.conf
```

Stock WeeWX 5 accepts this fixed URL or a downloaded copy of the same archive.
No catalog integration is required. Restart WeeWX yourself after configuration
changes.

## Use a serial sensor

Keep your existing station driver. The installer appends
`user.novaSDS011.NovaSDS011Service` to `process_services` in
`[Engine][[Services]]`. Configure:

```ini
[NovaSDS011]
    port = /dev/ttyUSB0
    timeout = 3.0
    read_period = 60
    sleep_period = 60
    sample_interval = 2
    json_output = /var/tmp/weewx-particles/particles.txt
    log_raw = false
```

Give the WeeWX service user serial-device access and permission to create/write
the JSON directory. Serial communication uses 9600 baud. Each reading period
keeps its last valid sample, then sleeps the sensor. Cached serial values
continue into LOOP packets between cycles, including after a read failure.

## Poll a local HTTP sensor

Replace the serial settings with your own device's URL:

```ini
[NovaSDS011]
    url = http://192.0.2.10/data.json
    timeout = 3.0
    http_poll_interval = 60
    json_output = /var/tmp/weewx-particles/particles.txt
```

The response must have `sensordatavalues` entries named `SDS_P1` for PM10 and
`SDS_P2` for PM2.5. HTTP mode bypasses serial acquisition and sleep/wake commands.
Invalid, negative and non-finite readings are rejected. Cached HTTP readings
expire after twice `http_poll_interval`; this is a local API input, not a cloud
account integration. No live endpoint or physical sensor was tested here.

## Output and removal

The service adds the latest concentrations to LOOP packets. Archive storage needs
`pm2_5` and `pm10_0` columns in your selected schema. It also replaces a JSON
snapshot after each successful cycle, for example:

```json
{"dateTime": 1738471834, "pm2_5": 12.3, "pm10_0": 24.7}
```

The file has mode 0644 and belongs to the writing process. The service does not
implement `file_owner` or `file_group` and does not need root for ownership
changes. The bundled `manifest.xml` is legacy metadata; its download, ownership
and service-list notes do not describe the catalog installer. Use this README
and `install.py` in the catalog archive.

Check your WeeWX logs for connection, sample and JSON-write errors. There is no
bundled manual test script or CLI. To remove the service:

```sh
weectl extension uninstall NovaSDS011 --config=/path/to/weewx.conf
```

Restart WeeWX afterwards. Original project:
[weewx-contrib/weewx-NovaSDS011](https://github.com/weewx-contrib/weewx-NovaSDS011).

<!-- Documentation updated 2026-10-23 by weewx-packages. -->
