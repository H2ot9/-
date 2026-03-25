import os
import yt_dlp
import asyncio
from pyrogram import Client, filters
from pyrogram.types import InlineKeyboardMarkup, InlineKeyboardButton

# ⚡ اضف متغيرات البيئة في السيرفر
API_ID = int(os.environ.get("API_ID"))
API_HASH = os.environ.get("API_HASH")
BOT_TOKEN = os.environ.get("BOT_TOKEN")

CHANNEL_NAME = "Николас"
CHANNEL_LINK = "https://t.me/H2OT3"

app = Client("fastbot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)

buttons = InlineKeyboardMarkup([
    [InlineKeyboardButton(f"📢 {CHANNEL_NAME}", url=CHANNEL_LINK)]
])

# 🎉 رسالة /start
@app.on_message(filters.command("start") & filters.private)
async def start(client, message):
    await message.reply(
        "🎵 اهلا بك في ميوزك گروع!\n\n"
        "📌 اكتب:\n"
        "yt اسم الاغنية\n"
        "او\n"
        "يوت اسم الاغنية\n",
        reply_markup=buttons
    )

# ⚡ تحميل متعدد المستخدمين
def download_audio(query):
    ydl_opts = {
        'format': 'bestaudio[ext=m4a]/bestaudio',
        'outtmpl': '%(title)s.%(ext)s',
        'noplaylist': True,
        'quiet': True,
        'nocheckcertificate': True,
        'ignoreerrors': True,
        'geo_bypass': True,
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(f"ytsearch:{query}", download=True)
        info = info['entries'][0]
        file = ydl.prepare_filename(info)

    return file, info

@app.on_message(filters.text & filters.private)
async def download(client, message):
    text = message.text.lower()

    if text.startswith("yt "):
        query = message.text[3:].strip()
    elif text.startswith("يوت "):
        query = message.text[4:].strip()
    else:
        return

    msg = await message.reply("⚡ جاري التحميل بسرعة...")

    try:
        loop = asyncio.get_event_loop()
        file, info = await loop.run_in_executor(None, download_audio, query)

        await msg.delete()

        await message.reply_audio(
            audio=file,
            title=info.get("title"),
            performer=info.get("uploader"),
            reply_markup=buttons
        )

        os.remove(file)

    except Exception:
        await msg.edit("❌ صار خطأ، جرّب غير اسم")

app.run()
