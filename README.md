<pre>

RAX3000M 使用的是 聯發科（MediaTek）MT7981B 
  
1. 【推薦】使用 QEMU 虛擬平台來測試你的固件功能
如果你主要是想測試 .config 編譯出來的軟體、套件或腳本是否正常運作，建議不要執著於模擬「RAX3000M」這個硬體。
做法： 在 make menuconfig 時，將 Target System 改選為 QEMU ARM Virt。
優點： 這種模式不需要 U-Boot，QEMU 可以直接引導 kernel。你可以把你在 RAX3000M 想要用的套件清單放進去編譯，測試軟體邏輯。
啟動指令範例：
bash
qemu-system-aarch64 -nographic -M virt -cpu cortex-a53 -m 512 \
  -kernel openwrt-armvirt-64-Image-initramfs
</pre>

<pre>
  
  qemu-system-aarch64 \
    -M virt \
    -cpu cortex-a53 \
    -smp 2 \
    -m 512 \
    -kernel <您的內核檔案路徑> \
    -append "root=/dev/vda rootwait console=ttyAMA0" \
    -drive file=<您的emmc鏡像路徑>,format=raw,if=virtio \
    -net nic,model=virtio \
    -net user,hostfwd=tcp::8080-:80 \
    -nographic

    關鍵參數說明：
-M virt: 使用通用虛擬化平台，這是模擬 ARM 路由器的標準做法。
-drive ...,if=virtio: 將您的 eMMC 鏡像模擬為虛擬磁碟。
-net user,hostfwd=tcp::8080-:80: 非常重要！ 這會將虛擬機的 80 埠（LuCI 界面）映射到您電腦的 8080 埠。
-nographic: 讓輸出直接顯示在您的終端機上。

qemu-system-aarch64 -M virt -cpu cortex-a53 -m 512 \
    -drive file=<変換したファイル名>.img,format=raw,if=virtio \
    -net nic -net user \
    -nographic

</pre>

<pre>
  以下是该设备的启动逻辑与结构分析：
1. 启动结构并非完全“固定”
虽然出厂和大多数第三方固件（如 OpenWrt、ImmortalWrt）默认使用 U-Boot，但启动链实际上分为几个阶段，你可以根据需求调整：
ROM Code (BL1): 固化在 MT7981 芯片内部，不可更改。
BL2 (Preloader): 负责初始化 DDR。
BL31 (ATF/TF-A): 负责安全环境和电源管理。
BL33 (U-Boot): 這是你提到的“固定结构”部分。 
OpenWRT
OpenWRT
 +1
2. 为什么通常“一定是” U-Boot？
在 RAX3000M 这种基于 MediaTek Filogic 平台的设备上，U-Boot 被广泛使用是因为：
引导兼容性： 它能正确加载 Linux 内核（uImage/itb 格式）并传递设备树（DTB）。
救砖功能： 第三方“不死 U-Boot”（如 hanwckf 的版本）提供了 Web 刷机界面，即使 Linux 系统崩溃，也能通过 U-Boot 恢复。
分区管理： 它定义了 NAND/eMMC 的分区表结构，决定了内核和 rootfs 存放在哪。 
3. 可选的替代结构
如果你想打破“固定”结构，理论和实践上有以下可能：
直接引导： 一些极简嵌入式系统可以跳过 U-Boot，由 BL2 直接跳转到精简后的内核，但这对 RAX3000M 的复杂驱动初始化非常不利。
UEFI (EDK2): 部分开发者正在为 MT798x 平台适配 UEFI 引导，但这在路由器固件中极少见。
OEM 限制： 移动原厂 U-Boot 带有签名校验，如果你想刷第三方 Linux（OpenWrt），必须更换为支持校验解除的第三方 U-Boot。
</pre>

4. 深度開發：本地 Clone + Quilt 補丁
3. 雲端編譯：GitHub Actions (與 .yml 結合)
2. 修改「特定軟體」：使用 SDK
如果你只想改某一個 App（例如 vlmcsd 或某個驅動）的代碼，而不想編譯整個作業系統
1. 修改「配置」：使用 Image Builder (推薦)
如果你只是想修改預設的網路設定、預裝外掛、或是加入自己的 .sh 腳本，不需要下載完整源碼



