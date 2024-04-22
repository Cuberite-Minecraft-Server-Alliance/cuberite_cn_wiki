## 使用 ChunkStays
插件可能需要以任意块为单位处理数据，因此需要一种方法来确保服务器内存中的块是可用的。
## 问题
通常情况下，当插件想要操作更大范围的世界数据时，它们需要确保服务器在内存中加载了相应的数据块。当被操作的数据距离连接的玩家较远，或者数据是通过控制台处理程序操作时，就很有可能无法加载数据块。

在使用 cBlockArea 类进行读写时，这一点变得更加重要。如果任何所需的块无效，这些函数就会失败。这意味着，要么块区域的数据不完整（Read() 失败），要么向世界写入的数据不完整（Write() 失败）。这种情况下几乎不可能恢复--你不能简单地稍后再读取或写入，因为在此期间世界可能已经发生了变化。
## 解决方案
最简单的解决方案是监控数据块的加载和卸载，并将操作推迟到所有数据块都可用时再进行。如果有多个代码路径都需要这种处理方法，那么这种方法将非常无效，而且很快就会变得难以维护。

我们采用了另一种方法，只需调用一个函数（有点隐蔽）即可访问：cWorld:ChunkStay()。这个调用的基本功能就是告诉服务器："为我加载这些块，一旦全部加载完毕，就调用这个回调函数"。服务器正是这样做的--它会记住回调函数，并要求世界加载器/生成器提供大块。一旦大块可用，它就会调用插件的回调函数。

不过也有一些小问题。如果请求读取或写入的代码曾访问过一些易失性对象，如 cPlayer 或 cEntity 对象，那么回调函数就不能再访问这些对象了，因为在此期间它们可能已经失效--玩家可能已经断开连接，实体可能已经消亡。因此，回调必须使用更长的方式访问这些对象，例如调用 cWorld:DoWithEntityByID() 或 cWorld:DoWithPlayer()。
##示例
举个简单的例子，理论上一个插件可以让玩家将产卵点周围的环境保存到示意图文件中。玩家发出命令启动保存，插件会将产卵点周围 50 x 50 x 50 块的区域读取到一个 cBlockArea 中，并以"_spawn.schematic "的形式保存到磁盘上。保存完成后，插件希望向玩家发送一条信息，让他们知道命令已成功执行。

第一次尝试显示的是最简单的方法。它只是读取块区域并保存，然后发送信息。我再重复一遍，这段代码是错误的！
``` bash
function HandleCommandSaveSpawn(a_Split, a_Player)
	-- Get the coords for the spawn:
	local SpawnX = a_Player:GetWorld():GetSpawnX()
	local SpawnY = a_Player:GetWorld():GetSpawnY()
	local SpawnZ = a_Player:GetWorld():GetSpawnZ()
	local Bounds = cCuboid(SpawnX - 25, SpawnY - 25, SpawnZ - 25, SpawnX + 25, SpawnY + 25, SpawnZ + 25)
	Bounds:ClampY(0, 255)

	-- Read the area around spawn into a cBlockArea, save to file:
	local Area = cBlockArea()
	local FileName = a_Player:GetName() .. "_spawn.schematic"
	Area:Read(a_Player:GetWorld(), Bounds, cBlockArea.baTypes + cBlockArea.baMetas)
	Area:SaveToSchematicFile(FileName)

	-- Notify the player:
	a_Player:SendMessage(cCompositeChat("The spawn has been saved", mtInfo))
	return true
end
```
现在，如果玩家继续探索，并使用命令保存他们的spawn，那么分块就不会被加载，因此 BlockArea 读取失败，BlockArea 包含错误数据。请注意，该插件没有进行任何错误检查，如果没有从世界中读取该区域，它就会很高兴地保存不完整的数据，然后说 "嘿，一切正常"，尽管它刚刚销毁了之前备份的带有错误数据的产卵示意图。

下面的脚本使用 ChunkStay 方法来缓解与大块相关的问题。这才是正确的方法：
``` bash
function HandleCommandSaveSpawn(a_Split, a_Player)
	-- Get the coords for the spawn:
	local SpawnX = a_Player:GetWorld():GetSpawnX()
	local SpawnY = a_Player:GetWorld():GetSpawnY()
	local SpawnZ = a_Player:GetWorld():GetSpawnZ()
	local Bounds = cCuboid(SpawnX - 25, SpawnY - 25, SpawnZ - 25, SpawnX + 25, SpawnY + 25, SpawnZ + 25)
	Bounds:ClampY(0, 255)

	-- Get a list of chunks that we need loaded:
	local MinChunkX = math.floor((SpawnX - 25) / 16)
	local MaxChunkX = math.ceil ((SpawnX + 25) / 16)
	local MinChunkZ = math.floor((SpawnZ - 25) / 16)
	local MaxChunkZ = math.ceil ((SpawnZ + 25) / 16)
	local Chunks = {}
	for x = MinChunkX, MaxChunkX do
		for z = MinChunkZ, MaxChunkZ do
			table.insert(Chunks, {x, z})
		end
	end  -- for x

	-- Store the player's name and world to use in the callback, because the a_Player object may no longer be valid:
	local PlayerName = a_Player:GetName()
	local World = a_Player:GetWorld()

	-- This is the callback that is executed once all the chunks are loaded:
	local OnAllChunksAvailable = function()
		-- Read the area around spawn into a cBlockArea, save to file:
		local Area = cBlockArea()
		local FileName = PlayerName .. "_spawn.schematic"
		if (Area:Read(World, Bounds, cBlockArea.baTypes + cBlockArea.baMetas)) then
			Area:SaveToSchematicFile(FileName)
			Msg = cCompositeChat("The spawn has been saved", mtInfo)
		else
			Msg = cCompositeChat("Cannot save the spawn", mtFailure)
		end

		-- Notify the player:
		-- Note that we cannot use a_Player here, because it may no longer be valid (if the player disconnected before the command completes)
		World:DoWithPlayer(PlayerName,
			function (a_CBPlayer)
				a_CBPlayer:SendMessage(Msg)
			end
		)
	end

	-- Ask the server to load our chunks and notify us once it's done:
	World:ChunkStay(Chunks, nil, OnAllChunksAvailable)

	-- Note that code here may get executed before the callback is called!
	-- The ChunkStay says "once you have the chunks", not "wait until you have the chunks"
	-- So you can't notify the player here, because the saving needn't have occurred yet.

	return true
end
```
请注意，这段代码会对 Area:Read() 函数进行错误检查，除非确实有正确的数据，否则不会覆盖之前的文件。如果你想知道我们已经加载了大块数据，读取怎么会失败，那么还有一个可用内存的问题--如果无法分配区域内存，即使所有大块数据都存在，也无法读取。因此，我们仍然需要进行检查。
## 结论
虽然 ChunkStay 会让代码变得稍长一些，一开始也更难掌握，但它是一项非常有用的技术。当你需要访问可能无法访问的数据块，并且确实需要数据时，就可以使用它。

使用 ChunkStay 可能遇到的最大障碍是它会在后台工作，从而使函数可能持有的所有 cPlayer 和 cEntity 对象失效，因此需要重新获取它们的 ID 和名称。这就是使用多线程代码的代价。
