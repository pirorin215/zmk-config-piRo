# piRo

## 目次
  - [1. ローカルビルド](#1-ローカルビルド)
    - [1-1. 環境設定](#1-1-環境設定)
    - [1-2. ビルド](#1-2-ビルド)

## 1. ローカルビルド

### 1-1. 環境設定

```
cd /Users/yoshi/zmk/app
git clone https://github.com/pirorin215/zmk-config-piRo.git
cp zmk-config-piRo/config/west.yml /Users/yoshi/zmk/app/
west update

sudo apt install ninja-build
```

### 1-2. ビルド

```
source ~/zmk/.venv/bin/activate
rm -rf build

west build -d build/piRo_reset -p -b seeeduino_xiao_ble -- -DSHIELD="settings_reset"
west build -d build/piRo_L     -p -b seeeduino_xiao_ble -- -DZMK_CONFIG="/Users/yoshi/zmk/app/zmk-config-piRo/config/" -DSHIELD="piRo_L"
west build -d build/piRo_R     -p -b seeeduino_xiao_ble -- -DZMK_CONFIG="/Users/yoshi/zmk/app/zmk-config-piRo/config/" -DSHIELD="piRo_R"

```
