# piRo

## 目次
  - [1. ローカルビルド](#1.-ローカルビルド)
    - [1.1. ライブラリインストール](#1.1.ライブラリインストール)
    - [1.2. west](#1.2.-west)
    - [1.3. zephyr-sdk](#1.3-zephyr-sdk)
    - [1.4. カスタムキーボードのリポジトリチェックアウト](#1.4-カスタムキーボードのリポジトリチェックアウト)
    - [1.5. ビルドスクリプト](#1.5-ビルドスクリプト)
    - [1.6. ビルド実行](#1.6-ビルド実行)

## 1. ローカルビルド

・ローカルビルドの公式セットアップ説明
https://zmk.dev/docs/development/local-toolchain/setup/native

・このメモを作った理由
独自キーボードのビルド環境を作る方法に手間取ったのでメモ
MacOS用

### 1.1. ライブラリインストール
```
brew install cmake ninja gperf python3 ccache qemu dtc wget libmagic wget
```

### 1.2. west

```
cd ~/
git clone https://github.com/zmkfirmware/zmk.git
cd zmk
python3 -m venv ~/zmk/.venv
source ~/zmk/.venv/bin/activate
pip install west
pip install --upgrade pip
```

```
west init -l app
west update
west zephyr-export
pip install -r ~/zmk/zephyr/scripts/requirements.txt
```

```
正式マニュアルはこうだけど、これだと独自のwest.ymlでビルドしづらい
west init ~/zephyrproject 
cd ~/zephyrproject
west update

west zephyr-export
pip install -r ~/zephyrproject/zephyr/scripts/requirements.txt
```

### 1.3. zephyr-sdk
```
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.3/zephyr-sdk-0.16.3_macos-x86_64.tar.xz
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.3/sha256.sum | shasum --check --ignore-missing
tar xvf zephyr-sdk-0.16.3_macos-x86_64.tar.xz
cd ~/zephyr-sdk-0.16.3/
./setup.sh
```

### 1.4. カスタムキーボードのリポジトリチェックアウト
```
cd ~/zmk/app
git clone https://github.com/pirorin215/zmk-config-piRo.git
cp zmk-config-piRo/config/west.yml ~/zmk/app/
west update
```

### 1.5. ビルドスクリプト

~/zmk/app/build.sh

```
source ~/zmk/.venv/bin/activate
rm -rf build

west build -d build/piRo_reset -p -b seeeduino_xiao_ble -- -DSHIELD="settings_reset"
west build -d build/piRo_L     -p -b seeeduino_xiao_ble -- -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" -DSHIELD="piRo_L rgbled_adapter"
west build -d build/piRo_R     -p -b seeeduino_xiao_ble -- -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" -DSHIELD="piRo_R rgbled_adapter"

ls -l build/*/*/*.uf2
cksum build/*/*/*.uf2
```

### 1.6. ビルド実行

```
cd ~/zmk/app
sh build.sh
```
