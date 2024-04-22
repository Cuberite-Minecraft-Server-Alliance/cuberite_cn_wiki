## 简介
长期以来，Cuberite 插件一直受到文档不完善的困扰。编写插件的人知道如何使用这些插件，但对于插件的新使用者来说，学习如何使用这些插件却是一种可怕的折磨。大多数情况下，插件作者只写了插件支持哪些命令，有时甚至连这些都不写。因此，人们呼吁采取行动结束这种状况，让插件文档的编写变得简单，同时也能集中管理。于是，Info.lua 文件诞生了。

大多数插件的某些部分在所有插件中都是相同的。这些部分是命令、控制台命令及其权限。如果一个插件实现了一个命令，它实际上会重复复制和粘贴相同的代码。因此，只提取独一无二的信息，将其集中起来，并将其周围的所有部分自动化是非常有意义的。这也是编写 Info.lua 文件的另一个原因--它是命令、控制台命令及其权限的中心枢纽。

最后但并非最不重要的一点是，我们希望将来能在网络上建立一个插件库，一个存储插件、插件描述和评论的库。信息库可以自动解析这些集中的信息，从而实现高级功能，例如根据命令搜索插件，或判断两个插件在命令上是否冲突。

我们已经编写了一个工具，可以轻松生成各种格式的插件文档。它能以最适合粘贴到论坛的格式输出文档。它可以生成标记格式的文档，用于 GitHub 和类似网站上的 README.md。巧妙之处在于，你无需手动保持所有这些格式同步，只需编辑 Info.lua 文件，该工具就会为你重新生成文档。
要为插件生成文档，请在安装了插件的 cuberite 服务器上激活 DumpInfo 插件，并使用 webadmin 界面 "转储 "插件信息。这将创建一个适合上传到 git 仓库的 README.md，以及一个可以复制粘贴到论坛帖子的 forum_info.txt。

总而言之，Info.lua 文件包含了插件的命令、控制台命令、它们的权限以及可能的整个插件文档，其结构可以被程序解析，但也可以被人类阅读和编辑。

## 整体结构
该文件包含一个 Lua 表 g_PluginInfo 的声明。该表包含作为其成员的所有结构化信息。每个成员本身都可以是一个结构。整个文件是一个有效的 Lua 源文件，因此任何语法检查 Lua 源文件的工具都可以对该文件进行语法检查。该文件具有一定的向前和向后兼容性，即可以任何方式进行扩展而不会中断。
下面是该文件的骨架：
``` bash
g_PluginInfo =
{
	Name = "Example Plugin",
	Date = "2014-06-12",
	Description = "This is an example plugin that shows how to use the Info.lua file",

	-- The following members will be documented in greater detail later:
	AdditionalInfo = {},
	Commands = {},
	ConsoleCommands = {},
	Permissions = {},
	Categories = {},
}
```
如您所见，结构非常简单。请注意，表内元素的顺序并不重要（Lua 属性）。

前几个元素用于记账。它们声明了插件的名称、ISO 格式的日期（代表插件的版本）和描述。我们的想法是，描述可以用两三句话概括插件的全部内容。

## 附加信息表
此表用于更详细地描述插件。如果有任何非繁琐的设置过程或依赖关系，请在此进行描述。这里的描述应该更加详细。如果能使插件更易于理解，也不必担心在此处使用几段文字。

表格应采用以下布局：
``` bash
AdditionalInfo =
{
	{
		Title = "Chapter 1",
		Contents = "Describe one big aspect of the plugin here",
	},
	{
		Title = "Chapter 2",
		Contents = "Describe another big topic",
	},
}
```
这里的想法是，用于从 Info.lua 文件生成文档的工具将创建一个链接化的目录，然后创建每个信息元素的内容。这些信息应该就是成功配置、运行和管理插件所需的全部内容。

命令表
命令表列出了插件实现的所有命令，以及它们的处理函数、所需权限、帮助字符串和更多信息。该表支持递归，允许插件轻松创建多字命令（如"//schematic load "和"//schematic save"），每个命令都有自己独立的处理程序。

该表使用的结构类似于下表：
``` bash
Commands =
{
	["/cmd1"] =
	{
		HelpString = "Performs the first action",
		Permission = "firstplugin.cmds.1",
		Alias = "/c1",
		Handler = HandleCmd1,
		ParameterCombinations =
		{
			{
				Params = "x y z",
				Help = "Performs the first action at the specified coordinates",
			},
			{
				Params = "-p",
				Help = "Performs the first action at the player's coordinates",
			}
		},
	},
	["/cmd2"] =
	{
		Alias = {"/c2", "//c2" },
		Category = "Something",
		Subcommands =
		{
			sub1 =  -- This declares a "/cmd2 sub1" command
			{
				HelpString = "Performs the second action's first subcommand",
				Permission = "firstplugin.cmds.2.1",
				Alias = "1",
				Handler = HandleCmd2Sub1,
				ParameterCombinations =
				{
					{
						Params = "x y z",
						Help = "Performs the second action's first subcommand at the specified coordinates",
					},
					{
						Params = "-p",
						Help = "Performs the second action's first subcommand at the player's coordinates",
					}
				},
			},
			sub2 =  -- Declares a "/cmd2 sub2" command
			{
				HelpString = "Performs the second action's second subcommand",
				Permission = "firstplugin.cmds.2.2",
				Handler = HandleCmd2Sub2,
			},
		},
	},
}
```
虽然一开始看起来可能会让人不知所措，但 "方法总比困难多"。命令表中的每个元素都定义了一条命令。大多数命令都以斜线开头，因此需要使用特殊的 Lua 语法（["/cmd1"] =）来处理具有非标准名称的表元素。命令既可以指定子命令，也可以指定处理函数（同时指定这两种函数将导致 UndefinedBehavior）。子命令使用与整个命令表相同的递归结构。

