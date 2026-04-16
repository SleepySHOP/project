import asyncio
from aiogram import Bot, Dispatcher
from aiogram.types import Message, CallbackQuery, InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.filters import Command

API_TOKEN = "8688289767:AAHcVO_oLH5Tm_DXkN16XTGD_iUe-jtRjPQ"

bot = Bot(token=API_TOKEN)
dp = Dispatcher()

# --- Пользователи ---
user_balances = {}
current_category = {}
current_index = {}

# --- Товары ---
accounts = {
    "PUBG Pro Account": {"price": 1200, "description": "High level PUBG account with rare outfits and weapons.", "rarity": "⭐⭐⭐"},
    "PUBG Stealth Account": {"price": 1500, "description": "Stealth themed PUBG account with camo gear.", "rarity": "⭐⭐"}
}

skins = {
    "Desperado Crate Skin": {"price": 300, "description": "Exclusive crate skin from PUBG crates.", "rarity": "⭐"},
    "Epic Weapon Skin": {"price": 450, "description": "Epic rarity weapon skin.", "rarity": "⭐⭐"}
}

gems = {
    25: {"price": 100, "description": "Pack of 25 gems.", "rarity": "⭐"},
    75: {"price": 250, "description": "Pack of 75 gems.", "rarity": "⭐⭐"},
    150: {"price": 450, "description": "Pack of 150 gems.", "rarity": "⭐⭐⭐"}
}

# --- Inline клавиатуры ---
main_kb = InlineKeyboardMarkup(
    inline_keyboard=[
        [
            InlineKeyboardButton(text="🛒 Menu", callback_data="menu"),
            InlineKeyboardButton(text="❓ Help", callback_data="help")
        ],
        [
            InlineKeyboardButton(text="ℹ️ Info", callback_data="info")
        ]
    ]
)

menu_kb = InlineKeyboardMarkup(
    inline_keyboard=[
        [InlineKeyboardButton(text="🎮 Game", callback_data="game")],
        [InlineKeyboardButton(text="💰 Balance", callback_data="balance")],
        [InlineKeyboardButton(text="⬅️ Back", callback_data="back_main")]
    ]
)

game_kb = InlineKeyboardMarkup(
    inline_keyboard=[
        [
            InlineKeyboardButton(text="🛍️ Sell", callback_data="sell"),
            InlineKeyboardButton(text="💰 Buy", callback_data="buy")
        ],
        [InlineKeyboardButton(text="⬅️ Back to Menu", callback_data="back_menu")]
    ]
)

category_kb = InlineKeyboardMarkup(
    inline_keyboard=[
        [InlineKeyboardButton(text="👤 Account", callback_data="account")],
        [InlineKeyboardButton(text="🎨 Skins", callback_data="skins")],
        [InlineKeyboardButton(text="💎 Gems", callback_data="gems")],
        [InlineKeyboardButton(text="⬅️ Back to Game", callback_data="back_game")]
    ]
)

# --- /start ---
@dp.message(Command("start"))
async def start_cmd(message: Message):
    user_id = message.from_user.id
    user_balances.setdefault(user_id, 1000)
    await message.answer("Добро пожаловать в PUBG магазин! 👋\nВыберите действие:", reply_markup=main_kb)

# --- Callback Handler ---
@dp.callback_query()
async def callback_handler(callback: CallbackQuery):
    user_id = callback.from_user.id
    data = callback.data
    user_balances.setdefault(user_id, 1000)
    current_index.setdefault(user_id, 0)

    if data == "menu":
        await callback.message.edit_text("Выберите раздел:", reply_markup=menu_kb)
    elif data == "game":
        await callback.message.edit_text("Раздел Game:", reply_markup=game_kb)
    elif data == "balance":
        await callback.message.answer(f"Ваш текущий баланс: 💰 {user_balances[user_id]} монет")
    elif data == "info":
        await callback.message.answer("Это тестовый PUBG магазин.\nЗдесь можно покупать аккаунты, скины и гемы.")
    elif data == "help":
        await callback.message.answer("Опишите вашу проблему. Мы свяжемся с вами в ближайшее время.")
    elif data in ["sell", "buy"]:
        await callback.message.edit_text("Выберите категорию:", reply_markup=category_kb)
    elif data in ["account", "skins", "gems"]:
        current_category[user_id] = data
        current_index[user_id] = 0
        await show_item(user_id, callback)
    elif data == "prev":
        current_index[user_id] = max(0, current_index[user_id]-1)
        await show_item(user_id, callback)
    elif data == "next":
        current_index[user_id] += 1
        await show_item(user_id, callback)
    elif data.startswith("buy:"):
        name = data.split(":", 1)[1]
        category = current_category[user_id]
        items = accounts if category=="account" else skins if category=="skins" else gems
        item = items[name] if category in ["account","skins"] else items[int(name)]
        price = item["price"]
        if user_balances[user_id] >= price:
            user_balances[user_id] -= price
            await callback.message.answer(f"Вы купили {name} за {price} монет! ✅\nВаш баланс: {user_balances[user_id]}")
        else:
            await callback.message.answer(f"Недостаточно монет для покупки {name}. 💰")

# --- Показ товара ---
async def show_item(user_id, callback):
    category = current_category[user_id]
    index = current_index[user_id]
    items = list(accounts.items()) if category=="account" else list(skins.items()) if category=="skins" else list(gems.items())
    if index >= len(items):
        current_index[user_id] = len(items)-1
        index = current_index[user_id]
    name, info = items[index]
    kb = InlineKeyboardMarkup(
        inline_keyboard=[
            [
                InlineKeyboardButton(text="⬅️", callback_data="prev"),
                InlineKeyboardButton(text="Купить", callback_data=f"buy:{name}"),
                InlineKeyboardButton(text="➡️", callback_data="next")
            ],
            [InlineKeyboardButton(text="⬅️ Back to Game", callback_data="back_game")]
        ]
    )
    await callback.message.answer(f"{name} {info.get('rarity','')}\n{info['description']}\nЦена: {info['price']} монет", reply_markup=kb)

# --- Запуск ---
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    import logging
    logging.basicConfig(level=logging.INFO)
    asyncio.run(main())
