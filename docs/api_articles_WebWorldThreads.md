## 网络服务器与世界线程
本文将解释插件作者关心的网络服务器与世界线程之间出现的线程问题。

一般来说，提供 webadmin 页面的插件在交互时应相当谨慎。对 Cuberite 对象的大多数操作都需要同步，而 Cuberite 会自动、透明地向插件提供同步--当写入一个块时，chunkmap 会被锁定，或者当操作一个实体时，实体列表会被锁定。每个插件还拥有一个互斥锁，因此一次只能有一个线程执行插件代码。

如果插件编写不慎，这种锁定可能会导致死锁。
## 示例情景
请看下面的示例。一个插件提供了一个 webadmin 页面，允许管理员将玩家踢出服务器。当管理员按下 "踢 "按钮时，插件会调用 cWorld:DoWithPlayer() 和一个回调来踢玩家。现在看来一切正常。

插件开发了一个新功能，现在插件添加了一个新的游戏内命令，这样管理员就可以在玩家玩游戏时踢玩家了。插件通过 cPluginManager.AddCommand() 注册了一个命令回调。现在必然会出现一些问题。

假设有两个管理员，一个在使用 webadmin，另一个在游戏中。两人同时试图踢一名玩家。网络管理员锁定了插件，因此可以执行插件代码，但就在此时，操作系统切换了线程。世界线程锁定了世界，这样它就可以访问游戏内命令列表，收到游戏内命令后，它试图锁定插件。插件已经锁定，因此世界线程被搁置。过了一会儿，网络管理员线程再次被唤醒并继续处理。它试图锁定 "世界"，以便遍历玩家列表，但 "世界 "线程已经锁定了该锁。现在两个线程各持有一个锁，并试图抢夺另一个锁，因此陷入了死锁。
## 如何避免死锁
避免这种死锁的方法主要有两种。第一种方法是使用任务： 每次需要在世界中执行任务时，不要执行任务，而是使用 cWorld:QueueTask()将其排队。这个方便的工具可以在世界的 TickThread 中调用给定的函数，从而消除死锁，因为现在只有一个线程。不过，这种方法无法让你获取数据。你无法查询玩家列表、实体或任何东西，因为当任务运行时，Webadmin 页面已经提供给了浏览器。

为了适应这种情况，你需要使用第二种方法--在 tick 线程中准备和缓存数据，可能使用回调。这意味着插件将拥有存储数据的全局变量，并在数据发生变化时更新这些变量；然后网络服务器线程将只读取这些变量，而不是调用世界函数。例如，如果一个网页要显示当前连接玩家的列表，插件就应该维护一个全局变量 g_WorldPlayers，它将是一个世界表，每项都是当前连接玩家的列表。webadmin 处理程序将读取该变量并据此创建页面；插件将使用 HOOK_PLAYER_JOINED 和 HOOK_DISCONNECT 来更新该变量。
## 如何避免
既然我们已经知道了危险是什么以及如何避免，那么如何知道我们的代码是否容易受到影响呢？

一般的经验法则是避免在网络服务器线程中调用任何读取或写入事物列表的函数。这意味着大多数 ForEach() 和 DoWith() 函数。只有 cRoot:ForEachWorld() 是安全的--因为世界列表预计不会发生变化，所以它不受互斥体的保护。获取和设置世界块自然是不安全的，调用其他插件或创建实体也是如此。

## 示例
核心程序具有使用网页界面踢玩家的功能。它使用以下代码进行踢球（在 webadmin 处理程序内）： cRoot:FindAndDoWithPlayer() 不安全，可能会导致死锁。新的解决方案是队列任务；但由于我们不知道玩家在哪个世界，因此需要将任务队列到所有世界：
``` bash
local KickPlayerName = Request.Params["players-kick"]
local FoundPlayerCallback = function(Player)
	if (Player:GetName() == KickPlayerName) then
	Player:GetClientHandle():Kick("You were kicked from the game!")
	end
end
cRoot:Get():FindAndDoWithPlayer(KickPlayerName, FoundPlayerCallback)
```
``` bash
cRoot:Get():ForEachWorld(    -- For each world...
	function(World)
	World:QueueTask(         -- ... queue a task...
		function(a_World)
		a_World:DoWithPlayer(KickPlayerName,  -- ... to walk the playerlist...
			function (a_Player)
			a_Player:GetClientHandle():Kick("You were kicked from the game!")  -- ... and kick the player
			end
		)
		end
	)
	end
```
