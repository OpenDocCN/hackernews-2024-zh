<!--yml

category: 未分类

date: 2024-05-27 14:56:28

-->

# 内部平台效应 - Wikipedia

> 来源：[https://en.wikipedia.org/wiki/Inner-platform_effect](https://en.wikipedia.org/wiki/Inner-platform_effect)

软件架构师倾向于复制他们的开发平台的倾向

**内部平台效应**是软件架构师倾向于创建一个可定制到成为软件开发平台副本（通常是低质量的）的系统。这通常是低效的，并且这样的系统通常被认为是[反模式](/wiki/Anti-pattern "Anti-pattern")的例子。

## 例子[[编辑](/w/index.php?title=Inner-platform_effect&action=edit&section=1 "编辑章节：例子")]

软件插件化的例子可以在基于插件的软件中看到，比如一些文本编辑器和网页浏览器，这些通常有开发人员创建插件，重新生成通常在操作系统上运行的软件。[Firefox](/wiki/Firefox "Firefox") 的插件机制已被用来开发多个[FTP](/wiki/File_Transfer_Protocol "File Transfer Protocol")客户端和[file browsers](/wiki/File_browser "File browser")，有效地复制了[操作系统](/wiki/Operating_system "Operating system")的部分功能，尽管在一个更受限制的平台上。

在[数据库](/wiki/Database "Database")领域，开发人员有时会试图绕过[RDBMS](/wiki/RDBMS "RDBMS")，例如将所有内容存储在一个带有三个[列](/wiki/Column_(database) "Column (database)")（实体ID、键和值）的大[表](/wiki/Table_(database) "Table (database)")中。虽然这种[实体-属性-值模型](/wiki/Entity-attribute-value_model "Entity-attribute-value model")允许开发人员摆脱[SQL](/wiki/SQL "SQL")数据库所施加的结构，但失去了所有[索引](/wiki/Index_(database) "Index (database)")和[查询优化器](/wiki/Query_optimizer "Query optimizer")等的好处，^([[1]](#cite_note-1)) 因为所有可以由RDBMS高效执行的工作都被迫由应用程序来完成。查询变得更加复杂，^([[2]](#cite_note-2)) 数据有效性约束也不再强制执行。性能和可维护性可能极差。

XML中也存在类似的诱惑，开发人员有时倾向于使用通用的元素名称，并使用属性存储有意义的信息。例如，每个元素都可能命名为*item*，并具有*type*和*value*属性。这种做法需要跨多个属性进行[连接](/wiki/Join_(relational_algebra) "Join (relational algebra")来提取意义。因此，[XPath](/wiki/XPath "XPath")表达式更加复杂，评估效率较低，并且结构验证提供的好处有限。

另一个例子是[Web桌面](/wiki/Web_desktop "Web desktop")现象，其中整个[桌面环境](/wiki/Desktop_environment "Desktop environment")——通常包括一个[Web浏览器](/wiki/Web_browser "Web browser")——在浏览器内运行（浏览器本身通常运行于由[操作系统](/wiki/Operating_system "Operating system")提供的桌面环境中）。桌面内的桌面对用户来说通常非常尴尬，因此通常只用来运行无法轻松部署到最终用户系统上的程序，或者通过隐藏外层桌面实现。

软件开发人员创建与其特定项目相关的自定义函数库是很正常的。当此库扩展到包括重复已作为编程语言或平台一部分可用的通用功能时，就会出现内部平台效应。由于这些新函数通常会调用多个原始函数之一，它们往往运行较慢，如果编码不当，还可能不够可靠。^([*[需要引用](/wiki/Wikipedia:Citation_needed "Wikipedia:Citation_needed")*])

另一方面，这类函数通常是为了在较低级别服务之上提供一个更简单（通常更可移植）的抽象层而创建的，这些服务要么具有笨拙的接口，要么过于复杂，不可移植或者移植性不足，或者与更高级别的应用代码简单地不匹配。

## 适当的用途[[编辑](/w/index.php?title=Inner-platform_effect&action=edit&section=3 "编辑章节：适当的用途")]

内部平台可以出于可移植性和权限分离的原因而有用——换句话说，使同一应用程序可以在多种外部平台上运行，而不影响内部平台管理的[沙盒](/wiki/Sandbox_(computer_security) "Sandbox (computer security")之外的任何内容。例如，Sun Microsystems设计了[Java平台](/wiki/Java_(software_platform) "Java (software platform")以满足这两个目标。

## 另请参见[[编辑](/w/index.php?title=Inner-platform_effect&action=edit&section=4 "编辑章节：另请参见")]

## 参考资料[[编辑](/w/index.php?title=Inner-platform_effect&action=edit&section=5 "编辑章节：参考资料")]

## 外部链接[[编辑](/w/index.php?title=Inner-platform_effect&action=edit&section=6 "编辑章节：外部链接")]
