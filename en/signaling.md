## Introduction
To communicate with another peer you simply need to exchange contact information and the rest will be done by WebRTC. The process of connecting to the other peers is also known as signaling. Please note that peers with different signaling address can not connect to each other. 

## Deploy by Yourself
Deploy your own Signaling Service (Recommended)

- [cbsignal](https://github.com/cdnbye/cbsignal) (Written by Golang, stand-alone version)
- [gosignaler-cluster](https://github.com/cdnbye/gosignaler-cluster) (Written by Golang, cluster version)
 

## Use free Signaling Service
- 'wss://signal.cdnbye.com' (China，default)
- 'wss://opensignal.cdnbye.com'  (America)
- 'wss://signalcloud.cdnbye.com'  (Germany)
- 'wss://opensignal.gcvow.top' (China)
- 'wss://sigyy.dianzhenkeji.com' (China)

## Example

### Web SDK
```javascript
var hlsjsConfig = {
    p2pConfig: {
        wsSignalerAddr: 'ws://your.signalservice.com',
        // Other p2pConfig options provided by hlsjs-p2p-engine
    }
};
// Hls constructor is overriden by included bundle
var hls = new Hls(hlsjsConfig);
// Use `hls` just like the usual hls.js ...
```

### Android SDK
```java
P2pConfig config = new P2pConfig.Builder()
    .wsSignalerAddr("ws://your.signalservice.com")
    .build();
P2pEngine.initEngine(ctx, YOUR_TOKEN, config);
```

### iOS SDK
```objectivec
CBP2pConfig *config = [CBP2pConfig defaultConfiguration];
config.wsSignalerAddr = @"ws://your.signalservice.com";
self.engine = [[CBP2pEngine alloc] initWithToken:YOUR_TOKEN andP2pConfig:config];
```