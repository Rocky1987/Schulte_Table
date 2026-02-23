# 🧠 Schulte Table 舒爾特方格訓練

一款專注力與反應速度訓練工具，基於經典的舒爾特方格（Schulte Table）設計。玩家需要按照數字順序（由小到大）依序點擊方格中的數字，藉此鍛鍊視覺搜尋能力與注意力集中度。

## ✨ 功能特色

- **可調式方格大小** — 支援 3×3、4×4、5×5、6×6 四種尺寸
- **精確計時器** — 點擊數字 1 自動開始計時，完成後停止，精確至毫秒
- **難度分級**
  - 🟢 **初級**：按過的按鈕會變暗，方便辨識已完成項目
  - 🔴 **進階**：按過的按鈕不會變暗，增加記憶與搜尋難度
- **隨機顏色模式** — 開啟後每個數字會隨機顯示不同顏色，增加視覺干擾
- **錯誤回饋** — 點擊錯誤數字時按鈕會產生抖動動畫提示
- **一鍵重新開始** — 隨時可以重新洗牌開始新一輪遊戲

## 🛠️ 技術棧

| 技術 | 用途 |
|------|------|
| [Vue 3](https://vuejs.org/) | 前端框架（Composition API + `<script setup>`） |
| [TypeScript](https://www.typescriptlang.org/) | 型別安全 |
| [Vite](https://vitejs.dev/) | 建置工具與開發伺服器 |
| [Tailwind CSS](https://tailwindcss.com/) | 樣式框架 |

## 📦 安裝與啟動

```bash
# 安裝依賴
npm install

# 啟動開發伺服器
npm run dev

# 建置生產版本
npm run build

# 預覽生產版本
npm run preview
```

## 🎮 遊戲說明

1. 從左側面板選擇方格大小（3×3 ~ 6×6）
2. 選擇難度（初級 / 進階）
3. 可選擇開啟「隨機顏色」模式增加挑戰
4. 點擊數字 **1** 開始遊戲，計時器自動啟動
5. 依照數字順序依序點擊，直到點完所有數字
6. 完成後計時器停止，顯示完成時間
7. 點擊 **Restart** 重新開始

## 📁 專案結構

```
Schulte_Table/
├── index.html            # 入口 HTML
├── package.json          # 專案設定與依賴
├── vite.config.ts        # Vite 設定
├── tailwind.config.js    # Tailwind CSS 設定
├── tsconfig.json         # TypeScript 設定
└── src/
    ├── App.vue           # 主應用元件（遊戲邏輯與 UI）
    ├── main.ts           # 應用程式入口
    └── style.css         # 全域樣式與 Tailwind 引入
```
