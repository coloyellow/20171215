# 伺服器操作命令指引

On 2022/06/11 at 140.122.104.20 by 曾元顯

## 目次

1. [在 Ubuntu 上安裝 Conda 給所有使用者](#create-conda)
    * [每位使用者需執行下列命令，以啟動 conda 指令](#init-conda)
2. [創建新的群組](#創建新的群組)
3. [創建新的使用者](#創建新的使用者)
    * [對已經有帳號者，增加其群組](#對已經有帳號者，增加其群組)
4. [新增使用者的命令摘要](#新增使用者的命令摘要)
5. [安裝 NVIDIA 驅動程式](#NVIDIA)
    * [以程式測試 GPU，看能否正常執行](#test-GPU)
6. [其他安裝或指令](#其他安裝或指令)
    * [安裝 tmux 給所有使用者](#tmux)
    * [How to Change Hostname on Ubuntu 20.04 (極少用到)](#rename-hostname)

## 在 Ubuntu 上安裝 Conda 給所有使用者 <a name="create-conda"></a>

參考：

1. https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html

2. Install Anaconda for all Users on Linux: https://www.youtube.com/watch?v=qc5dmXAolfY

```bash
# https://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade
sudo apt update
sudo apt upgrade
# 下載正確的安裝程式：
wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh

# 執行安裝程式：
sudo bash Miniconda3-py38_4.12.0-Linux-x86_64.sh
   Miniconda3 will now be installed into this location:
   /root/miniconda3  <<== 不要安裝在這裡!!!
     - Press ENTER to confirm the location
     - Or specify a different location below
   [/root/miniconda3] >>> /opt/miniconda3  <<== 建議安裝在此！！！

# 編輯環境設定檔案，增加 miniconda 的執行檔案路徑 到 環境變數 PATH 中：
sudo nano /etc/environment
#  add "/opt/miniconda3/bin:" to PATH
cat /etc/environment
#   PATH="/opt/miniconda3/bin:/usr/local/sbin: ...

# 讓 conda 即刻啟動：
conda init bash
source ~/.bashrc
conda --version
# 若失敗，只好登出，再重新登入，再執行：
conda --version
```

### 每位使用者需執行下列命令，以啟動 conda 指令 <a name="init-conda"></a>

```bash
# The next line should be run by the users who would like to have 
#   conda command without installation by themselves
conda init bash
source ~/.bashrc
```

## 創建新的群組

```bash
cat /etc/passwd   # 查看帳號名稱、群組id、登入的 shell 
cat /etc/group    # 查看群組 id 與 名稱 的對應

# 增加群組名稱：groupadd -g Group_ID Group_Name
sudo groupadd -g 5000 student
sudo groupadd -g 2000 crew
sudo groupadd -g 1001 admin
# if 1001 already exists, "sudo nano /etc/group" to update     "admin:x:1001:sam",
#   then "sudo systemctl restart sshd" to make it take effect
```

## 創建新的使用者

```bash
# 建立新帳號，且初始群組為 admin (1001)
sudo adduser --gid 1001 yang
#   會需要輸入 yang 的全名（可用中文）、初始密碼
# 或是初始設定時的群組為 crew (2000)
sudo adduser --gid 2000 gland
```

### 對已經有帳號者，增加其群組

```bash
sudo usermod -aG docker yang
# 查看 yang 帳號所屬的群組
id yang
# 使用者id=1002(yang) id群組=1001(admin) 組=1001(admin)
sudo usermod -aG sudo yang
sudo usermod -aG docker yang
id yang
# 使用者id=1002(yang) id群組=1001(admin) 組=1001(admin),27(sudo),135(docker)
```

## 新增使用者的命令摘要

```bash
# 以昭沂為例
sudo adduser --gid 1001 chaoyi
# 新增其 docker 群組
sudo usermod -aG docker chaoyi
# 新增其 admin 群組，視需要執行
sudo usermod -aG admin chaoyi
# 新增其 sudo 群組，視需要執行
sudo usermod -aG sudo chaoyi
# 查看其 所屬群組
id chaoyi
uid=1003(chaoyi) gid=1001(admin) groups=1001(admin),27(sudo),998(docker)

# 將以上的帳號、密碼，通知該名使用者，並告知若要修改個人密碼，只需執行：
passwd
# 另外，告知每位使用者啟動 conda 指令，只需第一次執行：
conda init bash
source ~/.bashrc
# 之後一旦再登入系統，自動有 conda 可用。
```

## 安裝 NVIDIA 驅動程式 <a name="NVIDIA"></a>

從 NVIDIA 網站下載 `NVIDIA-Linux-x86_64-515.48.07.run`，
執行 `sudo ./NVIDIA-Linux-x86_64-515.48.07.run` 結果其建議：

```bash
 The NVIDIA driver provided by Ubuntu can be installed by launching the "Software & Updates" application, and by selecting the NVIDIA driver from the "Additional Drivers" tab.
```

因此，要從 Ubuntu 的 GUI 來安裝。底下以遠端登入方式安裝，分幾個步驟達成。

### 步驟一：安裝 XRDP

為了要從遠端登入 GUI，參考：https://www.itproportal.com/guides/how-to-remote-desktop-into-ubuntu/
需要在 Ubuntu 上安裝 XRDP：

```bash
sudo apt autoremove
sudo apt install xrdp
sudo systemctl enable xrdp
```

### 步驟二：在自己的電腦上安裝 Microsoft Remote Desktop (MRD)

### 步驟三：執行 VPN 連到學校，在學校的網域中，MRD 才能連上學校的主機

### 步驟四：以MRD連上 Ubuntu 後，在其 GUI 上安裝 NVIDIA 驅動程式

```bash
 The NVIDIA driver provided by Ubuntu can be installed by launching the "Software & Updates" application, and by selecting the NVIDIA driver from the "Additional Drivers" tab.

在「額外驅動程式」的頁簽中，選擇第一個選項，亦即：
使用 NVIDIA driver metapackage 來自 nvidia-driver-510(專有，已通過測試)
按「套用變更」後，點選「重新啟動」系統。

註：若無法點選「重新啟動」按鈕，則從命令列重新啟動，請執行命令：
sudo reboot
```

### 步驟五：測試驅動程式

```bash
nvidia-smi
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 510.73.05    Driver Version: 510.73.05    CUDA Version: 11.6     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |                               |                      |               MIG M. |
   |===============================+======================+======================|
   |   0  Quadro RTX 6000     Off  | 00000000:37:00.0 Off |                  Off |
   | 34%   47C    P0    66W / 260W |      0MiB / 24576MiB |      0%      Default |
   |                               |                      |                  N/A |
   +-------------------------------+----------------------+----------------------+
   |   1  Quadro RTX 6000     Off  | 00000000:86:00.0 Off |                  Off |
   | 32%   41C    P0    55W / 260W |      0MiB / 24576MiB |      3%      Default |
   |                               |                      |                  N/A |
   +-------------------------------+----------------------+----------------------+
   +-----------------------------------------------------------------------------+
   | Processes:                                                                  |
   |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
   |        ID   ID                                                   Usage      |
   |=============================================================================|
   |  No running processes found                                                 |
   +-----------------------------------------------------------------------------+
```

### 以程式測試 GPU，看能否正常執行 <a name="test-GPU"></a>

#### [步驟一：請先安裝 Conda，以便建立虛擬環境](#create-conda)

#### 步驟二：建立虛擬環境，取得測試程式，進行測試

```bash
# 建立 python 虛擬環境
conda create -n SimTrans python=3.9
conda activate SimTrans
# 取得測試程式：
git clone https://github.com/NTNU-GLIS-AI-Lab/gpuTest
# 註：在新的機器上，第一次要安裝 git： sudo apt install git
cd gpuTest
# 安裝必要套件
pip install -r requirements.txt
# 執行測試程式
python SimTrain_1.py 2
#  It takes 96.1664 seconds
#  It takes 50.0423 seconds
```

#### 在執行測試程式時，請在另一個視窗執行 nvidia-smi 觀看 GPU 是否有執行起來！

```bash
# 繼續執行測試程式：
python SimTrain_1_predict.py 10 batch 
#   It takes 2.8571 seconds for loading model
#   It takes 17.5252 seconds to predict 10 examples for 10 times.
#   It takes 20.3823 seconds for loading and prediction.
python SimTrain_1_predict.py 10 batch noGPU
#   It takes 1.1063 seconds for loading model
#   It takes 10.0556 seconds to predict 10 examples for 10 times.
#   It takes 11.1619 seconds for loading and prediction.
python SimTrain_1_predict.py 10 each noGPU
#   It takes 1.1633 seconds for loading model
#   It takes 85.1152 seconds to predict 10 examples for 10 times.
#   It takes 86.2786 seconds for loading and prediction.
python SimTrain_1_predict.py 10 each 
#   It takes 2.8807 seconds for loading model
#   It takes 152.4110 seconds to predict 10 examples for 10 times.
#   It takes 155.2917 seconds for loading and prediction.
```

### 繼續測試 GPU

參考：https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker

```bash
echo $distribution
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
echo $distribution
sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
# At this point, a working setup can be tested by running a base CUDA container:
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi

# Now run the benchmark codes
./benchmark.sh 
# The above works without problems

./gpu-burn.sh
```

## 其他安裝或指令

### 安裝 tmux 給所有使用者 <a name="tmux"></a>

執行 `sudo apt install tmux` 即可安裝 tmux 供系統上的所有使用者運用。

常見 tmux 指令：

```bash
# To create a new session, called test-session
tmux new -s test-session
# To detach current session, press ctrl+b, d
# To list all sessions
tmux ls
# To attach to an existing session:
tmux attach -t test-session
# To kill a tmux session
tmux kill-session -t test-session

# 或者更簡化的指令：
tmux                # 產生 session No 為 0 的新 session
                    # 若已經有 0，則產生新的 session 1，依此類推
tmux ls
tmux a              # 進入 session No 為 0 的 session
tmux a -t 1         # 進入 session No 為 1 的 session
tmux kill-session      # 刪除 session No 為 0 的 session
tmux kill-session -t 1 # 刪除 session No 為 1 的 session
```

### How to Change Hostname on Ubuntu 20.04 (極少用到) <a name="rename-hostname"></a>

參考：https://phoenixnap.com/kb/ubuntu-20-04-change-hostname

```bash
sam@HP-DL380-1:~$ sudo hostnamectl set-hostname HP-DL380-2
[sudo] sam 的密碼： 
sam@HP-DL380-1:~$ hostname
HP-DL380-2
sam@HP-DL380-1:~$ hostnamectl
   Static hostname: HP-DL380-2
sam@HP-DL380-1:~$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	HP-DL380-1
sam@HP-DL380-1:~$ sudo reboot
```