1. 核心在於 .config 檔案
在 env 區段可以看到這行：
CONFIG_FILE: .config
這就是 OpenWrt 的「靈魂」。所有的架構選擇（例如是 x86_64、MT7621 還是 ARM）、插件選擇、內核設定，全部都儲存在這個 .config 檔案裡。
流程中發生了什麼：腳本在 Load custom configuration 步驟中，會執行 mv $CONFIG_FILE openwrt/.config。
這意味著：你必須在你的 GitHub 倉庫根目錄下放一個已經設定好的 .config 檔案。如果你沒放，編譯器就會使用 Lean LEDE 源碼 的默認設定。
2. diy-part2.sh 的輔助修改
你在腳本中看到的 DIY_P2_SH: diy-part2.sh 經常被用來透過指令直接修改架構或 IP。
許多人會在 diy-part2.sh 裡寫 sed 指令，例如：
sed -i 's/192.168.1.1/192.168.10.1/g' package/base-files/files/bin/config_generate
甚至可以用指令強制寫入特定的架構參數到 .config 中。
3. make defconfig 自動補全
在 Download package 步驟中，腳本執行了：
make defconfig
這個指令的作用是：檢查當前的 .config 檔案，如果缺少必要的相依性或架構設定，它會根據你提供的基礎設定自動補全剩餘的部分。




<pre>
  我想玩Uboot/ Linux的網絡編程/還有一些零散的内存技術  /還有就是我要用rust寫軟件
這是一個非常紮實的技術路徑。如果你想從 U-Boot（引導加載程序）、Linux 網絡內核 到 Rust 系統編程 全線打通，OpenWrt 確實是最佳的「實驗室」。
以下是你針對這三個方向可以深入研究切入點：
1. U-Boot 與嵌入式底層
U-Boot 是硬體啟動後運行的第一個程式，負責初始化內存並引導內核。
研究重點：
啟動流程：研究 SPL (Secondary Program Loader) 如何將內核從 Flash 搬運到 RAM。
環境變量 (Env)：了解如何通過 U-Boot 命令行修改啟動參數（例如 bootargs）。
救磚機制：學習如何編寫 httpd 或 tftp 模式，在不進入系統的情況下刷機。
實踐建議：去 GitHub - u-boot 查看針對你設備的 board 支持文件，嘗試自己編譯一個能啟動的 u-boot.bin。
2. Linux 網絡編程與內存技術
既然你在用 Lean 的固件，你可以直接觀察那些「激進」的優化是如何操作內核的。
網絡編程 (Kernel Space)：
Netfilter/Iptables：這是 OpenWrt 插件的核心。研究如何寫一個內核模組，攔截或修改經過路由器的封包。
XDP (Express Data Path)：如果你想追求極致性能，這是目前 Linux 網絡編程的最前沿技術，可以在數據包進入協議棧之前就進行處理。
內存技術 (Memory Management)：
DMA (Direct Memory Access)：研究網卡驅動如何不經過 CPU 搬運數據到內存，這在高性能路由器中至關重要。
Zram：OpenWrt 常用的壓縮內存技術，非常適合在內存小的嵌入式設備上練習。
3. 用 Rust 進行開發
Rust 是替代 C 語言進行系統編程的最佳選擇，特別適合編寫高效且安全的路由器插件。
Cross-Compilation (交叉編譯)：
你需要在 PC 上安裝 Rust 並添加目標平台的工具鏈（例如 mips-unknown-linux-musl 或 aarch64-unknown-linux-musl）。
你可以參考 OpenWrt SDK 官方文檔 來編譯你的 Rust 程序。
Rust 插件開發：
代碼實例：可以嘗試用 Rust 寫一個簡單的 DNS 轉發器或流量監控工具。
優勢：Rust 的內存安全特性（Ownership/Borrow Checker）能讓你免於處理嵌入式開發中最頭痛的 Memory Leak 和 Segmentation Fault。
下一步建議
既然你已經會用 SSH 手動配置 .config，你可以：
在 make menuconfig 中勾選 Build the OpenWrt SDK。這會生成一個獨立的開發環境，讓你在不重新編譯整個固件的情況下，快速編譯你寫的 Rust 程式。
搜尋並閱讀 The Rust Embedded Book，這是學習 Rust 進行底層開發的權威指南。
你需要我為你提供一個簡單的 Rust 交叉編譯到 OpenWrt 的 Hello World 流程嗎？



