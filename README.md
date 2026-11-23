# 线路票务售票演练

本地资料交换采用虚构机构、业务编号和文件，目录、原件与接收端响应均为合成数据。服务仅监听本机回环地址。

## 运行环境

使用 Node.js 22。启动前用 `npm ci` 恢复锁定清单，`npm start` 启动监听，`npm test` 检查来源接口及原件。监听端口可用 `PORT` 环境变量指定。申请示例保存于 `request.json`。

网页使用 Chromium。首次在独立环境配置时可运行 `npx playwright install chromium`。网页检查会操作真实 Chromium 中的查询与回执表单。

## 资料交换约定

售票页 `/portal/1` 的日期为运营服务日。班次标 `24:xx` 指服务日下一历日。站名后显示换乘节点和闸口，站内只有 `gates` 完整覆盖才可使用联程票。换乘产品可能作为单一票或按 `tickets` 出多张不同票。售票回条的报价号、有效期和占用状态以当前渠道出具，只有票务条款允许旅客类型及一次性优先顺序时可提交。超过时间的报价应重新查询，已确认支付不能当作自动释放。演练不收款。

报价项有 offer、两段实际运行区间及闸口。换乘最短耗时由 /fare/interchange 给出。无库存项也可能显示价格。演练钟来自 /book/time。settled 的占位到期由终端证实，和只看本地时钟超时不同。

来源入口用 HTTP 返回资料。选择 `first` 或 `change` 可以观察同一批业务材料的两个时点，请向 `PUT /_source/phase` 提交包含 `phase` 的 JSON 对象。启动时默认 `first`。每个监听实例的演练时点及回执相互独立，新进程不继承演练端的状态。

接口路径、请求键及返回材料由 `wire.json` 描述，附属原件在 `materials` 中。路径表列出本地演练内容，不含外部网络服务。对返回文件的地址应领取对应响应，获取时的状态、时间及内容归属以来源声明为准。

| 方法和路径 | 查询字段 | 录入字段 |
| --- | --- | --- |
| `GET /book/time` | 无 | 无 |
| `GET /fare/interchange` | 无 | 无 |
| `GET /timetable` | `channel`、`day` | 无 |
| `GET /fare/business` | 无 | 无 |
| `GET /book/active` | 无 | 无 |
| `POST /buy/hold` | 无 | `channel`、`journey`、`offer`、`requestId` |

网页 `/portal/1` 用于夜间列车报价。

回执可以按客户端办理号查询 `GET /receipts?requestId=…`。有时源端接受后会在返回前断开，查询仍能看到真实演练回执。
