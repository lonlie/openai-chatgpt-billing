# openai-chatgpt-billing

## 说明：
* 本项目实现实现免登录查询OpenAI ChatGPT用量及余额，包含前端html页面和后端的主要逻辑，简洁高效，可以自己部署实现。 
* 支持通过key(格式：sk-xxx)或者session key(格式：sees-xxx)查询。
* 同时适用于共享大号额度查询的场景，可以查询指定用户公共ID(user_public_id或user_id，格式：user-xxx)的用量情况。
* 支持指定日期用量查询。
* 查询结果包含：剩余额度、总额度、已用额度、是否绑卡、key到期时间。
* 针对官方每分钟5次的查询限制做了优化。 可以分别统计各模型分别的用量。