還有什麽好玩的幫我補充一下
既然你已經觸及了 U-Boot、內核網絡 和 Rust，這已經是路由器玩家的「天花板」級別了。要在這個基礎上玩得更深，這裡有幾個非常硬核且有趣的進階方向：
1. 玩轉 eBPF (Extended Berkeley Packet Filter)
這是目前 Linux 內核最火的技術。它允許你在不重新編譯內核的情況下，將自定義的代碼（可以用 Rust 寫）直接注入內核運行。
怎麼玩：在 OpenWrt 上實現一個「零延遲」的防火牆或監控工具。你可以用 Rust 的 Aya 庫 來編寫 eBPF 程序，直接攔截網卡上的原始數據包。
趣味點：這比傳統的 iptables 快得多，是現代高性能網絡開發的必修課。
2. 虛擬化與容器 (LXC/Docker)
如果你編譯的是 X86 或高性能 ARM（如 RK3588），不要只把它當路由器。
怎麼玩：在 OpenWrt 裡跑 LXC (Linux Containers)。你可以同時運行一個 Debian 桌面、一個 Home Assistant 自動化中心和一個 Rust 寫的後台服務，彼此內存隔離但共享內核。
內存技術：研究 KSM (Kernel Same-page Merging)，看看內核如何把多個容器中相同的內存頁合併，以節省路由器的內存。
3. 硬體接口通信 (GPIO / I2C / SPI)
路由器不只有網口。如果你有開發板或帶有擴展引腳的設備，可以玩硬體控制。
怎麼玩：用 Rust 寫一個驅動，通過 GPIO 控制風扇轉速（根據 CPU 溫度），或者外接一個 OLED 小螢幕顯示當前網速和內存佔用。
工具推薦：參考 rust-embedded/embedded-hal 庫。
4. 繞過內核協議棧 (DPDK)
如果你覺得 Linux 內核處理網絡包還是太慢（因為有上下文切換的開銷），你可以嘗試 用戶態網絡。
怎麼玩：研究 DPDK (Data Plane Development Kit)。它直接接管網卡，把數據包直接拉到用戶態（User Space）處理，完全跳過內核。
挑戰：這需要極強的內存管理能力，非常適合練習 Rust 的指針與內存安全特性。
5. 深入研究 OpenWrt 的 「Procd」 守護進程
這是 OpenWrt 的 init 系統（類似 PC 上的 systemd）。
怎麼玩：研究如何用 Rust 寫一個自啟動服務，並與 ubus（OpenWrt 的內部總線）通信。你可以開發一個能被官方 LuCI 界面調用的 Rust 後端，實現前後端完全分離。
6. 自製 U-Boot 啟動動畫或「救磚」網頁
怎麼玩：修改 U-Boot 源碼，在啟動時通過 GPIO 點亮特定編碼的 LED 燈，或者在 U-Boot 階段就實現一個簡易的 Web 服務器用於上傳固件。
總結建議：
你可以先從 「用 Rust 寫一個基於 ubus 的系統監控插件」 開始。這會讓你同時接觸到：
OpenWrt 的進程間通信 (ubus)
Rust 的交叉編譯
Linux 的 /proc 虛擬文件系統（內存與 CPU 數據來源）
你手上現在有哪款具體的硬體設備？（這決定了你是玩 eBPF 還是玩 GPIO 擴展）。




  內存部分：讀取 /proc/meminfo 並解析。
網絡部分：使用 pnet 庫抓取網卡上的原始數據包（Raw Sockets），統計每個 IP 的流量。
挑戰：嘗試將這個程序交叉編譯為你 LEDE 固件對應的架構（如 MIPS 或 ARM）。
</pre>




**English** | [中文](https://p3terx.com/archives/build-openwrt-with-github-actions.html)

# Actions-OpenWrt

[![LICENSE](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square&label=LICENSE)](https://github.com/P3TERX/Actions-OpenWrt/blob/master/LICENSE)
![GitHub Stars](https://img.shields.io/github/stars/P3TERX/Actions-OpenWrt.svg?style=flat-square&label=Stars&logo=github)
![GitHub Forks](https://img.shields.io/github/forks/P3TERX/Actions-OpenWrt.svg?style=flat-square&label=Forks&logo=github)

A template for building OpenWrt with GitHub Actions

## Usage

- Click the [Use this template](https://github.com/P3TERX/Actions-OpenWrt/generate) button to create a new repository.
- Generate `.config` files using [Lean's OpenWrt](https://github.com/coolsnowwolf/lede) source code. ( You can change it through environment variables in the workflow file. )
- Push `.config` file to the GitHub repository.
- Select `Build OpenWrt` on the Actions page.
- Click the `Run workflow` button.
- When the build is complete, click the `Artifacts` button in the upper right corner of the Actions page to download the binaries.

## Tips

- It may take a long time to create a `.config` file and build the OpenWrt firmware. Thus, before create repository to build your own firmware, you may check out if others have already built it which meet your needs by simply [search `Actions-Openwrt` in GitHub](https://github.com/search?q=Actions-openwrt).
- Add some meta info of your built firmware (such as firmware architecture and installed packages) to your repository introduction, this will save others' time.

## Credits

- [Microsoft Azure](https://azure.microsoft.com)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [coolsnowwolf/lede](https://github.com/coolsnowwolf/lede)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [Mattraks/delete-workflow-runs](https://github.com/Mattraks/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)
- [peter-evans/repository-dispatch](https://github.com/peter-evans/repository-dispatch)

## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/main/LICENSE) © [**P3TERX**](https://p3terx.com)
