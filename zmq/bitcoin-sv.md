# !/usr/bin/env python3
# Copyright (c) 2014-2017 The Bitcoin Core developers
# Distributed under the MIT software license, see the accompanying
# file COPYING or http://www.opensource.org/licenses/mit-license.php.

"""
    ZMQ example using python3's asyncio

    Bitcoin should be started with the command line arguments:
        bitcoind -testnet -daemon \
                -zmqpubrawtx=tcp://127.0.0.1:28332 \
                -zmqpubrawblock=tcp://127.0.0.1:28332 \
                -zmqpubhashtx=tcp://127.0.0.1:28332 \
                -zmqpubhashblock=tcp://127.0.0.1:28332

    We use the asyncio library here.  `self.handle()` installs itself as a
    future at the end of the function.  Since it never returns with the event
    loop having an empty stack of futures, this creates an infinite loop.  An
    alternative is to wrap the contents of `handle` inside `while True`.

    A blocking example using python 2.7 can be obtained from the git history:
    https://github.com/bitcoin/bitcoin/blob/37a7fe9e440b83e2364d5498931253937abe9294/contrib/zmq/zmq_sub.py
"""

import binascii
import asyncio
import json
from datetime import datetime, timedelta
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
import zmq
import zmq.asyncio
import signal
import struct
import sys
import requests
import smtplib

if (sys.version_info.major, sys.version_info.minor) < (3, 5):
    print("This example only works with Python 3.5 and greater")
    sys.exit(1)

REFRESH_INTERVAL = 1
WEBHOOK_AUTHEN_TOKEN = "asdasdasdasdasdsa"
CURRENT_REFRESH_TIME = None
WEBHOOK_DETAIL_API = "http://devapi.coinhe.io/v1/webhook-apis/bitcoin/"
WEBHOOK_API = None
NOTIFY_EMAILS = "oliver@coinhe.io, paul@coinhe.io, michael@coinhe.io"


EMAIL_HOST="email-smtp.us-east-1.amazonaws.com"
EMAIL_PORT=587
EMAIL_HOST_USER="AKIAIBH3ZDIFUL4JJSGA"
EMAIL_HOST_PASSWORD="Aq7blhk8WnSQNjWqibgNSnmCxc6/vY15eno9c5QF+fI/"

HAS_BLOCK = "has_block"
HAS_TX = "has_tx"
RAW_BLOCK_HEADER = "raw_block_header"
RAW_TX = "raw_tx"

SYMBOL = "BSV"

def get_headers():
    return {
        "Authorization": WEBHOOK_AUTHEN_TOKEN,
        "Content-Type": "application/json"
    }

def notify_erorr(error):
    try:
        server = smtplib.SMTP(EMAIL_HOST, EMAIL_PORT)
        server.ehlo()
        server.starttls()
        server.ehlo()
        server.login(EMAIL_HOST_USER, EMAIL_HOST_PASSWORD)
        msg = MIMEMultipart()
        msg['From'] = "noreply@coinhe.io"
        msg['To'] = NOTIFY_EMAILS
        msg['Subject'] = "BSV error stream"
        body = "BSV webhook error : %s" % error
        msg.attach(MIMEText(body, 'plain'))
        text = msg.as_string()
        #Send the mail
        server.sendmail("noreply@coinhe.io", NOTIFY_EMAILS, text)
        print("has error: %s" % error)
    except Exception as e:
        print("Send error %s" % str(e))


def is_need_refresh():
    global CURRENT_REFRESH_TIME
    if not CURRENT_REFRESH_TIME:
        return True
    if CURRENT_REFRESH_TIME + timedelta(hours=REFRESH_INTERVAL) > datetime.now():
        return True
    return False

def get_webhook_api():
    if is_need_refresh():
        global WEBHOOK_API
        global CURRENT_REFRESH_TIME
        print("refesh webhook api")
        try:
            response = requests.get(url=WEBHOOK_DETAIL_API, headers=get_headers())
            print(response.json())
            WEBHOOK_API = response.json()["bch_webhook"]
            CURRENT_REFRESH_TIME = datetime.now()
        except Exception as e:
            print(str(e))
            notify_erorr(str(e))
            return None
    return WEBHOOK_API

def send_webhook(event, data):
    data = {
        event: data.decode("utf-8"),
        "symbol": SYMBOL
    }
    data = json.dumps(data)
    try:
        print("send event %s" % event)
        url = get_webhook_api()
        if not url:
            notify_erorr("Cannot get webhook api")
            return
        response = requests.post(url=url, headers=get_headers(), data=data)
        print(response.json())
        if response.json()["status"] != "success":
            notify_erorr(str(response.json()))
        return
    except Exception as e:
        notify_erorr(str(e))

port = 28332

class ZMQHandler():
    def __init__(self):
        self.loop = asyncio.get_event_loop()
        self.zmqContext = zmq.asyncio.Context()

        self.zmqSubSocket = self.zmqContext.socket(zmq.SUB)
        self.zmqSubSocket.setsockopt_string(zmq.SUBSCRIBE, "hashblock")
        self.zmqSubSocket.setsockopt_string(zmq.SUBSCRIBE, "hashtx")
        self.zmqSubSocket.setsockopt_string(zmq.SUBSCRIBE, "rawblock")
        self.zmqSubSocket.setsockopt_string(zmq.SUBSCRIBE, "rawtx")
        self.zmqSubSocket.connect("tcp://127.0.0.1:%i" % port)

    async def handle(self) :
        msg = await self.zmqSubSocket.recv_multipart()
        topic = msg[0]
        body = msg[1]
        sequence = "Unknown"
        if len(msg[-1]) == 4:
          msgSequence = struct.unpack('<I', msg[-1])[-1]
          sequence = str(msgSequence)
        if topic == b"hashblock":
            print('- HASH BLOCK ('+sequence+') -')
            print(binascii.hexlify(body))
            send_webhook(HAS_BLOCK, binascii.hexlify(body))
        #elif topic == b"hashtx":
        #    print('- HASH TX  ('+sequence+') -')
        #    print(binascii.hexlify(body))
        #    send_webhook(HAS_TX, binascii.hexlify(body))
        #elif topic == b"rawblock":
        #    print('- RAW BLOCK HEADER ('+sequence+') -')
        #    print(binascii.hexlify(body[:80]))
        #    send_webhook(RAW_BLOCK_HEADER, binascii.hexlify(body))
        #elif topic == b"rawtx":
        #    print('- RAW TX ('+sequence+') -')
        #    print(binascii.hexlify(body))
        #    send_webhook(RAW_TX, binascii.hexlify(body))
        # schedule ourselves to receive the next message
        asyncio.ensure_future(self.handle())

    def start(self):
        self.loop.add_signal_handler(signal.SIGINT, self.stop)
        self.loop.create_task(self.handle())
        self.loop.run_forever()

    def stop(self):
        self.loop.stop()
        self.zmqContext.destroy()

daemon = ZMQHandler()
daemon.start()
