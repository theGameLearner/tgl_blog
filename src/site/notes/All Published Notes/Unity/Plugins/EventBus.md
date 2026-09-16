---
{"dg-publish":true,"permalink":"/all-published-notes/unity/plugins/event-bus/"}
---

created: 2026-09-15
updated: 2026-09-16

link: https://github.com/tglGames-Plugins/tgl-event-bus
# Event Bus
A simple Event Bus with Channel support to use in Unity applications.

### Limitation
Known Limitations:
- The Events are fire and forget, so we do not throw error if there are no listeners for the events being raised.


### Aim
A simple Event Bus plugin for Unity.  
Should be able to create custom events and channels for each of these custom events, then new events are created that follow the channels and treat the channel as a stand alone event bus.

Can later be improved into a MessageBus or we can add a message bus in same package later.
- *Primary Scope*: **Cross-Process / Distributed**: Can send data between threads, microservices, or networks.
- *Communication Style*: **Multi-Pattern**: Sub, Point-to-Point (Queues), Request-Response (RPC), and Command routing.
- *Payload Nature*: **Commands & Messages**: Can be Events, Commands ("Do X"), or Queries ("Get Y").
- *Reliability & Persistence*: Supports queuing, persistence, retries, and dead-letter channels for guaranteed delivery.
- *Coupling*: Low temporal coupling (producer can push a message even if the consumer is offline).

### How to Use
To use the package, you can follow the steps:
- Create a Interface which will be base for all your events, for e.g., `IAudioEventBase`  
- Create a Channel `AudioModuleEventChannel` on which all events of type `IAudioEventBase` will be published and subscribed to.
- Now, we can define events like `PlayAudioEvent` which will be published and subscribed for in Channel `AudioModuleEventChannel`
- This event `PlayAudioEvent` can now be used as seen in  `SampleTest` and `SampleTest2`








---

[^1]: 
[^2]: 

