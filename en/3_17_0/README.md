# Tuya iOS BizBundle

## Overview

Tuya iOS BizBundle is a vertical business bundle that contains logic modules and UI pages. It's designed to provide customers with the ability to quickly access the Tuya business module based on the Tuya Smart Home SDK.

Tuya iOS BizBundle currently offered includes:
- Mall
- Device Configuration
- Device Control
- IPC
- Scene
- Cloud Service
- FAQ
- Message Center

## Architecture

Tuya iOS BizBundle is opened in a service-oriented manner, and all function access is provided in the form of `Protocol`.

![Architecture](./pages/images/architecture.png)

There are two ways to use the protocol: **get service** and **provide service**

### Get Service

Obtain the instance which implement the service protocol provided by the BizBundle through `BizCore`, then call its service method to achieve the business purpose

```mermaid
classDiagram
    Protocol <|.. BizBundle : BizBundle implement servcie protocol
    Protocol .. BizCore
    BizCore <.. YourClass : Get instance of service
    class Protocol{
        +doSomeThing()
    }
    class BizCore{
        +serviceOfProtocol() id~Protocol~
    }
    class BizBundle{
    }
    class YourClass{
        +id~Protocol~ serviceImpl
    }
```

### Provide Service

Some BizBundle depend on services that are not implemented, At this time, you can provide your own class to implement the corresponding service, and register it to `BizCore` to improve the BizBundle functions

```mermaid
classDiagram
    Protocol <|.. YourClass : implement servcie protocol
    BizCore <.. YourClass : Register your service implement class or instance
    Protocol .. BizCore
    BizCore <.. BizBundle
    class BizBundle{
    }
    class Protocol{
        +doSomeThing()
    }
    class BizCore{
        +registerService(Protocol, Instance)
        +registerService(Protocol, Class)
        +registerRouteWithHandler(Block)
    }
    class YourClass{
        +doSomeThing()
    }
```









