# linux screen 常用命令

这并不是一本书的读书笔记。

## screen 外部

- `screen ls`查看列表
- `screen -r xxx`恢复 Detached 状态的 screen
- `screen -d xxx`将 Attached 状态改为 Detached 状态，即踢掉之前的连接
- `screen -S name`创建一个新的 screen

## screen 内部

- `ctrl+a d`断开连接
- `ctrl+a w`查看窗口列表
- `ctrl+a <n>`切换不同的窗口，`<n>`为数字
- `ctrl+a c`创建新窗口
- `ctrl+a n`切换到下一个窗口，`ctrl+a p`切换到上一个窗口
- `ctrl+a K`关闭窗口
- `exit`退出窗口（当所有窗口都 exit 之后，该 screen 自动销毁掉）

## 分屏

- `ctrl+a`然后输入`split`即可分屏，或者`ctrl+a S`
- `ctrl+a tab`多屏切换
- `ctrl+a X`关闭当前屏
- `ctrl+a Q`关闭其他屏


