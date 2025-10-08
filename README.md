# LuckFox Pico Zero 操作步驟

## 編譯檔案

* 安裝依賴

    ```
    sudo apt-get install repo git ssh make gcc gcc-multilib g++-multilib module-assistant expect g++ gawk texinfo libssl-dev bison flex fakeroot cmake unzip gperf autoconf device-tree-compiler libncurses5-dev pkg-config bc python-is-python3 passwd openssl openssh-server openssh-client vim file cpio rsync
    ```

* 下載 sdk

    ```
    cd ~
    git clone https://github.com/LuckfoxTECH/luckfox-pico.git
    ```

* 設定環境

    > 可以忽略 `.bash_profile` 錯誤 `sed: can't read /home/esa/.bash_profile: No such file or directory`

    ```
    cd ~/luckfox-pico/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/
    source env_install_toolchain.sh
    ```

    ```
    logout
    login
    ```

* 設定要編譯的開發版

    ```
    cd ~/luckfox-pico
    ./build.sh lunch
    ```

* 修改 DTS (Device Tree Source) [官網介紹 (強烈推薦先讀過再操作)](https://wiki.luckfox.com/zh/Luckfox-Pico-Zero/Device-Tree)

    * 打開 `dts_config`

        ```
        vim ~/luckfox-pico/config/dts_config
        ``` 

    * 添加到最後面
        ```


        
        /**********I2C**********/
        &i2c4 {
                status = "okay";
                pinctrl-0 = <&i2c4m1_xfer>;
                clock-frequency = <100000>;

                afe4404: afe4404@58 {
                        compatible = "ti,afe4404";
                        reg = <0x58>;
                };
        };
        ```
* BoardConfig_IPC

    ```
    vim ~/luckfox-pico/project/cfg/BoardConfig_IPC/BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Zero-IPC.mk
    ```

    * 設定預設 WiFi
        

        * 修改 `BoardConfig_IPC` 參數 `LF_WIFI_PASSWD` 和 `LF_WIFI_SSID`

            ```
            export LF_WIFI_SSID="Your wifi ssid"
            export LF_WIFI_PSK="Your wifi password"
            ```
    
    * ~~自訂 luckfox-config luckfox.cfg~~

        > 不用改

        * ~~建立系統檔案結構 `custom-overlay-luckfox-config`~~

            ```
            vim ~/luckfox-pico/project/cfg/BoardConfig_IPC/overlay/custom-overlay-luckfox-config/etc/luckfox.cfg
            ```

        * ~~設定以下參數~~

            ```
            SPI0_M0_CS_ENABLE=1
            SPI0_M0_MODE=1
            CSI_ENABLE=0
            CSI_UNITE_ENABLE=0
            I2C4_M1_STATUS=1
            I2C4_M1_SPEED=100000
            ```

        ---

        * ~~修改 `BoardConfig_IPC` 參數 `RK_POST_OVERLAY`~~

            * ~~原始~~

                ```
                # declare overlay directory
                export RK_POST_OVERLAY="overlay-luckfox-config overlay-luckfox-buildroot-init overlay-luckfox-buildroot-shadow"
                ```

            * ~~添加 `custom-overlay-luckfox-config`~~

                ```
                # declare overlay directory
                export RK_POST_OVERLAY="overlay-luckfox-config overlay-luckfox-buildroot-init overlay-luckfox-buildroot-shadow custom-overlay-luckfox-config"
                ```

* kernelconfig

    ```
    ./build.sh kernelconfig
    ```

    * AFE4404 驅動

        > 找到 `AFE4404` 按 `y`

        ```
        │ Symbol: AFE4404 [=y]                                                                                            │
        │ Type  : tristate                                                                                                │
        │ Defined at drivers/iio/health/Kconfig:24                                                                        │
        │   Prompt: TI AFE4404 heart rate and pulse oximeter sensor                                                       │
        │   Depends on: IIO [=y] && I2C [=y]                                                                              │
        │   Location:                                                                                                     │
        │     -> Device Drivers                                                                                           │
        │       -> Industrial I/O support (IIO [=y])                                                                      │
        │         -> Health Sensors                                                                                       │
        │ (1)       -> Heart Rate Monitors                                                                                │
        │ Selects: REGMAP_I2C [=y] && IIO_BUFFER [=y] && IIO_TRIGGERED_BUFFER [=y] 
        ```

* buildrootconfig

    ```
    ./build.sh buildrootconfig
    ```

    * vim

        ```
        │ Symbol: BR2_PACKAGE_VIM [=y]                                                                                    │
        │ Type  : bool                                                                                                    │
        │ Prompt: vim                                                                                                     │
        │   Location:                                                                                                     │
        │     -> Target packages                                                                                          │
        │ (7)   -> Text editors and viewers                                                                               │
        │   Defined at package/vim/Config.in:1                                                                            │
        │   Depends on: BR2_USE_MMU [=y] && BR2_USE_WCHAR [=y] && BR2_PACKAGE_BUSYBOX_SHOW_OTHERS [=y]                    │
        │   Selects: BR2_PACKAGE_NCURSES [=y]
        ```

    * 安裝 Python 套件

        * BR2_PACKAGE_PYTHON_PIP (python-pip)
        
            > 正式環境不要裝 pip，直接裝全部套件

            ```
            │ Symbol: BR2_PACKAGE_PYTHON_PIP [=y]                                                                             │
            │ Type  : bool                                                                                                    │
            │ Prompt: python-pip                                                                                              │
            │   Location:                                                                                                     │
            │     -> Target packages                                                                                          │
            │       -> Interpreter languages and scripting                                                                    │
            │         -> python3 (BR2_PACKAGE_PYTHON3 [=y])                                                                   │
            │ (5)       -> External python modules                                                                            │
            │   Defined at package/python-pip/Config.in:1                                                                     │
            │   Depends on: BR2_PACKAGE_PYTHON3 [=y]                                                                          │
            │   Selects: BR2_PACKAGE_PYTHON_SETUPTOOLS [=y] && BR2_PACKAGE_PYTHON3_SSL [=y]
            ```

        * BR2_PACKAGE_PYTHON_WEBSOCKETS (python-websockets)

            ```
            │ Symbol: BR2_PACKAGE_PYTHON_WEBSOCKETS [=y]                                                                      │
            │ Type  : bool                                                                                                    │
            │ Prompt: python-websockets                                                                                       │
            │   Location:                                                                                                     │
            │     -> Target packages                                                                                          │
            │       -> Interpreter languages and scripting                                                                    │
            │         -> python3 (BR2_PACKAGE_PYTHON3 [=y])                                                                   │
            │ (1)       -> External python modules                                                                            │
            │   Defined at package/python-websockets/Config.in:1                                                              │
            │   Depends on: BR2_PACKAGE_PYTHON3 [=y]                                                                          │
            │   Selects: BR2_PACKAGE_PYTHON3_ZLIB [=y] && BR2_PACKAGE_PYTHON3_SSL [=y]
            ```

    * 更改預設 shell

        > 只能用 `busybox` 或 `bash`，不能用 `zsh`，否則開機掛載會報錯 `/etc/init.d/S20linkmount:14: parse error near `}'`

        * bash
    
            ```
            System configuration  --->
            /bin/sh (??)  --->
            (X) bash
            ```
    
    * ~~Wi-Fi 相關~~

        * ~~wpa_supplicant~~

            ```
            │ Symbol: BR2_PACKAGE_WPA_SUPPLICANT [=y]                                                                         │
            │ Type  : bool                                                                                                    │
            │ Prompt: wpa_supplicant                                                                                          │
            │   Location:                                                                                                     │
            │     -> Target packages                                                                                          │
            │ (1)   -> Networking applications                                                                                │
            │   Defined at package/wpa_supplicant/Config.in:1                                                                 │
            │   Depends on: BR2_USE_MMU [=y]                                                                                  │
            │   Selects: BR2_PACKAGE_LIBOPENSSL_ENABLE_DES [=y] && BR2_PACKAGE_LIBOPENSSL_ENABLE_MD4 [=y]                     │
            │   Selected by [n]:                                                                                              │
            │   - BR2_PACKAGE_CONNMAN_WIFI [=n] && BR2_PACKAGE_CONNMAN [=n]
            ```
        * ~~dhcpcd~~

            ```
            │ Symbol: BR2_PACKAGE_DHCPCD [=y]                                                                                 │
            │ Type  : bool                                                                                                    │
            │ Prompt: dhcpcd                                                                                                  │
            │   Location:                                                                                                     │
            │     -> Target packages                                                                                          │
            │ (1)   -> Networking applications                                                                                │
            │   Defined at package/dhcpcd/Config.in:4                                                                         │
            │   Depends on: BR2_TOOLCHAIN_HEADERS_AT_LEAST_3_1 [=y]
            ```
        
        * ~~dnsmasq~~

            ```
            │ Symbol: BR2_PACKAGE_DNSMASQ [=y]                                                                                │
            │ Type  : bool                                                                                                    │
            │ Prompt: dnsmasq                                                                                                 │
            │   Location:                                                                                                     │
            │     -> Target packages                                                                                          │
            │ (1)   -> Networking applications                                                                                │
            │   Defined at package/dnsmasq/Config.in:1                                                                        │
            │   Depends on: BR2_USE_MMU [=y]                                                                                  │
            │   Selected by [n]:                                                                                              │
            │   - BR2_PACKAGE_LIBVIRT_DAEMON [=n] && BR2_PACKAGE_LIBVIRT [=n] && BR2_INSTALL_LIBSTDCPP [=y]
            ```

