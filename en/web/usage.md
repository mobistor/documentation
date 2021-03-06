
## Quick Start Demo

Copy the following code in your web page and run. Wait for a few seconds，and then open the same page from another browser. Now you have a direct P2P connection between two browsers without plugin! The first web peer will serve as a seed, if there is no one else in the same channel.
```html
<script src="https://cdn.jsdelivr.net/npm/cdnbye@latest"></script>
<video id="video" controls></video>
<p id="version"></p>
<h3>download info:</h3>
<p id="info"></p>
<script>
    document.querySelector('#version').innerText = `hls.js version: ${Hls.version}  cdnbye version: ${Hls.engineVersion}`;
    if(Hls.isSupported()) {
        var video = document.getElementById('video');
        var hls = new Hls({
            p2pConfig: {
                logLevel: true,
                live: false,        // set to true in live mode
                // Other p2pConfig options provided by CDNBye
            }
        });
        hls.loadSource('https://example.m3u8');
        hls.attachMedia(video);
        hls.on(Hls.Events.MANIFEST_PARSED, function(event, data) {
            video.play();
        });
        hls.p2pEngine.on('stats', function ({totalHTTPDownloaded, totalP2PDownloaded, totalP2PUploaded}) {
            var total = totalHTTPDownloaded + totalP2PDownloaded;
            document.querySelector('#info').innerText = `p2p ratio: ${Math.round(totalP2PDownloaded/total*100)}%, saved traffic: ${totalP2PDownloaded}KB, uploaded: ${totalP2PUploaded}KB`;
        });
    }
</script>
```

## Register Domain
[Register](/en/bindings.md) your domain to activate P2P service.

## Integrate to existing hls.js Project
Simply replace the hls.js script tag like:
 ```html
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
```
with
 ```html
<script src="https://cdn.jsdelivr.net/npm/cdnbye@latest"></script>
```
That's it!


## Integrate to HTML5 Players
See [Player Integration](/en/web/players.md)

## Include

#### Script
Include the pre-built script of latest version bundled with hls.js:
```html
<script src="https://cdn.jsdelivr.net/npm/cdnbye@latest"></script>
```
Or include the latest version without hls.js:
```html
<script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
<script src="https://cdn.jsdelivr.net/npm/cdnbye@latest/dist/hlsjs-p2p-engine.min.js"></script>
```

#### File
[Click me](https://cdnbye.oss-cn-beijing.aliyuncs.com/web_sdk/dist.zip)<br>This needs to be included before your player code. You can either prepend it to your compiled code or include it in a `<script>` before it.

#### Browserify / Webpack
```shell
npm install --save cdnbye
```
To include cdnbye you need to require it in the player module:
```javascript
var Hls = require('cdnbye');
```
If you are using ES6's import syntax:
```javascript
import Hls from 'cdnbye';
```

## Usage
#### Bundle
Add `p2pConfig` as a property of hlsjsConfig, then Create hls.js instance passing `hlsjsConfig` as constructor param.
```javascript
var hlsjsConfig = {
    debug: true,
    // Other hlsjsConfig options provided by hls.js
    p2pConfig: {
        logLevel: 'debug',
        // Other p2pConfig options if applicable
    }
};
// Hls constructor is overriden by included bundle
var hls = new Hls(hlsjsConfig);
// Use `hls` just like the usual hls.js ...
hls.loadSource(contentUrl);
hls.attachMedia(video);
hls.on(Hls.Events.MANIFEST_PARSED,function() {
    video.play();
});
```
#### Engine(the library without hls.js)

Create hls.js instance passsing `hlsjsConfig` as parameter. Create `P2PEngine` instance passing hls.js instance and `p2pConfig` as parameters. Call hls.js `loadSource` and `attachMedia` methods.
```javascript
var hlsjsConfig = {
    maxBufferSize: 0,       // Highly recommended setting
    maxBufferLength: 5,     // Highly recommended setting
    liveSyncDuration: 30,   // Highly recommended setting
};

var p2pConfig = {
    logLevel: 'debug',
    // Other p2pConfig options if applicable
};

var hls = new Hls(hlsjsConfig);
if (P2PEngine.isSupported()) {
    new P2PEngine(hls, p2pConfig);        // Key step
}

// Use `hls` just like your usual hls.js…
hls.loadSource(contentUrl);
hls.attachMedia(video);
hls.on(Hls.Events.MANIFEST_PARSED,function() {
    video.play();
});
```

## Support for IE
CDNBye use ES6 API which make it not supported for IE. @babel/polyfill can emulate a full ES2015+ environment. You can use CommonJS style to include required packages. First install these packages:
```bash
npm install --save @babel/polyfill
```
Then insert the code into the beginning of your entry file:
```javascript
require("@babel/polyfill");
```
Or include it in a `<script>` before the script of cdnbye:
```html
<script src="https://cdn.bootcss.com/babel-polyfill/7.4.4/polyfill.min.js"></script>
```

## Electron
CDNBye also supports [Electron](https://electronjs.org/)(CDNBye version>=0.10.0), you just need to register AppId and get a token from console:
```javascript
var hlsjsConfig = {
    p2pConfig: {
        token: YOUR_TOKEN,
        appName: YOUR_APP_NAME,
        appId: YOUR_APP_ID,
        // Other p2pConfig options if applicable
    }
};
```
Learn more [here](/en/bindings.md?id=app-id-and-token)


