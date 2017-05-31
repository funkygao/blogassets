---
title: VNC protocol
date: 2017-05-16 10:32:46
tags: protocol
---

## WebRTC

WebRTC提供了direct data and media stream transfer between two browsers without external server involved: P2P

浏览器上点击“Screen share”按钮后

```
// sender
利用OS的API获取screenshot，并以一定的FPS来进行发送
// 优化：把屏幕分成chunk，在把timer之间有变化的chunk生成frame
发送时，frame被编码成H.264或VP8
通过HTTPS发送

// receiver
对接收到的frame解码并显示
```

通过WebRTC实现的是只读的屏幕分享，receiver不能控制sender屏幕

实现
```
<body>
  <p><input type="button" id="share" value="Screen share" /></p>
  <p><video id="video" autoplay /></p>
</body>

<script>
navigator.getUserMedia = navigator.webkitGetUserMedia || navigator.getUserMedia;
$('#share').click(function() {
    navigator.getUserMedia({
         audio: false ,
         video: {
            mandatory: {
                chromeMediaSource: 'screen' ,
                maxWidth: 1280 ,
                maxHeight: 720
            } ,
            optional: [ ]
         }
    }, function(stream) {
       // we've got media stream
       // so the received stream can be transmitted via WebRTC the same way as web camera and easily played in <video> component on the other side
       document.getElementById('video').src = window.URL.createObjectURL(stream);
    } , function() {
       alert('Error. Try in latest Chrome with Screen sharing enabled in about:flags.');
    })
})
</script>
```

## VNC

Remote Frame Buffer，支持X11, Windows, Mac
远程终端用户使用机器（比如显示器、键盘、鼠标）的叫做客户端，提供帧缓存变化的被称为服务器

### 显示协议

pixel(x, y)  => 像素数据编码

### C->S消息类型

#### SetEncodings

Raw, CopyRect, RRE, Hextile, TRLE, ZRLE

#### FramebufferUpdateRequest

最重要的显示消息，表示client要server传回哪些区域的图像

```
client.send(messageType, incremental, x, y, width, height) => server
// incremental>0，表示该区域内容变化了才发给client；没有变化，就不用发

server.reply(messageType, rectangleN, [{x, y, with, height, color}, ...]) => client
```

#### KeyEvent

client端的键盘动作

#### PointerEvent

client端的鼠标动作

### vnc browser

http://guacamole.incubator.apache.org/
https://github.com/novnc/noVNC

## References

http://www.tuicool.com/articles/Rzqumu
https://github.com/macton/hterm
chrome://flags/#enable-usermedia-screen-capture
