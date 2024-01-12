# 1. 引用部分

```python
import inspect
import re
import six
```

1. `inspect` 模块：
   - `inspect` 模块提供了用于检查源文件中定义的类和函数的工具。它可以帮助程序员获取类和函数的参数、源代码、注释等信息，以及进行堆栈检查和解释器内省。这样可以在运行时获取或处理函数和类的信息，实现一些动态的操作。
2. `re` 模块：
   - `re` 模块是 Python 中用于操作正则表达式的模块。它提供了用于对字符串进行模式匹配和搜索、替换等操作的函数和工具。通过 `re` 模块，可以进行强大的字符串处理和匹配。
3. `six` 模块：
   - `six` 是一个用于在 Python 2 和 Python 3 之间进行兼容性处理的库。它提供了一些函数和工具，用于简化在不同 Python 版本中编写兼容性代码的工作。例如，可以使用 `six.moves` 模块来调用 Python 2 和 Python 3 中相同的模块和函数；还可以使用 `six.PY2` 和 `six.PY3` 来检查当前代码运行的 Python 版本，以便根据不同版本采取不同的行为。



# 2. class ExecutionError(Exception)

```python
class ExecutionError(Exception):
    pass
```

在 Python 中，可以通过定义自定义的异常类来表示特定的错误或异常情况。

通常情况下，自定义的异常类都是继承自内建的异常类（比如 `Exception`、`ValueError`、`TypeError`等）。通过继承内建异常类，可以利用异常处理机制对特定类型的错误进行捕获和处理。

在这个例子中，`ExecutionError` 继承自 `Exception`，这意味着它具有 `Exception` 的所有特性和行为。当发生与程序执行相关的错误时，可以引发 `ExecutionError` 异常实例，然后在代码的其他地方使用 `try...except` 块来捕获并处理该异常。

这个类定义中的 `pass` 语句表示类体为空。这意味着 `ExecutionError` 类本身没有定义任何额外的属性或方法，它只是简单地继承自 `Exception`，并且会继承 `Exception`类的所有方法和行为







# 3. class ConfigNode(object)

- 这里的 object 是 pytho2 中的写法，表示这个 ConfugNode 是一个基类





## 3.1 函数说明与变量定义

```python
	'''
    The ConfigNode class defines a common skeleton to be used by specific
    implementation.
    // 这句话解释了作者使用了一个非 python 语言中常用的术语"purely virtual"
    It is "purely virtual" (sorry for using non-pythonic vocabulary there ;-) ).
    '''
    # 分隔符
    _path_separator = '/'
    # 当前路径
    _path_current = '.'
    # fu
    _path_previous = '..'

    ui_type_method_prefix = "ui_type_"
    ui_command_method_prefix = "ui_command_"
    ui_complete_method_prefix = "ui_complete_"
    ui_setgroup_method_prefix = "ui_setgroup_"
    ui_getgroup_method_prefix = "ui_getgroup_"
```

- 作者说明 ConfigNode 是一个通用的结构，可以被特定的实现类继承和使用

- 在 C++ 和一些其他面向对象编程语言中，“purely virtual” 通常指的是纯虚拟类，即只有接口声明而没有具体实现的类。在 Python 中并没有严格的“纯虚拟类”这个概念，但通过作者的说明我们可以理解，作者是希望读者理解这个类可能没有完整的实现，只提供了一个框架或接口供其他类来实现。





