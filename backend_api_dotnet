using Kaliko.ImageLibrary;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using SixLabors.ImageSharp.Processing;
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;
using RestSharp;
using System.Text.Json;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Net;
using RestSharp.Extensions;
using Newtonsoft.Json.Linq;
using System.Threading.Tasks;
using System.ComponentModel.DataAnnotations;
using System.Reflection;
using System.Threading;
using System.Diagnostics;

namespace MyTest
{
    public class QueryBillDto
    {
        [Required]
        public string key { get; set; }
        [Required]
        public string ext { get; set; }
        public string publicId { get; set; }
        public DateTime? beginDate { get; set; }
        public DateTime? endDate { get; set; }
    }

    public class UsageChatData
    {
        public string organization_id { get; set; }
        public int aggregation_timestamp { get; set; }
        public int n_requests { get; set; }
        public string operation { get; set; }
        public string snapshot_id { get; set; }
        public int n_context_tokens_total { get; set; }
        public int n_generated_tokens_total { get; set; }
        public string user_id { get; set; }
    }

    public class UsageDalleData
    {
        public int timestamp { get; set; }
        public int num_images { get; set; }
        public int num_requests { get; set; }
        public string image_size { get; set; }
        public string operation { get; set; }
        public string user_id { get; set; }
        public string organization_id { get; set; }
    }

    /// <summary>
    /// 返回给前端的用量数据
    /// </summary>
    public class UsageData
    {
        public string date { get; set; }
        public List<UsageDataItem> data { get; set; }
    }

    /// <summary>
    /// 用量数据明细
    /// </summary>
    public class UsageDataItem
    {
        public string model { get; set; }
        public int context_tokens_total { get; set; }
        public int generated_tokens_total { get; set; }
        public double amount { get; set; }
        public int count { get; set; }
        public bool exact { get; set; }

    }

    [TestClass]
    public class BillTest
    {
        //chat和base相关模型的价格，Tuple<context,generated>
        static Dictionary<string, Tuple<double, double>> chatPrices = new Dictionary<string, Tuple<double, double>>
                {
                    { "gpt-3.5-turbo", new Tuple<double, double>(0.0015, 0.002) },
                    { "gpt-3.5-turbo-0301", new Tuple<double, double>(0.0015, 0.002) },
                    { "gpt-3.5-turbo-0613", new Tuple<double, double>(0.0015, 0.002) },
                    { "gpt-3.5-turbo-1106", new Tuple<double, double>(0.0015, 0.002) },
                    { "gpt-3.5-turbo-instruct", new Tuple<double, double>(0.0015, 0.002) },
                    { "gpt-3.5-turbo-instruct-0914", new Tuple<double, double>(0.0015, 0.002) },
                    { "gpt-3.5-turbo-16k", new Tuple<double, double>(0.003, 0.004) },
                    { "gpt-3.5-turbo-16k-0613", new Tuple<double, double>(0.003, 0.004) },
                    { "gpt-4", new Tuple<double, double>(0.03, 0.06) },
                    { "gpt-4-0314", new Tuple<double, double>(0.03, 0.06) },
                    { "gpt-4-0613", new Tuple<double, double>(0.03, 0.06) },
                    { "gpt-4-1106-preview", new Tuple<double, double>(0.01, 0.03) },
                    { "gpt-4-vision-preview", new Tuple<double, double>(0.01, 0.03) },
                    { "gpt-4-32k", new Tuple<double, double>(0.06, 0.12) },
                    { "gpt-4-32k-0314", new Tuple<double, double>(0.06, 0.12) },
                    { "gpt-4-32k-0613", new Tuple<double, double>(0.06, 0.12) },
                    { "text-ada-001", new Tuple<double, double>(0.0004, 0.0004) },
                    { "text-babbage-001", new Tuple<double, double>(0.0005, 0.0005) },
                    { "text-curie-001", new Tuple<double, double>(0.002, 0.002) },
                    { "text-davinci-001", new Tuple<double, double>(0.02, 0.02) },
                    { "text-davinci-002", new Tuple<double, double>(0.02, 0.02) },
                    { "text-davinci-003", new Tuple<double, double>(0.02, 0.02) },
                    { "text-embedding-ada-002", new Tuple<double, double>(0.0001, 0.0001) },
                    { "text-embedding-ada-002-v2", new Tuple<double, double>(0.0001, 0.0001) }
                };

