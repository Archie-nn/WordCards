# WordCards  1133334楊恩奇
## 功能實作 
開發重點在於引入多媒體播放功能、動態表單資料傳遞，以及實作檔案回寫機制，讓系統從單純的「唯讀瀏覽」升級為「具備儲存能力的圖書/單字卡管理系統」。

---

## 🛠️ 控制項與多媒體配置

### 1. Windows Media Player 核心整合 (`wmpSound`)
* 為了播放單字的發音音檔，專案引進了 Windows 內建的媒體播放器元件。
* **技術步驟**：在工具箱點擊右鍵 ➡️ 選擇項目 ➡️ 在 `COM 元件` 頁籤勾選 `Windows Media Player` (`wmp.dll`)。
* 將其拖曳至主表單上，命名為 `wmpSound`，並將其 `Visible` 屬性設為 `False`（隱藏後台播放）。

### 2. 動態編輯快顯表單 (`frmEditWord`)
為了解決單字修改需求，新增了第二個表單 `frmEditWord`，包含以下 UI 配置：
* **TextBox 控制項**：`txtWord`（單字）、`txtPhonogram`（音標）、`txtSoundPath`（音檔路徑）、`txtExplain`（解釋，設定 `MultiLine: True` 支援多行輸入）。
* **按鈕控制項**：`btnOK`（儲存，其 `DialogResult` 設為 `Yes`）、`btnCancel`（取消，其 `DialogResult` 設為 `No`）。

---

## 💻 核心程式碼架構擴充

### 一、 音檔播放邏輯 (Section 7.4 & 7.5)
當使用者點擊 `ListView` 中的任何一個單字，或者雙擊、按快捷鍵時，系統會自動捕捉該列對應的 `WordItem` 物件，並將音檔路徑指派給 Windows Media Player 進行發音：

```csharp
private void PlaySelectedWord()
{
    // 檢查是否有選取項目
    if (lvwWord.SelectedIndices.Count > 0)
    {
        int idx = lvwWord.SelectedIndices[0];
        WordItem item = _WordList[idx];

        // 取得絕對路徑（結合應用程式啟動路徑與相對路徑）
        string absolutePath = Path.Combine(Application.StartupPath, item.SoundPath);

        if (File.Exists(absolutePath))
        {
            wmpSound.URL = absolutePath; // 指派路徑後自動播放
        }
        else
        {
            tsslMessage.Text = $"找不到音檔：{item.SoundPath}";
        }
    }
}

// 監聽 ListView 的選取項變更事件
private void lvwWord_SelectedIndexChanged(object sender, EventArgs e)
{
    PlaySelectedWord();
} csharp```

### 二、 跨表單資料編輯傳遞 (Section 7.6)
當使用者在 `ListView` 上雙擊滑鼠時，系統會開啟 `frmEditWord` 編輯視窗，並採用「傳址（Pass by Reference）」的方式將選取的 `WordItem` 物件傳入，達到同步更新的效果。
