[TOC]

# 1. OS Install 

> 二つの方法がある、一つ目は下記のURLで`imager_x.x.x.exe`をダウンロードして、実行する。
>
> もう一つはWin32DiskImagerなどのツールを使って、OSをSDカードに書き込む。
>
> 今回は一つ目の方法を使っているため、二つ目の方法を紹介しない。

## 1.1 [Download for Windows](https://www.raspberrypi.org/software/)

![image-20210617132619391](../../AppData/Roaming/Typora/typora-user-images/image-20210617132619391.png)

## 1.2 SDカードに書き込む

![image-20210326211945147](https://raw.githubusercontent.com/EponaWang/MyImage/main/img/20210326211954.png)

## 1.3 SDカードのbootフォルダに追加

> 目的：① SSHを使う ② 起動時自動にwifiと接続する
>
> フォルダ `002_file` の中にファイルがある。適当に修正して、使ってください。

- 新規作成　`ssh`　空

- 新規作成　`wpa_supplicant.conf`

  内容：WIFIの設定 （priority  大きい数字の優先順位も高いです。)

  注意点：WIFIはスマートフォンから出した時、key_mgmtの設定を追加してください。
  
  ```
  country=GB
  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1
  network={
  	ssid="phone_wifi1_name"
  	psk="wifi1_password"
  	key_mgmt=WPA-PSK
  	priority=2 
  }
  network={
  	ssid="wifi2_name"
  	psk="wifi2_password"
  	priority=1
  }
  ```
  

## 1.4 IP Address を取得する

SDカードをRasp Pi に挿して、IP Address　を取得する。例えば：192.168.43.166

取得方法（例）：

- モニターと接続して、右上で直接見る。

- IP スキャンツールを使う

> ツール：`001_tool\Advanced IP Scanner\advanced_ip_scanner.exe`

# 2. リモートアクセス

## 2.1 SSHで接続する

> ツール：`001_tool\ssh putty.exe`

login as : `pi`

password：`raspberry`  

`sudo raspi-config` を入力して、VNCの設定をONにする

VNC設定：`3 Interface Options`  => `P3 VNC` 

設定終わったら、VNCを使って、画面で設定できる。

今SSHを使って、インタフェース、ロカールなどを設定してもいいし、

後でVNCを使って、画面で設定しても大丈夫です。

