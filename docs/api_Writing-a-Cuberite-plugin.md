## 编写 Cuberite 插件
本文将介绍如何编写基本插件。它详细介绍了插件的基本要求，解释了如何注册钩子和绑定命令，并给出了插件标准的详细信息。

让我们开始吧。  
要开始开发，我们首先必须获得 Cuberite 的编译副本，并确保 Core 插件位于 Plugins 文件夹中并已激活。核心插件将处理 Cuberite 的大部分终端用户体验，没有它，游戏将变得非常平淡。

创建基本模板  
插件是用 Lua 编写的。因此，请创建一个新的 Lua 文件。您可以创建任意多的文件，文件名不限，Cuberite 会在运行时将所有文件合并在一起，但我们现在先创建一个名为 main.lua 的文件。格式如下  
``` bash
PLUGIN = nil

function Initialize(Plugin)
	Plugin:SetName("NewPlugin")
	Plugin:SetVersion(1)

	-- Hooks

	PLUGIN = Plugin -- NOTE: only needed if you want OnDisable() to use GetName() or something like that

	-- Command Bindings

	LOG("Initialised version " .. Plugin:GetVersion())
	return true
end

function OnDisable()
	LOG("Shutting down...")
end
```
现在来解释一下基本原理。  

函数 Initialize 在插件启动时调用。它是设置插件的地方。  
Plugin:SetName 设置插件的名称。  
Plugin:SetVersion 设置插件的版本号。必须是整数。  
LOG 向控制台记录一条信息，在本例中，它将打印插件已初始化。这将添加一个带有插件名称的前缀。  
PLUGIN 变量只是存储该插件的对象，因此可以在 OnDisable 中调用 GetName()（因为没有传递插件参数，与 Initialize 相反）。只有在关闭时想知道插件的详细信息（名称等）时，才需要使用这个全局变量。
函数 OnDisable 在插件禁用时调用，通常是在服务器关闭时。在此执行清理和日志记录。  
请确保该函数返回 true，否则 Cuberite 会认为您的插件初始化失败，并打印带有错误信息的堆栈跟踪。  
## 注册钩子  
钩子是 Cuberite 在内部事件发生时调用的东西。例如，当玩家放置区块、移动、登录、吃东西等许多情况发生时，钩子就会被触发。有关完整列表，请参阅 API 文档。  

钩子可以是信息性的，也可以是可覆盖的。在任何情况下，返回 false 都不会触发响应，而返回 true 则会取消挂钩，并阻止它进一步传播到其他插件。可覆盖钩子只是意味着取消钩子会有可见的行为，例如阻止打开箱子。但也有一些例外情况，即只有更改钩子传递的值才会产生影响，而不是实际的返回值，例如 HOOK_KILLING 钩子。详情请查看 API 文档。

要注册钩子，请在前面代码示例的"--钩子 "区域插入以下代码模板。
``` bash
cPluginManager.AddHook(cPluginManager.HOOK_NAME_HERE, FunctionNameToBeCalled)
```	
这段代码的作用是什么？

cPluginManager.AddHook 注册钩子。钩子名称是第二个参数。有关所有钩子的列表，请参阅前面的 API 文档链接。
你会问第三个参数是什么？它是钩子触发时 Cuberite 调用的函数名称。你应该在这个函数中处理或取消钩子。
总的来说，这就是我们迄今为止所涉及内容的工作表述。
``` bash
function Initialize(Plugin)
	Plugin:SetName("DerpyPlugin")
	Plugin:SetVersion(1)

	cPluginManager.AddHook(cPluginManager.HOOK_PLAYER_MOVING, OnPlayerMoving)

	LOG("Initialised " .. Plugin:GetName() .. " v." .. Plugin:GetVersion())
	return true
end

function OnPlayerMoving(Player) -- See API docs for parameters of all hooks
	return true -- Prohibit player movement, see docs for whether a hook is cancellable
end
```
所以，这段代码可以阻止玩家移动。虽然不是特别有用，但还是可以的 :P。请注意，所有文档都可以在主 API 文档页面上找到，所以如果有任何疑问，请访问那里。