* 編譯系統

    ```
    ./build.sh
    ```


## 燒錄

> ⚠️線很重要，有些線可以進正常系統，但進不了燒錄模式
> ⚠️線很重要，有些線可以進正常系統，但進不了燒錄模式
> ⚠️線很重要，有些線可以進正常系統，但進不了燒錄模式

- [官網教學](https://wiki.luckfox.com/zh/Luckfox-Pico/Luckfox-Pico-SD-Card-burn-image)
- [SocToolKit](https://files.luckfox.com/wiki/Luckfox-Pico/Software/SocToolKit.zip)

## SSH 登入

> ip 可能是 `172.32.0.93 (busybox、bash)` 或 `172.32.0.70 (zsh)`
> [官網教學](https://wiki.luckfox.com/zh/Luckfox-Pico/SSH-Telnet-Login/#luckfox-picopico-mini-ab-%E7%99%BB%E5%BD%95)
> 註：adb 連線比 ssh server 更快開啟，有問題可以用 adb 來 debug

## ~~設定 PIN~~

> 不用改

* 關 CSI、開 I2C4_M1 (100000 MHz)

    ```
    luckfox-config
    ```

* 重開機

    ```
    reboot
    ```
* 確認 Sensor 狀態

    ```
    ls /sys/class/i2c-adapter/
    : i2c-4
    i2cdetect -y 4
    : '
        0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:          -- -- -- -- -- -- -- -- -- -- -- -- --
    10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    50: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- --
    60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    70: -- -- -- -- -- -- -- --
    '
    ls /sys/bus/iio/devices/
    : iio:device0  iio:device1
    cat /sys/bus/iio/devices/iio:device1/name
    : afe4404
    ```

## 連接裝置的使用說明

* 前往查看[連接裝置的使用說明](./devices/README.md)