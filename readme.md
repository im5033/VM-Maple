# 在 VMware Workstation 17 上安裝並優化 Windows 11 以運行楓之谷

本指南將帶你一步步完成在 VMware Workstation 17 虛擬機上安裝 Windows 11 並優化顯示卡資訊，以便順利運行楓之谷。

## 前置準備
1. **準備 Windows 11 ISO**
   - 建議先使用 [tiny11builder](https://github.com/ntdevlabs/tiny11builder) 處理 ISO 檔案，精簡系統。
2. **建立虛擬機**
   - 建立好虛擬機，但暫時不要開機。

## 虛擬機設定
1. 進入 Virtual Machine Settings > Hardware，將 **Trusted Platform Module (TPM)** 移除。
2. 在 Settings > Options > Advanced，將啟動模式改為 **BIOS**（不要使用 UEFI）。

## 安裝 Windows 11 並繞過 TPM 檢查
- 若未使用 tiny11，安裝時會遇到 TPM 限制：
  1. 當出現「無法安裝 Windows 11」提示時，按下 `Shift + F10` 開啟命令提示字元。
  2. 輸入 `regedit` 開啟登錄編輯器。
  3. 前往 `HKEY_LOCAL_MACHINE\SYSTEM\Setup`，新增機碼 `LabConfig`。
  4. 在 `LabConfig` 下新增 DWORD：
     - `BypassTPMCheck`，值設為 1
     - `BypassSecureBootCheck`，值設為 1
  5. 關閉登錄編輯器，繼續安裝即可。

## 提取與修改 VMware 顯示卡驅動
1. 你可以直接下載我已經處理好的驅動檔案：[Win8.zip](./Win8.zip)
2. 或者自行依下列步驟提取與修改：
   - 在 VMware 選單點選 **VM > Install VMware Tools**。
   - 於命令提示字元執行：
     ```
     cd /d d:
     setup64.exe /a /p
     ```
   - 選擇將 VMware Tools 安裝（解壓縮）到桌面。
   - 找到 `C:\Users\你的名字\Desktop\VMware\VMware Tools\VMware\Drivers\video_wddm\Win8`，這就是顯示卡驅動。
   - 使用文字編輯器批次將檔案內容：
     - 將「VMware」替換為「NVIDIA」
     - 將「vm」替換為「nv」
     - 檔名中的「vm」也一併改為「nv」

## 安裝自訂顯示卡驅動
1. 再次點選 **VM > Install VMware Tools**，選擇自訂安裝，安裝路徑可自訂。
2. 安裝完成後，**不要立即重啟**。
3. 點選「開始」> 按住 Shift 點選「重新啟動」。
4. 進入「疑難排解 > 進階選項 > 啟動設定 > 重新啟動」，開機時按 7 禁用驅動程式強制簽章。
5. 進入桌面後，開啟「裝置管理員 > 顯示卡」，右鍵更新驅動，選擇剛剛修改過的 Win8 資料夾。

## 修改註冊表顯示卡資訊
1. 你可以直接下載我已經處理好的註冊表檔案：[vmsb.reg](./vmsb.reg)
2. 或者依下列步驟自行修改：
   - 按 `Win + R`，輸入 `regedit`。
   - 前往：
     `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000`
   - 修改以下參數：
     - `HardwareInformation.AdapterString` 改為 `NVIDIA Geforce RTX 3060`
     - `HardwareInformation.ChipType` 改為 `NVIDIA GeForce RTX 3060`
     - `HardwareInformation.DacType` 改為 `Integrated RAMDAC`
     - `ProviderName` 改為 `NVIDIA`
   - 匯出該機碼為 `.reg` 檔，放在桌面，方便日後一鍵修改。
3. 可建立一個 BAT 檔，內容如下，開機自動套用：
   ```
   regedit /s C:\Users\你的使用者名\Desktop\XXXXXX.reg
   ```
   將 BAT 檔放到 `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup` 或執行 `shell:Startup`。

## 最後優化虛擬機 VMX 設定
1. 關閉虛擬機，找到 VM 的 `.vmx` 檔，用記事本開啟。
2. 在檔案最下方新增：
   ```
   SMBIOS.reflectHost="TRUE"
   hypervisor.cpuid.v0="FALSE"
   ```
3. 若遊戲有問題，可再加上：
   ```
   mks.enableDX12Renderer = "FALSE"
   mks.enableDX11Renderer = "TRUE"
   ```

---

如有問題，歡迎回報或討論！ 