![image-20210617002226364](https://raw.githubusercontent.com/EponaWang/MyImage/main/img/20210617002226.png)

![image-20210617002402409](https://raw.githubusercontent.com/EponaWang/MyImage/main/img/20210617002402.png)



## 2.2 VNCで接続する（画面あり）

> ツール：`001_tool\VNC-Viewer-6.17.731-Windows.exe`

welcome画面へ（国、言語などの設定）

ソフトの更新、時間がすごくかかります。

![](https://raw.githubusercontent.com/EponaWang/MyImage/main/img/20210617005510.png)

## 2.3 ソフトを更新する（コマンドで）

画面で更新した場合、コマンドを実行する必要がないです。

`sudo apt-get update`

`sudo apt-get upgrade`

## 2.4 TeamViewer をインストールする

> 目的：ライスパイをリモートアクセスする。

```
wget http://download.teamviewer.com/download/linux/version_11x/teamviewer-host_armhf.deb

sudo dpkg -i teamviewer-host_armhf.deb

sudo apt-get -f install

sudo apt-get install gdebi

sudo gdebi teamviewer-host_armhf.deb
```

インストール完了、登録して、IDを記録する。（後は、IDをPC側のteamviewerに入力して、ライスパイを制御できる）

![image-20210617140546980](../../AppData/Roaming/Typora/typora-user-images/image-20210617140546980.png)

今まで、ライスパイの設定は完了した。

# 3. GPIOの制御

```python
# プログラムを実行すると3回、1秒間隔でLチカします。
import RPi.GPIO as GPIO
import time

COUNT = 3
PIN = 17
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIN,GPIO.OUT)


for _ in range(COUNT):
    GPIO.output(PIN,GPIO.HIGH)
    time.sleep(1.0)
    GPIO.output(PIN,GPIO.LOW)
    time.sleep(1.0)


GPIO.cleanup()
```

# 4. WIFIの接続

```python
# search_wifi.py
# スキャン　結果表示　登録したリスト表示
import os

os.system('wpa_cli -i wlan0 scan')
os.system('wpa_cli -i wlan0 scan_result')
os.system('wpa_cli -i wlan0 list_network')
```

```python
# 登録したwifi設定を修正する
# リモートで制御する場合、慎重に設定してください
# wifi との接続が切れると、リモート制御もできなくなったかも。
# 使えないssid、pskに修正すると、再起動しても、wifiと接続できない。
import os

def set_wifi_ssid_psk(networkNo, ssid, psk, prority):
    os.system('sudo wpa_cli -i wlan0 set_network ' + networkNo + ' ssid ' + '\'"' + ssid + '"\'')
    os.system('sudo wpa_cli -i wlan0 set_network ' + networkNo + ' psk ' + '\'"' + psk + '"\'')
     os.system('sudo wpa_cli -i wlan0 set_network ' + networkNo + ' prority ' + '\'"' + prority + '"\'')
    # os.system('sudo wpa_cli -i wlan0 select_network ' + networkNo)
    os.system('sudo wpa_cli -i wlan0 enable_network ' + networkNo)
    os.system('sudo wpa_cli -i wlan0 save_config')
    
def add_new_network():
    new_networkNo = os.popen('wpa_cli -i wlan0 add_network')
    return new_networkNo.read()
    
def main():
    new_network_no = add_network();
    set_wifi_ssid_psk(new_network_no, 'iPhone', 'wangyuping')

if __name__ == '__main__':
    main()
```

このコマンドを使って、コマンドプロンプトで修正もできる。

` sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`

# 5. Bluetooth

## 5.1 Scan

> ファイル名は`bluetooth.py`の名前を設定しないでください。
>
> ライブラリのファイル名と同じにすれば、うまく動かない。

パッケージをインストール

```python
# Bluetooth機能 (BLE機能は別のライブラリが必要)
sudo apt-get install libbluetooth-dev
sudo pip install pybluez
```

```python
# BLE機能(Bluetooth Low Energy) DEMOが少ない
sudo apt-get install git build-essential libglib2.0-dev
git clone https://github.com/IanHarvey/bluepy.git
cd bluepy
python3 setup.py build
sudo python3 setup.py install
```


```python
# ブルートゥースの名前から探す
# No module named `bluetooth` のエラーが発生する時
# `python3 -m pip install pybluez` を実行する
import bluetooth
 
target_name = "My Device"
target_address = None
 
nearby_devices = bluetooth.discover_devices()
 
for bdaddr in nearby_devices:
    if target_name == bluetooth.lookup_name( bdaddr ):
        target_address = bdaddr
        break
 
if target_address is not None:
    print("found target bluetooth device with address ", target_address)
else:
    print("could not find target bluetooth device nearby")
```

```python
#スキャン
import bluetooth
 
nearby_devices = bluetooth.discover_devices(lookup_names=True)
for addr, name in nearby_devices:
    print("  %s - %s" % (addr, name))
 
    services = bluetooth.find_service(address=addr)
    for svc in services:
        print("Service Name: %s"    % svc["name"])
        print("    Host:        %s" % svc["host"])
        print("    Description: %s" % svc["description"])
        print("    Provided By: %s" % svc["provider"])
        print("    Protocol:    %s" % svc["protocol"])
        print("    channel/PSM: %s" % svc["port"])
        print("    svc classes: %s "% svc["service-classes"])
        print("    profiles:    %s "% svc["profiles"])
        print("    service id:  %s "% svc["service-id"])
        print("")
```

```python
# BLE スキャン
# 注意点：コマンドプロンプトsudoで実行する
# sudo python3 xxx.py
from bluepy.btle import Scanner, DefaultDelegate

class ScanDelegate(DefaultDelegate):
    def __init__(self):
        DefaultDelegate.__init__(self)

    def handleDiscovery(self, dev, isNewDev, isNewData):
        if isNewDev:
            print("Discovered device", dev.addr)
        elif isNewData:
            print("Received new data from", dev.addr)

scanner = Scanner().withDelegate(ScanDelegate())
devices = scanner.scan(10.0)

for dev in devices:
    print("Device %s (%s), RSSI=%d dB" % (dev.addr, dev.addrType, dev.rssi))
    for (adtype, desc, value) in dev.getScanData():
        print("  %s = %s" % (desc, value))
```



## 5.2 通信(未確認)

在pybluez中，先用discover devices函数扫描附近设备，然后调用find services函数扫描设备提供的服务。一般来说都会有rfcomm协议的服务。在树莓派上建立rfcomm的socket，然后连接相应服务的端口号(我选用的是端最小端口号，服务类型任意)。在socket请求连接的过程中，树莓派与远端设备会自然发生配对，手动确认配对即可。

```python
# RFCOMM 通信
# server
import bluetooth
 
server_sock=bluetooth.BluetoothSocket( bluetooth.RFCOMM )
 
port = 1
server_sock.bind(("",port))
server_sock.listen(1)
 
client_sock,address = server_sock.accept()
print "Accepted connection from ",address
 
data = client_sock.recv(1024)
print "received [%s]" % data
 
client_sock.close()
server_sock.close()
```

```python
# client
import bluetooth
 
bd_addr = "01:23:45:67:89:AB"
 
port = 1
 
sock=bluetooth.BluetoothSocket( bluetooth.RFCOMM )
sock.connect((bd_addr, port))
 
sock.send("hello!!")
 
sock.close()
```

# 6. 起動できない原因

ライスパイのランプ

| 緑のランプ | 赤のランプ | ステータス | 原因           |
| ---------- | ---------- | ---------- | -------------- |
| 点滅       | 常時ON     | 正常       | OK             |
| OFF        | 常時ON     | 不正常     | SDカードエラー |
| OFF        | 点滅       | 不正常     | 電源電圧不正常 |
| 常時ON     | 常時ON     | 不正常     | OSファイル損失 |

# 7. 参照（中国語URLあり）

[インストールとリモート制御など](https://github.com/TommyZihao/ZihaoTutorialOfRaspberryPi)

[GPIO制御]('https://qiita.com/masato/items/715e28e0c0c945a54297 )

[WIFI接続](https://www.cnblogs.com/hotwater99/p/12760261.html)

[Bluetooth Low Energy 機能](https://www.cnblogs.com/zjutlitao/p/10171913.html)

[Bluetooth Scanと通信](https://blog.csdn.net/zym326975/article/details/93897096)
