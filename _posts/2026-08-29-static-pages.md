---
layout: post
title: "從 GitHub Pages 到 Cloudflare Pages：嘗試新工具與雙平台建置"
description: "分享首次使用 Cloudflare Pages 的契機，並紀錄如何利用 Jekyll 多設定檔機制，讓 GitHub Pages 與 Cloudflare Pages 同時並存並各自擁有獨立的 Sitemap。"
date: 2026-08-29
tags: [職涯, 隨筆, 生活心得, Cloudflare Pages, GitHubPages]
project: "職涯、隨筆與生活心得"
---

# 從 GitHub Pages 到 Cloudflare Pages：嘗試新工具與雙平台建置

最近在參加 Google 數位人才探索計畫的課程時，看到有人提到：如果只是想部署靜態網頁，除了 GitHub Pages 之外，也可以試試看 **Cloudflare Pages**。

巧合的是在上課前兩天，我才剛註冊 Cloudflare 帳號，正在考慮買一個網域。當時符合預算的待選結尾名單有 `.com`、`.fyi`、`.page` 和 `.link`。在查找 Cloudflare Pages 資料時發現，Cloudflare Pages 提供預設的免費子網域結尾是 `.page.dev`，剛好是待選名單之一，也滿符合我網站的特性。

過去我只有使用過 GitHub Pages 的經驗，當下便馬上詢問 AI 實作的可行性。我請 AI 幫忙比較 GitHub Pages 與 Cloudflare Pages 之間的差異，Cloudflare Pages 有 **PR Preview URL (分支預覽網址) 功能**，這是讓我決定可以嘗試把專案部署到 Cloudflare Pages 上的動力。

## 主題與外掛套件缺失導致 Cloudflare Pages 建置失敗

在 Cloudflare Pages 介面上將 GitHub 專案連結到 Cloudflare Pages 後，多次無法成功建置。

查找了原因後發現：
1. 原來在 GitHub 專案的 `_config.yml` 裡面有指定 Jekyll 主題及外掛套件，並且不存在 `Gemfile` 這個檔案。因為在 GitHub Pages 的預設機制中，會自動處理主題與外掛建置。
2. 但在 Cloudflare Pages 上則需要 `Gemfile` 明確定義。預設在執行 Jekyll build 時，如果沒有透過 Bundler 載入 `Gemfile`，就會找不到主題套件。

而解決方法很直接：
1. 在專案根目錄補上 `Gemfile`，同步填入對應的主題及套件。
2. 將 Build command 修改為：`bundle exec jekyll build`。

## Google Search Console 的 Sitemap 網址指向問題

當準備把網站提交給 Google Search Console (GSC) 時，我先去檢查 Cloudflare Pages 上的 `sitemap.xml`，發現裡面的 URL 全都依然寫著 GitHub Pages 的網域。  
Jekyll 產生 Sitemap 時，會直接讀取 `_config.yml` 裡的 `url` 與 `baseurl`。透過 AI 建議調整 Build command 裡加上 `--url` 參數就能解決，結果 Cloudflare 直接跳出錯誤：

``` 
jekyll 4.4.1 | Error: invalid option: --url
```

原來 Jekyll 的 CLI 指令並**不支援 `--url` 參數**。

**雙平台各自獨立 Sitemap 的解決方式：**  
為了不破壞 GitHub Pages 的原本設定，又能讓 Cloudflare Pages 產出正確的 Sitemap，採用了多個設定檔覆寫機制：

1. GitHub Pages 維持使用 `_config.yml`。

2. 在專案根目錄新增 `_config_cloudflare.yml` 專門提供給 Cloudflare Pages 使用。

3. 修改 Cloudflare Pages 的 Build command：`bundle exec jekyll build --config _config.yml,_config_cloudflare.yml`。

Jekyll CLI 支援同時傳遞多個 `--config` 檔案，後方的設定會覆寫前方的設定。這個做法讓兩邊網址完全獨立，也順利完成 Google Search Console 的 Sitemap 提交。

從這次偶然在課程中聽聞 Cloudflare Pages，到順利完成部署與網域串接，雖然遇到幾次 CI/CD 設定問題，但學到了 Jekyll 在多平台建置時的配置邏輯。

利用多份設定檔覆寫的技巧後，就不需要為了單一平台的特殊需求而把主設定檔改得面目全非。同時，Cloudflare Pages 每次 Commit 產生的獨立預覽網址 (PR Preview URL / Immutable Preview)，在比對不同版本的建置狀況時也比想像中方便。目前兩邊的網站與 Sitemap 都各自正常運作，順利在原本的開發工作流裡多解開了一個新工具。