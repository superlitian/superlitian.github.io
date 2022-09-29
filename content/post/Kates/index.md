---
title: Kates - discord 机器人

description: 基于discord + 百度智能云自动对话unit平台实现社区治理机器人。

image: kates.jpg

slug: Kates

date: 2022-09-29

categories:
- Python

tags:
- Python
- Discord
---
### Kates
基于discord + 百度智能云自动对话unit平台实现社区治理机器人。
#### mian.py

```python
#!/usr/bin/env python3
import discord
import re
import unit

intents = discord.Intents.default()
intents.members = True
client = discord.Client(proxy="http://127.0.0.1:7890", intents=intents)
# discord 机器人 token
token = '' 


@client.event
async def on_ready():
    print("Bot client logged-in as: %s" % client.user)


@client.event
async def on_member_join(member):
    str = discord.Embed(title="👏🏻👏🏻👏🏻👏🏻👏🏻",description=f"Hi, {member.mention},"
                                                       f" 欢迎你来到{member.guild.name}社区", color=0x00ff00)
    # await member.create_dm()
    # await member.dm_channel.send(embed=str)
    # 频道管道
    channel = client.get_channel()
    await channel.send(embed=str)


@client.event
async def on_message(message):
    # 排除bot自身的消息
    if message.author == client.user:
        return

    # ping/pong测试，然后删除测试消息
    if message.content == 'welcome':
        await on_member_join(message.author)
    # 返回at机器人的消息
    elif client.user.mentioned_in(message):
        trim_mentioned_msg = re.sub("<@!?(\d+)>", "", message.content).strip()

        # 调用百度智能机器人
        await message.channel.send(
            f"{message.author.mention} {unit.talk(trim_mentioned_msg)}")


# www.baidu.com

if __name__ == "__main__":
    unit.get_token()
    client.run(token)

```



#### unit.py

```python
import json
import time

import requests

Token = ""


def talk(text):
    url = 'https://aip.baidubce.com/rpc/2.0/unit/service/v3/chat?access_token=' + Token
    request = {
        'terminal_id': '123456',
        'query': text
    }

    param = {
        'version': '3.0',
        'service_id': 'S71734',
        'session_id': '',
        'log_id': str(time.time()),
        'request': request
    }

    headers = {'content-type': 'application/json'}
    response = requests.post(url, data=json.dumps(param), headers=headers)
    if response:
        print(response.json())
        return response.json()['result']['context']['SYS_PRESUMED_HIST'][1]
    else:
        return "我也不是太明白~"


def get_token():
    global Token
    # 百度智能云鉴权信息
    ak = ''
    sk = ''
    host = f'https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id={ak}&client_secret={sk}'
    response = requests.get(host)
    if response:
        Token = response.json()['access_token']
```

