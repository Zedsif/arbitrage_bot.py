import requests
import time

# بيانات بوت تيليغرام
TELEGRAM_TOKEN = '7428237909:AAE-25PdnqyR9jpiEggYqo0k6FOeLyl76yg'
TELEGRAM_CHAT_ID = '6147633311'
ALERT_THRESHOLD = 2.0  # فرق الربح بالدينار

# دالة لإرسال رسالة على تيليغرام
def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    data = {"chat_id": TELEGRAM_CHAT_ID, "text": message}
    requests.post(url, data=data)

# جلب أفضل أسعار من Binance
def get_binance_price():
    url = "https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search"
    headers = {"Content-Type": "application/json"}
    data = {
        "page": 1,
        "rows": 1,
        "payTypes": [],
        "asset": "USDT",
        "tradeType": "BUY",
        "fiat": "DZD"
    }
    buy_resp = requests.post(url, headers=headers, json=data).json()
    buy_price = float(buy_resp['data'][0]['adv']['price'])

    data["tradeType"] = "SELL"
    sell_resp = requests.post(url, headers=headers, json=data).json()
    sell_price = float(sell_resp['data'][0]['adv']['price'])

    return buy_price, sell_price

# جلب أفضل أسعار من OKX
def get_okx_price():
    buy_url = "https://www.okx.com/v3/c2c/tradingOrders/books?quoteCurrency=dzd&baseCurrency=usdt&side=buy"
    sell_url = "https://www.okx.com/v3/c2c/tradingOrders/books?quoteCurrency=dzd&baseCurrency=usdt&side=sell"
    buy_resp = requests.get(buy_url).json()
    sell_resp = requests.get(sell_url).json()
    buy_price = float(buy_resp['data'][0]['price'])
    sell_price = float(sell_resp['data'][0]['price'])
    return buy_price, sell_price

# دالة للتحقق من فرصة الأربيتراج
def check_arbitrage():
    try:
        b_buy, b_sell = get_binance_price()
        o_buy, o_sell = get_okx_price()
        opportunities = []

        # فرصة شراء من OKX وبيع في Binance
        profit1 = b_sell - o_buy
        if profit1 >= ALERT_THRESHOLD:
            opportunities.append(f"💰 Buy from OKX @ {o_buy} DZD\nSell on Binance @ {b_sell} DZD\nProfit: {profit1:.2f} DZD")

        # فرصة شراء من Binance وبيع في OKX
        profit2 = o_sell - b_buy
        if profit2 >= ALERT_THRESHOLD:
            opportunities.append(f"💰 Buy from Binance @ {b_buy} DZD\nSell on OKX @ {o_sell} DZD\nProfit: {profit2:.2f} DZD")

        # إرسال التنبيهات عبر تيليغرام
        for msg in opportunities:
            send_telegram_message(msg)

    except Exception as e:
        send_telegram_message(f"❌ Error checking arbitrage:\n{str(e)}")

# تشغيل البوت بشكل دوري
while True:
    check_arbitrage()
    time.sleep(60)  # التحقق كل دقيقة

