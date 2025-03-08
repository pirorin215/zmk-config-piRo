# piRo

## 目次
  - [1. ローカルビルド環境構築](#1-ローカルビルド環境構築)
    - [1.1. ライブラリインストール](#11-ライブラリインストール)
    - [1.2. zmkfirmware](#12-zmkfirmware)
    - [1.3. west](#13-west)
    - [1.4. zephyr-sdk](#14-zephyr-sdk)
    - [1.5. カスタムキーボードのリポジトリチェックアウト](#15-カスタムキーボードのリポジトリチェックアウト)
    - [1.6. ビルドスクリプト](#16-ビルドスクリプト)
  - [2. ビルド実行](#2-ビルド実行)
  - [3. WARNING抑止](#3-WARNING抑止)

## 1. ローカルビルド環境構築

・ZMK公式のNative Setup説明

https://zmk.dev/docs/development/local-toolchain/setup/native

・このメモを作った理由

独自キーボードのビルド環境を作る方法に手間取ったのでメモ

> [!WARNING]
> MacOS用

### 1.1. ライブラリインストール
``` bash
brew install cmake ninja gperf python3 ccache qemu dtc wget libmagic wget
```

### 1.2. zmkfirmware

``` bash
cd ~/
git clone https://github.com/zmkfirmware/zmk.git
cd zmk
python3 -m venv ~/zmk/.venv
source ~/zmk/.venv/bin/activate
```

``` bash
pip install west
pip install --upgrade pip
```

### 1.3. west

> [!NOTE]
> 独自のwest.ymlを利用したビルドを行う為に変更した手順
 
``` bash
west init -l app
west update
west zephyr-export
pip install -r ~/zmk/zephyr/scripts/requirements.txt
```

> [!CAUTION]
> 公式手順は独自のwest.ymlを利用したビルドが行いづらい

<details>
<summary>公式手順</summary>

```bash
west init ~/zephyrproject 
cd ~/zephyrproject
west update
west zephyr-export
pip install -r ~/zephyrproject/zephyr/scripts/requirements.txt
```
</details>

### 1.4. zephyr-sdk
``` bash
cd ~
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.3/zephyr-sdk-0.16.3_macos-x86_64.tar.xz
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.3/sha256.sum | shasum --check --ignore-missing
tar xvf zephyr-sdk-0.16.3_macos-x86_64.tar.xz
cd ~/zephyr-sdk-0.16.3/
./setup.sh
```

### 1.5. カスタムキーボードのリポジトリチェックアウト
``` bash
cd ~/zmk/app
git clone https://github.com/pirorin215/zmk-config-piRo.git
cp zmk-config-piRo/config/west.yml ~/zmk/app/
west update
```

### 1.6. ビルドスクリプト

~/zmk/app/build.sh

``` bash
source ~/zmk/.venv/bin/activate
rm -rf build

west build -d build/piRo_reset -p -b seeeduino_xiao_ble -- -DSHIELD="settings_reset"
west build -d build/piRo_L     -p -b seeeduino_xiao_ble -- -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" -DSHIELD="piRo_L rgbled_adapter"
west build -d build/piRo_R     -p -b seeeduino_xiao_ble -- -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" -DSHIELD="piRo_R rgbled_adapter"

ls -l build/*/*/*.uf2
cksum build/*/*/*.uf2
```

## 2. ビルド実行

``` bash
cd ~/zmk/app
sh build.sh
```

## 3. WARNING抑止

~/zmk/app/CMakeLists.txtを編集

編集前
```
zephyr_cc_option(-Wfatal-errors)
```

編集後
```
zephyr_cc_option(-Wfatal-errors -Wno-unused-function -Wno-maybe-uninitialized -Wno-excess-initializers)
```

上記を設定することで、piRo_LとpiRo_Rのビルド時に警告（WARNING)が表示されなくなる。

settings_resetのビルド時はその他の警告（WARNING)がたくさん出るが抑止の仕方が分からなかった。

## 4. ZMK 日本語配列の主な記号の指定値メモ

ZMKで日本語配列のキーマップ作る情報が見当たらないでメモを作成

| 標準 | シフト | 指定値 |
|------|--------|--------|
| ! | ! | &kp LS(NUMBER_1) |
| " | " | &kp LS(NUMBER_2) |
| # | # | &kp LS(NUMBER_3) |
| $ | $ | &kp LS(NUMBER_4) |
| % | % | &kp LS(NUMBER_5) |
| & | & | &kp LS(NUMBER_6) |
| ' | ' | &kp LS(NUMBER_7) |
| ( | ( | &kp LS(NUMBER_8) |
| ) | ) | &kp LS(NUMBER_9) |
| - | = | &kp MINUS |
| ^ | ~ | &kp EQUAL |
| ¥ | \| | &kp INT3 |
| @ | ` | &kp LEFT_BRACKET |
| [ | { | &kp RIGHT_BRACKET |
| ; | + | &kp SEMICOLON |
| : | * | &kp SINGLE_QUOTE |
| ] | } | &kp NON_US_HASH |
| , | < | &kp COMMA |
| . | > | &kp DOT |
| / | ? | &kp SLASH |
| _ | _ | &kp INT1 |
