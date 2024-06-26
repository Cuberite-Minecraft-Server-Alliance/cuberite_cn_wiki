## 3 - 在没有插件的情况下连接世界

你可以通过修改文件`world.ini`来轻松地连接到一个世界，而无需使用额外的插件。但是，这种方法具有一定的限制：每个世界最多只能与两个不同的世界进行连接。

默认情况下，主世界与两个世界进行了连接：下界和末地。正常情况下，我们可以通过任何一个下界传送门进入下界，通过任何一个末地传送门进入末地。当然，你也可以调整下界和末地传送门的行为，并使每种传送门类型传送到你选择的世界。这可以通过编辑每个世界的world.ini文件中[LinkedWorlds]部分来实现。需要注意的是，使用这种方法，你无法通过两个同一类型的不同传送门将你传送到两个不同的世界。如果你想要这样的行为，你应该使用一个插件。请参阅下一小节。
