---
title: "棋牌遊戲開發| 遊戲伺服器篇"
date: 2020-01-26T00:00:00+08:00
# hero: /posts/introduction/hero.svg
description: "本篇內容為棋牌類型為主的遊戲伺服器架構心得筆記，涉及高併發、玩家佇列與自動佈署等。"
menu:
  sidebar:
    name: "棋牌遊戲開發| 遊戲伺服器篇"
    identifier: 58poker-backend
    parent: "experiences-2019"
    weight: 2020026002
---

---

### 前言

在系列文的第一篇 <a href="/projects/20200126-1-pokergame-client">棋牌遊戲開發| 前端篇</a> 中我們完整介紹了個遊戲前端的技術框架，接下來就要開始進入到後端的範疇了。  

由於遊戲後端涵蓋許多服務，包含玩家匹配、遊戲邏輯、PVE 系統等。本篇會先盡可能地將所有的服務都逐一提過，後續在針對其中的某些項目做進一步的介紹。  

另外，遊戲伺服器的僅負責所有與遊戲流程相關的業務需求；換句話說，玩家在牌桌結束後產生的對應需求 (e.g. 籌碼流動、點數轉移等)，均不在此處理。

---

### 遊戲後端技術框架結構圖

{{< img src="https://imgur.com/TW232Lm.png" align="center" >}}

#### 資料庫選擇

  - Redis：儲存需要在所有內部結點上共享的資料 (e.g. 玩家、牌局資料等)，以及需要 Real-time 支持的業務需求 (e.g. 正在進行遊戲中的牌局)。
  - MySQL：所有需要被永久保存的資料會在條件觸發後才進行寫入 (e.g. 牌局結算後)，減少在操作上的交換時間。

#### 通訊方式選擇

  - TCP：與外部服務 (e.g. Consul、Nats、API等) 採用短連接方式進行資料交換。
  - WebSocket/MQTT：透過 WebSocket 與遊戲前端建立連線後使用 MQTT 作為通訊格式。
  - RPC：內部結點 (e.g. GameServer、PacerServer、Gateway) 溝通時的通訊方式。
