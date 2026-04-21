# 🔐 Cryptography & Network Security Lab

This repository contains all experiments (EXP 3 → EXP 10) with only commands and program code.

---

## 📌 EXPERIMENT 3 – Image Steganography

### ▶️ Installation
```bash
pip install numpy pillow
▶️ Run
python steganography.py
💻 Program
import numpy as np
from PIL import Image

def Encode(src, msg, dest, pwd):
    img = Image.open(src)
    arr = np.array(img)

    msg = msg + pwd
    bits = ''.join(format(ord(i), '08b') for i in msg)

    if len(bits) > arr.size:
        print("ERROR: Need larger file size")
        return

    i = 0
    for x in np.ndindex(arr.shape):
        if i < len(bits):
            arr[x] = (arr[x] & 254) | int(bits[i])
            i += 1
        else:
            break

    Image.fromarray(arr).save(dest)
    print("Image Encoded Successfully")

def Decode(src, pwd):
    img = Image.open(src)
    arr = np.array(img)

    bits = ''.join(str(arr[x] & 1) for x in np.ndindex(arr.shape))
    chars = [bits[i:i + 8] for i in range(0, len(bits), 8)]

    msg = ""
    for b in chars:
        msg += chr(int(b, 2))
        if msg.endswith(pwd):
            break

    if msg.endswith(pwd):
        print("Hidden Message:", msg[:-len(pwd)])
    else:
        print("Wrong password or no hidden message found")

def Stego():
    print("--Image Steganography--")
    print("1: Encode")
    print("2: Decode")

    ch = input("Enter choice: ")

    if ch == '1':
        Encode(input(), input(), input(), input())
    elif ch == '2':
        Decode(input(), input())
    else:
        print("ERROR")

Stego()



📌 EXPERIMENT 4 – AES (CBC & ECB)
openssl version
touch plain.txt
gedit plain.txt
cat plain.txt

openssl enc -aes-128-cbc -e -in plain.txt -out cipher1.bin -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in cipher1.bin -out output1.txt -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
xxd cipher1.bin
cat output1.txt

openssl enc -aes-128-ecb -e -in plain.txt -out cipher2.bin -K 0123456789abcdeffedcba9876543210
openssl enc -aes-128-ecb -d -in cipher2.bin -out output2.txt -K 0123456789abcdeffedcba9876543210
xxd cipher2.bin
cat output2.txt



📌 EXPERIMENT 5 – ECB vs CBC (Image)
openssl enc -aes-128-ecb -e -in pic_original.bmp -out pic_ecb.bmp -K 0123456789abcdeffedcba9876543210
openssl enc -aes-128-cbc -e -in pic_original.bmp -out pic_cbc.bmp -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff

head -c 54 pic_original.bmp > header

tail -c +55 pic_ecb.bmp > body_ecb
cat header body_ecb > new_ecb.bmp

tail -c +55 pic_cbc.bmp > body_cbc
cat header body_cbc > new_cbc.bmp

eog new_ecb.bmp
eog new_cbc.bmp



📌 EXPERIMENT 6 – Padding
echo -n 12345 > f1.txt
echo -n 1234567890 > f2.txt
echo -n 1234567890abcdef > f3.txt

ls -l f*.txt

openssl enc -aes-128-cbc -e -in f1.txt -out f1.bin -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in f1.bin -out f1_dec.txt -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in f1.bin -out f1_nopad.txt -nopad -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
xxd f1_dec.txt
xxd f1_nopad.txt

openssl enc -aes-128-cbc -e -in f2.txt -out f2.bin -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in f2.bin -out f2_dec.txt -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in f2.bin -out f2_nopad.txt -nopad -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
xxd f2_dec.txt
xxd f2_nopad.txt

openssl enc -aes-128-cbc -e -in f3.txt -out f3.bin -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in f3.bin -out f3_dec.txt -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
openssl enc -aes-128-cbc -d -in f3.bin -out f3_nopad.txt -nopad -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
xxd f3_dec.txt
xxd f3_nopad.txt



📌 EXPERIMENT 7 – Error Propagation & IV
touch plain.txt
gedit plain.txt

openssl enc -aes-128-ecb -e -in plain.txt -out cipher_ecb.bin -K 0123456789abcdeffedcba9876543210
ghex cipher_ecb.bin
openssl enc -aes-128-ecb -d -in cipher_ecb.bin -out op_ecb.txt -K 0123456789abcdeffedcba9876543210

openssl enc -aes-128-cbc -e -in plain.txt -out cipher_cbc.bin -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff
ghex cipher_cbc.bin
openssl enc -aes-128-cbc -d -in cipher_cbc.bin -out op_cbc.txt -K 0123456789abcdeffedcba9876543210 -iv 00112233445566778899aabbccddeeff

cat op_ecb.txt
cat op_cbc.txt

echo "THIS IS SAMPLE DATA FOR AES IV TEST" > abc.txt

openssl enc -aes-128-cbc -e -in abc.txt -out cipher_iv1.bin -K 00112233445566778899AABBCCDDEEFF -iv 0102030405060708090A0B0C0D0E0F10
xxd cipher_iv1.bin

openssl enc -aes-128-cbc -e -in abc.txt -out cipher_iv2.bin -K 00112233445566778899AABBCCDDEEFF -iv 1112131415161718191A1B1C1D1E1F20
xxd cipher_iv2.bin



📌 EXPERIMENT 8 – RSA
openssl genrsa -out key.pri 2048
openssl rsa -in key.pri -out key.pub -pubout
openssl rsa -in key.pri -noout -text
cat key.pub

echo "HELLO" > msg.txt
openssl rsautl -encrypt -inkey key.pub -pubin -in msg.txt -out msg.enc
openssl rsautl -decrypt -inkey key.pri -in msg.enc -out msg.dec
cat msg.dec

openssl rand -hex 32 > secret.key
openssl rsautl -encrypt -inkey key.pub -pubin -in secret.key -out secret.key.enc
openssl rsautl -decrypt -inkey key.pri -in secret.key.enc -out secret.key.dec
cat secret.key
cat secret.key.dec



📌 EXPERIMENT 10 – TCPDump
tcpdump -D

sudo tcpdump -i any
Ctrl + C

sudo tcpdump -i any -w capture.pcap
Ctrl + C

# open new terminal
ping google.com
Ctrl + C

# back to first terminal
sudo tcpdump -i any tcp
Ctrl + C

sudo tcpdump -i any icmp
Ctrl + C

sudo tcpdump -i any port 80
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & tcp-syn != 0'
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & tcp-syn != 0' -w tcpsyn.pcap
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & tcp-ack != 0'
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & tcp-fin != 0' -w tcpfin.pcap
Ctrl + C

sudo tcpdump -r capture.pcap

sudo tcpdump -A -r capture.pcap

sudo tcpdump -xx -r capture.pcap

sudo tcpdump -xx -r tcpsyn.pcap

sudo tcpdump -xx -r tcpfin.pcap
 

sudo tcpdump -i any port 80
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & tcp-syn != 0'
Ctrl + C

sudo tcpdump -n -i any 'tcp[tcpflags] & tcp-syn != 0' -w tcpsyn.pcap
Ctrl + C
