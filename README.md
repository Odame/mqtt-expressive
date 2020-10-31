# mqtt-expressive

[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)
[![License](https://badgen.net/github/license/toddbluhm/ts-standard)](https://github.com/Odame/mqtt-expressive/blob/master/LICENSE)

A utility library to **express**ively handle incoming [`MQTT`](https://mqtt.org/) messages by redirecting to specific callbacks based on topics. Heavily inspired by the awesome routing pattern of [`Express`](https://expressjs.com/).

A wrapper around the popular [`mqtt`](https://www.npmjs.com/package/mqtt) npm package built for Node.JS server environment.

Just run `yarn add @Odame/mqtt-expressive` or `npm install @Odame/mqtt-expressive` in your nodejs project and reap benefits 😌.

📣 **This is a work in progress and as such not recommended for use in production yet.**

## Why is this needed

In NodeJS, setting up an mqtt client to listen for messages is done like this (taken from the ):

**The *[insert negative word here]* way** 🚫

```typescript
// index.ts
import mqtt from 'mqtt';
const client = mqtt.connect('mqtt://test.mosquitto.org')
    .on('message', function (topic, message) {
        switch (topic) {
            case '/home/outside/lights/status':
                console.log('outside lights status changed to', message);
                break;
            case '/home/garden/lights/status':
                console.log('garden lights status changed to', message);
                break;
            case '/home/bedroom/temperature':
                console.log('Bedroom temperature is ', message);
                break;
            case '/home/kitchen/temperature':
                console.log('Bedroom temperature is ', message);
                break;
            case '/sensors/temperature/001/state': {
                const data = JSON.parse(message.toString());
                console.log(
                    'temperature sensor 001 ',
                    data.batteryLevel, data.isBatteryCharging, data.uptime
                );
                break;
            }
            case '/sensors/humidity/735/state': {
                const data = JSON.parse(message.toString());
                console.log(
                    'humidity sensor 735 ',
                    data.batteryLevel, data.isBatteryCharging, data.uptime
                );
                break;
            }
            /* ... Handle other topics by adding specific cases here ... */
           default:
               console.error('Unhandled topic ', topic);
               break;
        }
    })
    .once('connect', function () {
        client.subscribe([
            '/home/+/lights/status',
            '/home/+/temperature',
            '/sensors/+/+/state'
        ]);
    });

```

This quickly becomes ugly and unmaintainable if there are messages from more than just a few topics to be handled.

**A much cleaner way with mqtt-expressive** 😍

`mqtt-expressive` allows you to write handlers for specific topics. It's like an onMessage callback, but for individual topics. This approach makes the intention/logic behind a topic (or set of topics) more expressive/declarative.

## Features

- [x] Supports both typescript and javascript
- [ ] Topic-based message handlers
- [ ] Params-based topics, like `express`
- [ ] Supports middlewares, like `express.use`
- [ ] Supports multi-nested routing, like `express.Router`
- [ ] Supports [shared subscriptions](https://docs.vernemq.com/configuration/shared_subscriptions)
- [ ] Supports sending response via `res.send()`
- [ ] Supports param-based middleware `router.param`
- [ ] Detailed documentation

## Basic Usage

```typescript
// index.ts
import mqttExpressive, {JSONMiddleWare} from '@Odame/mqtt-expressive';
import lightStatusController from './controllers/lightStatus';
import temperatureReadingController from './controllers/temperature';
import sensorStateController from './controllers/sensorState';
import {IClientOptions} from 'mqtt';

const app = mqttExpressive();

// same as app.sub(topic, handler) or app.subscribe(topic, handler)
app.get('/home/:section/lights/status', lightStatusController);
app.get('/home/:section/temperature', temperatureReadingController);
app.use(JSONMiddleware);
app.get('/home/sensors/:type/:id/state', sensorStateController);
// log unhandled topics (the default behaviour)
app.unhandled((topic, message) => console.log('Unhandled topic ', topic));
// optionally, attach handler for all (regardless of topic) messages
//   in addition to topic-specific handlers
app.all((topic, message) => console.log('app.all Received: ', topic, message))

app.start(
    'mqtt://test.mosquitto.org',
    (mqttOptions) => {
        console.log(`app listening on ${mqttOptions.host}`);
    }
);



// controllers/lightsStatus.ts
import {IMessageHandler} from '@Odame/mqtt-expressive';
type IMessage = string;
type IParams = {section: 'outside' | 'garden'};
const lightsStatusController: IMessageHandler<IMessage, IParams> = (req, res) => {
    console.log(`${req.params.section} lights status changed to ${req.body}`);
    res.end(); // calling res.end() is optional
}
export default lightsStatusController;



// controllers/temperatureReadings.ts
import {IMessageHandler} from '@Odame/mqtt-expressive';
type IMessage = string;
type IParams = {section: 'bedroom' | 'kitchen'};
const temperatureReadingsController: IMessageHandler<IMessage, IParams> = (req, res) => {
    console.log(`${req.params.section} temperature is ${req.body}`);
    res.end();
}
export default temperatureReadingsController;



// controllers/sensorsState.ts
import {IMessageHandler} from '@Odame/mqtt-expressive';
type IMessage = string;
type IParams = {type: 'temperature' | 'humidity', id: string};
const temperatureReadingsController: IMessageHandler<IMessage, IParams> = (req, res) => {
    console.log(
        `SensorType: ${req.params.type}, ID: ${req.params.id}`
        // req.body is parsed as json by the JSONMiddleware
        + ` Battery level ${req.body.batteryLevel} is ${req.body.isBatteryCharging}.`
        + `And has been on for ${req.body.uptime} minutes`
    );
    res.end();
}
export default temperatureReadingsController;
```

Based on the API of the popular `express` package, this is much cleaner and maintainable. Allows for easy testing.

## Installation

FIXME: Deploy to npm

Install from npm with yarn

```sh
 yarn add @Odame/mqtt-expressive
```

Or with npm

```sh
npm install @Odame/mqtt-expressive
```

## API Reference

TODO: Generate a more detailed API reference web page

## Tests

TODO: Add steps for running tests

## Built With

- [Typescript](https://www.typescriptlang.org/)

## Similar projects out there

- <https://github.com/taoyuan/mors>

## Contributing

Contributions are welcome! Check out the issues or the PRs, and make your own if you want something that you don't see there. See the [contributing guidelines](./CONTRIBUTING.md)

## License

[MIT](https://github.com/standard/standard/blob/master/LICENSE). Copyright &copy;
 [Prince Odame](https://princeodame.com)
