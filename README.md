import telebot
from telebot import types

TOKEN = "8548257231:AAH97Kc9BTIN9V_ViQK3RNisYsjLBCG_xZQ"
ADMIN_ID = 1428411847

bot = telebot.TeleBot(TOKEN)
user_state = {}

def main_menu():
    kb = types.ReplyKeyboardMarkup(resize_keyboard=True)
    kb.add("🛒 خرید VPN")
    kb.add("🧑‍💻 پشتیبانی", "💳 روش پرداخت")
    kb.add("📘 آموزش")
    return kb

def vpn_plans_kb():
    kb = types.InlineKeyboardMarkup()
    kb.add(types.InlineKeyboardButton("✅ ادامه خرید", callback_data="vpn_continue"))
    kb.add(types.InlineKeyboardButton("🔙 برگشت", callback_data="back_main"))
    return kb

def vpn_choose_plan_kb():
    kb = types.InlineKeyboardMarkup()
    kb.add(types.InlineKeyboardButton("VPN ۳ روزه", callback_data="vpn_3"))
    kb.add(types.InlineKeyboardButton("VPN ۶ روزه", callback_data="vpn_6"))
    kb.add(types.InlineKeyboardButton("VPN ۱۰ روزه", callback_data="vpn_10"))
    kb.add(types.InlineKeyboardButton("🔙 برگشت", callback_data="back_main"))
    return kb

def cancel_kb():
    kb = types.ReplyKeyboardMarkup(resize_keyboard=True)
    kb.add("❌ لغو")
    return kb

@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(
        message.chat.id,
        "سلام 👋\nبه ربات ثبت سفارش خوش اومدی ✅\nاز منو یکی رو انتخاب کن:",
        reply_markup=main_menu()
    )

@bot.message_handler(func=lambda m: m.text == "🛒 خرید VPN")
def buy_vpn(message):
    text = (
        "📌 قبل از خرید توجه کنید:\n\n"
        "✅ فعالسازی فقط از طریق برنامه NPV Tunnel انجام می‌شود.\n"
        "✅ نیاز به اینترنت همراه (دیتای موبایل) دارد.\n\n"
        "اگر شرایط را دارید روی «ادامه خرید» بزنید."
    )
    bot.send_message(message.chat.id, text, reply_markup=types.ReplyKeyboardRemove())
    bot.send_message(message.chat.id, "ادامه؟", reply_markup=vpn_plans_kb())

@bot.message_handler(func=lambda m: m.text == "💳 روش پرداخت")
def payment(message):
    text = (
        "💳 روش پرداخت:\n\n"
        "1) کارت به کارت\n"
        "2) پرداخت آنلاین (در صورت فعال بودن)\n\n"
        "📌 بعد از پرداخت، رسید را در بخش پشتیبانی ارسال کنید."
    )
    bot.send_message(message.chat.id, text, reply_markup=main_menu())

@bot.message_handler(func=lambda m: m.text == "📘 آموزش")
def tutorial(message):
    text = (
        "📘 آموزش اتصال:\n\n"
        "1) برنامه NPV Tunnel را نصب کنید.\n"
        "2) فایل/کانفیگ را وارد کنید.\n"
        "3) اینترنت همراه روشن باشد.\n"
        "4) اتصال را بزنید.\n\n"
        "اگر مشکل داشتید از بخش پشتیبانی پیام بدهید."
    )
    bot.send_message(message.chat.id, text, reply_markup=main_menu())

@bot.message_handler(func=lambda m: m.text == "🧑‍💻 پشتیبانی")
def support(message):
    user_state[message.chat.id] = {"mode": "support"}
    bot.send_message(
        message.chat.id,
        "🧑‍💻 پیام پشتیبانی خود را بنویسید (متن/عکس) 👇\nبرای لغو: ❌ لغو",
        reply_markup=cancel_kb()
    )

@bot.message_handler(func=lambda m: m.text == "❌ لغو")
def cancel(message):
    user_state.pop(message.chat.id, None)
    bot.send_message(message.chat.id, "لغو شد ✅", reply_markup=main_menu())

