# プロジェクトのGit登録

このプロジェクト全体をはGitに登録する手順について記載します。

## 0. Gitリポジトリ作成

- git@github.com:[yourname]/[project-root].git
- git@github.com:[yourname]/[project-root-backend].git
- git@github.com:[yourname]/[project-root-frontend].git

## 1. backend / frontend を「独立したGitリポジトリ」にする
 - backend 側の作業
  ```bash
  cd [project-root]/backend

  git init
  git add .
  git commit -m "Initial commit (backend)"
  git branch -M main
  git remote add origin git@github.com:[yourname]/[project-root-backend].git
  git push -u origin main

  .gitignore.srcを.gitignoreにリネーム
  ```

- frontend 側の作業
  ```bash
  cd [project-root]/frontend

  git init
  git add .
  git commit -m "Initial commit (frontend)"
  git branch -M main
  git remote add origin git@github.com:[yourname]/[project-root-frontend].git
  git push -u origin main

  .gitignore.srcを.gitignoreにリネーム
  ```

## 2. project-root を親リポジトリにする
```bash
cd [project-root]
git init
 ```

## 3. backend / frontend を submodule として登録
- すでに backend/frontend に .git がある場合は一度退避
  ```bash
  sudo mv backend /tmp/backend
  sudo mv frontend /tmp/frontend
  ```

- submodule 追加
  ```bash
  git submodule add git@github.com:[yourname]/[project-root-backend].git backend
  git submodule add git@github.com:[yourname]/[project-root-frontend].git frontend
 ```
 
- 退避していた中身を戻す（初回のみ）
  ```bash
  sudo rm -rf backend frontend
  sudo mv /tmp/backend ./backend
  sudo mv /tmp/frontend ./frontend
  ```

- 親リポジトリをコミット＆プッシュ
  ```bash
  #git add .gitmodules backend frontend
  git add .
  git commit -m "Add backend and frontend as submodules"
  git branch -M main
  git remote add origin git@github.com:[yourname]/[project-root].git
  git push -u origin main

  .gitignore.srcを.gitignoreにリネーム
  ```

## 4. clone する側の正しい手順
git clone --recursive git@github.com:[yourname]/[project-root].git
もしくは後から：
git submodule update --init --recursive

