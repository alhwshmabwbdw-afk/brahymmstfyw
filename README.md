‏import telebot
‏from telebot import types
‏
‏# --- إعداداتك الأساسية ---
‏API_TOKEN = 'YOUR_BOT_TOKEN_HERE'  # توكن البوت من BotFather
‏ADMIN_ID = 'YOUR_TELEGRAM_ID_HERE'    # الآيدي الخاص بك لاستقبال الطلبات
‏SHAM_CASH_LINK = "https://shamcash.me" # رابط دفع شام كاش
‏
‏bot = telebot.TeleBot(API_TOKEN)
‏user_data = {}
‏
‏# --- قوائم الباقات والأسعار ---
‏PRICES = {
‏    '🎮 ببجي موبايل': ['60 شدة', '325 شدة', '660 شدة', '1800 شدة'],
‏    '💎 فري فاير': ['100 جوهرة', '210 جوهرة', '530 جوهرة'],
‏    '📱 تيك توك': ['100 عملة', '500 عملة', '1000 عملة'],
‏    '✨ لايكي (Likee)': ['42 ماسة', '297 ماسة'],
‏    '🎲 يلا لودو': ['رصيد ذهبي', 'ماس لودو'],
‏    '🎬 شاهد VIP': ['اشتراك شهر', 'اشتراك سنة']
‏}
‏
‏# --- القائمة الرئيسية ---
‏def main_menu():
‏    markup = types.ReplyKeyboardMarkup(row_width=2, resize_keyboard=True)
‏    btns = [types.KeyboardButton(s) for s in PRICES.keys()]
‏    btns.append(types.KeyboardButton('📞 الدعم الفني'))
‏    markup.add(*btns)
‏    return markup
‏
‏@bot.message_handler(commands=['start'])
‏def start(message):
‏    bot.send_message(message.chat.id, "🌹 أهلاً بك في متجر الشحن السريع\nالرجاء اختيار التطبيق المراد شحنه من القائمة:", reply_markup=main_menu())
‏
‏# --- معالجة اختيار التطبيق ---
‏@bot.message_handler(func=lambda message: message.text in PRICES.keys())
‏def select_package(message):
‏    service = message.text
‏    user_data[message.chat.id] = {'service': service}
‏    
‏    markup = types.ReplyKeyboardMarkup(row_width=2, resize_keyboard=True)
‏    package_btns = [types.KeyboardButton(p) for p in PRICES[service]]
‏    package_btns.append(types.KeyboardButton('🔙 العودة للقائمة الرئيسية'))
‏    markup.add(*package_btns)
‏    
‏    bot.send_message(message.chat.id, f"اختر الكمية المطلوبة لـ {service}:", reply_markup=markup)
‏
‏# --- معالجة اختيار الكمية ---
‏@bot.message_handler(func=lambda message: any(message.text in p for p in PRICES.values()))
‏def get_id(message):
‏    if message.chat.id in user_data:
‏        user_data[message.chat.id]['package'] = message.text
‏        msg = bot.send_message(message.chat.id, "الآن أرسل رقم الـ ID الخاص بالحساب المراد شحنه:", reply_markup=types.ReplyKeyboardRemove())
‏        bot.register_next_step_handler(msg, request_payment)
‏
‏# --- طلب الدفع ---
‏def request_payment(message):
‏    user_data[message.chat.id]['game_id'] = message.text
‏    
‏    markup = types.InlineKeyboardMarkup()
‏    pay_btn = types.InlineKeyboardButton("💳 اضغط هنا للدفع عبر شام كاش", url=SHAM_CASH_LINK)
‏    markup.add(pay_btn)
‏    
‏    info = user_data[message.chat.id]
‏    text = (f"📝 **تفاصيل طلبك:**\n\n"
‏            f"🔹 التطبيق: {info['service']}\n"
‏            f"🔹 الكمية: {info['package']}\n"
‏            f"🔹 الآيدي: {info['game_id']}\n\n"
‏            f"⚠️ يرجى الدفع عبر الرابط أدناه، ثم **أرسل صورة التحويل** هنا لتأكيد الطلب.")
‏    
‏    bot.send_message(message.chat.id, text, reply_markup=markup, parse_mode="Markdown")
‏
‏# --- استلام صورة التحويل وإرسال الطلب للمدير ---
‏@bot.message_handler(content_types=['photo'])
‏def handle_photo(message):
‏    chat_id = message.chat.id
‏    if chat_id in user_data and 'game_id' in user_data[chat_id]:
‏        info = user_data[chat_id]
‏        username = f"@{message.from_user.username}" if message.from_user.username else "مخفي"
‏        
‏        caption = (f"🚨 **طلب جديد وصل!**\n\n"
‏                   f"👤 العميل: {username}\n"
‏                   f"🎮 التطبيق: {info['service']}\n"
‏                   f"📦 الكمية: {info['package']}\n"
‏                   f"🆔 الآيدي: {info['game_id']}")
‏        
‏        # إرسال الصورة والبيانات لك
‏        bot.send_photo(ADMIN_ID, message.photo[-1].file_id, caption=caption, parse_mode="Markdown")
‏        
‏        bot.send_message(chat_id, "✅ تم استلام طلبك وصورة التحويل.\nسيتم مراجعة الطلب والشحن لك في أسرع وقت ممكن! 🚀", reply_markup=main_menu())
‏        del user_data[chat_id]
‏
‏@bot.message_handler(func=lambda message: message.text == '🔙 العودة للقائمة الرئيسية')
‏def go_back(message):
‏    start(message)
‏
‏bot.polling(none_stop=True)
‏