@bot.callback_query_handler(func=lambda call: True)
def callbacks(call):
    chat_id = call.message.chat.id

    if call.data == "back_main":
        bot.edit_message_text("به منو برگشتی ✅", chat_id, call.message.message_id)
        bot.send_message(chat_id, "منوی اصلی:", reply_markup=main_menu())
        return

    if call.data == "vpn_continue":
        bot.edit_message_text("یکی از پلن‌ها رو انتخاب کن 👇", chat_id, call.message.message_id)
        bot.send_message(chat_id, "پلن VPN:", reply_markup=vpn_choose_plan_kb())
        return

    if call.data in ["vpn_3", "vpn_6", "vpn_10"]:
        plan_map = {"vpn_3": "۳ روزه", "vpn_6": "۶ روزه", "vpn_10": "۱۰ روزه"}
        plan = plan_map[call.data]

        user_state[chat_id] = {"mode": "vpn_order", "plan": plan}

        bot.send_message(
            chat_id,
            f"✅ پلن انتخاب شد: VPN {plan}\n\n"
            "حالا لطفاً اطلاعات زیر رو یکجا ارسال کن:\n"
            "1) نام و نام خانوادگی\n"
            "2) یوزرنیم تلگرام (اگر داری)\n"
            "3) توضیحات (مثلاً مدل گوشی/کشور/هرچی لازم می‌دونی)\n\n"
            "برای لغو: ❌ لغو",
            reply_markup=cancel_kb()
        )
        return

@bot.message_handler(content_types=['text', 'photo'])
def handle_all(message):
    chat_id = message.chat.id

    if chat_id not in user_state:
        bot.send_message(chat_id, "از منو انتخاب کن 👇", reply_markup=main_menu())
        return

    mode = user_state[chat_id].get("mode")

    if mode == "support":
        if message.content_type == "text":
            bot.send_message(
                ADMIN_ID,
                f"📩 پیام پشتیبانی جدید\n\n"
                f"👤 کاربر: {message.from_user.first_name}\n"
                f"🆔 ID: {chat_id}\n"
                f"📌 متن:\n{message.text}"
            )
        else:
            caption = message.caption if message.caption else ""
            bot.send_photo(
                ADMIN_ID,
                message.photo[-1].file_id,
                caption=(
                    f"📩 پیام پشتیبانی (عکس)\n\n"
                    f"👤 کاربر: {message.from_user.first_name}\n"
                    f"🆔 ID: {chat_id}\n"
                    f"📌 کپشن:\n{caption}"
                )
            )

        bot.send_message(chat_id, "پیامت ارسال شد ✅", reply_markup=main_menu())
        user_state.pop(chat_id, None)
        return

    if mode == "vpn_order":
        plan = user_state[chat_id].get("plan", "نامشخص")

        if message.content_type != "text":
            bot.send_message(chat_id, "لطفاً اطلاعات رو به صورت متن ارسال کن ✍️", reply_markup=cancel_kb())
            return

        bot.send_message(
            ADMIN_ID,
            f"🛒 سفارش جدید VPN\n\n"
            f"👤 کاربر: {message.from_user.first_name}\n"
            f"🆔 ID: {chat_id}\n"
            f"📦 پلن: {plan}\n\n"
            f"📝 اطلاعات:\n{message.text}"
        )

        bot.send_message(chat_id, "✅ سفارش ثبت شد. منتظر پیام ادمین باشید.", reply_markup=main_menu())
        user_state.pop(chat_id, None)
        return

print("Bot is running...")
bot.infinity_polling()    kb.add(types.InlineKeyboardButton("VPN ۱۰ روزه", callback_data="vpn_10"))
    kb.add(types.InlineKeyboardButton("🔙 برگشت", callback_data="back_main"))
    return kb

def cancel_kb():
    kb = types.ReplyKeyboardMarkup(resize_keyboard=True)
    kb.add("❌ لغو")
    return kb

@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(
        message.chat.id,
        "سلام 👋\nبه ربات ثبت سفارش خوش اومدی ✅\nاز منو یکی رو انتخاب کن:",
        reply_markup=main_menu()
    )

@bot.message_handler(func=lambda m: m.text == "🛒 خرید VPN")
def buy_vpn(message):
    text = (
        "📌 قبل از خرید توجه کنید:\n\n"
        "✅ فعالسازی فقط از طریق برنامه NPV Tunnel انجام می‌شود.\n"
        "✅ نیاز به اینترنت همراه (دیتای موبایل) دارد.\n\n"
        "اگر شرایط را دارید روی «ادامه خرید» بزنید."
    )
    bot.send_message(message.chat.id, text, reply_markup=types.ReplyKeyboardRemove())
    bot.send_message(message.chat.id, "ادامه؟", reply_markup=vpn_plans_kb())

@bot.message_handler(func=lambda m: m.text == "💳 روش پرداخت")
def payment(message):
    text = (
        "💳 روش پرداخت:\n\n"
        "1) کارت به کارت\n"
        "2) پرداخت آنلاین (در صورت فعال بودن)\n\n"
        "📌 بعد از پرداخت، رسید را در بخش پشتیبانی ارسال کنید."
    )
    bot.send_message(message.chat.id, text, reply_markup=main_menu())

