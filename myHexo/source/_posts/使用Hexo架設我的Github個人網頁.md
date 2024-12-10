---
title: 使用Hexo架設我的Github個人網頁
date: 2024-12-10 19:40:32
tags: "web"
---

之前就打算架設一個個人網頁，但是考慮到需要租借伺服器還要申請網域之類的，十分麻煩，於是一直遲遲沒有動手。直到最近知道可以用 GitHub Pages 來架站，所以稍微研究了一下。這篇文章主要紀錄我從頭開始架設網站的過程，以及在過程中遇到的一些問題以及如何解決。

## 1. 選擇並架設網站框架
我選擇使用 [Hexo](https://hexo.io/zh-tw/)。Hexo 是一個快速的靜態網頁產生器，且支援 Markdown 文件，對於像我這種懶得設計及排版的人來說可謂是十分方便。以下整理一下架設流程：

### 安裝 `Node.js` 和 `Hexo`

先去 [Node.js 官網](https://nodejs.org/zh-tw) 安裝 `Node.js` 和 `npm`，並確定兩個都有裝成功：
```bash
node -v
npm -v
```
接著使用 npm 安裝 Hexo，並確認是否成功安裝：
``` bash
npm install -g hexo-cli
hexo -v
```

### 建立一個 Hexo 的文件夾
首先選一個位置建立一個空的 hexo project 資料夾
> 這裡注意資料夾一定要是空的，不然在 `hexo init` 的時候會報錯。

```bash
hexo init myHexo
cd myHexo
npm install
```
接著就可以選擇主題了，我這裡使用的是 [frame](https://github.com/zoeingwingkei/frame/) 這個主題。
把主題放至 `theme/` 底下即可。
```bash
git clone https://github.com/zoeingwingkei/frame.git theme/frame
```

接著需要修改 `_config.yml` 來選擇所要使用的主題。像我這邊把 `theme` 中的內容改成 `frame` 就好。
```yaml
theme: frame
```

到這裡算是基本設定完成，可以再local端簡單測試一下自己的網頁長怎樣。
```bash
hexo server
```

## 2. 設定 GitHub Pages
接下來，我在 GitHub 上創建了一個新的 repo 並啟用了 GitHub Pages。然後，我將 `gh-pages` 分支設置為網站源；而原來的分支 `main` 則做為放 hexo 框架程式碼的地方，順便作為備份。在 GitHub 儲存庫的 Settings 中選擇 `gh-pages` 作為 Pages 的來源分支即可。

這樣，我的 GitHub Pages 顯示的就會是 hexo 部屬後的靜態網頁，我只需要在修改完我的程式後， hexo 就會幫我把網頁資料轉換成靜態資料並部屬到 gh-pages 分支上。

## 3. 創建和部屬網站
在成功設置了 Hexo 和 GitHub Pages 後，就可以把這些資料轉換成靜態網頁並部屬上去了。

首先需要在 `_config.yml` 中設定 deploy 的資訊：
```yaml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: https://github.com/ychsiao0809/ychsiao0809.github.io.git
  branch: gh-pages
```

再來透過 hexo 部屬：
```bash
hexo generate
hexo deploy
```

在這步驟之前過程一切順利，但是在使用 `hexo deploy` 時，我遇到了一些身份驗證的問題。由於啟用了 GitHub 的二階段認證，我需要設置 Personal Access Token (PAT) 來解決這個問題。於是乎我從 GitHub 生成了一筆 PAT 並將它放在我的 `_config.yml` 的 `deploy.repo` 裡面：
```yml
deploy:
  type: git
  repo: https://{{ github_token }}@github.com/ychsiao0809/ychsiao0809.github.io.git
  branch: gh-pages
```

這樣配置後總算是可以順利 deploy 至 GitHub Pages 上了，但之後在我想把整包 hexo 推回我的 `main` 分支的時候又出現狀況：
```bash
remote: error: GH013: Repository rule violations found for refs/heads/dev.
remote:
remote: - GITHUB PUSH PROTECTION
remote:   —————————————————————————————————————————
remote:     Resolve the following violations before pushing again
remote:
remote:     - Push cannot contain secrets
.
.
.
remote:       —— GitHub Personal Access Token ——————————————————————
remote:        locations:
remote:          - commit: {{}}
remote:            path: _config.yml:105
remote:
remote:        (?) To push, remove secret from commit(s) or follow this URL to allow the secret.
```
這段訊息表示我的提交中可能包含敏感信息，簡單來說就是我把我自己生成的 PAT 寫在 `_config.yml` 裡面又直接推回我的 repo 裡面，簡直超級危險。現在回想感覺自己真的挺笨的，只好感謝 GitHub 有考量到像我這種笨蛋行為有特別設計防呆。

解決方式也十分簡單，就不要把 PAT 寫進 `_config.yml` 裡，寫在 git local config 裡面就好了。
```bash
git remote set-url origin https://{{ github_token }}@github.com/ychsiao0809/ychsiao0809.github.io.git
```

另外還有遇到 submodule 的問題：我原本 hexo 專案底下的 `frame` 主題是直接用 `git clone`。我決定將它轉換為 Git submodule 來更好地管理它。我原先的做法是刪除舊的 frame 目錄，然後使用 git submodule add 命令重新添加過來，但後來發現好像只要自己在寫一份 `.gitmodules` 就可以解決這個問題。
```git
[submodule "myHexo/themes/frame"]
	path = myHexo/themes/frame
	url = https://github.com/zoeingwingkei/frame.git
```

6. 結語
總體來說，這次設置 GitHub Pages 的過程雖然遇到了一些問題，但最終順利完成。這次經歷讓我學會了如何處理 GitHub 的身份驗證、子模塊管理和推送保護規則。

希望我的經驗能夠幫助到你們，並且鼓勵大家嘗試設置自己的 GitHub 網頁！