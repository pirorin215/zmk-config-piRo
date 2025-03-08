# ZMKローカルビルド環境構築メモ

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
  - [4. ZMK日本語配列の主な記号の指定値メモ](#4-ZMK日本語配列の主な記号の指定値メモ)
  - [5. USBデバッグの方法](#5-USBデバッグの方法)
    - [5.1. ビルドの引数変更](#51-ビルドの引数変更)
    - [5.2. confの変更](#52-confの変更)
    - [5.3. デバイスポイントの確認](#53-デバイスポイントの確認)
    - [5.4. ログ出力](#54-ログ出力)

## 1. ローカルビルド環境構築

ZMK公式のNativeSetup説明<br/>
https://zmk.dev/docs/development/local-toolchain/setup/native

<pre>
ZMK公式のローカルビルド構築手順は上記URL。
独自キーボードのビルド環境を作る方法に手間取ったのでメモを作成。  
</pre>

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

~/zmk/app/build.sh を新規作成

``` bash
source ~/zmk/.venv/bin/activate
rm -rf build

west build -d build/piRo_reset -p -b seeeduino_xiao_ble -- -DSHIELD="settings_reset"
west build -d build/piRo_L     -p -b seeeduino_xiao_ble -- ¥
    -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" ¥
    -DSHIELD="piRo_L rgbled_adapter"

west build -d build/piRo_R     -p -b seeeduino_xiao_ble -- ¥
    -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" ¥
    -DSHIELD="piRo_R rgbled_adapter"

ls -l build/*/*/*.uf2
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

## 4. ZMK日本語配列の主な記号の指定値メモ

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

## 5. USBデバッグの方法

### 5.1. ビルドの引数変更

ビルドコマンドに<br/>
**-DSNIPPET=zmk-usb-logging** <br/>
を追加する

変更前
```
west build -d build/piRo_L     -p -b seeeduino_xiao_ble -- ¥
    -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" ¥
    -DSHIELD="piRo_L rgbled_adapter"
```

変更後
```
west build -d build/piRo_L     -p -b seeeduino_xiao_ble -- ¥
    -DZMK_CONFIG="$HOME/zmk/app/zmk-config-piRo/config/" ¥
    -DSHIELD="piRo_L rgbled_adapter" ¥
    -DSNIPPET=zmk-usb-logging
```
### 5.2. confの変更

zmk-config-piRo/config/boards/shields/piRo/piRo_R.conf に設定を追加

```
CONFIG_LOG=y
CONFIG_ZMK_LOG_LEVEL_DBG=y
CONFIG_USB_DEVICE_STACK=y
CONFIG_SERIAL=y
CONFIG_CONSOLE=y
CONFIG_UART_INTERRUPT_DRIVEN=y
CONFIG_UART_LINE_CTRL=y
```

### 5.3. デバイスポイントの確認

``` bash
$ ls /dev/tty.usb*
/dev/tty.usbmodem12101
$
```
上記の様に「/dev/tty.usb〜」が表示されない場合、</br>
デバッグ用のビルドに失敗してる。

### 5.4. ログ出力

> [!TIP]
> 「Ctrl-t q」で終了
``` bash
sudo tio /dev/tty.usbmodem12101
```

キースイッチを押した場合の出力例
```
$ sudo tio /dev/tty.usbmodem12101
[23:28:20.765] tio 3.8
[23:28:20.765] Press ctrl-t q to quit
[23:28:20.768] Connected to /dev/tty.usbmodem12101
[00:05:05.391,357] <err> zmk: Failed to set output 0 to 0: -11
[00:05:05.396,636] <dbg> zmk: kscan_matrix_read: Sending event at 3,3 state on
[00:05:05.396,728] <dbg> zmk: zmk_physical_layouts_kscan_process_msgq: Row: 3, col: 3, position: 43, pressed: true
[00:05:05.396,789] <dbg> zmk: position_state_changed_listener: 43 bubble (no undecided hold_tap active)
[00:05:05.396,881] <dbg> zmk: zmk_keymap_apply_position_state: layer_id: 0 position: 43, binding name: key_press
[00:05:05.396,911] <dbg> zmk: on_keymap_binding_pressed: position 43 keycode 0x70051
[00:05:05.396,942] <dbg> zmk: hid_listener_keycode_pressed: usage_page 0x07 keycode 0x51 implicit_mods 0x00 explicit_mods 0x00
[00:05:05.396,942] <dbg> zmk: zmk_hid_implicit_modifiers_press: Modifiers set to 0x00
[00:05:05.396,972] <dbg> zmk: zmk_endpoints_send_report: usage page 0x07
[00:05:05.408,630] <dbg> zmk: bvd_sample_fetch: ADC raw 1446 ~ 1270 mV => 3760 mV
[00:05:05.408,660] <dbg> zmk: bvd_sample_fetch: Percent: 42
[00:05:05.463,714] <dbg> zmk: kscan_matrix_read: Sending event at 3,3 state off
[00:05:05.464,080] <dbg> zmk: zmk_physical_layouts_kscan_process_msgq: Row: 3, col: 3, position: 43, pressed: false
[00:05:05.464,111] <dbg> zmk: position_state_changed_listener: 43 bubble (no undecided hold_tap active)
[00:05:05.464,172] <dbg> zmk: zmk_keymap_apply_position_state: layer_id: 0 position: 43, binding name: key_press
[00:05:05.464,202] <dbg> zmk: on_keymap_binding_released: position 43 keycode 0x70051
[00:05:05.464,233] <dbg> zmk: hid_listener_keycode_released: usage_page 0x07 keycode 0x51 implicit_mods 0x00 explicit_mods 0x00
[00:05:05.464,263] <dbg> zmk: zmk_hid_implicit_modifiers_release: Modifiers set to 0x00
[00:05:05.464,263] <dbg> zmk: zmk_endpoints_send_report: usage page 0x07
[00:05:05.478,485] <dbg> zmk: zmk_hid_mouse_movement_set: Mouse movement set to -2/0
[00:05:05.478,546] <dbg> zmk: zmk_hid_mouse_scroll_set: Mouse scroll set to 0/0
[00:05:05.478,576] <dbg> zmk: zmk_hid_mouse_movement_set: Mouse movement set to 0/0
[00:05:05.486,450] <dbg> zmk: zmk_hid_mouse_movement_set: Mouse movement set to -2/0
```
