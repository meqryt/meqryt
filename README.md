import random
import os
from threading import Thread
from telegram import Bot, Update
from telegram.ext import Application, CommandHandler, ContextTypes
from telegram.constants import ParseMode
from flask import Flask

# Ваш токен от BotFather
TOKEN = "7688688149:AAHxTF3DyjEHQ8BktYshmm1GRYJXBKAkfc0"

# Список путей к GIF (добавлены расширения .gif)
gif_paths = [
    r"C:\Users\meqry\Downloads\Новая папка\preview1.gif",
    r"C:\Users\meqry\Downloads\Новая папка\preview2.gif",
    r"C:\Users\meqry\Downloads\Новая папка\preview3.gif",
    r"C:\Users\meqry\Downloads\Новая папка\preview4.gif",
    r"C:\Users\meqry\Downloads\Новая папка\preview5.gif",
]

# Хранение статуса запущенной задачи для каждого чата
running_jobs = set()

# Функция для отправки напоминания
async def send_reminder(context: ContextTypes.DEFAULT_TYPE):
    chat_id = context.job.data['chat_id']  # Получаем chat_id из данных задачи

    # Случайный выбор GIF из списка
    media_path = random.choice(gif_paths)

    reminder_message = (
        "Напоминание!\n\n"
        "@kzfdrules - правила.\n\n"
        "Прекращаем троллиться, оскорблять друг друга без причины, обсуждать людей, обсуждать группы "
        "и вести себя неадекватно. Нарушение правил может привести вплоть до бана.\n\n"
        "Актив = преф."
    )

    try:
        # Проверка существования файла
        if os.path.exists(media_path):
            # Отправляем только один случайный GIF как анимацию
            await context.bot.send_animation(chat_id=chat_id, animation=open(media_path, 'rb'), caption=reminder_message, parse_mode=ParseMode.HTML)
        else:
            print(f"Файл {media_path} не найден.")
    except Exception as e:
        print(f"Error sending reminder: {e}")

# Функция для проверки прав администратора
async def is_admin(bot, chat_id, user_id):
    member = await bot.get_chat_member(chat_id, user_id)
    return member.status in ("administrator", "creator")

# Команда /command1 для запуска напоминаний
async def command1(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = update.effective_chat.id  # Получаем ID чата
    user_id = update.effective_user.id  # Получаем ID пользователя, отправившего команду

    # Проверяем, является ли пользователь администратором
    if not await is_admin(context.bot, chat_id, user_id):
        await update.message.reply_text("Эта команда доступна только администраторам.")
        return

    # Проверяем, не запущена ли уже задача для этого чата
    if chat_id in running_jobs:
        await update.message.reply_text("Напоминания уже запущены для этого чата.")
        return

    await update.message.reply_text("Бот запущен! Напоминания будут отправляться каждые 30 минут.")

    # Запуск задачи каждые 30 минут, передаем chat_id в данных задачи
    context.job_queue.run_repeating(send_reminder, interval=1800, first=0, data={'chat_id': chat_id})

    # Добавляем чат в список активных задач
    running_jobs.add(chat_id)

# Flask приложение
app = Flask('')

@app.route('/')
def home():
    return "I'm alive"

def run_flask():
    app.run(host='0.0.0.0', port=80)

def keep_alive():
    t = Thread(target=run_flask)
    t.start()

# Основной код для Telegram-бота
def main():
    # Создаём приложение
    application = Application.builder().token(TOKEN).build()

    # Добавляем обработчик команды /command1
    application.add_handler(CommandHandler("command1", command1))

    # Запускаем приложение Telegram
    application.run_polling()

if __name__ == "__main__":
    keep_alive()
    main()
