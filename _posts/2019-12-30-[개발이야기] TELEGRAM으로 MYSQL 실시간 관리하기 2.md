---
layout:     post
title:      "[개발이야기] Telegram으로 MYSQL 실시간 관리하기 2"
subtitle:   " \"와들 쇼핑 개발 과정\""
date:       2019-12-30 12:00:00
author:     "이즈"
catalog: true
tags:
    - Telegram
    - Chatbot
    - MYSQL
    - MariaDB
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

#### 챗봇 개발 2편

본 글은 텔레그램의 기본 작동법을 설명한 1편과 이어진 글이다.  
이를 읽지 않았다면, 이해에 어려움이 있을 수 있으니 1편을 읽고 오길 바란다.  
링크는 [여기](https://waddlecorp.github.io/2019/12/23/%EA%B0%9C%EB%B0%9C%EC%9D%B4%EC%95%BC%EA%B8%B0-TELEGRAM%EC%9C%BC%EB%A1%9C-MYSQL-%EC%8B%A4%EC%8B%9C%EA%B0%84-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0-1/)에 걸어두었다. 

본 글에서는 텔레그램의 기본 동작보다는  
MYSQL과의 실시간 연동 위주의 글을 쓸 예정이다.

#### MYSQL과의 실시간 연동

##### 1. MYSQL과의 연동

실시간 연동에 앞서, 먼저 연동하는 방법을 짚고 넘어가겠다.  
main함수에 mysql과의 연결을 추가한다.

```
    db = pymysql.connect(host=**본인의 host**,
                        port=**본인의 port**,
                        user=**본인의 user**,
                        passwd=**본인의 passwd**,
                        db=**본인의 db**,
                        charset='utf8')
```
그리고 db를 실행시키기 위해 execute라는 함수를 만든다.

```
def execute(sql, flag=False):
    with db.cursor() as cursor:
        cursor.execute(sql)
        if flag:
            result = cursor.fetchall()
            db.commit()
            return result
        db.commit()
        return None
```
그렇다면 이 이후로는 다음과 같이 함수를 작성할 수 있다.  
return 값이 있는 경우 **sql = execute(f"""SELECT * FROM User;""", True)**  
return 값이 없는 경우 **execute(f"""UPDATE User SET Name='와들';""")**  
MYSQL의 기본 문법을 설명하지는 않겠다.     

사용자가 결제를 완료해서 우리한테 알림이 와야하는 상태를 'payed'라고 하겠다.    
그러면 우리는 원래의 check_order 함수를 다음과 같이 수정할 수 있다.  

원래의 check_order 함수

```
def check_order(bot, args):
    try:
        set_menu = []
        set_menu.append(InlineKeyboardButton("응 내가 할게", callback_data="yes")) 
        set_menu.append(InlineKeyboardButton("지금 바빠", callback_data="no"))
        set_menu_markup = InlineKeyboardMarkup(build_menu(set_menu, len(set_menu) - 1)) 

        waddle.sendMessage(bot.message.chat.id, "결제 요청이 들어왔습니다.\n맡아 결제 하시겠습니까?", reply_markup=set_menu_markup)
    
    except:
        print("error from check_order")
```
바꿀 check_order 함수

```
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

##### 2. 실시간 연동

이제 MYSQL과의 기본적인 연동 방법에 대해서는 알았으니 실시간 연동에 대해 알아보자.   
딱히 어렵지는 않다. check_order 함수를 1초에 한번씩 계속 실행시켜주면 된다.  
Thread를 하나 만들어 1초마다 반복시켜보자.   
단 check_order 함수는 원래 handler이므로 다음과 같이 변경했다.  
watch 안에 있는 check_order2(123456)은 id가 123456인 사용자에게 알림을 준다는 가정 하에 임의로 적은 것이다.

```
def check_order(bot, args):
    check_order2(bot.message.chat.id)

def check_order2(id):
    try:
        order_request = execute(f"""SELECT * FROM Ordered WHERE Status='payed';""", True)
        request_len = len(order_request)

        set_menu = []
        set_menu.append(InlineKeyboardButton("응 내가 할게", callback_data="yes")) 
        set_menu.append(InlineKeyboardButton("지금 바빠", callback_data="no"))
        set_menu_markup = InlineKeyboardMarkup(build_menu(set_menu, len(set_menu) - 1)) 

        waddle.sendMessage(id, "결제 요청 "+str(request_len)+"건이 들어왔습니다.\n맡아 결제 하시겠습니까?", reply_markup=set_menu_markup)
    
    except:
        print("error from check_order")

def watch():
    while True: 

        order_request = execute(f"""SELECT * FROM Ordered WHERE Status='payed';""", True)
        if len(order_request) != 0:
            check_order2(123456)
            
        time.sleep(1)

if __name__ == '__main__':

    # 다른 쓰레드에서 watch를 실행한다
    t = threading.Thread(target=watch)
    t.start()

    waddle = ChatBotModel.WaddleBot()

    waddle.add_handler('order', check_order)
    waddle.add_query_handler(callback_get)
    waddle.add_message_handler(text)
    waddle.add_error_handler(error)
    waddle.start()
```

##### 3. 정보 업데이트

위에서는 MYSQL에서 정보를 얻어오는 것만 얘기했는데,  
답장 기능을 이용하면 MYSQL을 업데이트 할 수도 있다.  
나는 이 때 답장 기능을 쏠쏠히 활용했다.  
하는 방법은 대부분 설명했으니 이 부분은 건너 뛰도록 하겠다.  

##### 최종

다 합치면 다음과 같다.
내가 처음 챗봇 개발을 시작할 때 너무 헤매서,  
조금 도움이 됐으면 하는 생각에 글을 써보았는데  
많은 사람에게 도움이 됐으면 좋겠다.

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
import pymysql
from telegram import InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater
import json
import threading
import time

### execute SQL Query
def execute(sql, flag=False):
    with db.cursor() as cursor:
        cursor.execute(sql)
        if flag:
            result = cursor.fetchall()
            db.commit()
            return result
        db.commit()
        return None


### make button
def build_menu(buttons, n_cols, header_buttons=None, footer_buttons=None):
    menu = [buttons[i:i + n_cols] for i in range(0, len(buttons), n_cols)]
    if header_buttons:
        menu.insert(0, header_buttons)
    if footer_buttons:
        menu.append(footer_buttons)
    return menu


def check_order2(id):
    try:
        order_request = execute(f"""SELECT * FROM Ordered WHERE Status='payed';""", True)
        request_len = len(order_request)

        set_menu = []
        set_menu.append(InlineKeyboardButton("응 내가 할게", callback_data="yes")) 
        set_menu.append(InlineKeyboardButton("지금 바빠", callback_data="no"))
        set_menu_markup = InlineKeyboardMarkup(build_menu(set_menu, len(set_menu) - 1)) 

        waddle.sendMessage(id, "결제 요청 "+str(request_len)+"건이 들어왔습니다.\n맡아 결제 하시겠습니까?", reply_markup=set_menu_markup)
    
    except:
        print("error from check_order")

def check_order(bot, args):
    check_order2(bot.message.chat.id)
    
def manager_order(id):
    try:
        order_request = execute(f"""SELECT * FROM Ordered WHERE Status='payed';""", True)
        execute(f"""UPDATE Ordered SET Status='order request' WHERE Status='payed';""")

        comment = "order_request를 사용해서 보내고 싶은 메세지를 보내주세요."
        waddle.sendMessage(id, comment)

    except:
        print("error from manager_order")

def complete_order(id, reply_text, text):
    try:
        comment = "reply_text와 text를 사용해서 보내고 싶은 메세지를 보내주세요."        
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
    if bot.message.reply_to_message is not None and len(bot.message.text) == 14:
        complete_order(bot.message.chat.id, bot.message.reply_to_message.text, bot.message.text)
    elif bot.message.reply_to_message is not None:
        waddle.sendMessage(bot.message.chat.id, "양식을 잘 못 입력했습니다")


### print log
def error(bot, update):
    """Log Errors caused by Updates."""
    logger.warning('Update "%s" caused error "%s"', bot, update.error)


def watch():
    while True: 

        order_request = execute(f"""SELECT * FROM Ordered WHERE Status='payed';""", True)
        if len(order_request) != 0:
            check_order2(123456)

        time.sleep(1)


if __name__ == '__main__':
    
    db = pymysql.connect(host=**본인의 host**, # ex. chatbot-telegram.abcdefg.ap-northeast-2.rds.amazonaws.com
                        port=**본인의 port**, # ex. 3306
                        user=**본인의 user**, # ex. admin
                        passwd=**본인의 passwd**, # ex. Telegram!
                        db=**본인의 db**, # ex. User
                        charset='utf8')

    t = threading.Thread(target=watch)
    t.start()

    waddle = ChatBotModel.WaddleBot()
    waddle.add_handler('order', check_order)
    waddle.add_query_handler(callback_get)
    waddle.add_message_handler(text)
    waddle.add_error_handler(error)
    waddle.start()
```