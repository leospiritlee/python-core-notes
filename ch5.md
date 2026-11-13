# 第5章 GUI 编程

**一句话**：Tkinter 是 Python 默认的 GUI 库，用「控件（widget）拼界面 + 回调（callback）响应事件」的方式，把脚本包成能点的图形程序。

**为什么需要它**：命令行脚本再强，给非程序员用也得有个界面。GUI 把功能变成按钮、输入框、列表，用户点一下就触发逻辑。书里引用 Jim Ahlstrom：「GUI 的东西应该会很难，它甚至可以塑造性格」——但 Tkinter 恰恰证明，简单 GUI 没那么难。

**核心结构**：Tkinter 是 Python 对 **Tk** 工具包的封装（Tk 源自 Tcl 语言，后被 Perl/Tk、Ruby/Tk、Python/Tkinter 复用）。核心概念是**控件（widget）**——按钮、标签、文本框等基础积木，用布局管理器（pack/grid/place）摆好后，再给控件绑**回调（callback）**函数，事件发生时自动调用。

**关键设计**：书里强调 Tk「不是最新最强，但足够易用、跨平台」。开发节奏是：先 `import tkinter`、建顶层窗口 `Tk()`、用 widget 拼界面、给按钮 `command=` 绑回调、最后 `mainloop()` 进入事件循环。事件是「用户点了什么 / 输入了什么」，回调是「对应的处理函数」。Tkinter 还支持菜单、对话框、画布等，足以做出商用品质的中等应用。

**拆解**：

### Tk/Tcl 与 Tkinter
Tk 是 Tcl 的 GUI 工具包，Tkinter 是它在 Python 里的接口。理解这层关系，就能看懂为什么很多 Tk 教程用 Tcl 术语（如 `pack`、`bind`）。

### 控件 widget
按钮 `Button`、标签 `Label`、输入框 `Entry`、文本区 `Text`、列表 `Listbox` 等是界面积木。每个 widget 是一个对象，有自己的属性（文字、颜色、尺寸）和方法。

### 回调与事件循环
`Button(command=on_click)` 把函数绑到点击事件；`mainloop()` 启动后程序进入「等事件→派发回调」的循环。业务逻辑全写在回调里，界面只是触发入口。

### 其他 GUI 工具包
书里简介 wxPython、PyQt 等现代工具包，以及 GUI 构建工具（wxGlade、Glade）。Tkinter 适合原型，复杂应用可平滑迁移到更现代的绑定。

**今天怎么做**：现代桌面 GUI 可选 PyQt/PySide（Qt 绑定，功能强）、wxPython、或干脆用 Web 技术（Electron/本地起服务）。Tkinter 仍随 Python 发行、零安装成本，适合内部小工具——属延伸，非语料原文。

<div class="chapend" markdown="1">

**容易误解的地方**：新手以为「`mainloop()` 之后的代码会顺序执行」，其实 `mainloop()` 是**阻塞的事件循环**，不退出就不会往下走；所有「用户操作后的逻辑」必须写在回调里，不能写在 `mainloop()` 之后指望它顺序跑。另一个坑：直接 `print` 调试 GUI 看不到界面卡顿原因，要用日志或把状态写进 `Label`。

**记住这一句**：Tkinter = 控件拼界面 + `command` 绑回调 + `mainloop()` 转事件循环；`mainloop()` 是阻塞的，用户操作的全部逻辑都得在回调里实现。

</div>
