# The Coral USB Accelerator with Raspberry Pi


## `1. 從序列埠進到 Raspberry Pi`

#### a. 依照下圖將pi裝上"PL2303HXD USB轉TTL傳輸線"，並接上筆電
<img src="https://i.imgur.com/gB9atUg.png" width="50%" height="50%">

#### b. 將pi連接好電源線，開機(紅燈會亮，黃光會閃)
#### c. 查詢筆電的Device Manager確認你的pi接上到COM幾(以下圖為例 為COM3)
<img src="https://i.imgur.com/3jWZh2M.png" width="50%" height="50%">

#### d. 開putty，點選Serial
Serial line填你在上一部查的COM，Speed填115200，如下圖所示
點選右下角的Open開啟連結
<img src="https://i.imgur.com/tKZJdT2.png" width="50%" height="50%">

### Hint: 如果連上後畫面全黑，可以試試按Enter，或重開pi

-------------

## `2. 設定網路`
    $ ifconfig wlan0
    => 發現沒網路

    $ sudo iwlist wlan0 scan
    => 查詢現有的網路，找到想要的那個

    $ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

    在裡頭填上資料，如下:
    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
        ssid="hahaha"           =>網路名稱
        psk="1234567890123"     =>密碼
        proto=RSN               =>WPA2填RSN，WPA1填WPA
        key_mgmt=WPA-PSK        =>PSK填WPA-PSK，EAP填WPA-EAP
        pairwise=CCMP           =>AES cipher(WPA2)填CCMP，TKIP cipher(WPA1)填TKIP
        auth_alg=OPEN
    }
    =>按ctrl+X =>按Y 儲存跳離檔案

    $ sudo ifdown wlan0
    $ sudo ifup wlan0
    $ sudo kill -9 $(ps -ef | grep wpa | awk '{print $2}')
    $ sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
    $ sudo dhclient
    =>這5個指令中間有錯誤訊息皆忽略

    $ ifconfig wlan0
    => 查看網路有沒有了

### Hint1: sudo reboot試試
### Hint2: 可以把ifdown和ifup兩行程式換成 ⇒
    $ sudo ip link set wlan0 down
    $ sudo ip link set wlan0 up   
### Hint3: 出錯的話 試試以下作法
    $ sudo nano /etc/network/interfaces

    在裡頭填上資料，如下:
    auto lo

    iface lo inet loopback
    iface eth0 inet dhcp

    allow-hotplug wlan0
    iface wlan0 inet manual
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
    iface default inet dhcp
    =>按ctrl+X =>按Y 儲存跳離檔案

-------------

## `3. vnc遠端控制`
#### a. 在pi上打以下指令
    $ sudo apt-get install tightvncserver -y
    => 安裝tightvncerver

    $ vncserver
    => 設定密碼等等等

    $ ps a | grep vnc
    => 確認vnc server是開在哪一個DISPLAY上
#### b. 在筆電安裝VNC viewer，步驟如下
在Chrome上安裝 VNC Viewer for Google Chrome套件
在Chrome上打 "chrome://apps"，點選 VNC Viewer

#### c. 用VNC連線，步驟如下
在address那欄填上"PI's IP address:5900+DISPLAY NUMBER"
例如：
PI's IP address == 192.168.0.1
DISPLAY NUMBER == 1
那addres那欄要填 => `192.168.0.1:5901`

### Hint1: 可以用`$ vncpasswd`來設定新密碼
### Hint2: 電腦和raspberry要在同一個網路下
### Hint3: 出錯的話 試試以下作法
    $ ps aux | grep -i apt
    =>查看有沒有process在工作

    $ sudo kill -9 <process id>
    => 把他們停掉

    $ sudo killall apt apt-get
    => 停掉所有的工作
### Hint4: 出錯的話，不知道有沒有用但可以試試看
    $ sudo killall apt apt-get
    $ sudo rm /var/lib/apt/lists/lock
    $ sudo rm /var/cache/apt/archives/lock
    $ sudo rm /var/lib/dpkg/lock*
    $ sudo dpkg --configure -a
    $ sudo apt update

----------

## `4. 相機設定`
#### a. 軟體設定

    $ sudo raspi-config

    選擇Interfacing Options
    將Camera和I2C都開啟    //I2C記得開，很重要
    她會自動重開機，就讓她做

#### b. 硬體安裝

按照這個 https://www.youtube.com/watch?v=GImeVqHQzsE&feature=emb_title 影片裝上相機

我的裝好的樣子:
<img src="https://i.imgur.com/4H47wgX.jpg" width="50%" height="50%">

    $ raspistill -o haha.png
    => 拍照

    $ eog haha.png
    => 查看拍的照片有沒有成功

### Hint: png不行，就換jpg試試

------------------

## `5. 實作 USB Accelerator`
#### a. 安裝Edge TPU runtime，如下:
    $ echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list

    $ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

    $ sudo apt-get update
    $ sudo apt-get install libedgetpu1-std
