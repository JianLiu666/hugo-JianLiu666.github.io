---
title: "棋牌遊戲開發| 前端篇"
date: 2020-01-24T00:00:00+08:00
hero: /posts/experiences/2019/game-platform-frontend/image/banner.jpeg
description: "本篇內容為棋牌類型為主的網頁遊戲前端架構心得筆記，涉及遊戲引擎、狀態機與幀同步等。"
menu:
  sidebar:
    name: "棋牌遊戲開發| 前端篇"
    identifier: 58poker-frontend
    parent: "experiences-2019"
    weight: 20200124001
---

---

### 前言

本篇為棋牌遊戲開發系列文的第一篇，主要是想把自己經手一年從零到有的專案重點記錄下來，也藉此機會讓自己重新審視過整個專案的架構。

這個系列主要會先以前後端兩篇來簡述整個棋牌專案的架構，接著再以獨立的篇幅逐步介紹其中細節，例如 PVE 系統、高併發處理或是數據追蹤等功能實現的重點紀錄。

---

### 遊戲前端技術框架結構圖

{{< img src="/posts/experiences/2019/game-platform-frontend/image/mindmap.svg" align="center" >}}

#### 通訊方式選擇

  - HttpRequest：大部分與外部服務 (e.g. 會員系統) 溝通時採用的傳輸協定。
  - WebSocket/MQTT：透過 WebSocket 與後端伺服器建立連線後使用 MQTT 作為通訊格式。

#### 音訊模組
   
  - AudioSource：包含背景音樂、音效等。
  - GameVoice：包含多人群聊、錄製發話等。

#### 動畫模組
   
  - Base Action：給定一系列運動軌跡組合而成的單次動畫演示 (e.g. 洗牌、派牌等)。
  - Schedule Action：泛指有固定頻率與運動軌跡的動畫演示 (e.g. 輪播牆、跑馬燈等)。
  - Spine/DragonBones：載入利用第三方工具實現的動畫演示 (e.g. 圖示特效)。

#### 操作模組
   
  - 智能拖曳：協助玩家在透過單指觸控實現拖曳需求時的智能篩選 (e.g. 依當前情境從手牌中篩選有利出牌組合)。
  - 碰撞回饋：概括所有在玩家操作/視覺上造成影響流暢度的部分 (e.g. 按鈕點擊回饋、滑動經過物件時的回饋等)。

---

### 遊戲前端系統架構圖

{{< img src="https://imgur.com/VwE9JcX.png" align="center" >}}

如圖所示，整個前端架構主要是由一個遊戲大廳 (Game Lobby) 與旗下遊戲 (e.g. Dice、Poker等) 所組成，我們提供客戶自主選擇要串接的渠道，例如遊戲大廳或是直接訪問遊戲。

圖中每個圖示代表一個獨立專案，如此一來當A遊戲未來在實現版本升級時，其餘遊戲不會連帶受到影響。

---

### 遊戲前端遊工作流程

{{< img src="https://imgur.com/ldTUMTi.png" align="center" >}}

該圖為遊戲前端在進行一局遊戲時的時序圖，內容著重在建立牌局時的渲染步驟以及與後端溝通的過程。

  - 當畫面載入時會先將必要檔案優先處理，後續批次動態載入其餘資源，盡量降低玩家在網路速度較差時的等待時間。