        //dalle相关模型的价格，image_size Tuple<price,是否为精确价格>
        static Dictionary<string, Tuple<double, bool>> dallePrices = new Dictionary<string, Tuple<double, bool>>
                {
                    { "1792x1024", new Tuple<double, bool>(0.1, false) },
                    { "1024x1792", new Tuple<double, bool>(0.1, false) },
                    { "1024x1024", new Tuple<double, bool>(0.04, false) },
                    { "1024x1024>2", new Tuple<double, bool>(0.04, true) },//两张以上的是dalle2
                    { "512x512", new Tuple<double, bool>(0.018, true) },
                    { "256x256", new Tuple<double, bool>(0.016, true) }
                };

        static string apiDomain = "https://api.openai.com";

        [TestMethod]
        public void TesQueryBase()
        {
            QueryBillDto dto = new QueryBillDto() { key = "sk-LE4XuFtT17c6kFJwLdeLUF4zOWOyjDXeNYiXo1BqYnuT3Blb", publicId = "user-nNchTlf0bOsxdsaTUFuZdPaw", ext = "1652831557593", beginDate = DateTime.Now.AddDays(-5), endDate = DateTime.Now.AddDays(-5) };
            var x = TesQuery(dto).Result;
        }

        public async Task<ResultSingle<dynamic>> TesQuery(QueryBillDto dto)
        {
            var result = new ResultSingle<dynamic>();

            if (dto.ext != "1652831557593")
            {
                result.Status = RStatus.S9999;
                result.Desc = "非法请求";
                return result;
            }

            // 计算起始日期和结束日期
            DateTime now = DateTime.Now;
            DateTime startDate = now.AddDays(-90);
            DateTime endDate = now.AddDays(1);

            // 设置API请求URL和请求头
            string urlSubscription = apiDomain + "/v1/dashboard/billing/subscription"; // 查是否订阅
            string urlUsage = $"{apiDomain}/v1/dashboard/billing/usage?start_date={startDate.ToString("yyyy-MM-dd")}&end_date={endDate.ToString("yyyy-MM-dd")}"; // 查使用量

            try
            {
                var useKey = dto.key.StartsWith("sk-");

                HttpClient client = new HttpClient();
                // 获取API限额
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", dto.key);
                HttpResponseMessage response = await client.GetAsync(urlSubscription);

                string subscriptionDataString = await response.Content.ReadAsStringAsync();
                JObject subscriptionData = JObject.Parse(subscriptionDataString);

                if (!response.IsSuccessStatusCode)
                {
                    if (response.StatusCode == HttpStatusCode.Unauthorized)
                    {
                        if ((string)subscriptionData["error"]["code"] == "invalid_api_key")
                        {
                            result.Status = RStatus.S9999;
                            result.Desc = "输入的key或会话密钥无效。";
                            return result;
                        }
                        else if ((string)subscriptionData["error"]["code"] == "expired_session_key")
                        {
                            result.Status = RStatus.S9999;
                            result.Desc = "会话密钥过期，请重新获取后查询。";
                            return result;
                        }
                        else if ((string)subscriptionData["error"]["code"] == "account_deactivated")
                        {
                            result.Status = RStatus.S9999;
                            result.Desc = "对应账户已被封禁。";
                            return result;
                        }
                    }
                }

                //按原逻辑走
                if (!useKey)
                {
                    // 判断是否过期
                    long timestamp_now = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
                    long timestamp_expire = (long)subscriptionData["access_until"];
                    if (timestamp_now > timestamp_expire)
                    {
                        result.Status = RStatus.S9999;
                        result.Desc = "对应账户额度已过期, 请登录OpenAI进行查看。";
                        return result;
                    }

                    double totalAmount = (double)subscriptionData["hard_limit_usd"];
                    bool is_subscribed = (bool)subscriptionData["has_payment_method"];

                    // 获取已使用量
                    response = await client.GetAsync(urlUsage);
                    string usageDataString = await response.Content.ReadAsStringAsync();
                    JObject usageData = JObject.Parse(usageDataString);
                    double totalUsage = (double)usageData["total_usage"] / 100;

                    // 如果用户绑卡，额度每月会刷新
                    if (is_subscribed)
                    {
                        // 获取当前月的第一天日期
                        int day = now.Day;  // 本月过去的天数
                        startDate = now.AddDays(-(day - 1)); // 本月第一天
                        urlUsage = $"{apiDomain}/v1/dashboard/billing/usage?start_date={startDate.ToString("yyyy-MM-dd")}&end_date={endDate.ToString("yyyy-MM-dd")}"; // 查使用量
                        response = await client.GetAsync(urlUsage);
                        usageDataString = await response.Content.ReadAsStringAsync();
                        usageData = JObject.Parse(usageDataString);
                        totalUsage = (double)usageData["total_usage"] / 100;
                    }

                    // 计算剩余额度
                    double remaining = totalAmount - totalUsage;

                    result.ReturnObject = new { total = totalAmount, usage = totalUsage, remaining, bind = is_subscribed, util = timestamp_expire, useKey, detail = new List<UsageData>() };
                }
                //使用key按天查询
                else
                {
                    //useKey=true，不显示总额和到期时间、显示详情
                    //useKey=false，显示总额和到期时间、不显示详情

                    //按天查询，汇总结果
                    var usageData = GetUsageByKey(client, dto.key, dto.publicId, dto.beginDate, dto.endDate);

                    result.ReturnObject = new { total = 0, usage = usageData.Sum(m => m.data.Sum(i => i.amount)), remaining = 0, bind = true, util = 0, useKey, detail = usageData };
                }
            }
            catch (Exception e)
            {
                result.Status = RStatus.S9999;
            }
            return result;
        }

