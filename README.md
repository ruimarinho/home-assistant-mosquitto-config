# home-assistant-mosquitto-config

[![build status][travis-image]][travis-url]

A [mosquitto](https://mosquitto.org) config for integrating with [Home Assistant's MQTT component](https://home-assistant.io/docs/mqtt/).

## Introduction

[Home Assistant](https://home-assistant.io/docs/mqtt/broker/) ships with an embedded MQTT broker called [HBMQTT](https://pypi.python.org/pypi/hbmqtt). It works fine for most situations but if you're looking for something more powerful, including improved debug utilities, you'll have to integrate with an independently running MQTT broker, such as `mosquitto`.

`mosquitto` is a fast and lightweight open source broker that implements MQTT versions 3.1 and 3.1.1, the latter being the preferred one by Home Assistant. The integration is seamless but if you'd like to take advantage of the fenomenal [MQTT discovery](https://home-assistant.io/docs/mqtt/discovery/) of Home Assistant, then you need to configure it properly.

Persistence needs to be enabled otherwise if the `mosquitto` server is restarted it may lose the auto-discovery MQTT messages left by connected devices.

## Usage

Start by adding the [necessary configuration](https://github.com/ruimarinho/home-assistant-config/blob/master/includes/mqtt.yaml) to Home Assistant's `mqtt` component.

My config working dir is at `/volume1/applications/mosquitto/`, so all steps take that into consideration.

1. Create a `passwd` file to store the hash of the Home Assistant user.

```sh
❯ touch /volume1/applications/mosquitto/passwd
```

2. Generate a password for the user (e.g. `homeassistant`):

```sh
❯ docker run --rm -it -v /volume1/applications/mosquitto/config/passwd:/etc/mosquitto/passwd ruimarinho/mosquitto mosquitto_passwd -c /etc/mosquitto/passwd homeassistant
```

3. The docker image [ruimarinho/mosquitto](https://hub.docker.com/r/ruimarinho/mosquitto/) runs the mosquitto server with the user `mosquitto` (id=100, group=nogroup), which means we should map our files accordingly:

```sh
❯ mkdir /volume1/applications/mosquitto/{data,config}
❯ chown -R 100:65533 /volume1/applications/mosquitto/
❯ chmod 744 -R /volume1/applications/mosquitto/
```

Copy the contents of [mosquitto.conf](mosquitto.conf) to `/volume1/applications/mosquitto/config`.

4. Run the mosquitto server with the mapped folders:

```sh
❯ docker run --restart always \
  --name mosquitto \
  -p 1883:1883 \
  -v /volume1/applications/mosquitto/config:/etc/mosquitto \
  -v /volume1/applications/mosquitto/data:/var/lib/mosquitto \
  -d \
  ruimarinho/mosquitto -c /etc/mosquitto/mosquitto.conf -v
```

## Websockets

Currently, Home Assistant only supports MQTT websocket connections to the built-in broker. This configuration is already prepared for the eventuality of it supporting for external brokers as well (port `9001`).

## License

MIT

[travis-image]: https://img.shields.io/travis/ruimarinho/home-assistant-mosquitto-config.svg?style=flat-square
[travis-url]: https://travis-ci.org/ruimarinho/home-assistant-mosquitto-config
