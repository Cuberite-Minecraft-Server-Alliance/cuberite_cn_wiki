## 编写 Cuberite 插件
本文将介绍如何编写基本插件。它详细介绍了插件的基本要求，解释了如何注册钩子和绑定命令，并给出了插件标准的详细信息。

## 让我们开始吧。  
要开始开发，我们首先必须获得 Cuberite 的编译副本，并确保 Core 插件位于 Plugins 文件夹中并已激活。核心插件将处理 Cuberite 的大部分终端用户体验，没有它，游戏将变得非常平淡。

## 创建基本模板  
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
## 绑定命令
格式
现在我们知道了如何连接到 Cuberite，那么如何绑定一个命令，如 /explode，让玩家输入呢？这就比较复杂了。首先，我们在初始示例的"--命令绑定 "部分添加以下模板：

-- 如果命令不需要参数（/explode），则添加此模板
``` bash
cPluginManager.BindCommand("/commandname", "permissionnode", FunctionToCall, " - 命令描述")
```
-- 如果命令需要参数，请添加此参数（/explode Notch）
``` bash
cPluginManager.BindCommand("/commandname", "permissionnode", FunctionToCall, " ~ 命令描述和参数")
```
			
它的作用是什么，为什么有两个？

PluginManager:BindCommand 绑定命令。它包含命令名称（带斜线）、玩家执行命令所需的权限、执行命令时要调用的函数以及命令描述。
命令名称不言自明。权限节点基本上只是玩家所在组需要拥有的一个字符串，因此你可以在其中包含任何内容，不过我们推荐使用 "derpyplugin.explode "这样的样式。要调用的函数与钩子函数类似，但有一些固定参数，我们稍后会介绍。
为什么会有两个？标准。接受参数的插件必须使用"~ 命令和参数描述 "的描述格式，而不接受参数的命令必须使用"- 命令描述 "的格式。请务必在斜线或破折号前留一个空格。此外，请尽量在客户端上使用一行简短的说明。

## 参数
执行命令时，Cuberite 调用的函数中有哪些参数？一个 "Split"数组和一个 "Player "对象。

## 分割数组
Split 数组是一个包含提交给服务器的所有文本的数组，其中包括实际命令。Cuberite 会自动将文本分割到数组中，因此插件作者无需担心。命令"/derp zubby explode "的 Split 数组示例如下
``` bash
   /derp (Split[1])
   zubby (Split[2])
   explode (Split[3])
```
   传递的参数总数为 3 (#Split)
## 玩家对象和向他们发送信息
播放器对象基本上是一个指向已执行命令的播放器的指针。你可以用它们做一些事情，但最常见的是发送信息。详情请再次参阅 API 文档。但是，你会问，我们如何向客户端发送消息呢？

有专门的函数用于向播放器发送格式化的信息。所谓格式，指的是彩色前缀/彩色文本（取决于配置），这些前缀/文本可以清晰地分类玩家发送的信息类型。例如，信息信息的前缀是黄色的 [INFO]，警告信息的前缀是玫瑰色的 [WARNING]。这里列出了几个最常用的函数，更多详情请查看 API 文档。请在 cRoot、cWorld 和 cPlayer 部分查看分别向整个服务器、整个世界和单个玩家广播的函数。

--格式 §黄色[INFO]§white%text%（黄色[INFO]，后面是白色文字）
-- 用途：信息消息，例如命令的使用说明
Player:SendMessageInfo（"使用方法：/explode [玩家]"） --格式： /explode [玩家]

-- 格式： §green[INFO] §white%text% （绿色 [INFO] 等）
-- 使用：成功信息，如命令成功执行时
Player:SendMessageSuccess("缺口被炸毁！")

-- 格式： §rose[INFO]§white%text%（玫瑰色[INFO]等）
-- 使用： 失败消息，比如命令输入正确但运行失败，比如在 /tp com 中找不到目标播放器
-- 格式： §rose[INFO]§white%text%（玫瑰色[INFO]等）。
-- 用途： 失败信息，比如命令输入正确但运行失败，比如在 /tp 命令中没有找到目标玩家
Player:SendMessageFailure（"未找到玩家 Salted")
			
这些是最基本的。如果出于上述三种原因之外的其他原因，你想向玩家输出文本，并且想给文本着色，只需将 "cChatColor.*colorhere*"与你想要的文本连接起来，连接词为"..."。有关所有颜色的详细信息，以及使用 LOG("Text") 向控制台记录日志的详细信息，请参阅 API 文档。

## 最后的示例和结论
这就是一个可以检查命令有效性、炸死玩家以及拒绝向 ping >100ms 的玩家收集拾取信息的示例。
``` bash
function Initialize(Plugin)
	Plugin:SetName("DerpyPluginThatBlowsPeopleUp")
	Plugin:SetVersion(9001)

	cPluginManager.BindCommand("/explode", "derpyplugin.explode", Explode, " ~ Explode a player");

	cPluginManager:AddHook(cPluginManager.HOOK_COLLECTING_PICKUP, OnCollectingPickup)

	LOG("Initialised " .. Plugin:GetName() .. " v." .. Plugin:GetVersion())
	return true
end

function Explode(Split, Player)
	if (#Split ~= 2) then
		-- There was more or less than one argument (excluding the "/explode" bit)
		-- Send the proper usage to the player and exit
		Player:SendMessage("Usage: /explode [playername]")
		return true
	end

	-- Create a callback ExplodePlayer with parameter Explodee, which Cuberite calls for every player on the server
	local HasExploded = false
	local ExplodePlayer = function(Explodee)
		-- If the player name matches exactly
		if (Explodee:GetName() == Split[2]) then
			-- Create an explosion of force level 2 at the same position as they are
			-- see API docs for further details of this function
			Player:GetWorld():DoExplosionAt(2, Explodee:GetPosX(), Explodee:GetPosY(), Explodee:GetPosZ(), false, esPlugin)
			Player:SendMessageSuccess(Split[2] .. " was successfully exploded")
			HasExploded = true;
			return true -- Signalize to Cuberite that we do not need to call this callback for any more players
		end
	end

	-- Tell Cuberite to loop through all players and call the callback above with the Player object it has found
	cRoot:Get():FindAndDoWithPlayer(Split[2], ExplodePlayer)

	if not(HasExploded) then
		-- We have not broken out so far, therefore, the player must not exist, send failure
		Player:SendMessageFailure(Split[2] .. " was not found")
	end

	return true
end

function OnCollectingPickup(Player, Pickup) -- Again, see the API docs for parameters of all hooks. In this case, it is a Player and Pickup object
	if (Player:GetClientHandle():GetPing() > 100) then -- Get ping of player, in milliseconds
		return true -- Discriminate against high latency - you don't get drops :D
	else
		return false -- You do get the drops! Yay~
	end
end
```
请务必阅读注释，了解所有操作的说明。除非你想让 Cuberite 在执行命令时打印出 "未知命令 "的信息，否则请确保所有命令处理程序都返回 true :P。请确保遵循标准--使用 CoreMessaging.lua 函数发送消息，使用破折号表示无参数命令，反之亦然，最后，API 文档是您的朋友！

祝您编码愉快）
