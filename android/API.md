
## P2P配置
用建造者模式实例化`CBP2pConfig`，以下的参数是默认值：
```java
P2pConfig config = new P2pConfig.Builder()
    .logEnabled(false)                                // 是否打印日志
    .logLevel(LogLevel.WARN)                          // 打印日志的级别
    .announce("https://tracker.cdnbye.com/v1")        // tracker服务器地址
    .wsSignalerAddr("wss://signal.cdnbye.com")        // 信令服务器地址
    .downloadTimeout(10_000, TimeUnit.MILLISECONDS)   // HTTP下载ts文件超时时间
    .dcDownloadTimeout(6_000, TimeUnit.MILLISECONDS)  // datachannel下载二进制数据的最大超时时间
    .localPort(52019)                                 // 本地代理服务器的端口号
    .diskCacheLimit(1024*1024*1024)                   // 点播模式下P2P在磁盘缓存的最大数据量(设为0可以禁用磁盘缓存)
    .memoryCacheCountLimit(30)                        // P2P在内存缓存的最大数据量，用ts文件个数表示
    .p2pEnabled(true)                                 // 开启或关闭p2p engine
    .wifiOnly(false)                                  // 是否只在wifi和有线网络模式上传数据（建议在云端设置）
    .withTag("unknown")                               // 用户自定义的标签，可以在控制台查看分布图
    .webRTCConfig(null)                               // 通过webRTCConfig来修改WebRTC默认配置
    .maxPeerConnections(20)                           // 最大连接节点数量
    .useHttpRange(true)                               // 在可能的情况下使用Http Range请求来补足p2p下载超时的剩余部分数据
    .isSetTopBox(false)                               // 是否机顶盒设备，如果在机顶盒运行设为true，提高兼容性
    .setUserAgent(null)                               // 设置请求ts时候的User-Agent
    .build();  
P2pEngine.initEngine(getApplicationContext(), token, config);
```

## P2P Engine
实例化P2pEngine，获得一个全局单例：
```java
P2pEngine engine = P2pEngine.initEngine(Context context, String token, P2pConfig config);
```
参数说明:
<br>

| 参数 | 类型 | 是否必须 | 说明 |
| :-: | :-: | :-: | :-: |
| `context` | Context | 是 | 建议使用Application 的 Context 对象。                                                                                      
| `token` | String | 是 | CDNBye分配的token。
| `config` | P2pConfig | 否 | 自定义配置。

### P2pEngine API
#### `P2pEngine.Version`
当前插件的版本号。

#### `P2pEngine.getInstance()`
获取`P2pEngine`的单例。

#### `engine.parseStreamUrl(String url)`
将原始播放地址(m3u8)转换成本地代理服务器的地址。

#### `engine.isConnected()`
是否已与CDNBye后台建立连接。

#### `engine.stopP2p()`
停止P2P加速并释放资源，一般不需要调用。SDK采用"懒释放"的策略，只有在重启p2p的时候才释放资源。对于性能较差的设备起播耗时可能比较明显，建议在视频播放之前提前调用`engine.stopP2p()`。

#### `engine.restartP2p()`
重启P2P加速服务，一般不需要调用。

#### `engine.getPeerId()`
获取对等连接的id。

### P2P统计
通过`P2pStatisticsListener`来监听P2P下载信息：
```java
engine.addP2pStatisticsListener(new P2pStatisticsListener() {
            @Override
            public void onHttpDownloaded(long value) {
            }

            @Override
            public void onP2pDownloaded(long value) {
            }

            @Override
            public void onP2pUploaded(long value) {
            }

            @Override
            public void onPeers(List<String> peers) {
            }
            
            @Override
            public void onServerConnected(boolean connected) {
            }
        });
```
PS：下载和上传数据量的单位是KB。

## 高级用法
### 切换源
当播放器切换到新的播放地址时，只需要将新的播放地址(m3u8)传给`P2pEngine`，从而获取新的本地播放地址：
```java
String newParsedURL = P2pEngine.getInstance().parseStreamUrl(url);
```

### 切换信令
某些场景下需要动态修改信令地址，防止单个信令负载过大，例如根据播放地址的哈希值选择信令。可以通过调用`engine.setConfig(config)`运行时动态调整配置，示例如下：
```java
P2pConfig config = new P2pConfig.Builder()
        .wsSignalerAddr("wss://yoursignal2.com")
        .build();
P2pEngine.getInstance().setConfig(config);
```
需要注意的是这个方法会重置`P2pEngine`的所有config，因此之前已经修改的字段需要再设置一次以保持一致。

### 自行配置 STUN 和 TURN 服务器地址
STUN用于p2p连接过程中获取公网IP地址，TURN则可以在p2p连接不通时用于中转数据。本SDK已内置公开的STUN服务，开发者可以通过P2pConfig来更换STUN地址。TURN服务器则需要开发者自行搭建，可以参考[coturn](https://github.com/coturn/coturn)。
```java
import org.webrtc.PeerConnection;
import org.webrtc.PeerConnection.RTCConfiguration;

List<PeerConnection.IceServer> iceServers = new ArrayList<>();
iceServers.add(PeerConnection.IceServer.builder(YOUR_STUN_OR_TURN_SERVER).createIceServer());
RTCConfiguration rtcConfig = new RTCConfiguration(iceServers);
P2pConfig config = new P2pConfig.Builder()
    .webRTCConfig(rtcConfig)
    .build();
```
### 解决动态m3u8路径问题
某些流媒体提供商的m3u8是动态生成的，不同节点的m3u8地址不一样，例如example.com/clientId1/file.m3u8和example.com/clientId2/file.m3u8, 而本插件默认使用m3u8作为channelId。这时候就要构造一个共同的chanelId，使实际观看同一直播/视频的节点处在相同频道中。在构造P2pConfig时通过在ChannelIdCallback的onChannelId方法中重新构造channelId，再把config传给P2pEngine即可：
```java
import com.cdnbye.sdk.ChannelIdCallback;

P2pConfig config = new P2pConfig.Builder()
    .channelId(new ChannelIdCallback() {
        @Override
        public String onChannelId(String urlString) {
            return formatChannelId(urlString);
        }
    }).build();
```
`强烈建议在chanelId中加入唯一标识符，防止与其他频道产生冲突。`
