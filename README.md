# Authing APP 扫码登录 Demo

文档：[Authing - APP 扫码登录 Web](https://docs.authing.cn/authing/scan-qrcode/app-qrcode)

## 使用

```shell
$ npm install http-server -g
$ http-server
```

然后打开浏览器扫码调试即可。

## 简介

引入 Authing CDN：

```
<script src="https://cdn.jsdelivr.net/npm/authing-js-sdk/dist/authing-js-sdk-browser.min.js"></script>
```

> 这里用到的 SDK 为 [authing-js-sdk](https://github.com/authing/authing.js)

初始化：

```javascript
const authing = new Authing({
  userPoolId: '59f86b4832eb28071bdd9214',
  host: {
    user: "http://localhost:3000/graphql",
    oauth: "http://localhost:3000/graphql"
  }
});
```

此 Sample 一共包含两个 Demo，[一个自动生成生成扫码表单](./examples/sample1/index.html)：

```javascript
authing.startAppAuthScanning({
  onSuccess(userInfo) {
    alert('扫码成功，请打开控制台查看用户信息')
    console.log(userInfo);
    // 存储 token 到 localStorage 中
    localStorage.setItem('token', userInfo.token);
  }
})
```

[一个手动调用生成二维码、轮询二维码状态、换取用户信息](./examples/sample2/index.html)：

```javascript
authing.geneQRCode({ scence: 'APP_AUTH' }).then(res => {
  if (res.code === 200) {
    const { qrcodeId, qrcodeUrl } = res.data
    $("#qrcode").attr('src', qrcodeUrl);
    $("#qrcode").show()
    authing.startPollingQRCodeStatus({
      qrcodeId,
      scence: 'APP_AUTH',
      interval: 1000,
      onPollingStart: function (intervalNum) {
        console.log("Start polling for qrcode status: ", intervalNum)
        // clearInterval(intervalNum)
      },
      onResult: function (res) {
        console.log("Got qrcode latest result: ", res)
      },
      onScanned: function (userInfo) {
        console.log("User scanned qrcode: ", userInfo)
      },
      onSuccess: function (data) {
        const { ticket, userInfo } = data;
        console.log(`User confirmed authorization: ticket = ${ticket}`, userInfo)
        // 获取用户信息
        authing.exchangeUserInfoWithTicket(ticket).then(res => {
          const { code } = res
          if (code === 200) {
            console.log("Exchange userInfo success: ", res)
          } else {
            console.log("Exchange userInfo failed: ", res)
          }
        })
      },
      onCancel: function () {
        console.log("User canceled authorization")
      },
      onExpired: function () {
        console.log("QRCode has expired.")
      },
      onError: function (data) {
        console.log("Chcek qrcode status failed: ", data)
      }
    })
  }
})
```
