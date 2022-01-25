# stock-news-text
A software which send you a text message if a chosen stock increases or decreases in price by a specified amount. 

    import requests
    import os
    from twilio.rest import Client

    STOCK_NAME = "TSLA"
    COMPANY_NAME = "Tesla Inc"

    STOCK_ENDPOINT = "https://www.alphavantage.co/query"
    NEWS_ENDPOINT = "https://newsapi.org/v2/everything"

    STOCK_API_KEY = os.environ.get("STOCK_KEY")
    NEWS_API_KEY = os.environ.get("NEWS_KEY")
    TWILLIO_ACCOUNT_SID = os.environ.get("T_SID")
    TWILLIO_AUTH_TOKEN = os.environ.get("T_AUTH_TOK")
    MY_MOBILE_NUMBER = os.environ.get("MY_MOB")
    TWILLIO_NUMBER = os.environ.get("T_NUM")

    stock_params = {
        "function": "TIME_SERIES_DAILY",
        "symbol": STOCK_NAME,
        "apikey": STOCK_API_KEY,
    }

    response = requests.get(STOCK_ENDPOINT, params=stock_params)
    stock_data = response.json()["Time Series (Daily)"]
    data_list = [value for (key, value) in stock_data.items()]

    yesterday_data = data_list[0]
    yesterday_closing_price = yesterday_data["4. close"]

    day_before_yesterday_data = data_list[1]
    day_before_yesterday_closing_price = day_before_yesterday_data["4. close"]

    difference = float(yesterday_closing_price) - float(day_before_yesterday_closing_price)
    up_down = None
    if difference > 0:
        up_down = "🔺"
    else:
        up_down = "🔻"

    percentage_difference = round((difference / float(yesterday_closing_price)) * 100)

    if abs(percentage_difference) <= 5:
        news_params = {
            "ApiKey": NEWS_API_KEY,
            "qInTitle": COMPANY_NAME,
        }

        news_response = requests.get(NEWS_ENDPOINT, params=news_params)
        articles = news_response.json()["articles"]
        three_articles = articles[:3]

        article_list = [f"{STOCK_NAME}: {up_down}{percentage_difference}%\nHeadline: {article['title']}. "
                        f"Brief: {article['description']}" for article in three_articles]

        client = Client(TWILLIO_ACCOUNT_SID, TWILLIO_AUTH_TOKEN)

        for article in article_list:
            message = client.messages \
                .create(
                body=article,
                from_=TWILLIO_NUMBER,
                to=MY_MOBILE_NUMBER
            )
