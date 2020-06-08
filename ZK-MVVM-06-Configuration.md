# **Configuration**

## **Register Application Level Converters(注册应用级转换器)**

> Available for ZK: EE
>
> Since 6.0.1

您可以通过在zk.xml中设置library-property（org.zkoss.bind.appConverters）来注册应用程序级别转换器[1]。

```xml
<library-property>
    <name>org.zkoss.bind.appConverters</name>
    <value>foo=my.FooConverter,bar=my.BarConverter</value>
</library-property>
```

然后按转换器名称使用它们。

```xml
<label value="@load(vm.message) @converter('foo')"/> 
<label value="@load(vm.message) @converter('bar')"/>
```

> [1]：应用程序级别转换器只有一个实例，并且在所有绑定程序之间共享。



## **Register Application Level Validators(注册应用级验证器)**

> Available for ZK: EE
>
> Since 6.0.1

您可以通过在zk.xml中设置library-property（org.zkoss.bind.appValidators）来注册应用程序级别验证程序[1]。

```xml
<library-property>
    <name>org.zkoss.bind.appValidators</name>
    <value>foo=my.FooValidator,bar=my.BarValidator</value>
</library-property>
```

然后按验证者名称使用它们。

```xml
<textbox value="@bind(vm.name) @validator('foo')"/> 
<textbox value="@bind(vm.value) @validator('bar')"/>
```

> [1]：应用程序级别验证程序只有一个实例，并且在所有绑定程序之间共享。

## **Print ZK Bind Debugging Information(打印ZK绑定调试信息)**

> Since 6.5.2

打开打印ZK绑定调试信息会在运行时打印很多消息，这可能有助于调试。请不要在生产环境中启用它。

要启用它，请在zk.xml中添加以下配置（默认为false）：

```xml
<library-property>
    <name>org.zkoss.bind.DebuggerFactory.enable</name>
    <value>true</value>
</library-property>
```

打印的消息如下所示：

```
[0]ADD-BINDING[add-binding:prop-load] [0]ADD-BINDING[add-binding:prop-load] [0]LOAD_BINDING
[0] *[load:prop-init] vm.element.name [0]LOAD_BINDING
[0] *[load:prop-load] vm.element.name [0]LOAD_BINDING
[0] *[load:prop-load] vm.element.name [0]LOAD_BINDING
vm.escValue1 > value <label uuid="j5VUn" id="" /> vm.escValue2 > value <label uuid="j5VUo" id="" />
at [file:/data/zk/git/zk/zktest/src/a at [file:/data/zk/git/zk/zktest/src/a
[0] *[load:prop-load] vm.escValue1 > value ' [0]LOAD_BINDING
[0] *[load:prop-load] vm.escValue2 > value "
======================================= [6431]ON_EVENT
<label uuid="j5VUn" id="" /> <label uuid="j5VUo" id="" />
*[event] [onClick]
+
<button uuid="j5VUl" id="" />
[onClick] ['cmd1'] cmd1 <button uuid="j5VUl" id="" />
vm.validator1 org.zkoss.zktest.bind.basic.AllFunctionVM$1@6ec135d6 result = true <textbox uuid=
[before = 'cmd1'] value > vm.element.name A <textbox uuid="j5VUh" id="" />
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431]
[6431] [6431]NOTIFY_CHANGE
+
[after = 'cmd1'] vm.element.name > value A <label uuid="j5VUj" id="" /> *[command:post-global] [onClick] ['gcmd1'] gcmd1 <button uuid="j5VUl" id="" />
COMMAND *[command:on-command] + VALIDATE
*[validation:prop] + SAVE_BEFORE
+ SAVE_BINDING *[save:prop-save]
+ LOAD_BEFORE
+ EXECUTE
*[command:execute] cmd1
+ SAVE_AFTER
+ LOAD_AFTER
<button uuid="j5VUl" id="" />
public void org.zkoss.zktest.bind.basic.AllFunctionVM.cmd1()
+ LOAD_BINDING *[load:prop-load]
POST_GLOBAL_COMMAND
[6431]
[6431]
[6431]
[6431]
[6431] [6431]GLOBAL_COMMAND
vm.element.name > value <label uuid="j5VUb" id="" /> vm.element.name > value <textbox uuid="j5VUd" id="" /> value > vm.element.name <textbox uuid="j5VUd" id="" /> vm.element.name > value <label uuid="j5VUf" id="" />
at [file:/data/zk/git/zk/zktest/src/a at [file:/data/zk/git/zk/zktest/src/a at [file:/data/zk/git/zk/zktest/src/a at [file:/data/zk/git/zk/zktest/src/a
[before = 'cmd1'] [before = 'cmd2'] [after = 'cmd1']
value > vm.element.name <textbox uuid="j5VUh" id="" />
value > vm.element.name <textbox uuid="j5VUh" id="" />
vm.element.name > value <label uuid="j5VUj" id="" />
'cmd1' <button uuid="j5VUl" id="" /> at [file:/data/zk/git/zk/zktest/s
> value > value > value
item 1 <label uuid="j5VUb" id="" /> item 1 <textbox uuid="j5VUd" id="" /> item 1 <label uuid="j5VUf" id="" />
*[notify-change] [org.zkoss.zktest.bind.basic.AllFunctionVM$Element@41a8dfb3][name] <window uuid="j5VU0" id="" /> Size=2 + LOAD_BINDING
*[load:prop-load] vm.element.name > + LOAD_BINDING
value A <textbox uuid="j5VUd" id="" /> value A <label uuid="j5VUf" id="" />
*[load:prop-load] vm.element.name >
*[command:on-command-global] gcmd1 + EXECUTE
[6431]
[6431]
[6431] [6431]NOTIFY_CHANGE
[6431] [6431] [6431] [6431] [6431]
*[notify-change] [org.zkoss.zktest.bind.basic.AllFunctionVM$Element@41a8dfb3][*] <window uuid="j5VU0" id="" /> Size=2 + LOAD_BINDING
*[load:prop-load] vm.element.name > value A-GCMD1 <textbox uuid="j5VUd" id="" /> + LOAD_BINDING
*[load:prop-load] vm.element.name > value A-GCMD1 <label uuid="j5VUf" id="" />
```

## **Debugging Tool: ZK Binding Tracker(调试工具:绑定跟踪器)**

上面的配置将在服务器控制台上打印消息，并且它是系统范围的，这意味着将打印所有页面的调试信息，并且可能变得难以阅读。 因此，我们在此为您提供一个工具：ZK Binding Tracker，它是可以安装在Chrome浏览器中的Chrome扩展程序。 它可以以丰富的格式显示当前ZUL页面的调试信息。

![image-20200607141952791](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162331.png)

更多详细信息,请参阅: [Small_Talks](http://books.zkoss.org/wiki/Small_Talks/2013/June/ZK_Binding_Tracker_-_A_Chrome_Extension)/2013/June/ZK_Binding_Tracker_-_A_Chrome_Extension.

