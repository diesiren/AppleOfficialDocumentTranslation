# IAP 服务器通知类型

原文请查看[notification_type](https://developer.apple.com/documentation/appstoreservernotifications/notification_type)

描述AppStore发送该通知的IAP事件。类型为字符串。

## 可能的值
### CANCEL

表示苹果客户支持取消了该订阅或者用户升级了他们的订阅。cancellation_date的key包含了该更改的日期和时间。

### DID_CHANGE_RENEWAL_PREF

表示客户对其订阅的计划进行了更改，该更改将在下次续订时生效。当前激活的计划不受影响。

### DID_CHANGE_RENEWAL_STATUS

表示订阅续订状态的一个更改。在JSON响应中，检查`auto_renew_status_change_date_ms`字段来了解最新状态更新的日期和时间。检查`auto_renew_status`字段来了解续订的状态。

### DID_FAIL_TO_RENEW

表示一个订阅因为账单问题而续订失败了。检查`is_in_billing_retry_period`来了解该订阅的当前重试状态。如果订阅处于账单宽限期内，检查`grace_period_expires_date`来了解新的服务到期日期。

###  DID_RECOVER
表示过去续订失败的过期订阅已经成功续订。检查`expires_date`以确定下一个续订的日期和时间。

### DID_RENEW
表示客户的订阅已经成功自动续订了新的交易周期

### INITIAL_BUY
在用户最初购买订阅时发生。在你的服务端存储`latest_receipt`最为一个令牌来随时通过App Store验证用户的订阅状态。

### INTERACTIVE_RENEWAL
表示客户以交互的方式续订了一个订阅。可能用你的App的界面或者在AppStore的账户订阅设置里。立即提供服务。

### PRICE_INCREASE_CONSENT
表示AppStore已经开始询问用户同意您App的订阅价格上涨，在`unified_receipt.Pending_renewal_info`对象里，`price_consent_status`是`0`，表示AppStore正在征求用户的同意，并且尚未收到。除非用户同意新的价格，否则订阅不会自动续订。当顾客同意涨价时，系统设置`price_consent_status`为1。使用`verifyReceipt`来检查`receipt`用以查看更新的价格同意状态。

### REFUND
表示AppStore成功退款了一笔交易。`cancellation_date_ms`包含退款交易的时间戳。`original_transaction_id`和`product_id`标识原始交易和产品。`cancellation_reason`包含该原因。

### RENEWAL （沙盒下已弃用）
表示在过去失败的已过期的订阅已经成功自动续订。检查`expires_date`来确定下一个续订日期和时间。此通知在沙盒下已弃用，计划于2021年3月在生产环境中弃用。更新你现有的代码来依赖`DID_RECOVER`通知作为替代。

## 讨论
对于这些通知类型值描述的订阅和退款事件，你可以实时接收服务器通知并对其做出反应。该`notification_type`出现在响应体中。

```
重要
RENEWAL通知类型已经计划于2021年3月在生产环境中弃用，并在沙盒环境中已经弃用。更新任何现有的代码以改为依赖DID_RECOVER通知类型。
```

## 处理通知事件的用例
根据你从AppStore接收到的通知，处理客户产品和订阅生命周期的各种用例。以下是一些产品事件的例子和您可能希望收到的服务器通知：

|订阅或IAP事件|触发的通知类型|
|:--:|:--:|
|顾客完成了一个订阅的初始购买| INITIAL_BUY |
|订阅是激活的，用户升级到另一个SKU|CANCEL, DID_CHANGE_RENEWAL_STATUS, INTERACTIVE_RENEWAL|
|订阅是激活的，用户降级到另一个SKU|DID_CHANGE_RENEWAL_PREF|
|订阅已过期，用户重新订阅相同的SKU|DID_CHANGE_RENEWAL_STATUS|
|订阅已过期，用户重新订阅了另一个SKU(升级或降级)|INTERACTIVE_RENEWAL, DID_CHANGE_RENEWAL_STATUS|
|用户在AppStore账户的订阅设置中禁用了自动续期的订阅，从而有效地取消了订阅|DID_CHANGE_RENEWAL_STATUS|
|AppleCare退款了一个订阅|CANCEL, DID_CHANGE_RENEWAL_STATUS|
|因结算问题，订阅无法续订| DID_FAIL_TO_RENEW |
|AppStore通过一个账单重试恢复了过期的订阅| DID_RECOVER |
|账单重试失败后，订阅流失| DID_CHANGE_RENEWAL_STATUS |
|AppleCare成功退款了消耗型，非消耗型或非续期订阅的交易| REFUND |
|您已经提高了订阅价格，用户必须同意涨价才能自动续订| PRICE_INCREASE_CONSENT |
|订阅成功续订| DID_RENEW |

## 在沙盒中测试通知事件
当你使用沙盒苹果ID登录到AppStore时，你开发签名的App将使用沙盒环境。要在App Store Connect中创建一个沙盒苹果ID或测试账号，请参阅[创建沙盒测试账号](https://help.apple.com/app-store-connect/#/dev8b997bee1)

如果你为App启用了AppStore服务器通知，请在沙盒环境中测试交易逻辑。要确定一个订阅通知事件是否发生在沙盒环境，请检查在JSON响应体对象中的`environment`字段是否等于`Sandbox`。

以下通知类型在沙盒中是可用的：`INITIAL_BUY`, `DID_CHANGE_RENEWAL_PREF`, `DID_CHANGE_RENEWAL_STATUS`, `DID_RENEW`, 和 `INTERACTIVE_RENEWAL`.

有关测试IAP的更新信息，请参阅[使用沙盒测试应用内购买](https://developer.apple.com/documentation/storekit/in-app_purchase/testing_in-app_purchases_with_sandbox)
