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
}
```

### 二、 跨表單資料編輯傳遞 (Section 7.6)
當使用者在 `ListView` 上雙擊滑鼠時，系統會開啟 `frmEditWord` 編輯視窗，並採用「傳址（Pass by Reference）」的方式將選取的 `WordItem` 物件傳入，達到同步更新的效果。

**1. frmEditWord.cs 內部結構**
```csharp
public partial class frmEditWord : Form
{
    private WordItem _item; // 儲存外部傳入的單字物件引用

    public frmEditWord(WordItem item)
    {
        InitializeComponent();
        _item = item; // 建立引用連結
    }

    private void frmEditWord_Load(object sender, EventArgs e)
    {
        // 將原本的資料填入 UI 欄位中
        txtWord.Text = _item.Word;
        txtPhonogram.Text = _item.Phonogram;
        txtSoundPath.Text = _item.SoundPath;
        txtExplain.Text = _item.Explain;
    }

    private void btnOK_Click(object sender, EventArgs e)
    {
        // 使用者點擊儲存，將 UI 修改後的值寫回該物件
        _item.Word = txtWord.Text;
        _item.Phonogram = txtPhonogram.Text;
        _item.SoundPath = txtSoundPath.Text;
        _item.Explain = txtExplain.Text;
        
        this.DialogResult = DialogResult.Yes; // 關閉表單並回傳 Yes
    }
}
```

**2. 主表單 frmTSVFile.cs 觸發雙擊編輯**
```csharp
private void lvwWord_MouseDoubleClick(object sender, MouseEventArgs e)
{
    if (lvwWord.SelectedIndices.Count > 0)
    {
        int idx = lvwWord.SelectedIndices[0];
        
        // 將對應的 WordItem 傳入新表單
        frmEditWord editForm = new frmEditWord(_WordList[idx]);
        DialogResult result = editForm.ShowDialog(this);

        // 如果使用者在編輯視窗按下了「儲存」
        if (result == DialogResult.Yes)
        {
            UpdateListView();    // 重新整理主畫面的 ListView
            PlaySelectedWord();  // 重新播放該單字確認發音
            tsslMessage.Text = $"單字 [{_WordList[idx].Word}] 已成功修改。";
        }
    }
}
```

### 三、 資料儲存與還原回寫機制 
為了將記憶體中修改過的單字永久儲存，必須將 WordItem 重新格式化為帶有 \t (Tab鍵) 的字串，並利用 StreamWriter 覆寫回原檔案。

### 📖 進階操作步驟指引
- 點擊發音：載入單字檔後，在 ListView 清單中滑鼠單擊任意單字，系統會透過後台的 wmpSound 自動尋找對應的 .mp3 檔案並播放發音。

- 雙擊彈窗修改：在想要修改的單字上雙擊滑鼠左鍵，會跳出強制回應的「修改單字」視窗。

- 資料同步：在修改視窗變更內容後點擊「確定」，主視窗的 ListView 與下方的狀態列會即時連動更新。

- 存檔：點選功能表的儲存選項，即可將整份經過編輯、校正後的單字清單，完整寫回指定的 TSV 文字檔中。
