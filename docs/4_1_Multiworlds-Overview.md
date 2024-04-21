## 1 - 多世界概述

Cuberite支持多个世界。每个世界都有自己的`world.ini`文件。可以通过编辑`settings.ini`文件来添加其他的世界。下面的示例中对此进行了解释。

### 添加世界

``` ini
[Worlds]
DefaultWorld=world
World=world_nether
World=world_the_end
World=myNewWorld
World=HappyLand
```

在上面的示例中，向服务器添加了2个额外的世界。请注意，这将自动创建2个额外的配置文件，即`myNewWorld/world.ini`和`HappyLand/world.ini`。

### 维度（世界类型）

要创建一个下界类型的世界，您应该将_nether后缀添加到您的世界名称中，例如"`World=myHellishWorld_nether`"。这将创建一个预配置为下界的世界和`world.ini`。对于末地世界，亦是如此，将`_the_end`后缀添加到世界名称中。一旦创建了默认`world.ini`，你可以根据自己的喜好进行调教。

当不存在world.ini时，"_nether"和"_the_end"后缀才会被使用，并指导服务器选择要创建的默认world.ini的内容。当存在world.ini时，后缀就不再重要了，维度由每个world.ini中的选项来确定。后缀的处理方式仅仅只是为了与之前的Cuberite版本兼容。

剩下的部分将涉及连接世界和在它们之间旅行的内容。