        private List<UsageData> GetUsageByKey(HttpClient client, string key, string publicId, DateTime? beginDate, DateTime? endDate)
        {
            var result = new List<UsageData>();

            int requestCount = 0;

            if (beginDate == null || endDate == null)
            {
                DateTime now = DateTime.Now;
                beginDate = new DateTime(now.Year, now.Month, 1);
                endDate = now;
            }

            if ((endDate - beginDate).Value.Days > 31)
            {
                return result;
            }

            for (var date = beginDate.Value; date <= endDate; date = date.AddDays(1))
            {
                try
                {
                    // 用于限制请求速率的计数器和时间记录
                    if (requestCount > 4)
                    {
                        Thread.Sleep(TimeSpan.FromMinutes(1));
                        requestCount = 0;
                    }

                    string urlUsage = $"{apiDomain}/v1/usage?date={date:yyyy-MM-dd}{(string.IsNullOrWhiteSpace(publicId) ? "" : ("&user_public_id=" + publicId))}";

                    var response = client.GetAsync(urlUsage).Result;
                    var usageDataString = response.Content.ReadAsStringAsync().Result;
                    var usageData = JObject.Parse(usageDataString);

                    var chatData = ((JArray)usageData["data"]).ToObject<List<UsageChatData>>();
                    var dalleData = ((JArray)usageData["dalle_api_data"]).ToObject<List<UsageDalleData>>();

                    var usageResult = chatData.
                        Select(m => new { m.snapshot_id, m.n_context_tokens_total, m.n_generated_tokens_total, amount = chatPrices[m.snapshot_id].Item1 * m.n_context_tokens_total + chatPrices[m.snapshot_id].Item2 * m.n_generated_tokens_total }).GroupBy(m => m.snapshot_id).
                        Select(m => new UsageDataItem() { model = m.Key, context_tokens_total = m.Sum(t => t.n_context_tokens_total), generated_tokens_total = m.Sum(t => t.n_generated_tokens_total), amount = Math.Round(m.Sum(t => t.amount) / 1000, 2, MidpointRounding.AwayFromZero), count = m.Count(), exact = true }).ToList();

                    var dalleUsage = dalleData.Select(m =>
                    {
                        var size = (m.image_size == "1024x1024" && m.num_images > 2) ? "1024x1024>2" : m.image_size;
                        return new { amount = dallePrices[size].Item1 * m.num_images, m.num_requests, exact = dallePrices[size].Item2 };
                    }).ToList();

                    //增加dalle相关的数据
                    usageResult.Add(new UsageDataItem() { model = "dalle", context_tokens_total = 0, generated_tokens_total = 0, amount = Math.Round(dalleUsage.Sum(m => m.amount), 2, MidpointRounding.AwayFromZero), count = dalleUsage.Count, exact = !dalleUsage.Any(m => !m.exact) });

                    result.Add(new UsageData() { date = date.ToString("yyyy-MM-dd"), data = usageResult.OrderBy(m => m.model).ToList() });

                    // 更新请求计数
                    requestCount++;
                }
                catch (Exception ex)
                {

                }
            }

            return result;
        }
    }
}
