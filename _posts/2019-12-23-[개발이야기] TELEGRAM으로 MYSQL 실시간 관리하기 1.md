---
layout:     post
title:      "[개발이야기] Telegram으로 MYSQL 실시간 관리하기 1"
subtitle:   " \"와들 쇼핑 개발 과정\""
date:       2019-12-23 12:00:00
author:     "이즈"
catalog: true
tags:
    - Telegram
    - Chatbot
    - MYSQL
    - DataBase
    - DB
    - 텔레그램
    - 챗봇
    - 연동
    - 실시간
    - 관리
    - 알림
---

### 블루펭귄 개발 과정

주식회사 와들의 첫 iOS 앱인 **[블루펭귄](https://www.waddlelab.com/)**을 개발하는 과정을 담은 글이다.

#### 챗봇 개발이 왜 시작됐지?  

먼저 우리의 iOS 앱에서 왜 챗봇 개발이 시작됐는지부터 짚고 넘어가보겠다. 이를 말하려면 우리의 앱에 대한 간단한 설명이 필요하다. 와들 쇼핑은 시각장애인을 위한 쇼핑 앱이다. 그러나 우리가 직접 물건을 파는 것은 아니고, 다른 사이트에서 파는 정보를 **시각장애인이 읽기 쉽게 바꾸어 쇼핑을 돕는 앱**이다.  

나중에는 사용자가 물건을 주문할 시 원래 물건이 있던 사이트로 자동으로 주문이 들어가게 하는 것이 가장 좋겠지만, 아직까지는 우리가 중간 단계를 직접 손으로 해주고 있다. 사용자가 우리 앱을 통해 물건을 주문하면, 우리가 원래 사이트에서 다시 주문을 하는 식이다.  

따라서 사용자에게서 주문이 들어왔을 때 우리에게 실시간으로 알림이 들어와야한다. 그 기능을 간단하게 구현하기 위해 챗봇 개발을 시작했다.  

#### 그렇다면 왜 굳이 Telegram을 썼지?  

나는 챗봇에게 두가지 기능을 원했다.
1. 자연어 처리보다는, 정해진 요청에 정해진 응답을 할 수 있는 챗봇
2. 들어온 채팅에만 대답할 수 있는 챗봇이 아닌 **사용자에게 먼저 연락을 할 수 있는 챗봇**  

두번 째 조건을 만족하는 챗봇을 찾기가 굉장히 어려웠다.  

카카오톡 플러스친구, 페이스북 메시지 등 다양한 방법을 알아보았지만, 두가지 조건을 만족하는 챗봇은 텔레그램 밖에 없었다. 그래서 텔레그램을 시작했다.

#### 본격적으로 코드를 뜯어보자

##### 1. 구조 만들기

이번 편에서는 텔레그램의 작동법에 대해 설명하고,  
다음 편에서는 이를 사용해서 MYSQL과 연동하는 법 위주로 설명하겠다.  

먼저, ChatBotModel.py 파일을 만들어 구조를 만든다.  
먼저 코드에 넣을 토큰을 발급 받아야한다. 토큰은 [다음 링크](https://www.siteguarding.com/en/how-to-get-telegram-bot-api-token)를 참고하여 만들면 된다.  

add_handler은 /order 처럼 '/'+문자열 을 인식하는 핸들러다.  
query_handler은 버튼 클릭 등을 인식하는 핸들러다.   
message_handler은 채팅을 인식하는 핸들러다.  
error_handler은 에러를 인식하는 핸들러다.  

```
import telegram
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, MessageHandler, Filters

class TelegramBot:
    def __init__ (self, name, token):
        self.core = telegram.Bot(token)
        self.updater = Updater(token, use_context=True)
        self.name = name

    def sendMessage(self, id, text, reply_markup=None):
        self.core.sendMessage(chat_id = id, text=text, reply_markup=reply_markup)

    def stop(self):
        self.updater.start_polling()
        self.updater.dispatcher.stop()
        self.updater.job_queue.stop()
        self.updater.stop()

class WaddleBot(TelegramBot):
    def __init__(self):
        self.token = **본인의 토큰**
        TelegramBot.__init__(self, '와들', self.token)
        self.updater.stop()

    def add_handler(self, cmd, func):
        self.updater.dispatcher.add_handler(CommandHandler(cmd, func))

    def add_query_handler(self, func):
        self.updater.dispatcher.add_handler(CallbackQueryHandler(func))

    def add_message_handler(self, func):
        self.updater.dispatcher.add_handler(MessageHandler(Filters.text, func))

    def add_error_handler(self, func):
        self.updater.dispatcher.add_error_handler(func)

    def start(self):
        print('start')
        self.updater.start_polling()
        self.updater.idle()
```

##### 2. 챗봇 시작하기

ChatBot.py를 만들어 챗봇을 시작한다.  

/order가 들어오면 check_order을 실행한다  
버튼이 눌리면 callback_get을 실행한다  
메세지가 들어오면 text을 실행한다      
에러가 들어오면 error을 실행한다  

```
import ChatBotModel
from telegram import InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater

if __name__ == '__main__':

    waddle = ChatBotModel.WaddleBot()

    waddle.add_handler('order', check_order)
    waddle.add_query_handler(callback_get)
    waddle.add_message_handler(text)
    waddle.add_error_handler(error)
    waddle.start()
```

이 다음부터는 방금 언급된 함수 네개를 설명하겠다.

##### 3. check_order

먼저 check_order 함수를 살펴보자.  
이는 '/order'이라는 메세지가 들어왔을 때 
다음 사진처럼 메세지에 "응 내가 할게", "지금 바빠"  
이렇게 두개의 버튼을 만들어서 보내는 방법이다.
버튼이 눌렸을 때의 챗봇 처리는 뒤에 다시 설명 할 예정이다.  
<img class="shadow" width="600" src="/img/02-button.png" alt="버튼이 있는 메세지 사진"/>

```
def build_menu(buttons, n_cols, header_buttons=None, footer_buttons=None):
    menu = [buttons[i:i + n_cols] for i in range(0, len(buttons), n_cols)]
    if header_buttons:
        menu.insert(0, header_buttons)
    if footer_buttons:
        menu.append(footer_buttons)
    return menu

def check_order(bot, args):
    try:
        order_request = execute(f"""SELECT * FROM Ordered WHERE Status='payed';""", True)
        request_len = len(order_request)

        set_menu = []
        set_menu.append(InlineKeyboardButton("응 내가 할게", callback_data="yes")) 
        set_menu.append(InlineKeyboardButton("지금 바빠", callback_data="no"))
        set_menu_markup = InlineKeyboardMarkup(build_menu(set_menu, len(set_menu) - 1)) 

        waddle.sendMessage(bot.message.chat.id, "결제 요청 "+str(request_len)+"건이 들어왔습니다.\n맡아 결제 하시겠습니까?", reply_markup=set_menu_markup)
    
    except:
        print("error from check_order")
```

다른 함수는 어렵지 않겠지만, 갑자기 나타난 bot.message.chat.id는 의아할 수도 있다.
핸들러는 항상 두가지 파라미터 bot, args을 받아야한다.  
그리고 bot을 print 하면 모든 정보가 담겨 나온다.  
다음은 내가 /order을 입력했을 때 bot.message 안에 담긴 정보이다.  

```
{
    'message_id': 1564, 
    'date': 1577015232, 
    'chat': {
        'id': 123456, 
        'type': 'private', 
        'first_name': 'Izz', 
        'last_name': 'Kim'
    }, 
    'text': '/order', 
    'entities': [], 
    'caption_entities': [], 
    'photo': [], 
    'new_chat_members': [], 
    'new_chat_photo': [], 
    'delete_chat_photo': False, 
    'group_chat_created': False, 
    'supergroup_chat_created': False, 
    'channel_chat_created': False, 
    'from': {
        'id': 123456, 
        'first_name': 'Izz', 
        'is_bot': False, 
        'last_name': 'Kim', 
        'language_code': 'ko'
    }
}
```
이 때 내 id는 chat.id에 담겨있다.  
따라서 내 아이디는 bot.message.chat.id를 통해 알 수 있다.  
그래서 메세지를 보내는 id를 bot.message.chat.id에 넣은 것이다.

##### 5. callback_get

다음은 버튼이 눌렸을 때의 처리이다.  
위에서 말했던 것 처럼 bot.message을 출력해보면   
callback_data의 값이 bot.callback_query.data 안에,  
id 값이 bot.callback_query.message.chat.id 안에 담겨있는 것을 확인할 수 있다.  
이를 통해 다음과 같은 함수를 짤 수 있다.  

<img class="shadow" width="600" src="/img/02-button-callback.png" alt="버튼이 있는 메세지에 답장을 한 사진"/>

```
def callback_get(bot, update):
    if bot.callback_query.data=="yes":
        manager_order(bot.callback_query.message.chat.id)
    elif bot.callback_query.data=="no": 
        waddle.sendMessage(bot.callback_query.message.chat.id, "다음 기회에..")
```
manager_order에서는 주문 정보를 보내주었다.

```
def manager_order(id):
    try:
        comment = "주문 정보는 다음과 같습니다.\n어쩌구저쩌구.."
        waddle.sendMessage(id, comment)

    except:
        print("error from manager_order")
```

#### 6. text

나는 크게 메세지를 두가지로 나눴다.   
첫번째는 일반 메세지가 온 경우,  
두번째는 답장 메세지가 온 경우이다.  

마찬가지로 bot.message을 출력해보면 둘은 reply_to_message의 유무로 비교할 수 있고,  
id는 bot.message.chat.id, 메세지는 bot.message.text,  
답장으로 온 경우 기존 메세지는 bot.message.reply_to_message.text에  
담겨있다는 것을 알 수 있다.  

```
def text(bot, update):
    if bot.message.reply_to_message is not None:
        complete_order(bot.message.chat.id, bot.message.reply_to_message.text, bot.message.text)
    else:
        waddle.sendMessage(bot.message.chat.id, "환영합니다")
```
complete_order은 주문정보를 보고 주문을 한 다음 답장으로 완료를 보내면 일어난다.  
나는 이때 추가적인 mysql처리를 해주었지만, 여기서는 따로 나타내지 않겠다.  
함수는 다음과 같이 나타낼 수 있다.  

```
def complete_order(id, reply_text, text):
    try:
        comment = "reply_text와 text를 사용해서 보내고 싶은 메세지를 보내주세요."        
        waddle.sendMessage(id, comment)

    except:
        print("error from complete_order")
```

##### 7. error

마지막으로 에러처리이다.  
챗봇에서 에러가 나면 자동으로 다음 함수가 실행된다.  

```
def error(bot, update):
    """Log Errors caused by Updates."""
    logger.warning('Update "%s" caused error "%s"', bot, update.error)
```

##### 최종

다 합치면 다음과 같다.

**ChatBotModel.py**
```
import telegram
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, MessageHandler, Filters

class TelegramBot:
    def __init__ (self, name, token):
        self.core = telegram.Bot(token)
        self.updater = Updater(token, use_context=True)
        self.name = name

    def sendMessage(self, id, text, reply_markup=None):
        self.core.sendMessage(chat_id = id, text=text, reply_markup=reply_markup)

    def stop(self):
        self.updater.start_polling()
        self.updater.dispatcher.stop()
        self.updater.job_queue.stop()
        self.updater.stop()

class WaddleBot(TelegramBot):
    def __init__(self):
        self.token = **본인의 토큰**
        TelegramBot.__init__(self, '와들', self.token)
        self.updater.stop()

    def add_handler(self, cmd, func):
        self.updater.dispatcher.add_handler(CommandHandler(cmd, func))

    def add_query_handler(self, func):
        self.updater.dispatcher.add_handler(CallbackQueryHandler(func))

    def add_message_handler(self, func):
        self.updater.dispatcher.add_handler(MessageHandler(Filters.text, func))

    def add_error_handler(self, func):
        self.updater.dispatcher.add_error_handler(func)

    def start(self):
        print('start')
        self.updater.start_polling()
        self.updater.idle()
```

**ChatBot.py**
```
import sys
import ChatBotModel
import requests
import logging
from telegram import InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater
import json
import threading
import time

### make button
def build_menu(buttons, n_cols, header_buttons=None, footer_buttons=None):
    menu = [buttons[i:i + n_cols] for i in range(0, len(buttons), n_cols)]
    if header_buttons:
        menu.insert(0, header_buttons)
    if footer_buttons:
        menu.append(footer_buttons)
    return menu


def check_order(bot, args):
    try:
        set_menu = []
        set_menu.append(InlineKeyboardButton("응 내가 할게", callback_data="yes")) 
        set_menu.append(InlineKeyboardButton("지금 바빠", callback_data="no"))
        set_menu_markup = InlineKeyboardMarkup(build_menu(set_menu, len(set_menu) - 1)) 

        waddle.sendMessage(bot.message.chat.id, "결제 요청이 들어왔습니다.\n맡아 결제 하시겠습니까?", reply_markup=set_menu_markup)
    
    except:
        print("error from check_order")
    
def manager_order(id):
    try:
        comment = "주문 정보는 다음과 같습니다.\n어쩌구저쩌구.."
        waddle.sendMessage(id, comment)

    except:
        print("error from manager_order")

def complete_order(id, reply_text, text):
    try:
        comment = reply_text + "의 답장은 " + text + "입니다."        
        waddle.sendMessage(id, comment)

    except:
        print("error from complete_order")
    

### button clicked
def callback_get(bot, update):
    if bot.callback_query.data=="yes":
        manager_order(bot.callback_query.message.chat.id)
    elif bot.callback_query.data=="no": 
        waddle.sendMessage(bot.callback_query.message.chat.id, "다음 기회에..")

### text came
def text(bot, update):
    print(bot.message)
    if bot.message.reply_to_message is not None:
        complete_order(bot.message.chat.id, bot.message.reply_to_message.text, bot.message.text)
    else:
        waddle.sendMessage(bot.message.chat.id, "환영합니다")

### print log
def error(bot, update):
    """Log Errors caused by Updates."""
    logger.warning('Update "%s" caused error "%s"', bot, update.error)


if __name__ == '__main__':

    waddle = ChatBotModel.WaddleBot()
    waddle.add_handler('order', check_order)
    waddle.add_query_handler(callback_get)
    waddle.add_message_handler(text)
    waddle.add_error_handler(error)
    waddle.start()
```