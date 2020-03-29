[TOC]

#### **安装 tmux**

#### **tmux的会话**

1. tmux new -s [session-name] 新建会话
2. ctrl+b d 退出会话，回到shell的终端环境
3. tmux ls 终端环境查看会话列表
4. ctrl+b s 会话环境查看会话列表
5. tmux a -t [session-name] 从终端环境进入会话
6. tmux kill-session -t [session-name] 销毁会话（终端环境） ctrl+b : kill-session -t [session-name] 销毁会话（在会话环境中）
7. tmux rename -t old_session_name new_session_name 重命名会话（终端环境） ctrl+b $ 重命名会话 (在会话环境中)

#### **tmux的window**

1. ctrl+b c 创建window
2. crtl+b , 修改窗口名
3. ctrl+b & 关闭window

#### **tmux的pane**

1. ctrl+b % 垂直分屏(组合键之后按一个百分号)，用一条垂线把当前窗口分成左右两屏。
2. ctrl+b " 水平分屏(组合键之后按一个双引号)，用一条水平线把当前窗口分成上下两屏。
3. ctrl+b x 关闭pane