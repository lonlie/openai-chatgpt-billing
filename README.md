# openai-chatgpt-billing

## 说明：
* 本项目实现实现免登录查询OpenAI ChatGPT用量及余额，包含前端html页面和后端的主要逻辑，简洁高效，可以自己部署实现。 
* 支持通过key(格式：sk-xxx)或者session key(格式：sees-xxx)查询。
* 同时适用于共享大号额度查询的场景，可以查询指定用户公共ID(user_public_id或user_id，格式：user-xxx)的用量情况。
* 支持指定日期用量查询。
* 查询结果包含：剩余额度、总额度、已用额度、是否绑卡、key到期时间。
* 针对官方每分钟5次的查询限制做了优化。 可以分别统计各模型分别的用量。

## 已部署好的版本，可直接在线使用：
### [https://gptbill.lonlie.cn/](https://gptbill.lonlie.cn/)
![image](https://github.com/lonlie/openai-chatgpt-billing/assets/12546332/d465d66a-89cd-4f49-ada3-a57f10882a22)

## 接口依据：
相关功能基于官方的接口实现，主要是下面两个：
> 账号订阅的情况（通过sees）：https://api.openai.com/v1/dashboard/billing/subscription

> 用量清单（通过sees）：https://api.openai.com/v1/dashboard/billing/usage?start_date=2023-12-1&end_date=2023-12-12

> 指定日期用量情况（通过key）：https://api.openai.com/v1/usage?date=2023-12-1&user_public_id=user-xxx

## 可参考的文章：
* 免登录在线查询OpenAI ChatGPT API key余额，https://blog.csdn.net/lonliecom/article/details/130564423
* OpenAI ChatGPT余额查询又不行了？2023-7-21，https://blog.csdn.net/lonliecom/article/details/131857048

## 现存问题：
* 通过key查询用量时，由于usage接口中没有提供具体使用了哪个dalle及图片质量参数，导致dalle相关相关模型的用量结果可能不精确

## 联系作者及充值购买相关账号额度资源：
![image](https://github.com/lonlie/openai-chatgpt-billing/assets/12546332/2c6168ae-f832-4640-84e3-18c69a5c30a9)
![image](https://github.com/lonlie/openai-chatgpt-billing/assets/12546332/063eead4-e1ed-4c0a-8721-da99e2637771)