把USB Accelerator接上raspberry pi
我接好的雛形:
<img src="https://i.imgur.com/SvIH12c.jpg" width="50%" height="50%">



#### b. 查詢欲下載的版本，並下載。步驟如下:
查詢規格: https://www.raspberrypi.com.tw/10684/55/

    nano一個py檔(EX: nano haha.py )
    -------------------haha.py------------------------------------
    #!/usr/bim/env python
    import pip
    print (pip.pep425tags.get_supported())
    ------------------------------------------------------------------
    python3 haha.py
    =>你會看到pip執行的版本，像我的話會看到有 ('cp35', 'cp35m', 'linux_armv7l')
    =>那我就知道要下載pip3 install tflite_runtime-1.14.0-cp35-cp35m-linux_armv7l.whl 這個版本

到這個 https://www.tensorflow.org/lite/guide/python 網站
點選下載你規格的whl檔

    $ pip3 install tflite_runtime-1.14.0-cp37-cp37m-linux_armv7l.whl
    => 這行可能會跑很久，要有耐心

#### c. 額外弄一些東東，不然會出錯

`*切記: 以下所有指令，如果發現出錯`
`sudo就加-H, wget就加no-check-certificate，可能本來不行的就成功了`

    $ strings /usr/lib/arm-linux-gnueabihf/libstdc++.so.6 | grep GLIBCXX
    => 查看當前GLIBCXX的版本
    => 若缺少GLIBCXX_3.4.22，做以下指令

    $ sudo apt-get install libstdc++6
    $ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt-get dist-upgrade

    $ strings /usr/lib/arm-linux-gnueabihf/libstdc++.so.6 | grep GLIBCXX
    => 看看有沒有新加了

#### d. 這個斷落的內容，是我在實作時有加的東西，但我不知道有沒有影響，所以等你出錯時，再試著加加看

    $ sudo apt-get install -y libhdf5-dev libc-ares-dev libeigen3-dev
    $ sudo pip3 install keras_applications==1.0.7 --no-deps
    $ sudo pip3 install keras_preprocessing==1.0.9 --no-deps

    $ wget -O numpy-1.16.1-cp35-cp35m-linux_armv7l.whl https://www.piwheels.hostedpi.com/simple/numpy/numpy-1.16.1-cp35-cp35m-linux_armv7l.whl
    $ pip3 install numpy-1.16-1.whl

    $ wget -O h5py-2.9.0-cp35-cp35m-linux_armv7l.whl https://www.piwheels.org/simple/h5py/h5py-2.9.0-cp35-cp35m-linux_armv7l.whl
    $ pip3 install h5py-2.9.0-cp35-cp35m-linux_armv7l.whl --no-deps
    $ sudo apt-get install -y openmpi-bin libopenmpi-dev
    -----------------------------------------------------------------
    $ virtualenv --python=python3 /tmp/v
    $ sudo apt-get install libtiff-dev libjpeg-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl-dev tk-dev python-tk


#### e. 跑一個model試試看(出錯基本上都是這裡出錯，心臟要夠大顆QQ)

    $ mkdir coral && cd coral
    $ git clone https://github.com/google-coral/tflite.git

    $ cd tflite/python/examples/classification
    $ bash install_requirements.sh
    => 這行可能會跑很久，要有耐心(也可能很快拉XD)

    $ python3 classify_image.py \
    --model models/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite \
    --labels models/inat_bird_labels.txt \
    --input images/parrot.jpg

如果跑出以下類似訊息，就代表成功了!

    INFO: Initialized TensorFlow Lite runtime.
    ----INFERENCE TIME----
    Note: The first inference on Edge TPU is slow because it includes loading the model into Edge TPU memory.
    11.8ms
    3.0ms
    2.8ms
    2.9ms
    2.9ms
    -------RESULTS--------
    Ara macao (Scarlet Macaw): 0.76562

我們的成果:
<img src="https://i.imgur.com/G27pr5N.png" width="80%" height="80%">


### Hint: 出錯的話，試試這個給他run，可能會有奇效
    $ apt-get install -y \
    python3 \
    python-dev \
    python3-dev

-----------------------------



### `參考資料`
https://coral.withgoogle.com/docs/accelerator/get-started/

https://www.raspberrypi.com.tw/1999/connect-to-raspberry-pi-via-serial/

https://www.raspberrypi.com.tw/2152/setting-up-wifi-with-the-command-line/?fbclid=IwAR3RQd2CvjVx8sOiPf_l2ixpUwcDIszvgRf6L5_ZEk9aufDOeYXnkLPL3wg

https://blog.cavedu.com/2018/03/23/raspberry-pi-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%E9%81%A0%E7%AB%AF%E6%A1%8C%E9%9D%A2/

https://blog.wuct.me/raspberry-pi-100abbe7a1fd
