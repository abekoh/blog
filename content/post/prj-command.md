---
title: "fish,tmux,neovim,ghq,pecoで開発スペース構築を快適にする"
date: 2020-03-14T01:43:00+09:00
draft: false
categories: [ "tech" ]
tags: [ "Vim", "Neovim", "fish", "tmux", "shell" ]
---

ちょっとしたコード編集とかだったらターミナル上だけで完結させるのが好きでして。
効率よくプロジェクト選択、そのまま編集したりしやすくなる `prj` というコマンドを自作しています。
また、これに加えて使いやすいように他にも設定盛り込んでます。

なんだかんだ1年以上運用していて、満足しているのでアウトプットしておきます。

## 環境
次のような環境が必要です
- [Neovim](https://github.com/neovim/neovim)
  - [neovim-remote](https://github.com/mhinz/neovim-remote)
- [tmux](https://github.com/tmux/tmux)
- [fish](https://fishshell.com/)
- [peco](https://github.com/peco/peco)

Neovim,tmuxは必須ですが、fish,pecoはzsh,fzfに書き換えても良いと思います。
インストール方法はそれぞれリンク先を参照してください。

## デモ
![prj command](/images/prj-command.gif)

わかりにくいかもですが、次のような特徴をみせてます。
1. `prj`で、ghqにより管理されたgitリポジトリをpecoをつかって選択
2. リポジトリを選択するとそのプロジェクト用のtmuxセッションが開かれ、カレントディレクトリがそのリポジトリ配下になる
3. tmuxセッション内で、Neovimは1タブでしか開けない
4. tmuxセッション内でもう一度`prj`を実行すると、別のプロジェクトも開ける

次の項でその実現方法を解説します。

## 解説

### prjコマンド

ghqとpecoの連携についてはこちらの記事の考えをそのまま使っています。

[ghq, peco, hubで快適Gitライフを手に入れよう！ - Qiita](https://qiita.com/itkrt2y/items/0671d1f48e66f21241e2)

この組み合わせによって、レポジトリ管理・選択が劇的に楽になります。
さらに小細工を入れてtmuxセッションを開くようにしました。

prjコマンドはfish scriptで書いています。こんな感じ
```fish
# prj.fish
function prj -d "start project"
  # 引数が設定されていれば、それをpecoにわたす
  if test (count $argv) -gt 0
    set prjflag --query "$argv"
  end
  set PRJ_PATH (ghq root)/(ghq list | peco $prjflag)
  # プロジェクトが選択されなければ終了
  if test -z $PRJ_PATH
    return
  end
  # プロジェクト名は 所有者/リポジトリ名 の形式。その名前に`.`を含む場合は`_`に置換
  set PRJ_NAME (echo (basename (dirname $PRJ_PATH))/(basename $PRJ_PATH) | sed -e 's/\./_/g')
  # プロジェクトのtmuxセッションが存在しなければ作成
  if not tmux has-session -t $PRJ_NAME
    tmux new-session -c $PRJ_PATH -s $PRJ_NAME -d
    tmux setenv -t $PRJ_NAME TMUX_SESSION_PATH $PRJ_PATH
  end
  # tmuxセッション外であればattach
  if test -z $TMUX
    tmux attach -t $PRJ_NAME
  # tmuxセッション内であればswitch
  else
    tmux switch-client -t $PRJ_NAME
  end
end
```
これでtmux連携が可能になりました。

### セッション内で開くNeovimを1つに
セッション内で自分は複数タブを使い、一方でNeovim、一方でファイル確認、一方でサーバー起動とかやります。そうやってると、ついついNeovimを複数タブで開いてしまい、どこでどのファイルを編集していたかわからなくなります。  
それを回避するために、neovim-remoteを採用しました。

neovim-remoteを使うと、すでに起動してあるNeovimに開くファイルを追加する、といった動きをさせることができます。

socketファイルをわかりやすくプロジェクト名として/tmpに保存し、同プロジェクトで新た`nvim`すると`nvr --remote-tab --servername /tmp/{プロジェクト名}`で既存Neovimの新規タブとして開かれるようになります。

`nvim`コマンドをfishでラッピングし実現しました。
```fish
# nvim.fish
function nvim -d "neovim wrapping"
  # tmuxセッション内でなければそのまま
  if test -z $TMUX
    command nvim $argv
  else
    # /tmp/にtmuxセッションを保管
    set socket_path /tmp/(echo (tmux display-message -p '#S') | sed 's/\//_/g' )
    if test -S $socket_path
      # すでにソケットが存在してたらそれに接続
      nvr --remote-tab --servername $socket_path $argv
      # 該当のnvimに移動
      set session_id (tmux list-panes -F '#{session_id}')
      set pane_ids (tmux list-panes -a -F "#{session_id},#{window_index},#{pane_index},#{pane_current_command}" | grep "^$session_id,.*,nvim\$" | string split ',')
      tmux select-window -t $pane_ids[2] && tmux select-pane -t $pane_ids[3]
    else
      # ソケットがなければ作成して起動
      command env NVIM_LISTEN_ADDRESS=$socket_path nvim $argv
    end
  end
end
```

### cdコマンド空打ちでプロジェクトルートに移動
デモに記録していませんでしたが、これも便利なので紹介。  
tmuxセッション内で`cd`空打ちすると、homeではなくプロジェクトルートに移動するようにしています。

例のごとく、fishでラッピング
```fish
# cd.fish
function cd --description "Change directory"
  if test -n "$TMUX" -a -z "$argv"
    set session_path (tmux show-environment | grep TMUX_SESSION_PATH | string replace "TMUX_SESSION_PATH=" "")
    if test $session_path
      builtin cd $session_path
      return $status
    end
  end
end
```

自分の設定では、[標準のcd.fish](https://github.com/fish-shell/fish-shell/blob/master/share/functions/cd.fish)をコピーしてきて[書き加えた](https://github.com/abekoh/dotfiles/blob/master/config/fish/functions/cd.fish)かたちにしています。

## 設定全体
dotfilesを公開しているので、全体像はこちらから。今回のfishコマンドたちはconfig/fish/functionsに置いてます。

[abekoh/dotfiles](https://github.com/abekoh/dotfiles)

ただ完全に自分用で、使ってない機能、保守していないところもあるのであしからず。。

## まとめ
fishでコマンド上書きでゴリ押しですが、使い勝手よくて気に入っています。
こういった作業改善、凝りだすと止まらなくなってしまうので危険です。。笑
