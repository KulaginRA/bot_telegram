# bot_telegram
import logging
import requests
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext

# Включаем логирование
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# Замените 'YOUR_TELEGRAM_BOT_TOKEN' на токен вашего Telegram-бота
TELEGRAM_BOT_TOKEN = 'YOUR_TELEGRAM_BOT_TOKEN'

# Замените 'YOUR_OPENWEATHERMAP_API_KEY' на ваш API ключ OpenWeatherMap
OPENWEATHERMAP_API_KEY = 'YOUR_OPENWEATHERMAP_API_KEY'

def start(update: Update, context: CallbackContext) -> None:
    update.message.reply_text('Привет! Отправь мне название города, и я скажу тебе погоду!')

def get_weather(update: Update, context: CallbackContext) -> None:
    city = ' '.join(context.args)
    if not city:
        update.message.reply_text('Пожалуйста, укажите название города.')
        return

    url = f'http://api.openweathermap.org/data/2.5/weather?q={city}&appid={OPENWEATHERMAP_API_KEY}&units=metric&lang=ru'
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        weather_description = data['weather'][0]['description']
        temperature = data['main']['temp']
        feels_like = data['main']['feels_like']
        humidity = data['main']['humidity']
        wind_speed = data['wind']['speed']

        weather_message = (
            f"Погода в городе {city}:\n"
            f"Описание: {weather_description}\n"
            f"Температура: {temperature}°C\n"
            f"Ощущается как: {feels_like}°C\n"
            f"Влажность: {humidity}%\n"
            f"Скорость ветра: {wind_speed} м/с"
        )
        update.message.reply_text(weather_message)
    else:
        update.message.reply_text('Не удалось получить данные о погоде для указанного города. Пожалуйста, проверьте название города.')

def main() -> None:
    # Создаем Updater и передаем ему токен вашего бота
    updater = Updater(TELEGRAM_BOT_TOKEN)

    # Получаем диспетчера для регистрации обработчиков
    dispatcher = updater.dispatcher

    # Регистрируем обработчики команд
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("weather", get_weather))

    # Запуск бота
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
