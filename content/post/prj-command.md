---
title: "neovim,fish,tmux,ghq,pecoを使った開発環境構築"
date: 2020-03-13T23:29:56+09:00
draft: false
categories: [ "tech" ]
tags: [ "vim", "neovim", "fish", "tmux", "shell" ]
---

ちょっとしたコード編集とかだったらターミナル上だけで完結させるのが好きでして、
それを効率よくプロジェクト選択、そのまま編集できる `prj` というコマンドを自作しています。

なんだかんだ1年以上運用していて、満足しているのでアウトプットしておきます。

## 環境
次のような環境が必要です
- [Neovim](https://github.com/neovim/neovim)
  - [neovim-remote](https://github.com/mhinz/neovim-remote)
- [tmux](https://github.com/tmux/tmux)
- [fish](https://fishshell.com/)
- [peco](https://github.com/peco/peco)

Neovim,tmuxは必須ですが、fish,pecoはうまく書き換えればzsh,fzfでも良いかと思います。

## デモ