## 3.2 `def __init__()`

 ```python
     def __init__(self, name, parent=None, shell=None):
         '''
         # 如果一个新对象的父 ConfigNode 为 Node，那么这个新对象就是一个新的根节点
         @param parent: The parent ConfigNode of the new object. If None, then
         the ConfigNode will be a root node.
         # 父对象类型：ConfigNode 或者 None（自己本身就是根节点）
         @type parent: ConfigNode or None
         @param shell: The shell to attach a root node to.
         @type shell: ConfigShell
         '''
         
         self._name = name
         self._children = set([])
         # 根节点必须有与之关联的 shell
         if parent is None:
             if shell is None:
                 raise ValueError("A root ConfigNode must have a shell.")
             else:
                 self._parent = None
                 self._shell = shell
                 shell.attach_root_node(self)
         else:
             # 如果该节点不是根节点，就不能有自己的 shell
             if shell is None:
                 self._parent = parent
                 self._shell = None
             else:
                 raise ValueError("A non-root ConfigNode can't have a shell.")
 		
         # 检查是不是根节点，如果不是就继续执行
         if self._parent is not None:
             # 遍历父节点的子节点集合（相当于查看所有兄弟节点）
             for sibling in self._parent._children:
                 # 比较兄弟节点和自己的名称是否重名
                 if sibling.name == name:
                     # 如果重名就报错
                     raise ValueError("Name '%s' already used by a sibling."
                                      % self._name)
             # 如果不重名，就将自己添加进父节点的子节点集合中，确保了在同一级别下节点名称的唯一性
             self._parent._children.add(self)
 		
         # 创建一个空字典，用于存储配置参数组
         self._configuration_groups = {}
 		# 之后的一系列 self.define_config_group_param 方法是用于定义特定的配置参数
         # 每次调用这个方法，都会为配置参数组添加一个新的参数
         
         # 例如，此方法就添加了名为 "tree_round_nodes" bool 类型参数，后面的 "Tree node 			# display style" 是该参数的描述信息
         self.define_config_group_param(
             'global', 'tree_round_nodes', 'bool',
             'Tree node display style.')
         # 是否展示 tree 的状态
         self.define_config_group_param(
             'global', 'tree_status_mode', 'bool',
             'Whether or not to display status in tree.')
         # 显示节点树的最大深度
         self.define_config_group_param(
             'global', 'tree_max_depth', 'number',
             'Maximum depth of displayed node tree.')
         # 是否展示根节点
         self.define_config_group_param(
             'global', 'tree_show_root', 'bool',
             'Whether or not to display tree root.')
         # console 的颜色模式
         self.define_config_group_param(
             'global', 'color_mode', 'bool',
             'Console color display mode.')
         # 日志消息到控制台的级别
         self.define_config_group_param(
             'global', 'loglevel_console', 'loglevel',
             'Log level for messages going to the console.')
         # 日志消息到日志文件的级别
         self.define_config_group_param(
             'global', 'loglevel_file', 'loglevel',
             'Log level for messages going to the log file.')
         # 用于指定要使用的日志文件
         self.define_config_group_param(
             'global', 'logfile', 'string',
             'Logfile to use.')
         # 默认的文本显示颜色样式
         self.define_config_group_param(
             'global', 'color_default', 'colordefault',
             'Default text display color.')
         # 路径补全的颜色样式
         self.define_config_group_param(
             'global', 'color_path', 'color',
             'Color to use for path completions')
         # 命令补全的颜色样式
         self.define_config_group_param(
             'global', 'color_command', 'color',
             'Color to use for command completions.')
         # 参数补全的颜色样式
         self.define_config_group_param(
             'global', 'color_parameter', 'color',
             'Color to use for parameter completions.')
         # 关键字补全的颜色样式
         self.define_config_group_param(
             'global', 'color_keyword', 'color',
             'Color to use for keyword completions.')
         # shell 提示路径的最大长度，0 表示无限制
         self.define_config_group_param(
             'global', 'prompt_length', 'number',
             'Max length of the shell prompt path, 0 for infinite.')
         
 		# 如果用户没有 bookmarks，就设置为空
         if self.shell.prefs['bookmarks'] is None:
             self.shell.prefs['bookmarks'] = {}
 ```



## 3.3 `def ui_type_number()`

```python
def ui_type_number(self, value=None, enum=False, reverse=False):
        '''
        UI parameter type helper for number parameter type.
        # 指出该参数要进行类型检查的值
        @param value: Value to check against the type.
        # 指出该参数可以是任何类型的值
        @type value: anything
        # 只有当 value 被省略时，enum 才有意义；如果设置了这个参数，就返回类型可能的取值列表或者一个空列表；如果没有设置，就返回类型格式的文本描述
        @param enum: Has a meaning only if value is omitted. If set, returns
        a list of the possible values for the type, or [] if this is not
        possible. If not set, returns a text description of the type format.
        # enum：bool 类型
        @type enum: bool
        # 如果设置了 reverse，就会将内部值转为 UI 字符串表示？
        @param reverse: If set, translates an internal value to its UI
        string representation.
        @type reverse: bool
        @return: c.f. parameter enum description.
        @rtype: str|list|None
        # 如果值与类型不匹配就抛出异常
        @raise ValueError: If the value does not check ok against the type.
        '''
        if reverse:
            if value is not None:
                return str(value)
            else:
                return 'n/a'

        type_enum = []
        syntax = "NUMBER"
        if value is None:
            if enum:
                return type_enum
            else:
                return syntax
        elif not value:
            return None
        else:
            try:
                value = int(value)
            except ValueError:
                raise ValueError("Syntax error, '%s' is not a %s."
                                 % (value, syntax))
            else:
                return value
```

