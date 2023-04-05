# OKEx Auto Trade Script

> 我的另外一个项目是利用 GitHub Actions 的定时任务来做自动交易，工作流程是读取当前交易状态 -> 查询价格 -> 是否触发交易并创建订单 -> 创建订单成功后通知到 XX 应用 -> 结束。  
> 有兴趣的话可以联系我，代码没有开源，同时也是一个老项目，只是比这个新一些。

---

目前本项目处于开发初期阶段，可能会有部分异常未能处理。

> 在开始使用之前请你必须先**认真**完整阅读这份说明。

## 协议

- **个人使用遵守 [GNU GPL 3.0](./LICENSE)**
- **商业使用必须咨询作者以获得商业授权**

## 免责声明

- 本项目本质上只负责低买高卖，不构成任何投资建议，你需要自行承担一切风险！
- 从机制上看你完全可以手动在交易所挂单实现低买高卖，不一定需要依靠任何自动交易！
- 脚本终究是脚本！

## 简介

本项目是利用 OKEx API 自动交易，实现行情跌时买入，行情涨时卖出。有且只有币币交易。

你必须使用 OKEx 账户才能使用本项目；如果你没有 OKEx 账户可以[点击这里注册](https://www.okex.com/join/4710873)。

## 特点

- [x] 易用，JSON 文件配置
- [x] 安全交易，不先买入则不卖出
- [x] 可设置根据行情涨跌比率调整买入卖出价
- [x] 可设置每单买卖最小盈利 USDT 数量，用于保证盈利
- [x] 支持 Telegram 和 钉钉 交易通知

## 开始使用

> 先下载代码

### 0. 安装 Node.js 以及 NPM

你需要先[下载安装 Node.js 和 NPM](https://nodejs.org/zh-cn/)才可以运行，Node.js 默认包含 NPM。  
随后在本项目运行 `npm i` 命令安装必要的依赖包。

### 1. 创建环境变量文件(必须)

在项目根目录创建一个 `.env` 文件(注意有点)，接下来会用到。

### 2. 填写 OKEx API

打开并修改 `.env` 文件，这三个值在 [OKEx 我的账户 API](https://www.okex.com/account/my-api) 申请所得。

申请时权限需要勾选 `交易` 权限，`提现` 不需要勾选。

```bash
passphrase = "passphrase"
apiKey = "apiKey"
secretKey = "secretKey"
```

### 3. 设置代理

OKEx 的交易所 API 是需要大陆以外的 IP 才能访问的，如果你能正常访问可以跳过次步骤。

同样打开 `.env` 文件，修改这项参数。

如果你用的是 Clash 应该可以在设置界面找到端口，修改端口即可。

```bash
http_proxy = "http://127.0.0.1:7892"
```

### 4. 调整你的策略

打开并修改 `strategy.json` 文件。每个参数详细解释可以参考 [state.js](./app/store/state.js)。

其中标注解释：

- `@private` 属于内部参数，不允许出现在策略文件中，也不允许修改
- `@requires` 属于必填参数，必须在策略文件中填写正确

以 `ETH` 为例，基本策略配置如下所示。

```json
{
  "symbol": "ETH",
  "basePrice": 1950,
  "amounts": [0.001, 0.002, 0.003],
  "allow24hFluctuationRatio": 0.05,
  "baseDecreaseRatio": 0.02,
  "baseIncreaseRatio": 0.02,
  "dynamicNextPrice": true,
  "maxStep": 10,
  "safeSellAmount": false
}
```

### 5. 开始自动交易

当你调整好你理想的策略后，运行这条命令：

```console
npm run start
```

随后你会看到如下界面，证明它在运行中。

![](https://cdn.jsdelivr.net/gh/2nthony/github-itself-image-hosting-service@main/uPic/fvCujfB6fN8G.png)

如果你相信这个脚本，那么可以将它扔在服务并让它一直运行。

## 事件通知

### 钉钉

请阅读 [如何开启钉钉交易通知](https://2nthony.notion.site/66c43cd0d9054a6b98adb1798055eef3)

### Telegram

打开并修改 `.env` 文件，这两个参数需要在 Telegram 申请机器人获得。你可以 Google 如何申请 Telegram 机器人，很简单。

```bash
telegram_bot_token = "token"
telegram_bot_chat_id = "chat_id"
```

通知如图所示

![](https://cdn.jsdelivr.net/gh/2nthony/github-itself-image-hosting-service@main/uPic/R2PZ0iOpeoy5.png)

## 注意事项

### 初期使用建议从小尝试

在刚开始使用本项目时，建议你设置购买数量 `amounts` 比较小的值，防止风险，但也尽量避免是目标币种的最小交易数量，理由阅读下面一条注意事项；待你觉得可行的时候再适当调节。

### 最小盈利 USDT 数量

最小盈利 USDT `minProfitUSDT` 本意是让你的每一单买卖都能最小盈利你设定的 USDT 数量以避免亏本买卖，但并**不建议**依靠此功能。其中包括手续费等各项扣除计算，最终得出合适的卖出价位并变更下次卖出价格，实际盈利可能会有略微偏差，你可以适当提高 `amounts` 或提高 `baseDecreaseRatio` 和 `baseIncreaseRatio` 来避免。

### 买入手续费可能是挂单或吃单

原因是达到触发买入的价格，在调用 API 下单的时候，行情仍然会跌或涨。

### 交易账户上有足够的 USDT 和目标币种

你的交易账户需要足够的 USDT 使得可以在触发买入价时正常买入；有足够的目标币种使得可以在触发卖出价时正常卖出，或者你将 `safeSellAmount` 设置为 `true`，只卖出你上次扣除手续费后所得的数量。

### 需要大陆以外的 IP 访问

在 开始使用-3.设置代理 中已经提到。

### 每次触发下单请求会冷却 33 秒(买入卖出共享)

除了是 OKEx 下单的 API 限速之外，也防止短时间跌幅过大而频繁买入。

### 单价差异较大币种切换注意

在切换单价差异较大的币种，如 `BTC` 切换到 `ETH` 是请务必注意你设定的 `basePrice` 是否合理。

## 捐赠

如果你认同本项目或对你有帮助，不妨考虑捐赠以支持我。

| METHOD | ADDRESS                                    |
| ------ | ------------------------------------------ |
| BTC    | bc1qa6m9yqe26k223zwak4cmnyv9axq0dxg6thdcuh |
| ETH    | 0x42f8c423d71dD30b42DA09F6FB8683b6a5a4A524 |
| DOGE   | A2X5K6X4NgYdRvoLkqqp4mptDtcJ88JU1r         |

## Sponsors

[![sponsors](https://cdn.jsdelivr.net/gh/2nthony/sponsors-image/sponsors.svg)](https://github.com/sponsors/2nthony)

## LICENSE

GNU GPL 3.0 © [2nthony](https://github.com/2nthony)
