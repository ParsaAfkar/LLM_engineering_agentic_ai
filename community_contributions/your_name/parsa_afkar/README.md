Final Project
import streamlit as st
import requests
from openai import OpenAI
from dotenv import load_dotenv
import os

load_dotenv()

AVAL_API_KEY = os.getenv("aval_api_key")
WEATHER_API_KEY = os.getenv("weather_api_key")


client = OpenAI(
    api_key="aval_api_key",
    base_url="https://api.avalai.ir/v1"
)

def get_weather(city):
    url = "https://api.openweathermap.org/data/2.5/weather"
    params = {
        "q": city,
        "appid": "weather_api_key",
        "units": "metric",
        "lang": "fa"
    }
    r = requests.get(url, params=params)
    data = r.json()
    return f"{data['weather'][0]['description']}، دما {data['main']['temp']} درجه"
def travel_agent(prompt):
    response = client.chat.completions.create(
        model="gpt-4.1-mini",
        messages=[
            {"role": "system", "content": "تو یک دستیار برنامه‌ریز سفر هستی."},
            {"role": "user", "content": prompt}
        ]
    )
    return response.choices[0].message.content
st.set_page_config(page_title="دستیار سفر", layout="centered")

st.title("✈️ دستیار هوشمند برنامه‌ریزی سفر")
origin = st.text_input("مبدا سفر")
destination = st.text_input("مقصد سفر")
budget = st.number_input("بودجه (تومان)", min_value=0)
days = st.number_input("تعداد روز سفر", min_value=1)
travel_type = st.selectbox("نوع سفر", ["هواپیما", "قطار", "زمینی"])
if st.button("ساخت برنامه سفر"):
    weather = get_weather(destination)

    prompt = f"""
    مقصد: {destination}
    مبدا: {origin}  
    بودجه: {budget} تومان
    مدت: {days} روز
    نوع سفر: {travel_type}
    وضعیت آب‌وهوا: {weather}

    یک برنامه سفر پیشنهادی با هزینه تقریبی ارائه بده.
    """

    result = travel_agent(prompt)
    st.text_area("برنامه پیشنهادی سفر", result, height=300)
streamlit run final_project/app.py
