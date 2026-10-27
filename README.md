---server.py

import socket, threading

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.bind(("0.0.0.0",8080))
s.listen(5)
clients=[]

def broadcast(msg,sender):
    for c in clients:
        if c!=sender:
            try: c.send(msg)
            except: clients.remove(c)

def handle_client(c):
    while True:
        try:
            msg=c.recv(1024)
            if not msg: break
            print(msg.decode())
            broadcast(msg,c)
        except: break
    if c in clients: clients.remove(c)
    c.close()

print("Chat Server Running...")
while True:
    c,a=s.accept()
    print("Connected:",a)
    clients.append(c)
    threading.Thread(target=handle_client,args=(c,)).start()

  ---client.py

import socket, threading

ip=input("Enter server IP: ")
c=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
c.connect((ip,8080))
name="blessing"

def receive():
    while True:
        try:
            msg=c.recv(1024).decode()
            if not msg: break
            print(msg)
        except: break

threading.Thread(target=receive,daemon=True).start()
while True:
    msg=input()
    if msg.lower()=="exit": break
    c.send((name+": "+msg).encode())
c.close()
