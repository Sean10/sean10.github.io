---
title: hammerspoon神器扩展
subtitle: tech
date: 2022-02-20 00:01:59
updated:
tags: [hammerspoon, mac, Alfred, extension]
categories: [专业]
---



# 概览

[ashfinal/awesome\-hammerspoon: awesome configuration for Hammerspoon\.](https://github.com/ashfinal/awesome-hammerspoon)
开源,有点厉害,初步看着像是提供了整套操纵mac桌面的Lua API. 基本上是等同于alfred的效果了吧?

参照[ashfinal/awesome\-hammerspoon: awesome configuration for Hammerspoon\.](https://github.com/ashfinal/awesome-hammerspoon)部署, 然后修改`~/.hammerspoon/init.lua`即可


# 功能扩展
## 剪切板历史

[Hammerspoon docs: TextClipboardHistory](http://www.hammerspoon.org/Spoons/TextClipboardHistory.html)

注册快捷键"alt+"v"

``` lua
hs.loadSpoon('TextClipboardHistory')
spoon.TextClipboardHistory.show_in_menubar = false
spoon.TextClipboardHistory.paste_on_select = true
spoon.TextClipboardHistory.honor_ignoredidentifiers = true
spoon.TextClipboardHistory:start()
hs.hotkey.bind("alt", "V", function()
    spoon.TextClipboardHistory:toggleClipboard()
    mode:exit()
end)
```

[Hammerspoon docs: TextClipboardHistory](http://www.hammerspoon.org/Spoons/TextClipboardHistory.html)

## hsearch: 类似alfred

实现类似alfred的自定义能力. 

### 疑似卡顿

[Hammerspoon “Freezes” After a While · Issue \#2219 · Hammerspoon/hammerspoon](https://github.com/Hammerspoon/hammerspoon/issues/2219)
但是hammerspoon的Choooser好像实在是太卡了, hidden和显示都会转圈圈.

提了个issue,看看官方大佬有没有人能够解决, 我试了下我编译老是报错,感觉像是objective-c里还有什么依赖不在那个cocoa里的.

TODO:后来没怎么用这个功能, 后续遇到再定位看看




## resizeM: 平铺窗口管理器

实现将当前转换快速修改窗口大小以及在桌面的平铺位置. 默认用的`awesome-hammerspoon`里的


`Option+R`进入hammerspoon模式, 通过方向键右切换到左右屏幕, 然后通过F来进行全屏化.

### 另一种实现方式

找的大佬的脚本[^resize]
``` lua
-- Window management

-- Defines for window maximize toggler
local frameCache = {}
local logger = hs.logger.new("windows")

-- Resize current window

function winresize(how)
   local win = hs.window.focusedWindow()
   local app = win:application():name()
   local windowLayout
   local newrect

   if how == "left" then
      newrect = hs.layout.left50
   elseif how == "right" then
      newrect = hs.layout.right50
   elseif how == "up" then
      newrect = {0,0,1,0.5}
   elseif how == "down" then
      newrect = {0,0.5,1,0.5}
   elseif how == "max" then
      newrect = hs.layout.maximized
   elseif how == "left_third" or how == "hthird-0" then
      newrect = {0,0,1/3,1}
   elseif how == "middle_third_h" or how == "hthird-1" then
      newrect = {1/3,0,1/3,1}
   elseif how == "right_third" or how == "hthird-2" then
      newrect = {2/3,0,1/3,1}
   elseif how == "top_third" or how == "vthird-0" then
      newrect = {0,0,1,1/3}
   elseif how == "middle_third_v" or how == "vthird-1" then
      newrect = {0,1/3,1,1/3}
   elseif how == "bottom_third" or how == "vthird-2" then
      newrect = {0,2/3,1,1/3}
   end

   win:move(newrect)
end

function winmovescreen(how)
   local win = hs.window.focusedWindow()
   if how == "left" then
      win:moveOneScreenWest()
   elseif how == "right" then
      win:moveOneScreenEast()
   end
end

-- Toggle a window between its normal size, and being maximized
function toggle_window_maximized()
   local win = hs.window.focusedWindow()
   if frameCache[win:id()] then
      win:setFrame(frameCache[win:id()])
      frameCache[win:id()] = nil
   else
      frameCache[win:id()] = win:frame()
      win:maximize()
   end
end

-- Move between thirds of the screen
function get_horizontal_third(win)
   local frame=win:frame()
   local screenframe=win:screen():frame()
   local relframe=hs.geometry(frame.x-screenframe.x, frame.y-screenframe.y, frame.w, frame.h)
   local third = math.floor(3.01*relframe.x/screenframe.w)
   logger.df("Screen frame: %s", screenframe)
   logger.df("Window frame: %s, relframe %s is in horizontal third #%d", frame, relframe, third)
   return third
end

function get_vertical_third(win)
   local frame=win:frame()
   local screenframe=win:screen():frame()
   local relframe=hs.geometry(frame.x-screenframe.x, frame.y-screenframe.y, frame.w, frame.h)
   local third = math.floor(3.01*relframe.y/screenframe.h)
   logger.df("Screen frame: %s", screenframe)
   logger.df("Window frame: %s, relframe %s is in vertical third #%d", frame, relframe, third)
   return third
end

function left_third()
   local win = hs.window.focusedWindow()
   local third = get_horizontal_third(win)
   if third == 0 then
      winresize("hthird-0")
   else
      winresize("hthird-" .. (third-1))
   end
end

function right_third()
   local win = hs.window.focusedWindow()
   local third = get_horizontal_third(win)
   if third == 2 then
      winresize("hthird-2")
   else
      winresize("hthird-" .. (third+1))
   end
end

function up_third()
   local win = hs.window.focusedWindow()
   local third = get_vertical_third(win)
   if third == 0 then
      winresize("vthird-0")
   else
      winresize("vthird-" .. (third-1))
   end
end

function down_third()
   local win = hs.window.focusedWindow()
   local third = get_vertical_third(win)
   if third == 2 then
      winresize("vthird-2")
   else
      winresize("vthird-" .. (third+1))
   end
end

function center()
   local win = hs.window.focusedWindow()
   win:centerOnScreen()
end

-------- Key bindings

-- Halves of the screen
hs.hotkey.bind({"ctrl","cmd"}, "Left",  hs.fnutils.partial(winresize, "left"))
hs.hotkey.bind({"ctrl","cmd"}, "Right", hs.fnutils.partial(winresize, "right"))
hs.hotkey.bind({"ctrl","cmd"}, "Up",    hs.fnutils.partial(winresize, "up"))
hs.hotkey.bind({"ctrl","cmd"}, "Down",  hs.fnutils.partial(winresize, "down"))

-- Center of the screen
hs.hotkey.bind({"ctrl", "cmd"}, "C", center)

-- Thirds of the screen
hs.hotkey.bind({"ctrl", "alt"}, "Left",  left_third)
hs.hotkey.bind({"ctrl", "alt"}, "Right", right_third)
hs.hotkey.bind({"ctrl", "alt"}, "Up",    up_third)
hs.hotkey.bind({"ctrl", "alt"}, "Down",  down_third)

-- Maximized
hs.hotkey.bind({"ctrl", "alt", "cmd"}, "F",     hs.fnutils.partial(winresize, "max"))
hs.hotkey.bind({"ctrl", "alt", "cmd"}, "Up",    hs.fnutils.partial(winresize, "max"))

-- Move between screens
hs.hotkey.bind({"ctrl", "alt", "cmd"}, "Left",  hs.fnutils.partial(winmovescreen, "left"))
hs.hotkey.bind({"ctrl", "alt", "cmd"}, "Right", hs.fnutils.partial(winmovescreen, "right"))
```

[^resize]: [\(つェ⊂\)咦\!又好了\!](https://thinkhard.tech/2019/04/08/hammerspoon-introduce/)


## hints: 窗口切换器

实现根据前缀快速切换到指定的窗口. 

这里显示的图标位置按照窗口的左上角索引位置排列, 所以只要能记住窗口的位置, 就选择对应的前缀输入就可以完成切换了[^switch]

[^switch]:[Switching between windows with Alt \+ Tab · Issue \#856 · Hammerspoon/hammerspoon](https://github.com/Hammerspoon/hammerspoon/issues/856)


### windowhints 经常卡顿几秒才弹出切换的窗口和对应的按键

根据[WindowHints Performance · Issue \#233 · Hammerspoon/hammerspoon](https://github.com/Hammerspoon/hammerspoon/issues/233)这篇来看, 这个问题早早已经存在, 且主要受限于各自电脑装上的某些会阻塞的程序.

根据下面的代码, 可以发现关键点是`windows = windows or window.allWindows()`这段的用时长.

``` lua
--- hs.hints.windowHints([windows, callback, allowNonStandard])
--- Function
--- Displays a keyboard hint for switching focus to each window
---
--- Parameters:
---  * windows - An optional table containing some `hs.window` objects. If this value is nil, all windows will be hinted
---  * callback - An optional function that will be called when a window has been selected by the user. The function will be called with a single argument containing the `hs.window` object of the window chosen by the user
---  * allowNonStandard - An optional boolean.  If true, all windows will be included, not just standard windows
---
--- Returns:
---  * None
---
--- Notes:
---  * If there are more windows open than there are characters available in hs.hints.hintChars, multiple characters will be used
---  * If hints.style is set to "vimperator", every window hint is prefixed with the first character of the parent application's name
---  * To display hints only for the currently focused application, try something like:
---   * `hs.hints.windowHints(hs.window.focusedWindow():application():allWindows())`
function hints.windowHints(windows, callback, allowNonStandard)

  if hints.style == "vimperator" then
    hintChars = hints.hintCharsVimperator
  else
    hintChars = hints.hintChars
  end

  windows = windows or window.allWindows()
  selectionCallback = callback

  if (modalKey == nil) then
    modalKey = hints.setupModal()
  end
  hints.closeHints()
  hintDict = {}
  for _, win in ipairs(windows) do
    local app = win:application()
    if app and app:bundleID() and isValidWindow(win, allowNonStandard) then
      if hints.style == "vimperator" then
        local appchar = string.upper(string.sub(app:title(), 1, 1))
        if hintDict[appchar] == nil then
          hintDict[appchar] = {}
        end
        hints.addWindow(hintDict[appchar], win)
      else
        hints.addWindow(hintDict, win)
      end
    end
  end
  takenPositions = {}
  if next(hintDict) ~= nil then
    hints.displayHintsForDict(hintDict, "", nil, allowNonStandard)
    modalKey:enter()
  end
end
```


在`opt+z`呼出的窗口中输入, 获取到自己环境中导致耗时长的程序


```
> hs.window._timed_allWindows()
2022-02-19 23:15:05: took 0.12s for com.sogou.inputmethod.sogou
2022-02-19 23:15:05: took 0.07s for com.apple.inputmethod.EmojiFunctionRowItem
2022-02-19 23:15:05: took 1.68s for com.apple.quicklook.QuickLookUIService
2022-02-19 23:15:05: took 0.06s for N/A
2022-02-19 23:15:05: took 4.81s for com.apple.WebKit.WebContent
2022-02-19 23:15:05: took 0.12s for com.apple.ViewBridgeAuxiliary
2022-02-19 23:15:05: took 0.08s for com.apple.studentd
2022-02-19 23:15:05: took 5.66s for com.apple.appkit.xpc.openAndSavePanelService
2022-02-19 23:15:05: took 1.10s for com.apple.LookupViewService
2022-02-19 23:15:05: took 0.11s for com.apple.controlstrip
2022-02-19 23:15:05: took 1.25s for com.apple.ActivityMonitor
2022-02-19 23:15:05: took 0.35s for com.apple.PressAndHold
2022-02-19 23:15:05: took 0.90s for com.apple.amp.devicesui
2022-02-19 23:15:05: table: 0x600000245f40
```



根据下文这段里的, 可以发现allWindows默认已经通过`SKIP_APPS`跳过了一些程序.

``` lua 
local SKIP_APPS={
  ['com.apple.WebKit.WebContent']=true,['com.apple.qtserver']=true,['com.google.Chrome.helper']=true,
  ['org.pqrs.Karabiner-AXNotifier']=true,['com.adobe.PDApp.AAMUpdatesNotifier']=true,
  ['com.adobe.csi.CS5.5ServiceManager']=true,['com.mcafee.McAfeeReporter']=true}
-- so apparently OSX enforces a 6s limit on apps to respond to AX queries;
-- Karabiner's AXNotifier and Adobe Update Notifier fail in that fashion
function window.allWindows()
  local r={}
  for _,app in ipairs(application.runningApplications()) do
    if app:kind()>=0 then
      local bid=app:bundleID() or 'N/A' --just for safety; universalaccessd has no bundleid (but it's kind()==-1 anyway)
      if bid=='com.apple.finder' then --exclude the desktop "window"
        -- check the role explicitly, instead of relying on absent :id() - sometimes minimized windows have no :id() (El Cap Notes.app)
        for _,w in ipairs(app:allWindows()) do if w:role()=='AXWindow' then r[#r+1]=w end end
      elseif not SKIP_APPS[bid] then
        for _,w in ipairs(app:allWindows()) do
          r[#r+1]=w
        end
      end
    end
  end
  return r
end

```


理论上使用`hs.window.filter`可能有帮助?


最后短时间没搞明白`filter`的注入到`window.allWindows`里的防范, 先手动在`/Applications/Hammerspoon.app/Contents/Resources/extensions/hs/window.lua`底下的`SKIP_APPS`直接增加了自己环境里耗时多的程序, 跳过后基本呼出`alt+tab`的延时正常了.[^vscode][^chooser]

[^vscode]: [Hammerspoon get sluggish with vscode running · Issue \#2289 · Hammerspoon/hammerspoon](https://github.com/Hammerspoon/hammerspoon/issues/2289)


[^chooser]: [windowHints are slow · Issue \#2970 · Hammerspoon/hammerspoon](https://github.com/Hammerspoon/hammerspoon/issues/2970)
> Yabai is capable of doing what you want using only the window id, but that solution requires disabling SIP and injecting code into Dock.app.

根据这句来判断, hammerspoon在关闭SIP之后还是会存在这个问题的.

``` lua
local SKIP_APPS={
  ['com.apple.WebKit.WebContent']=true,['com.apple.qtserver']=true,['com.google.Chrome.helper']=true,
  ['org.pqrs.Karabiner-AXNotifier']=true,['com.adobe.PDApp.AAMUpdatesNotifier']=true,
  ['com.adobe.csi.CS5.5ServiceManager']=true,['com.mcafee.McAfeeReporter']=true,
['com.apple.appkit.xpc.openAndSavePanelService']=true, 
['com.apple.quicklook.QuickLookUIService']=true,
['com.apple.PressAndHold']=true,
['com.apple.ActivityMonitor']=true,
['com.apple.amp.devicesui']=true}
```

## 快捷方式打开指定目录下最新的文件(如截图)

```
hs.hotkey.bind("alt", "o", function()
    logger = hs.logger.new('preview newest screenshot')
    max = 0
    newest_file = nil
    for file in hs.fs.dir("~/Desktop") do
        filepath = hs.fs.pathToAbsolute(table.concat({"~/Desktop", file}, "/"))
        time = hs.fs.attributes(filepath)['creation']

        if max < time
        then
            max = time
            newest_file = filepath
        end
    end

    if (max ~= 0)
    then
        logger.i("open \""..newest_file.."\"")
        res = os.execute("open \""..newest_file.."\"")
        logger.i(res)
    end
end)

```


#### TODO:os.execute会自动补全路径?

```
> os.execute("open \"~/Desktop/temp.sh\"")
The file /Users/sean10/Desktop/~/Desktop/temp.sh does not exist.
nil	exit	1
```

### 快速切换显示器信号源, 基于DDC工具

增加了`shift+F11`的快速切换信号源的方案.

``` lua
hs.hotkey.bind("shift", "f11", function()
    hidden_status = os.execute("/usr/local/bin/ddcctl -d 1 -i 15")
end)
```

# 日志调试方式

[How to debug Hammerspoon scripts? · Issue \#989 · Hammerspoon/hammerspoon](https://github.com/Hammerspoon/hammerspoon/issues/989)

[pkulchenko/MobDebug: Remote debugger for Lua\.](https://github.com/pkulchenko/MobDebug

根据上述链接, 应该可以用lua的调试方式来操作, `mobdebug.lua`


### 日志

``` lua
    logger = hs.logger.new('preview snap')
    logger.i("hello info")
    -- 默认hammerspoon是error等级日志
    logger.e("error)
    
```



# Reference