---
title: "在 GitHub Pages 上建立一個 Hugo Blog"
date: 2018-07-10T02:07:00+08:00
tags: [
    "git",
    "hugo",
]
categories: [
    "blog", 
]

---

[Hugo](https://gohugo.io/) 是一個靜態網站生成器，雖然還有很多亂七八糟的功能，不
過我主要看上了他能拿來產生部落格的功能。以下將介紹如何使用 Hugo 在 Github Pages 
上搭建專題頁面。

<!--more-->

要從頭開始搭建 Hugo blog 在 github 的步驟如下：

1. 搭建一個 Hugo 網站
2. 部署到 GitHub 及 GitHub Pages 上

## 搭建一個 Hugo 網站
### 安裝 Hugo

Hugo 提供[多種安裝方式](https://gohugo.io/getting-started/installing/)，在 Arch
Linux 中，是由 Community Repo 維護，可直接安裝：
```sh
# pacman -Sy hugo
```

### 建立 Hugo 網站

```sh
$ hugo new site <project-name>
$ cd <project-name>
```

### 選擇想要的 Theme

Hugo 上有[不少 theme](https://themes.gohugo.io/)可供挑選，如果沒有能力自行修
改，那一個 theme 基本上就代表了你的網站的能力，所以挑選前要注意。

在這邊我花費了很久的時間，主要有幾個原因：

1. 沒有我看得上眼的 dark theme，不是功能不齊就
是[太醜](https://themes.gohugo.io/after-dark/)或[太花](https://themes.gohugo.io/bluestnight/)。
2. 最喜歡的 [Hikari](https://themes.gohugo.io/hikari/) 沒有標籤也沒有類別。
3. 最後就先暫時選定 [Whiteplain](https://themes.gohugo.io/whiteplain/)，因為佈
   景看起來夠簡單修改應該會比較容易，不過目前試用時還是有看到一些問題，例如那堆
   分享按鈕……

以我最後選的 `taikii/whiteplain` 為例，安裝 theme。不過因為未來要把整個靜態資料
夾一併 push，因此改用 `git submodule` 而不是一般的 `git clone`

```sh
$ git submodule add https://github.com/taikii/whiteplain.git themes/whiteplain
```

### 調整網站設定

通常都是複製 theme 中的 `exampleSite/config.toml` 並加以修改，詳細依各 theme 而
定。以下有提到設定時也會以 toml 做為設定格式。

要注意的是 `baseurl`，如果像我一樣打算先以 Project Pages 做為 blog，那麼應該要
設定成： `baseurl = "https://<username>.github.io/<project-name>"`
如此一來一個基本的網站就算建完了，可以用 `$ hugo serve` 先行預覽。預設的網址為
：`https://localhost:1313/<project-name>`

## 部署到 Github 及 GitHub Pages 上
### 初始化專案

針對一般專案的初始化就不用多提，比較特別的是我們打算將 Hugo 產生出來的檔案直接
推送至 `gh-pages` 分支，因此要讓 `master` 分支忽略 `public/`：

```sh
$ git init

# Ignore `public`
$ echo "public" >> .gitignore

$ git add .
$ git commit -m `Initial commit.`

# add remote site
$ git remote add origin https://github.com/<user-name>/<project-name>.git
```

### 初始化 `gh-pages`

GitHub Pages 使用 `gh-pages` 分支做為網站，因此我們利用 orphan branch[^ob] 建
立：

```sh
# make a orphan branch and push to origin
$ git checkout --orphan gh-pages
$ git reset --hard
$ git commit --allow-empty -m "Initializing gh-pages branch"
$ git push origin gh-pages
$ git checkout master
```

[^ob]: 和原本 repo 無關的分支。可見[官方解釋](https://git-scm.com/docs/git-checkout/#git-checkout---orphanltnewbranchgt)

### 建立並部署至 GitHub Page 上

利用 github 可以建立多重[工作樹](https://git-scm.com/docs/git-worktree)的功能，
將 `public/` 做為 `gh-pages` 的工作樹：

```sh
$ rm -rf public
$ git worktree add -B gh-pages public origin/gh-pages
```

接著只要重新建立網站並 `commit` 至 `origin/gh-pages` 就算佈署完成了。

```sh
$ hugo
$ cd public && git add --all && git commit -m "Publishing to gh-pages" && cd ..

# commit to GitHub Pages
$ git push origin gh-pages
```

### 建立 publish script

由於 Hugo 每次產生檔案時不會自動刪除 `public/`，必須手動砍除再把 `public/` 加入
工作樹，工作繁瑣。建議直接建立個可執行的 script `public.sh` 置於
`<project-name>` 根目錄：

```sh
#!/bin/sh

# Those 2 line assume you are in content/*/
#DIR=$(dirname "$0")
#cd $DIR/..

if [[ $(git status -s) ]]
then
    echo "The working directory is dirty. Please commit any pending changes."
    exit 1;
fi

echo "Deleting old publication"
rm -rf public
mkdir public
git worktree prune
rm -rf .git/worktrees/public/

echo "Checking out gh-pages branch into public"
git worktree add -B gh-pages public origin/gh-pages

echo "Removing existing files"
rm -rf public/*

echo "Generating site"
hugo

echo "Updating gh-pages branch"
cd public && git add --all && git commit -m "Publishing to gh-pages (publish.sh)"

echo "pushing to publish"
git push origin gh-pages -f
```

如此每次寫完文章後，只要執行下兩行指令即可重編網站並佈署：

```sh
$ git push
$ sh publish_changes.sh
```


## 參考

* [Host on GitHub | Hugo](https://gohugo.io/hosting-and-deployment/hosting-on-github/#put-it-into-a-script-1)
* [Hugo website hosted on GitHub Pages | keithp's blog](https://keithpblog.org/post/hugo-website-on-githubpages/)
