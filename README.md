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