权限元素指定该命令只有在指定权限的情况下才能使用。请注意，调用子命令时不会检查子命令父级的权限。这意味着，为有子命令的命令指定权限不会有任何影响，但我们不鼓励这样做，因为我们将来可能会添加相关处理。

可选的类别表为生成的文档中的命令类别提供了说明。文档生成器将按指定的类别（默认为 "常规"）对命令进行分组，每个类别都将写入指定的说明。

ParameterCombinations 表仅用于生成文档，它列出了命令支持的各种参数组合。即使命令只支持一种组合，也值得指定，因为这种组合将被记录在案。

别名成员指定命令的任何可能别名。每个别名都会单独注册，如果有子命令表，则会将其应用到所有别名，正如人们所期望的那样。您可以指定一个字符串作为值（如果只有一个别名），也可以为多个别名指定一个字符串表。没有别名的命令无需指定该成员。

控制台命令表
该表的作用与命令表类似，只是这些命令是为服务器控制台提供的。因此，没有为这些命令指定权限。由于大多数控制台命令不使用前导斜线，因此命令名称不需要使用特殊语法。此外，处理函数不会接收 Player 参数。

下面是 ConsoleCommands 表的示例：
``` bash
ConsoleCommands =
{
	concmd =
	{
		HelpString = "Performs the console action",
		Subcommands =
		{
			sub1 =
			{
				HelpString = "Performs the console action's first subcommand",
				Handler = HandleConCmdSub1,
				ParameterCombinations =
				{
					{
						Params = "x y z",
						Help = "Performs the console action's first subcommand at the specified coordinates",
					},
				},
			},
			sub2 =
			{
				HelpString = "Performs the console action's second subcommand",
				Handler = HandleConCmdSub2,
			},
		},
	},
}
Permissions table
The purpose of this table is to document permissions that the plugin uses. The documentation generator automatically collects the permissions specified in the Command table; the Permissions table adds a description for these permissions and may declare other permissions that aren't specifically included in the Command table.

Permissions =
{
	["firstplugin.cmd.1.1"] =
	{
		Description = "Allows the players to build high towers using the first action.",
		RecommendedGroups = "players",
	},
	["firstplugin.cmd.2.1"] =
	{
		Description = "Allows the players to kill entities using the second action. Note that this may be misused to kill other players, too.",
		RecommendedGroups = "admins, mods",
	},
}
The RecommendedGroup element lists, in plain English, the intended groups for which the permission should be enabled on a typical server. Plugin authors are advised to create reasonable defaults, prefering security to openness, so that admins using these settings blindly don't expose their servers to malicious users.

Categories
The optional Categories table provides descriptions for categories in the generated documentation. Commands can have categories with or without category descriptions in this table. The documentation generator will output a table of listed categories along with their description.

Categories = 
{
	General =
	{
		Description = "A general, yet somehow vague description of the default category."
	},
	Something =
	{
		Description = "Some descriptive words which form sentences pertaining to this set of commands use and goals."
	},
},
```
## 在代码中使用文件
仅仅编写 Info.lua 文件并将其保存到插件文件夹中，还不足以让它被实际使用。你的插件需要包含以下模板代码，最好是在其 Initialize() 函数中：

``` bash
-- 使用 InfoReg 共享库处理 Info.lua 文件：
dofile(cPluginManager:GetPluginsPath() .. "/InfoReg.lua")
RegisterPluginInfoCommands()
RegisterPluginInfoConsoleCommands()
```
当然，如果您的插件没有任何控制台命令，它就不需要调用 RegisterPluginInfoConsoleCommands() 函数，同样，如果它没有任何游戏内命令，它也不需要调用 RegisterPluginInfoCommands() 函数。

示例
有几个插件已经实现了这种方法。您可以访问它们以获得灵感，并查看生成的文档是什么样子的：

Gallery 插件： [Info.lua](https://github.com/cuberite/gallery/blob/master/Info.lua)、[论坛](https://forum.cuberite.org/thread-1306.html)文档
WorldEdit 插件： [Info.lua](https://github.com/cuberite/WorldEdit/blob/master/Info.lua)、[论坛](https://forum.cuberite.org/thread-870.html)和 [MarkDown](https://github.com/cuberite/WorldEdit) 文档
