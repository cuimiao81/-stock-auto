import tushare as ts
import requests
import json
import os

# ========== 你只需要改这里 ==========
# 从 Tushare 个人中心获取你的 token
TUSHARE_TOKEN = os.getenv("TUSHARE_TOKEN")
# 从飞书开放平台「凭证与基础信息」获取
APP_ID = "cli_a9d0bce7acb89ccb"
APP_SECRET = "OCm7MPTNdTKnZH6nYLibTbGTPuizqCht"
# 从飞书App个人资料页获取你的用户ID
USER_ID = "用户742053"
# 你想关注的ETF代码，这里以沪深300ETF（510300）为例
ETF_TS_CODE = "510300.SH"
# ====================================

def get_tenant_access_token():
    """获取飞书应用的 tenant_access_token"""
    url = "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal"
    headers = {"Content-Type": "application/json"}
    data = {"app_id": APP_ID, "app_secret": APP_SECRET}
    response = requests.post(url, headers=headers, json=data)
    if response.status_code == 200:
        return response.json()["tenant_access_token"]
    else:
        raise Exception(f"获取 tenant_access_token 失败: {response.text}")

def send_to_feishu(content):
    """发送文本消息到飞书"""
    tenant_access_token = get_tenant_access_token()
    url = "https://open.feishu.cn/open-apis/im/v1/messages"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {tenant_access_token}"
    }
    data = {
        "receive_id": USER_ID,
        "msg_type": "text",
        "content": json.dumps({"text": content})
    }
    response = requests.post(url, headers=headers, json=data)
    if response.status_code == 200:
        print("✅ 飞书消息发送成功")
    else:
        print(f"❌ 飞书消息发送失败: {response.text}")

def get_etf_fund_info(ts_code):
    """获取ETF基金的基本信息"""
    pro = ts.pro_api(TUSHARE_TOKEN)
    df = pro.fund_basic(ts_code=ts_code)
    if not df.empty:
        return df.iloc[0]
    return None

def get_etf_daily(ts_code, trade_date):
    """获取ETF的日行情数据"""
    pro = ts.pro_api(TUSHARE_TOKEN)
    df = pro.fund_daily(ts_code=ts_code, trade_date=trade_date)
    if not df.empty:
        return df.iloc[0]
    return None

if __name__ == "__main__":
    try:
        import datetime
        today = datetime.date.today().strftime("%Y%m%d")
        
        # 1. 获取ETF基本信息
        etf_info = get_etf_fund_info(ETF_TS_CODE)
        if not etf_info:
            send_to_feishu(f"⚠️ 未找到ETF {ETF_TS_CODE} 的基本信息")
            exit()
        
        # 2. 获取ETF今日行情数据
        etf_daily = get_etf_daily(ETF_TS_CODE, today)
        if not etf_daily:
            send_to_feishu(f"⚠️ 今日 {today} 暂无 {etf_info['name']} 数据")
            exit()
        
        # 3. 格式化消息内容
        msg = (
            f"📊 ETF 行情播报\n"
            f"名称: {etf_info['name']} ({ETF_TS_CODE})\n"
            f"日期: {today}\n"
            f"开盘: {etf_daily['open']:.4f}\n"
            f"收盘: {etf_daily['close']:.4f}\n"
            f"最高: {etf_daily['high']:.4f}\n"
            f"最低: {etf_daily['low']:.4f}\n"
            f"涨跌: {etf_daily['change']:.4f}\n"
            f"涨跌幅: {etf_daily['pct_chg']:.2f}%\n"
            f"成交额: {etf_daily['amount']:.2f} 万元\n"
            f"基金类型: {etf_info['fund_type']}\n"
            f"管理人: {etf_info['management']}"
        )
        
        # 4. 发送到飞书
        send_to_feishu(msg)
        
    except Exception as e:
        send_to_feishu(f"❌ ETF数据抓取失败: {str(e)}")
