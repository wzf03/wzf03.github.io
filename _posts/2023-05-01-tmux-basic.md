---
title: "Tmux基本用法"
date: 2023-05-01T20:09:50+08:00
image: assets/img/covers/tmux.png
categories: [Linux]
tags: [linux, tmux]
---

## 会话管理

### 新建会话

```sh
tmux new -s <session-name>
```

### 分离

`Ctrl+b d` 或者 `tmux detach`

### 列出会话

`tmux ls` or `tmux list-session`

### 接入会话

```sh
# 使用编号
tmux attach -t 0
# 使用名称
tmux attach -t <session-name>
```

### 杀死会话

```sh
tmux kill-session -t 0
tmux kill-session -t <session-name>
```