@bot.message_handler(func=lambda m: m.text == "📘 آموزش")
def tutorial(message):
    text = (
        "📘 آموزش اتصال:\n\n"
        "1) برنامه NPV Tunnel را نصب کنید.\n"
        "2) فایل/کانفیگ را وارد کنید.\n"
        "3) اینترنت همراه روشن باشد.\n"
        "4) اتصال را بزنید.\n\n"
        "اگر مشکل داشتید از بخش پشتیبانی پیام بدهید."
    )
    bot.send_message(message.chat.id, text, reply_markup=main_menu())

@bot.message_handler(func=lambda m: m.text == "🧑‍💻 پشتیبانی")
def support(message):
    user_state[message.chat.id] = {"mode": "support"}
    bot.send_message(
        message.chat.id,
        "🧑‍💻 پیام پشتیبانی خود را بنویسید (متن/عکس) 👇\nبرای لغو: ❌ لغو",
        reply_markup=cancel_kb()
    )

@bot.message_handler(func=lambda m: m.text == "❌ لغو")
def cancel(message):
    user_state.pop(message.chat.id, None)
    bot.send_message(message.chat.id, "لغو شد ✅", reply_markup=main_menu())

@bot.callback_query_handler(func=lambda call: True)
def callbacks(call):
    chat_id = call.message.chat.id

    if call.data == "back_main":
        bot.edit_message_text("به منو برگشتی ✅", chat_id, call.message.message_id)
        bot.send_message(chat_id, "منوی اصلی:", reply_markup=main_menu())
        return

    if call.data == "vpn_continue":
        bot.edit_message_text("یکی از پلن‌ها رو انتخاب کن 👇", chat_id, call.message.message_id)
        bot.send_message(chat_id, "پلن VPN:", reply_markup=vpn_choose_plan_kb())
        return

    if call.data in ["vpn_3", "vpn_6", "vpn_10"]:
        plan_map = {"vpn_3": "۳ روزه", "vpn_6": "۶ روزه", "vpn_10": "۱۰ روزه"}
        plan = plan_map[call.data]

        user_state[chat_id] = {"mode": "vpn_order", "plan": plan}

        bot.send_message(
            chat_id,
            f"✅ پلن انتخاب شد: VPN {plan}\n\n"
            "حالا لطفاً اطلاعات زیر رو یکجا ارسال کن:\n"
            "1) نام و نام خانوادگی\n"
            "2) یوزرنیم تلگرام (اگر داری)\n"
            "3) توضیحات (مثلاً مدل گوشی/کشور/هرچی لازم می‌دونی)\n\n"
            "برای لغو: ❌ لغو",
            reply_markup=cancel_kb()
        )
        return

@bot.message_handler(content_types=['text', 'photo'])
def handle_all(message):
    chat_id = message.chat.id

    if chat_id not in user_state:
        bot.send_message(chat_id, "از منو انتخاب کن 👇", reply_markup=main_menu())
        return

    mode = user_state[chat_id].get("mode")

    # پشتیبانی
    if mode == "support":
        if message.content_type == "text":
            bot.send_message(
                ADMIN_ID,
                f"📩 پیام پشتیبانی جدید\n\n"
                f"👤 کاربر: {message.from_user.first_name}\n"
                f"🆔 ID: {chat_id}\n"
                f"📌 متن:\n{message.text}"
            )
        else:
            caption = message.caption if message.caption else ""
            bot.send_photo(
                ADMIN_ID,
                message.photo[-1].file_id,
                caption=(
                    f"📩 پیام پشتیبانی (عکس)\n\n"
                    f"👤 کاربر: {message.from_user.first_name}\n"
                    f"🆔 ID: {chat_id}\n"
                    f"📌 کپشن:\n{caption}"
                )
            )

        bot.send_message(chat_id, "پیامت ارسال شد ✅", reply_markup=main_menu())
        user_state.pop(chat_id, None)
        return

    # سفارش VPN
    if mode == "vpn_order":
        plan = user_state[chat_id].get("plan", "نامشخص")

        if message.content_type != "text":
            bot.send_message(chat_id, "لطفاً اطلاعات رو به صورت متن ارسال کن ✍️", reply_markup=cancel_kb())
            return

        bot.send_message(
            ADMIN_ID,
            f"🛒 سفارش جدید VPN\n\n"
            f"👤 کاربر: {message.from_user.first_name}\n"
            f"🆔 ID: {chat_id}\n"
            f"📦 پلن: {plan}\n\n"
            f"📝 اطلاعات:\n{message.text}"
        )

        bot.send_message(chat_id, "✅ سفارش ثبت شد. منتظر پیام ادمین باشید.", reply_markup=main_menu())
        user_state.pop(chat_id, None)
        return

print("Bot is running...")
bot.infinity_polling()
