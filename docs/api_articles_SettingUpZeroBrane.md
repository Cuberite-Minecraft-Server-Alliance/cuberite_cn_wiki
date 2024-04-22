## 设置 ZeroBrane Studio IDE
本文将介绍如何设置用于编写 Lua 代码的IDE ZeroBrane Studio，这样你就可以使用集成开发环境开发 Cuberite 插件了。
## 关于 ZeroBrane Studio
ZeroBrane Studio是一个用于编写Lua代码的集成开发环境。它具备集成开发环境所应具备的基本功能--它允许你以项目的形式管理文件组，你可以在标签式编辑器中编辑多个文件，代码是语法高亮显示的。代码自动补全、符号浏览等。它还有一个 Lua 调试器，允许你在任何使用 Lua 并能加载 Lua 包的应用程序中调试你的 Lua 代码。它使用多平台 WxWidgets 工具包编写，可在 Windows、Linux 和 MacOS 等多个平台上运行。

正如你所看到的，你可以在代码中设置断点，检查变量值，查看 Lua 调用栈。

ZBS 是开源的，源代码在 GitHub 上：https://github.com/pkulchenko/ZeroBraneStudio  。项目主页：https://studio.zerobrane.com/  。
## 首次设置
由于 ZBS 是一个通用的 Lua 集成开发环境，因此您需要首先对其进行设置，以便为 Cuberite 插件开发做好准备。为此，您需要从 ZBS 插件库中下载一个文件 cuberite.lua。将该文件放入 ZBS 文件夹中的 "packages "文件夹。请注意，插件库中还有其他有用的插件，您可能需要稍后再去看看，以便进一步定制您的 ZBS。要安装这些插件，只需将它们保存到同一文件夹即可。

下一步是安装 Cuberite 的代码自动完成支持。由于 API 会随着时间的推移而不断发展，因此会经常添加新的函数和类，因此应不时重复此步骤。您的 Cuberite 安装中应包含 APIDump 插件。在服务器设置中启用 APIDump 插件，这样做非常省钱，而且在正常游戏过程中也不会降低性能。要生成代码完成支持文件，请在服务器控制台中输入命令。这会在 Cuberite 可执行文件旁边创建一个新文件 "cuberite_api.lua"。将该文件移到 ZBS 文件夹内的 "api/lua "子文件夹中。(请注意，如果您以前的版本中有 "mcserver_api.lua "文件，则应将其删除）。

下载 cuberite.lua 文件并安装完成支持后，需要重启 ZBS 才能加载插件。如果没有错误，你应该会在 Project -> Lua Interpreter 子菜单中看到两个新项目： "Cuberite - 调试模式 "和 "Cuberite - 发布模式"。两者之间唯一的区别是启动 Cuberite 时使用的文件名--调试模式使用 cuberite_debug(.exe)，发布模式使用 "cuberite(.exe)"。如果您自己创建了 Cuberite 可执行文件，并且是在调试模式下创建的，则应选择调试模式选项。在所有其他情况下，包括从网上下载已编译好的 Cuberite 可执行文件，都应选择发布模式选项。

对于初次使用 Cuberite 的用户来说，ZBS 中没有图形用户界面设置可能会有点不知所措，但集成开发环境的可配置性很强。你可以编辑两个文件来更改设置，可以是全系统范围的（计算机的所有用户共享这些设置），也可以是全用户范围的（这些设置只针对计算机的特定用户）。这些文件是普通的 Lua 源文件，您可以在集成开发环境中快速找到并编辑它们，从菜单中选择 "编辑"->"首选项"->"设置"： 从菜单中选择 XYZ，XYZ 可以是系统或用户。

ZBS 网页上有关于大多数设置的文档，请访问 https://studio.zerobrane.com/documentation.html  ，尤其是首选项部分。我个人建议将 editor.usetabs 设置为 true，并在可能的情况下调整 editor.tabwidth，关闭 editor.smartindent 功能，调试时应将 debugger.alloweditting 选项设置为 true，除非你想惩罚自己。
## 项目管理
ZBS 使用项目工作，它将特定文件夹中的所有文件和子文件夹都视为一个项目。无需专门的项目文件，也无需将单个文件添加到工作区，所有文件都会自动添加。要将 Cuberite 插件作为项目打开，请单击 "项目 "窗格中的三点按钮，或从菜单中选择 "项目"->"项目目录"->"选择..."。浏览并选择 Cuberite 插件的文件夹。ZBS 将加载插件文件夹中的所有文件，然后您就可以开始编辑代码了。

请注意，虽然 ZBS 允许您在插件中使用子文件夹（您应该这样做，尤其是大型插件），但除非您在编辑器中打开了 Cuberite 插件文件夹根目录下的文件，否则当前的 Cuberite ZBS 插件将无法开始调试。

调试
现在您可以调试代码了。不过在此之前，别忘了保存项目文件。如果尚未启用插件，请在 settings.ini 文件中启用插件。如果想让程序在某一行中断，最好在启动程序前设置断点。将光标放在该行上，然后点击 F9（或使用菜单 Project -> Toggle Breakpoint）在该行上切换断点。最后，点击 F5，或选择菜单 Project -> Start Debugging，在调试器下启动 Cuberite。Cuberite 窗口会弹出并加载插件。如果窗口没有出现，请检查 ZBS 中的 "输出 "窗格，失败的原因通常有两个：

当前打开文件中的代码有语法错误。这些错误在输出窗格中报告为 "编译错误"，双击该行可查看错误信息
ZBS 无法找到 Cuberite 可执行文件。确保您编辑的文件位于 Cuberite 可执行文件文件夹层次结构的下两三级，并且 Cuberite 可执行文件的命名正确（cuberite[.exe] 或 cuberite_debug[.exe]）。还要确保选择了正确的解释器（菜单 Project -> Lua Interpreter）。

运行时，如果执行过程中遇到断点，ZBS 窗口就会出现，断点行旁边会显示一个绿色箭头。您可以使用 F10（Step Into）和 Shift+F10 （Step Over）逐步查看代码。你还可以使用观察窗口检查变量值，或者直接将鼠标悬停在变量上，在工具提示中显示变量值。使用远程控制台窗格可直接在 Cuberite 的插件上下文中执行命令。

还可以使用 Project -> Break 菜单项，尽快进入调试器。还可以在 Cuberite 插件运行时设置断点。请注意，由于调试器的实现方式，Cuberite 可能会在中断/断点生效前执行更多的 Lua 代码。如果 Cuberite 没有在插件中执行任何 Lua 代码，它将不会中断，直到插件代码再次启动。这可能会导致错过断点，并延迟断点命令生效。因此，最好在运行程序前设置断点，或在程序等待另一个断点时设置断点。
