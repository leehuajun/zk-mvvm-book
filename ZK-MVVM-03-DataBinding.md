# 数据绑定（Data Binding）

## Overview

当使用MVVM模式开发Web应用程序时，数据绑定机制在View和ViewModel之间的数据同步中起着关键作用。

 

数据绑定是一种机制，可确保根据预定义的绑定关系将对UI组件中的数据所做的任何更改自动传递到目标ViewModel。 （反之亦然）。 应用程序开发人员只需通过数据绑定注解表达式来定义UI组件的属性和目标对象（通常是ViewModel）之间的数据绑定关系。

 

`ZK Bind`是一种全新的数据绑定机制，具有新的规范和实现。 基于从数据绑定1中获得的经验，并考虑到用户和贡献者的建议和反馈，我们在ZK6中创建了这个易于使用，灵活，功能丰富的新数据绑定系统。

![image-20200607140456327](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161928.png)

开发人员使用ZK绑定注解定义各种数据绑定关系。绑定源是组件的属性，绑定目标是ViewModel的属性或Command。 当我们将组件绑定到ViewModel时，该组件将成为ViewModel的Root View组件。 可以通过ZK绑定注解将所有子组件的属性和根视图组件本身绑定到ViewModel的属性（或Command）。 此根视图组件不必是页面的根组件

 

For example:

```xml
<vlayout>
    <vbox apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MainViewModel')">
        <label value="@load(vm.msg)"/>
    </vbox>
</vlayout>
```

> `vbox`是 ViewModel "foo.MainViewModel"的根视图组件
>
> label的value属性是绑定源（binding source）, ViewModel的msg属性是绑定目标 （binding target）

## 快速预览（Quick Preview）

### 注解表达式（Annotation Expression）

- zul中的Java样式注解表达式：新的ZK注解与Java的注解样式一致。 如果您知道Java样式，就知道ZK样式

- 一组协作注解：ZK Bind使用一组注解使数据绑定的使用尽可能直观和清晰

  @id(...)：								用于标识实例的id，例如视图模型或表单。

  @init(...)：							 用于初始化实例

  @load(...)：						   用于绑定数据和命令以及将数据加载到目标的参数

  @save(...)：						   用于绑定数据和命令以及用于保存数据的参数。 

  @bind(...)：							用于将数据与参数绑定在一起以进行加载和保存。

  @ref(...)：							  用于引用表达式（自6.0.1开始）。 

  @converter(...)：				用于指定转换器以及参数

  @validator(...)：				 用于指定验证器以及参数

  @command(...)：				用于将事件与参数绑定到命令。

  @global-command(...)：  用于将事件与参数绑定到全局命令。

  @template(...)：				 用于确定用于渲染容器组件的模板。

### 弹性表达（EL 2.2 Flexible Expressions）

- ZK Bind接受EL 2.2语法表达式，您可以在其中轻松提供灵活的操作。
- 无缝绑定到bean属性，索引属性和Map键。
- 自动绑定到组件自定义属性。
- 自动绑定到Spring，CDI和Seam托管bean。

```xml
<image src="@load(vm.person.boy ? 'boy.png' : 'girl.png')"/>
<button onClick="@command(vm.add ? 'add' : 'update')" label="@load(vm.add ? 'Add' : 'Update'"/>
<button onClick="@command('subscribe')" disabled="@load(empty vm.symbol)" label="Subscribe"/>
```

### 单向 Load

- Bean属性更改时加载
- 执行command之后/之前的条件加载
- 执行不同/相同command之前/之后的多个条件加载

```xml
<label value="@load(vm.person.fullname)"/>
<label value="@load(vm.person.firstname, after='update')"/>
<label value="@load(vm.person.firstname, before='delete')"/>
<label value="@load(vm.person.firstname, after={'update','delete'})"/>
<label value="@load(vm.person.firstname, after='update') @load(vm.person.message, after='delete')"/>
```

### 单向 Save

- UI组件属性更改时保存
- 多次保存到不同目标bean的属性
- 执行command之前/之后的条件保存
- 执行不同/相同command之前/之后的多个条件保存

```xml
<textbox value="@save(vm.person.firstname)"/>
<textbox value="@save(vm.person.firstname) @save(vm.tmpperson.firstname)"/>
<textbox value="@save(vm.person.firstname, before='update')"/>
<textbox value="@save(vm.person.firstname, after='delete')"/>
<textbox value="@save(vm.selected.firstname, before={'update','add'})"/>
```

### 初始绑定（Initial Binding）

在首次将UI组件添加到绑定系统时加载

```xml
<label value="@init(vm.selected.firstname)"/>
```

### 双向数据绑定

多个条件加载并在执行不同/相同commands之前/之后保存在不同的后端Bean上

```xml
<textbox value="@load(vm.selected.firstname) @save(vm.selected.firstname) @save(vm.newperson.firstname,before='add'）"/>
```

保存和加载绑定的简短表达式，无命令条件

```xml
<textbox value="@bind(vm.person.firstname)"/>
```

> `@bind(...)`是"`@load(...)  @save(...)`"的缩写，并且@save如果该属性不支持则`自动被忽略`

### 绑定到任何属性（Bind to Any Attributes）

绑定到UI组件的所有属性

```xml
<textbox value="@bind(vm.symbol)" instant="true"/>
<button disabled="@load(empty vm.symbol)" label="Subscribe" />
```

### 事件命令绑定（Event Command Binding）

- 将ZK事件桥接到命令
- 自动事件监听器注册
- 简单的命令名称调用

```xml
<button onClick="@command('subscribe')" disabled="@load(empty vm.symbol)" label="Subscribe" />
```

### 集合绑定（Collection Binding）

- 绑定Listbox/Grid/Tree/Combobox
- 局部变量范围仅限于容器组件
- 支持索引属性（以下示例为fooStatus.index）

```xml
<listbox width="300px" model="@load(vm.albumList)" selectedItem="@bind(vm.selectedAlbum)" vflex="true">
    <template name="model" var="foo">
        <listitem label="@load(foo.title)"/>
        <listitem label="${fooStatus.index}"/>
    </template>
</listbox>
```

### 引用绑定（Reference Binding）

> Since 6.0.1

- 使用自定义名称引用EL表达式
- 简写
- 模块化视图以使其可重用

```xml
<vlayout p="@ref(vm.person)">
    <hlayout>
        First Name:<textbox value="@bind(p.firstName)"/>
    </hlayout>
    <hlayout>
        Last Name:<textbox value="@bind(p.lastName)"/>
    </hlayout>
</vlayout>
```

### 嵌入式验证周期（Embedded Validation Cycle）

- 通过名称或EL表达式绑定验证器
- 嵌入式系统验证器：提供常用的验证器，用户只需指定名称即可直接在其中使用
- 验证单个属性或表单
- 验证command

```xml
<textbox value="@save(vm.selected.firstname) @validator('beanValidator')"/>
<grid form="@id('fx') @load(vm.selected) @save(vm.selected, before='update') @validator(vm.passwordValidator)">
    <row>username<textbox value="@bind(fx.username)"/></row>
    <row>password<textbox value="@bind(fx.password)" type="password"/></row>
    <row>retype password<textbox value="@bind(fx.retypePassword)" type="password"/></row>
</grid>
```

### 增强的转换器机制Enhanced Converter Mechanism）

- 按名称或EL表达式绑定转换器
- 嵌入式系统转换器：提供常用的转换器，用户只需指定名称即可直接使用

```xml
<datebox value="@bind(vm.selected.birthday) @converter('formattedDate', format='yyyy/MM/dd')"/>
```

### 动态模板（Dynamic Template）

动态确定要渲染的模板。

```xml
<grid model="@bind(vm.nodes) @template(vm.type='foo'?'template1':'template2')">
    <template name="template1">
        <!-- child components -->
    </template>

    <template name="template2">
        <!-- child components -->
    </template>
</grid>
```

### Debugging

ZK支持调试功能，例如打印可以手动启用的ZK绑定调试信息。 请参阅 MVVM/Configuration.

## EL表达式（EL Expression）

在ZK绑定注解中，我们采用EL表达式指定绑定目标并引用隐式对象。 绑定目标主要是ViewModel的（嵌套）属性。 您可以在`ZUL开发人员参考/ UIComposed / ZUML / EL表达式`部分中介绍的ZUL中使用EL表达式。 但是在ZK绑定注解中使用EL的格式和评估有些不同。

### 基础格式

所有ZK绑定注解都具有通用格式：

```xml
@[Annotation](value=[EL-expression], [arbitraryKey]=[EL-expression])
```

它以“ @”符号开头，后跟注解名称，例如“ id”或“ bind”。 在括号内，我们可以编写多个用逗号分隔的键/值对。 键是一个自定义名称（不是EL表达式），就像Map中的键一样。 该值是一个EL表达式，但不包含在“ $ {”和“}”中。 默认键名称为“值”。 如果仅编写EL表达式而未指定其键名，则将其隐式设置为名为“值”的键。 因此，在编写ZK绑定注解时，我们通常会忽略此默认键名称。 在大多数情况下，我们可以编写如下注解：

```
@[Annotation]( [EL-expression])
```

我们只需要编写一个键-值对并省略默认键名即可。 我们经常使用多个键值对将参数传递给命令，转换器和验证器。
尽管所有ZK绑定注解都具有通用格式，但绑定器解析和评估EL表达式的方式在不同注解之间是不同的。 我们将在以下各节中描述差异。

### 运行时评估

binder通常在每次要访问目标对象时都对EL表达式求值。 因此，评估结果可能有时会有所不同。

#### 根据运行时值进行命令绑定

```xml
<button label="Cmd" onClick="@command(vm.checked?'command1':'command2')"/>
<groupbox visible="@load(not empty vm.selected)"/>
```

> 单击该按钮时，绑定程序将根据“ `vm.checked`”的值执行命令。

#### 间接参考

```xml
<label value="@bind(vm.person[vm.currentField])"/>
```

> 如果`vm.currentField`的评估结果为`firstName`，则绑定程序将加载`vm.person.firstName`。 因此，binder加载的`vm.person属性`取决于`vm.currentField`的评估结果。

### 调用ViewModel方法

> 您可以使用EL表达式在ViewModel中调用方法。

#### 调用ViewModel的concat()

```xml
<label value="@load(vm.concat(vm.foo,'postfix'))"/>
```

#### 调用标签库的方法

我们可以在带有EL表达式的zul中平等地调用标签库的方法，也可以在数据绑定表达式中调用它们，并且此调用可以嵌套。 语法如下：

```xml
<?taglib uri="http://www.zkoss.org/dsp/web/core" prefix="c"?> 
        ...
<label value="@load(c:cat('A-',vm.value))" />
<label value="@load(c:cat3('$',c:formatNumber(vm.price, '00'), ' per person'))" />
```

> 第1行：声明核心方法的标签库。
> 第3行：核心的cat方法。
> 第4行：使用内核的cat3方法和formatNumber进行嵌套调用。

#### 调用Xel方法

xel方法允许开发人员在EL表达式中调用Java类的静态方法，而无需定义标签库。 您还可以在数据绑定表达式中使用此功能

```xml
<?xel-method prefix="x" name="max" class="java.lang.Math" signature="int max(int,int)"?>
        ...
<intbox value="@bind(vm.value1)"/>
<intbox value="@bind(vm.value2)"/>
<label value="@load(x:max(vm.value1,vm.value2))"/>
```

提示：某些EL字符在XML属性或ZK注解中是非法的，您应将其替换为其他EL运算符。

| Character | Replacement |
| --------- | ----------- |
| =         | eq          |
| !=        | ne          |
| &&        | and         |
| <         | lt          |
| <=        | le          |
| >         | gt          |
| >=        | ge          |

For example:

```xml
<image src="@load(vm.picture ne null ? 'images/'.concat(vm.picture) : 'images/NoImage.png')"/>
<label value="@bind(vm.age lt 18?'true':'false')"/>
```

### BindComposer

要在ZUL中启用数据绑定，必须在组件（此数据绑定的Root组件）上应用BindComposer。 BindComposer实现Composer，并充当激活组件及其子组件的数据绑定机制的角色。 它还会初始化Binder和ViewModel，并将ViewModel对象的引用传递给Binder。

#### 应用BindComposer（Apply BindComposer）

要使用ViewModel，您必须通过将“ org.zkoss.bind.BindComposer”设置为组件的“ apply”属性来应用BindComposer。

```xml
<window id="win" apply="org.zkoss.bind.BindComposer">
    <!-- other components inside will have data binding ability-->
</window>
```

> Since 8.0.0

在ZK 8中，当使用viewModel的属性时，将自动应用BindComposer。

#### 初始化ViewModel（Initialize a ViewModel）

您还必须指定ViewModel完全限定的类名，以对其进行初始化并为其指定ID。 典型用法如下：

```xml
<window id="win" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MyViewModel')">
    <label value="@load(vm.message)"/> <!-- other components -->
</window>
```

在上面的代码示例中，在组成目标组件window之后，BindComposer将尝试将`@init()`中的字符串文字解析为`Class`对象并将其实例化。 如果成功，它将把ViewModel对象存储为根组件的属性以备将来使用，并存储`@id`中指定的键`vm`。 因此，`<window>`的任何子组件都可以通过id `vm`引用ViewModel。。

如果未指定ViewModel属性，则`绑定器本身`将成为`ViewModel`。 这意味着您可以在不指定ViewModel属性的情况下应用继承BindComposer的ViewModel，但我们仅建议有经验的ZK用户使用此用法。

#### Wire Variable Automatically

如果在ViewModel中使用`@WireVariable`注解成员字段，则在连接Binder和ViewModel之前，变量（如果存在）将自动连接到该字段中。 阅读`Wire Variable` 以获取有关接线变量的更多详细信息。 以下是显示如何将`messagService`变量连接到ViewModel的示例。

```java
public class OrderVM {
    @WireVariable
    MessageService messageService;
    ...
}
```

#### Initialize Binder

如果未指定根组件的`binder`属性，则默认情况下，`BindComposer`创建`AnnotateBinder`。 除非您要使用自己的`binder`，否则通常不必指定`binder`属性。 您可以通过在根组件上调用`getAttribute("binder")`来获取`Binder`对象。 在上面的示例中，它是win.getAttribute("binder")。 我们只建议有经验的ZK用户使用此用法。

#### Initialize Validation Message Holder（初始化验证消息持有者）

`验证消息持有者`也是由`BindComposer`创建的。 它是验证器在验证过程中生成的验证消息的`容器`。 在使用验证消息持有者之前，我们必须在`validationMessages`属性中为其指定一个`ID`。 我们可以使用组件作为密钥从中检索验证消息。 我们将在本节中详细介绍

```xml
<window title="Order Management" border="normal" width="600px" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init(orderVm)" validationMessages="@id('vmsgs')">
    <hlayout>
        <intbox id="qbox"
                value="@load(vm.selected.quantity) @save(vm.selected.quantity, before='saveOrder') @validator(quantityValidator)"
        <label value="@bind(vmsgs[qbox])" sclass="red"/>
    </hlayout>
</window>
```

> 给验证消息持有者一个ID以便引用它。 （第3行）
> 我们可以使用该组件作为键来检索输入组件qbox的验证消息。 （第7行）

### Binder

`Binder`是整个数据绑定机制中的关键。 当我们在组件上应用BindComposer时，它将创建一个绑定器并将组件和ViewModel的对象引用传递给该绑定器。

`Binder`解析ZUL上的ZK绑定注解，以在组件和ViewModel之间建立数据绑定关系。 它还基于绑定关系在ZUL（View）和ViewModel之间同步数据，并将事件转发到ViewModel的command方法。

通常，ViewModel只是一个POJO。 由于对其他人一无所知，因此ViewModel只能通过Binder与另一个对象进行通信。 binder就像一个代理，并使用ViewModel的元数据（注解）和方法来帮助它与其他对象进行通信。 默认情况下，binder订阅桌面范围的事件队列，因此事件队列是binder之间的常见通信机制。 有2个功能：基于此机制运行的`全局Command绑定`和`动态更改通知`。

![image-20200607140629942](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161942.png)

ZK还允许您更改绑定器预订的默认队列名称（queue name）和作用域（scope）。 您可以根据需要更改队列的名称和作用域，将绑定程序分为不同的组。语法如下：

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm')    @init('foo.MyViewModel')" binder="@init(queueName='myqueue')">
    
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') 	      @init('foo.MyViewModel')" 
        binder="@init(value='xxx.MyBinder', queueScope='session')">
```

> 有关可用范围，请参阅`ZK开发人员的参考/事件处理/事件队列`。

对于Binder类自定义（此功能仅供框架设计师使用，不建议应用程序开发人员使用！），您的实现必须扩展AnnotateBinder

### 初始化（Initialization）

我们可以通过初始绑定来初始化任何组件的属性：`@init`。 它会在开始时加载一次ViewModel的属性，然后在用户交互期间该属性将不会被`binder`同步。但是我们期望初始绑定注解中的EL评估结果对于属性“ `viewModel`”和“ `form`”而言是不同的。

#### @Init在viewModel属性

我们通常使用此注解的第一个地方是将ViewModel分配给viewModel属性中的组件。 在viewModel属性中使用时，应在`@init`的字符串文字中指定`完全限定类名称`。

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MyViewModel')">
</window>
```

composer将解析字符串“ foo.MyViewModel”并创建一个对象。

#### @init在组件的属性

通常使用它来初始化具有恒定值或ViewModel属性的组件属性。 binder将其加载一次，并且在随后的用户交互过程中不会随后进行同步。

```xml
<label value="@init(vm.message)"/> 
<label value="@init(123)"/> 
<checkbox checked="@init(true)"/>
```

#### @init在表单绑定

我们还可以在表单绑定上使用初始绑定来填充预定义的值。

```xml
<div form="@id('fx') @init(vm.defaultOrder) @load(vm.order) @save(vm.order, before='compute')"> 
</div>
```

> 我们将在`数据绑定/表单绑定`部分中使用`@init`描述更高级的用法。

### Command Binding（Command绑定）

#### 概述

Command绑定是一种无需编写代码即可将UI组件的事件（例如按钮的onClick链接到ViewModel的Command [1]）的机制。 此绑定是通过@command在ZUL中使用命令名称建立的。 我们只能将一个事件绑定到一个Command，但是可以将多个事件绑定到同一Command。
Command的名称是默认情况下带有Java注解@Command的ViewModel方法的名称。 您还可以在@Command的元素中指定命令的名称。

#### Command method example（Command方法例子）

```java
public class OrderVM {
    // create and add a new order to a list
    // command name is set to method name by default @Command
    public void newOrder() {
        Order order = new Order();
        getOrders().add(order);
        selected = order;//select the new one
    }

    // save an order
    // command name is specified @Command('save')
    public void saveOrder() {
        orderService.save(selected);
    }
}
```

#### Command binding example（Command绑定例子）

```xml
<toolbar>
    <button label="New" onClick="@command('newOrder')"/>
    <button label="Save" onClick="@command('save')"/>
</toolbar>
<menubar>
    <menu label="Order">
        <menupopup>
            <menuitem label="New" onClick="@command('newOrder')"/>
            <menuitem label="Save" onClick="@command('save')"/>
        </menupopup>
    </menu>
</menubar>
```

单击“保存”按钮或菜单项时，binder将在ViewModel中调用saveOrder()。

> 您可以将参数传递给命令方法。 请参考`高级/参数`。

#### Empty Command（空Command）

如果我们使用`空字符串文字`指定命令名称，或者`@command()`中EL的评估结果为`null`，则binder将`忽略该命令`。 如果您在某些情况下不愿做任何事情，这很方便。 例如：

```xml
<button onClick="@command(empty vm.selection?'add':''"/> 
<button onClick="@command(empty vm.selection?'save':null"/>
```

#### Default Command（默认Command）

> since 6.5.1

我们可以应用`@DefaultCommand`将方法标记为`默认command方法`，仅当command绑定与任何其他命令方法都不匹配时才调用该方法。 binder收到命令后，便开始通过匹配其名称来查找ViewModel的命令方法。 仅当binder找不到匹配的方法时，它才会调用默认命令方法。

假定下面的ViewModel中只有两个命令方法。 如果我们触发命令“ `exit`”，则绑定程序将调用默认命令方法`defaultAction()`，因为它找不到名为“ `exit`”的命令方法。

**ViewModel with default command method**

```java
public class OrderVM {
    @Command
    public void newOrder() { 
        ...
    }

    @DefaultCommand
    public void defaultAction() { 
        ...
    }
}
```

#### Command Execution（Command执行）

Command执行是ZK Bind的一种机制，它在ViewModel上执行方法调用。 它绑定到组件的事件，并且当绑定事件发生时，绑定程序将遵循生命周期来完成执行。 COMMAND执行分为6个阶段：VALIDATION , SAVE-BEFORE , LOAD-BEFORE , EXECUTE , SAVE-AFTER , LOAD-AFTER。

**Saving and Loading in Command Execution(在Command执行中保存和加载)**

您可以使用@save（expression，before | after ='a-command'）语法（使用此语法，在执行阶段之前或之后）（例如，调用ViewModel的command方法）之前或之后将多个值同时保存到ViewModel中。 该值在用户编辑后将不会立即保存）。 您还可以使用@load（expression，before | after ='a-command'）语法在EXECUTE阶段之前或之后将值加载到组件。

**Validation in Command Execution(在Command执行中验证)**

验证也包含在command执行中。 它在任何其他阶段之前的“验证”阶段中执行。 如果有多个保存绑定依赖于同一命令，则将在`VALIDATION`阶段调用所有绑定验证器。 如果验证者说无效，则将中断执行，并在其余阶段中将其忽略。

**Phases of Command Execution(命令执行阶段)**

以下是命令执行的阶段：

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161951.png" alt="image-20200607140709544" style="zoom:50%;" />

- 当绑定的ZK事件进入绑定程序时，将调用COMMAND阶段，并且COMMAND阶段内的所有阶段将开始一个接一个地执行。
- 在`VALIDATION`阶段，binder首先收集所有需要验证的特性。 然后，它调用与此命令相关的每个`save-binding`验证器。 在对验证器的每次调用中，binder提供一个新的ValidationContext，其中包含主要属性和其他收集的属性。 这意味着，您可以对收集的属性进行依赖验证，例如，检查发货日期是否大于创建日期。 如果任何验证器通过调用ValidationContext.setInvalid()报告无效，则binder将忽略所有后续阶段，并加载已通知更改的其他属性。
- 在`SAVE-BEFORE`阶段中，binder调用与命令相关的所有`save-binding`，并将其标记`before`以将值保存到表达式中
- 在`LOAD-BEFORE`阶段中，binder调用与命令相关的所有`load-binding`，并标记`before`以将值从表达式加载到组件
- 在`EXECUTE`阶段，binder调用ViewModel的命令方法。 例如，如果命令为`saveOrder`，它将尝试查找具有注解`@Command('saveOrder')`的方法或名为`viewOrder()`的viewModel的方法，然后调用该方法。 如果没有要执行的方法，则会报错。
- 在`SAVE-AFTER`阶段中，binder调用与命令相关的所有`save-binding`，并标记`after`以将值保存到表达式中
- 在`LOAD-AFTER`阶段，binder调用与命令相关的所有加载绑定，并标记`after`以将值从表达式加载到组件

### Property Binding（属性绑定）

#### Two Way Data Binding(双向绑定)

属性绑定使开发人员可以将任何组件的属性绑定到ViewModel的属性，并指定其访问权限。 有3种访问权限：save-load（`@bind`），仅保存（`@save`）和仅加载（`@load`）。
我们通常在输入组件（例如文本框）上使用仅保存绑定，以将用户输入收集到JavaBean中，并在显示数据时仅加载绑定。 如果您同时有这两种需求，则可以使用保存加载绑定。 我们也称其为非条件属性绑定，因为您不能在此绑定中指定条件。

**Save-load binding（读写绑定/双向绑定）**

```xml
<window id="window" title="maximize test" border="normal" maximizable="true" maximized="@bind(vm.maximized)">
</window>
```

通过双向绑定（save-load），当更改窗口的最大化属性时，它将值保存回ViewModel的`maximized`属性，反之亦然。

**Save only binding（只写绑定）**

```xml
<textbox value="@save(vm.person.firstname)"/> 
<textbox value="@save(vm.person.lastname)"/> 
<intbox value="@save(vm.person.age)"/>
```

在save-only绑定中，仅文本框值更改将保存到ViewModel。 ViewModels的更改将不会加载到文本框中。

**Load only binding（只读绑定）**

```xml
<label value="@load(vm.welcomeMessage)" />
```

在load-only绑定中，只有ViewModel的属性更改将加载到标签。

当与组件相关的事件触发时，将组件属性的值保存到ViewModel。 例如，文本框的值保存在onChange事件触发时。 在文本框中完成输入并将光标移动到下一个输入字段之后，输入的数据此时将保存到ViewModel。 加载的时间由@NotifyChange指定。

如果ViewModel的属性是map，则语法如下：

```xml
<label value="@load(vm.myMapping['myKey'])" />
```

#### **Limitation(局限性)**

**Can't save to variable**

在下面的示例中，person.name的值被加载到文本框中，并且在编辑之后，该值随后被保存到person.name。

```xml
<textbox value="@bind(person.name)"/>
```

但是，如果表达式仅包含first-term，则“ load from”仍然有效，但不允许“ save to”执行。 在下面的示例中，值myname被加载到文本框中，但是在编辑值之后，将值保存回时会发生EL异常。

```xml
<textbox value="@bind(myname)"/>
```

> first-term是来自变量解析器的变量，其值不能由EL直接保存

#### Conditional Binding（按条件绑定）

有时我们需要控制保存或加载的时间，而不是取决于与属性相关的事件触发。 我们可以在ViewModel的Command的`before`或`after`指定属性来控制时间。 因此，仅在执行指定的Command之前或之后，才将属性的值保存（或加载）到ViewModel。

假设有一个带有数据库的订单管理应用程序。 以下zul示例是其页面之一，列表框显示订单列表，而doublebox用于编辑订单明细。

```xml
<listbox model="@load(vm.orders)" selectedItem="@bind(vm.selected)" hflex="true" height="200px">
    <template name="model" var="item">
        <listitem>
            <listcell label="@load(item.id)"/>
            <listcell label="@load(item.quantity)"/>
            <listcell label="@load(item.price)"/>
        </listitem>
    </template>
</listbox>
<toolbar>
    <button label="New" onClick="@command('newOrder')"/>
    <button label="Save" onClick="@command('saveOrder')"
            disabled="@bind(empty vm.selected)"/> 
    <!-- show confirm dialog when selected is persisted -->
    <button label="Delete" onClick="@command(empty vm.selected.id?'deleteOrder':'confirmDelete')"
            disabled="@load(empty vm.selected)"/>
</toolbar>
<grid hflex="true">
    <columns>
        <column width="120px"/>
        <column/>
    </columns>
    <rows>
        <row>Quantity
            <intbox value="@load(vm.selected.quantity) @save(vm.selected.quantity,before='saveOrder') "/>
        </row>
        <row>Price
            <doublebox value="@load(vm.selected.price) @save(vm.selected.price,before='saveOrder') "
                       format="###,##0.00"/>
        </row>
    </rows>
</grid>
```

- `listcell`和`doublebox`都绑定到`vm.selected.price`
- 如果在用户输入后立即保存`doublebox`的值，则绑定到相同ViewModel属性的`listcell`的标签也会更改。 此效果可能会误导用户该值已保存到数据库。
- 为了消除这种误导效果，开发人员可能希望在用户单击“Save”按钮后批量保存所有编辑结果。 我们可以通过在@save的`before`属性指定Command的名称来实现。 当用户单击“保存”按钮时，intbox和doublebox的值将批量保存。 

#### **Execution Order（执行顺序）**

如果一个组件同时具有非条件属性和命令绑定。 执行顺序为：

1. Save binding
2. Execute the command 
3. Load binding

具有多重绑定的组件

```xml
<textbox value="@bind(vm.filter)" onChange="@command('doSearch')"/>
```

当onChange事件触发时，绑定程序首先保存vm.filter，执行命令“ doSearch”，然后加载vm.filter。

#### **Multiple Conditions（多条件）**

我们还可以在属性`before`或`after`的字符串文字数组中指定多个Command的名称，如下所示：

**Load after multiple commands**

```xml
<label value="@load(vm.person.firstname, after={'update','delete'})"/>
```

当我们使用属性绑定来收集用户输入时，我们可能需要先进行验证，然后再保存到ViewModel中。 ZK通过称为`validator`的可重用元素提供了一种标准验证机制。 后面会描述其细节。

#### **Collection Binding（集合绑定）**

当我们绑定容器组件的的`model`属性时，我们需要集合绑定，例如`listbox`或`grid`，以连接到ViewModel。 ViewModel的目标属性必须是Collection对象，例如 `List`或`Set`。 当getter方法返回`List`时，binder会自动将其包装为`ListModel`。 如果getter已经返回`ListModel`对象，则binder将不会包装它。

我们使用以下ViewModel来演示集合绑定的用法。

**ViewModel for collection binding**

```java
public class OrderVM { 
    //the order list
    private List<Order> orders; 
    //the selected order
    private Order selected;

    public List<Order> getOrders() {
        return orders;
    }

    public Order getSelected() {
        return selected;
    }

    public void setSelected(Order selected) {
        this.selected = selected;
    }
}
```

**Iteration Variable（迭代变量）**

**```var``` Attribute**

我们通常将Collection Binding与`<template>`一起使用，并指定其`var`属性来命名表示该集合的每个条目的迭代变量：

```xml
<template name="model" var="item"> 
    ...
</template>
```

如果未在var中指定迭代变量名称，则var的默认名称为：`each`。

**Implicit Iteration Status Variable（隐式迭代状态变量）**

ZK会根据迭代变量的名称自动设置迭代状态的变量名称，并且此变量存储迭代索引。 例如。 如果设置`var ="item"`，则迭代索引将为`itemStatus.index`：

```xml
<listbox>
    <template name="model" var="item">
        <listitem>
            <listcell label="@load(itemStatus.index)"/>
        </listitem>
    </template>
</listbox>
```

如果未在`var`中指定迭代变量名称，则隐式迭代状态的默认变量名称为：`forEachStatus`。

**Binding on Listbox（绑定到 Listbox）**

**Collection binding with listbox（使用列表框绑定集合）**

```xml
<listbox model="@load(vm.orders)" selectedItem="@bind(vm.selected)" hflex="true" height="200px">
    <listhead>
        <listheader label="Row Index"/>
        <listheader label="Id"/>
        <listheader label="Quantity" align="center" width="80px"/>
        <listheader label="Price" align="center" width="80px"/>
        <listheader label="Creation Date" align="center" width="100px"/>
        <listheader label="Shipping Date" align="center" width="100px"/>
    </listhead>
    <template name="model" var="item">
        <listitem>
            <listcell label="@load(itemStatus.index)"/>
            <listcell label="@load(item.id)"/>
            <listcell label="@load(item.quantity)" style="@load(item.quantity lt 3?'color:red':'')"/>
            <listcell label="@load(item.price)"/>
            <listcell label="@load(item.creationDate)"/>
            <listcell label="@load(item.shippingDate)"/>
        </listitem>
    </template>
</listbox>
```

- 我们将列表框的模型绑定到Collection类型的属性：`List<Order>`。 （第1行）
- 我们将`seletedItem`绑定到集合中相同对象类型（order）的ViewModel的属性。 每当用户每次选择列表框的项目时，binder都会将所选对象保存到ViewModel。 因此，我们可以通过这种方式获得用户的选择。 （第1行）
- `var`属性的值：`item`表示模型的对象，因此使用点表示法引用其属性，例如`item.quantity`。它在集合中的索引可以由`itemStatus.index`引用。 （第12行）
- 借助EL的强大功能，我们可以在ZUL上实现简单的表示逻辑。 当数量少于3时，我们用红色显示数量。（第14行）

**Binding multiple selections（绑定多个选择）**

```xml
<listbox selectedItems="@bind(vm.selected)" model="@load(vm.model)"> 
    <!-- other components -->
</listbox>
```

**ViewModel for multiple selections（ViewModel定义多个选择）**

```java
public class ListModelSelection {
    private List<String> model;
    private Set<String> selected;
    //getter and setter
}
```

我们可以通过将属性`selectedItems`绑定到`Set`来检索多个选择。

**Binding on Grid（绑定到 Grid）**

grid的用法与此类似。

**Collection binding usage for grid（网格的集合绑定用法）**

```xml
<grid model="@load(vm.orders)">
    <columns>
        <column label="Id"/>
        <column label="Quantity"/>
        <column label="Price"/>
    </columns>
    <template name="model" var="item">
        <row>
            <label value="@bind(item.id)"/>
            <label value="@bind(item.quantity)"/>
            <label value="@bind(item.price)"/>
        </row>
    </template>
</grid>
```

**Dynamic Template（动态模板）**

动态模板使开发人员可以指定要在不同条件下应用的模板，例如 grid。 它在`grid`，`listbox`，`combobox`，`selectbox`和`tree`中受支持。 您可以将此功能与`@template([EL-expression])`结合使用。 binder将EL表达式评估为模板的名称，并寻找相应的模板来渲染子组件。 如果指定的模板在当前组件中不存在，它将查找父组件的模板。 如果未分配@template，则默认使用名为model的模板。

**默认情况**

```xml
<grid model="@bind(vm.orders)">
    <template name="model"> 
        !-- child components --> 
    </template>
</grid>
```

**指定的模板名称**

```xml
<grid model="@bind(vm.orders) @template('myTemplate')">
    <template name="myTemplate">
        <!-- child components -->
    </template>
</grid>
```

您可以使用EL来确定要使用的模板

**Conditional（按条件选择模板）**

```xml
<grid model="@bind(vm.orders) @template(vm.type='foo'?'template1':'template2')">
    <template name="template1">
        <!-- child components -->
    </template>
    <template name="template2"> 
        <!-- child components --> 
    </template>
</grid>
```

它还为每行评估模板时提供了隐式变量：`each`（绑定集合中项目的实例）和`forEachStatus`（迭代状态）。 也就是说，您可以请求对绑定集合中的每个项目(行)使用不同的`<template>`

```xml
<grid model="@bind(vm.orders) @template(each.type='A'?'templateA':'templateB')">
    <template name="templateA">
        <!-- child components -->
    </template>
    <template name="templateB"> 
        <!-- child components --> 
    </template>
</grid>
```

> 假设绑定集合中的对象具有属性`type`。 它的值可以是`A`或`B`。（第1行）

### Children Binding（子绑定）

#### **Create Children Dynamically with Template(使用模板动态创建子代)**

子绑定允许我们将子组件绑定到集合，然后可以使用`<template>`在集合上动态创建一组相似的组件。 通常，我们也可以通过`listbox`或`grid`来执行此操作，但是它们具有固定的结构和布局。 借助此功能，我们可以创建`更灵活的子组件`，例如：`一组来自列表的复选框`或`动态生成的菜单项`。

使用此功能的步骤：

1. 创建一个List对象作为ViewModel的属性（在CE版本下，它必须是一个List对象）。
2. 将父组件的children属性与@init或@load绑定到ViewModel的属性
3. 使用<template>封闭子组件并将其name属性设置为child。 因为子级绑定选择名称为子级的默认模板。

#### **基本用法示例**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.ChildrenSimpleVM')">
    Simple - Init
    <vlayout id="init" children="@init(vm.nodes)">
        <template name="children" var="node">
            <label value="@bind(node.name)" style="padding-left:10px"/>
        </template>
    </vlayout>
    Simple - load
    <vlayout id="load" children="@load(vm.nodes)">
        <template name="children" var="node">
            <label value="@bind(node.name)" style="padding-left:10px"/>
        </template>
    </vlayout>
    Simple - load after cmd
    <vlayout id="aftercmd" children="@load(vm.nodes, after='cmd')">
        <template name="children" var="node">
            <label value="@bind(node.name)" style="padding-left:10px"/>
        </template>
    </vlayout>
    <!-- other components -->
</window>
```

**Basic usage screenshot**

![image-20200607140837832](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162009.png)

#### **与动态模板结合**

如果将此功能与动态模板结合使用，则甚至可以在不同条件下渲染不同的子组件。
这是创建动态菜单栏的示例。 如果菜单项没有子菜单，则使用menuitem，否则使用menu。

**动态菜单栏示例**

```xml
<menubar id="mbar" children="@bind(vm.nodes) @template(empty each.children?'menuitem':'menu')">
    <template name="menu" var="node">
        <menu label="@bind(node.name)">
            <menupopup children="@bind(node.children) @template(empty each.children?'menuitem':'menu')"/>
        </menu>
    </template>
    <template name="menuitem" var="node">
        <menuitem label="@bind(node.name)" onClick="@command('menuClicked',node=node)"/>
    </template>
</menubar>
```

**Dynamic menu bar screenshot**

![image-20200607140911510](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162022.png)

#### **Default Converter for Children Binding(子绑定的默认转换器)**

> Avaliable for ZK: EE
>
> Since 6.0.1

用户通常将属性“子级”绑定到Collection对象，但是您也可以将其绑定到Array，Enum甚至Object。 隐式默认转换器会将它们转换为Collection对象。 当然，您可以应用定制的转换器，我们将在数据绑定/转换器中讨论它。

### **Form Binding(表单绑定)**

#### 概述

表单绑定就像一个缓冲区。 它会自动创建一个中间对象。 在保存到ViewModel之前，所有输入数据都保存到中间对象。 这样，我们可以防止脏数据在用户确认之前保存到ViewModel中。 假设用户在Web应用程序中填写表单，则用户输入的数据将直接保存到ViewModel的属性（目标对象）中。 然后，用户在提交表单之前取消填充操作，因此不赞成使用ViewModel中存储的数据。 如果我们进一步处理已弃用的数据，将会带来麻烦，因此开发人员可能会先将输入数据存储在中间位置，然后在用户确认后将其移至实际目标对象。 表单绑定提供了一个中间对象来存储未经确认的输入，而无需您自己实现。

在执行命令进行确认之前，表单绑定可以使ViewModel中的目标对象保持不变。 在通过Command保存到ViewModel的属性（目标对象）之前，我们可以将输入保存在Form绑定的中间对象中。 执行命令时（例如单击按钮），输入数据实际上会保存到ViewModel的属性中。 开发人员只需编写ZK绑定表达式即可轻松实现整个过程，它减轻了开发人员手动清除脏数据或自己实现缓冲区的负担。

#### **ZUL、中间对象和目标对象之间的数据流如下所示：**

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162028.png" alt="image-20200607140948537" style="zoom: 50%;" />

步骤：

1. 在表单属性中使用`@id`为中间对象提供一个id。
   - 在ZK绑定表达式中可以使用id引用中间对象。
2. 指定要使用`@load`加载的ViewModel的属性
3. 指定要保存的ViewModel的属性，并在`@save`中使用`before`绑定保存Command
   - 这意味着binder将在命令之前将中间对象的属性保存到ViewModel
4. 像在属性绑定中一样，将组件的属性绑定到中间对象的属性。
   - 您应该使用@id中指定的中间对象的id来引用其属性。

#### **使用常规的属性绑定示例**

```xml
<groupbox>
    <grid hflex="true">
        <columns>
            <column width="120px"/>
            <column/>
        </columns>
        <rows>
            <row>Id
                <hlayout>
                    <label value="@load(vm.selected.id)"/>
                </hlayout>
            </row>
            <row>Description
                <textbox value="@bind(vm.selected.description)"/>
            </row>
            <row>Quantity
                <intbox value="@bind(vm.selected.quantity)"/>
            </row>
            <row>Price
                <doublebox value="@bind(vm.selected.price)" format="###,##0.00"/>
            </row>
        </rows>
    </grid>
</groupbox>
```

> 输入数据将直接保存到vm.selected的属性中。 无中间对象

#### **同样例子，但是用form绑定（使用中间对象绑定）**

```xml
<groupbox form="@id('fx') @load(vm.selected) @save(vm.selected, before='saveOrder')">
    <grid hflex="true">
        <columns>
            <column/>
            <column/>
        </columns>
        <rows>
            <row>Id
                <hlayout>
                    <label value="@load(fx.id)"/>
                </hlayout>
            </row>
            <row>Description
                <textbox value="@bind(fx.description)"/>
            </row>
            <row>Quantity
                <intbox value="@bind(fx.quantity) "/>
            </row>
            <row>Price
                <doublebox value="@bind(fx.price)" format="###,##0.00"/>
            </row>
        </rows>
    </grid>
</groupbox>
```

> - 我们给中间对象一个id: `fx`。 （第1行）
> - 我们可以访问的`fx属性`与`vm.selected`相同。 （第14、17、20行）
> - 每次用户触发`onChange`事件时，Binder会将输入数据保存到`中间对象fx`中。 单击绑定到Command '`saveOrder`'的按钮时，它将把`中间对象`的数据保存到`vm.selected`

#### **Middle Object(中间对象)**

表单绑定会自动为您创建一个中间对象，以存储您指定的ViewModel对象的属性。 但它仅存储属性绑定到的那些属性。 如果仍然需要访问未存储在中间对象中的属性，则可以从ViewModel的原始对象访问它。 假设`vm.selected`具有4个属性：`id`，`description`，`quantity`，`price`。 对于以下示例：

```xml
<groupbox form="@id('fx') @load(vm.selected) @save(vm.selected, before='saveOrder')">
    <grid hflex="true">
        <rows>
            <row>Id
                <label value="@load(fx.id)"/>
            </row>
            <row>Description
                <textbox value="@bind(fx.description)"/>
            </row>
            <row>Quantity
                <intbox value="@bind(fx.quantity) "/>
            </row>
        </rows>
    </grid>
</groupbox>
```

因为我们仅将属性绑定到3个属性：`id`，`description`和`quantity`，所以中间对象仅存储这3个属性。 因此，`price`属性不会存储在中间对象中，如果不先绑定它就无法引用它。 例如，我们不能通过@command('cmd'，currentPrice = fx.price)将其作为参数传递，因为`fx.price在中间对象中不存在`。

#### **Form Status Variable(表单状态变量)**

表单绑定还记录中间对象的修改状态。这是用户知道是否已修改表单数据（脏状态）的常见要求，因此开发人员添加了一项功能，该功能可以通过UI效果提醒用户。 例如，某些文本编辑器在标题栏上添加星号“ *”，以提醒用户修改后的文本文件。 “表单绑定”将脏状态保留在我们可以利用的变量中。

脏状态存储在自动创建的表单状态变量中，其命名约定为：`[middleObjectId]Status`

继续上面的示例，我们在Id值旁边添加一个感叹号图标。 如果用户修改了任何输入数据，将显示惊叹号图标。

```xml
<groupbox form="@id('fx') @load(vm.selected) @save(vm.selected, before='saveOrder')">
    <grid hflex="true">
        <columns>
            <column width="120px"/>
            <column/>
        </columns>
        <rows>
            <row>Id
                <hlayout>
                    <label value="@load(fx.id)"/>
                    <image src="@load(fxStatus.dirty?'exclamation.png':'')"/>
                </hlayout>
            </row>
            <row>Description
                <textbox value="@bind(fx.description)"/>
            </row>
            <row>Quantity
                <intbox value="@bind(fx.quantity) "/>
            </row>
            <row>Price
                <doublebox value="@bind(fx.price)" format="###,##0.00"/>
            </row>
            <row>Total Price
                <label value="@load(fx.totalPrice)"/>
            </row>
            <row>Creation Date
                <datebox value="@bind(fx.creationDate)"/>
            </row>
            <row>Shipping Date
                <datebox value="@bind(fx.shippingDate)"/>
            </row>
        </rows>
    </grid>
</groupbox>
```

在此示例中，表单ID为`fx`，表单状态变量为`fxStatus` 。 它的`dirty`属性指示该表单是否已被用户修改。

![image-20200607141032658](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162036.png)

用户修改字段后，“ Id”字段旁边会出现一个感叹号图标。 如果用户单击“保存”按钮或将数据更改回原始值，则惊叹号图标消失。

#### **Initialize with Form Object(用表单对象初始化)**

如果您想更好地控制表单绑定，例如 要操纵中间对象，可以在@init()中提供一个实现Form接口的对象（或者可以使用SimpleForm方便使用）。 这将使用您自己的中间对象（Form对象）初始化表单绑定，然后您可以执行任何想在中间对象的属性更改时通知其他人的操作。

**用表单对象初始化**

```xml
<groupbox form="@id('fx') @init(vm.myForm) @load(vm.selected) @save(vm.selected, before='saveOrder') ">
```

准备一个Form对象

```java
public class OrderVM {
    Form myForm = new SimpleForm();

    public Form getMyForm() {
        return myForm;
    }
}
```

#### **Form Validation表单验证**

在将数据保存到表单的中间对象之前，我们还可以使用验证器来验证用户输入。 请参考`数据绑定/验证器`。

### **Reference Binding(引用绑定)**

> Available for ZK: PE, EE 
>
> Since 6.0.1

**Make an expression reference(作表达式参考)**

引用绑定使我们可以引用具有自定义名称的EL表达式。 我们可以在绑定到该引用的组件中嵌套的另一个EL表达式中使用该引用。 通常，我们使用引用绑定来缩短表达式或模块化视图[1]。

使用该特性的步骤：

1. 使用@ref绑定组件上的自定义属性。
2. 在子组件中嵌套的其他EL表达式中使用custom属性

> 使用此功能时，必须避免保存为first-term的表达式。

#### **基本使用示例**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MyVM')">
    <vlayout p="@ref(vm.person)">
        <hlayout>
            First Name:
            <textbox value="@bind(p.firstName)"/>
        </hlayout>
        <hlayout>
            Last Name:
            <textbox value="@bind(p.lastName)"/>
        </hlayout>
    </vlayout>
</window>
```

如上面的示例所示，我们在vlayout节点，为vm.person添加了一个@ref引用，并命名为p。 这样，vlayout中的p可以被其他EL表达式使用。

#### **模块化视图示例**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MyVM')">
    <include p="@ref(vm.person1)" src="person.zul"/>
    <include p="@ref(vm.person2)" src="person.zul"/>
</window>
```

模块化视图示例 - person.zul

```xml
<vlayout>
    <hlayout>
        First Name:<textbox value="@bind(p.firstName)"/>
    </hlayout>
    <hlayout>
        Last Name:<textbox value="@bind(p.lastName)"/>
    </hlayout>
</vlayout>
```

在上面的示例中，我们有一个视图-person.zul，它将组件绑定到p.firstName和p.lastName。 此视图可以被提供p引用的其他视图重用。

### **Converter(转换器)**

Converter在ViewModel的属性和UI组件属性值之间执行两种转换。 在将ViewModel的属性加载到组件时，它将数据转换为组件属性，并在保存到ViewModel时将其转换回。 在不同的上下文中以不同的格式显示数据是很普遍的要求。 使用转换器开发人员可以实现此目的，而无需实际更改ViewModel中的实际数据。

#### **实现一个转换器(Converter)**

开发人员可以通过实现Converter接口来创建自定义转换器。 将ViewModel的属性加载到组件时，将调用coerceToUi()方法，并且其返回类型应与绑定的组件属性的value [1]相对应。 保存时将调用coerceToBean()。 如果只需要单向转换，则可以将未使用的方法留空。

> [1]从6.0.1开始，转换器可以返回IGNORED_VALUE来指示绑定器应忽略以将该值加载到组件。 如果要在命令或值更改通知之后在组件上进行一些额外的方法调用，这将很有用。

以下是内置转换器`formattedDate`的实现方式。

```java
public class MyDateFormatConverter implements Converter {
    /**
     * Convert Date to String.
     *
     * @param val  date to be converted
     * @param comp associated component
     * @param ctx  bind context for associate Binding and extra parameter (e.g. format) 
     * @return the converted String
     */
    public Object coerceToUi(Object val, Component comp, BindContext ctx) {
        //user sets format in annotation of binding or args when calling binder.addPropertyBinding() 
        final String format = (String) ctx.getConverterArg("format");
        if (format == null) throw new NullPointerException("format attribute not found");
        final Date date = (Date) val;
        return date == null ? null : new SimpleDateFormat(format).format(date);
    }

    /**
     * Convert String to Date.
     *
     * @param val  date in string form
     * @param comp associated component
     * @param ctx  bind context for associate Binding and extra parameter (e.g. format) 
     * @return the converted Date
     */
    public Object coerceToBean(Object val, Component comp, BindContext ctx) {
        final String format = (String) ctx.getConverterArg("format");
        if (format == null) throw new NullPointerException("format attribute not found");
        final String date = (String) val;
        try {
            return date == null ? null : new SimpleDateFormat(format).parse(date);
        } catch (ParseException e) {
            throw UiException.Aide.wrap(e);
        }
    }
}
```

> 我们通过ctx.getConverterArg("format")检索"format"参数的值。 这使您可以传递任意参数。

根据上面的代码，我们可以将"format"参数传递给"formattedDate"转换器。

```xml
<label value="@load(item.creationDate) @converter(vm.myConverter, format='yyyy/MM/dd')"/>
```

#### **Use Custom Converter(使用自定义转换器)**

应用转换器的最常见方法是将组件属性绑定到ViewModel的属性（这是自定义转换器），或者您也可以指定自定义转换器的全限定类名。

把转换器作为一个属性返回

```java
public class MyViewModel {
    private Converter MyConverter = new MyConverter();

    public Converter getMyConverter() {
        return myConverter;
    }
}
```

使用自定义转换器示例

```xml
<label value="@load(vm.message) @converter(vm.myConverter)"/>
<label value="@load(vm.message) @converter('com.foo.MyConverter')"/>
```

#### **Register Application Level Converters(注册应用级别的转换器)**

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

然后按转换器名称使用它们

```xml
<label value="@load(vm.message) @converter('foo')"/> 
<label value="@load(vm.message) @converter('bar')"/>
```

> [1]：应用程序级别转换器只有一个实例，并且在所有绑定程序之间共享。

#### **使用内置转换器**

ZK提供了一些常用的内置应用程序级别转换器。 您可以将它们与名称和参数一起使用：@converter('converterName'，parameterKey ='value')。

目前，我们提供的内置转换器是：

##### formattedNumber

##### formattedDate

```xml
<label value="@load(item.price) @converter('formatedNumber', format='###,##0.00')"/> 
<label value="@load(item.creationDate) @converter('formatedDate', format='yyyy/MM/dd')"/>
```

您应为formattedNumber转换器或formattedDate转换器的格式参数值指定数字或日期模式

> since 8.0.0

##### **formattedTime**

```xml
<label value="@load(item.time) @converter('formattedTime', format='hhmmss')"/>
```

您应该为formattedTime转换器的format参数值指定时间模式，该值仅与Date对象的小时/分钟/秒/毫秒值相关。

#### **Use Converter in Children Binding(在子绑定使用转换器)**

> since 6.0.1

子绑定也支持转换器，它具有默认转换器（如果用户未分配），可以将用户提供的对象转换为Collection对象。 用户可以像其他绑定一样使用定制的转换器。 如果为子绑定实现转换器，请记住在coerceToUi()中返回Collection对象。

```xml
<hlayout children="@init(vm.items) @converter(vm.itemConverter)">
    <template name="children">
        <label value="@load(each) "/>
    </template>
</hlayout>
```

### **Validator(验证器)**

#### 用户输入验证

用户输入验证是Web应用程序必不可少的功能。 ZK的验证器可以帮助开发人员完成此任务。 验证器是一个可重用的元素，它执行验证并将验证消息存储到验证消息保存器中。 应用后，在将数据保存到绑定目标（ViewModel或中间对象）之前会先调用它。 当您将组件的属性绑定到验证器时，绑定器将使用它来自动验证属性的值，然后再保存到ViewModel或中间对象。 如果验证失败，则ViewModel（或中间对象）的属性将保持不变。

我们通常为自定义验证器提供ViewModel的属性。

```java
public class MyViewModel {
    private Validator emailValidator = new EmailValidator();

    public Validator getEmailValidator() {
        return emailValidator;
    }
}
```

然后可以通过ViewModel的属性或带有验证器的全限定类名的字符串文字在ZUL上引用它

```xml
<textbox value="@save(vm.account.email) @validator(vm.emailValidator)"/> 
<textbox value="@save(vm.account.email) @validator('org.foo.EmailValidator')"/>
```

binder使用`vm.emailValidtor`验证文本框的值，然后再保存到`vm.account.email`。 如果验证失败，则ViewModel的属性：`vm.account.email` 将保持不变。

```xml
<textbox value="@save(vm.account.email, before='save') @validator(vm.emailValidator)"/>
```

binder使用`vm.emailValidtor`来验证文本框的值，然后再执行命令`save`。

以下是上述两种保存语法的比较表：

|                       | Property Binding                                             | Save Before Command                                          |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Syntax Example**    | @bind(vm.account.email) @validator(vm.emailValidator)        | @load(vm.account.email)<br/>@save(vm.account.email, before= 'save')<br/>@validator(vm.emailValidator) |
| **Save When**         | a component's attribute related event fires (e.g.   onChange for value) | Before executing a command                                   |
| **Validation Timing** | Immediately validate for single field<br/>No validation when executing a command | Batch validate all fields when triggering specified command<br/>No immediate validation of single fields after a user input |
| **Validation Fails**  | **Not** save data to ViewModel                               | Not save data to ViewModel & Not execute the command         |

在表单绑定中，您可以在`Form`属性或输入组件上应用验证器。 如果您在两个地方都申请，则可以双重验证用户输入。 第一次是将数据保存到中间对象时，这可以在输入后给用户立即响应。 第二次是在命令上保存到ViewModel时，这可以验证用户输入，即使他没有输入任何内容并直接提交空数据。

```xml
...
<toolbar>
    ...
    <button label="Save" onClick="@command('saveOrder')" disabled="@bind(empty vm.selected)"/>
    ...
</toolbar>
<groupbox form="@id('fx') @load(vm.selected) @save(vm.selected, before='saveOrder') @validator(vm.formValidator)">
    <grid hflex="true">
        <columns>
            <column width="120px"/>
            <column/>
        </columns>
        <rows>
            <row>Id
                <hlayout>
                    <label value="@load(fx.id)"/>
                </hlayout>
            </row>
            <row>Description
                <textbox value="@bind(fx.description)"/>
            </row>
            <row>Quantity
                <intbox value="@bind(fx.quantity) @validator(vm.quantityValidator)"/>
            </row>
            <row>Price
                <doublebox value="@bind(fx.price) @validator(vm.priceValidator)" format="###,##0.00"/>
            </row>
        </rows>
    </grid>
</groupbox>
```

> 第7行：ZK将在执行命令'saveOrder'之前调用vm.formValidator。
>
> 第23行：intbox上触发onChange事件时（用户模糊焦点时），ZK将验证输入值。
>
> 第7、20、23行：您可以分别在表单属性和每个输入组件上应用验证器以执行双重验证。

**Validation Message Holder(验证消息持有人)**

数据绑定提供了一种存储和显示验证消息的标准机制。 在执行验证之后，验证器可以将验证消息存储在验证消息持有者中。 要使用它，您必须通过使用`@id`在`validationMessages`属性中指定其ID来对其进行初始化。 然后，您可以使用此ID引用它。

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MyViewModel')"
        validationMessages="@id('vmsgs')">
</window>
```

**Display a Message(显示验证消息)**

验证消息持有者像带有键-值对的映射一样存储消息。 该值是执行验证后验证程序生成的验证消息，默认键是绑定到验证程序的绑定源组件（对象本身，没有ID）。 要在ZUL中检索并显示验证消息，您可以将显示组件（即标签）绑定到以组件为键的验证消息持有人。

绑定器将在每次验证后重新加载验证消息。

**Display validation message with default key**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm')@init('foo.MyViewModel')"
        validationMessages="@id('vmsgs')">
    <hlayout>Value1:
        <textbox id="tb1" value="@bind(vm.value1) @validator(vm.validator1)"/>
        <label id="m1" value="@bind(vmsgs[tb1])"/>
    </hlayout>
    <hlayout>Value2:
        <intbox id="tb2" value="@bind(vm.value2) @validator(vm.validator2)"/>
        <label id="m2" value="@bind(vmsgs[self.previousSibling])"/>
    </hlayout>
</window>
```

> 您可以使用组件的ID来引用组件对象。 （第5行）
>
> 您可以使用组件的属性来引用具有相对位置的组件对象，从而消除了为组件提供ID的可能性。（第9行）

**Display a Message of a Self-defined Key**

您可以显示绑定到扩展AbstractValidator的验证器的自定义键的第一条验证消息。 显示带有自定义键的验证消息

```xml
<vbox form="@id('fx') @load(vm) @save(vm,before='submit') @validator(vm.formValidator)">
    <hbox>
        <textbox id="t41" value="@bind(fx.value1)"/>
        <label id="l41" value="@bind(vmsgs['fkey1'])"/>
    </hbox>
    <hbox>
        <textbox id="t42" value="@bind(fx.value2)"/>
        <label id="l42" value="@bind(vmsgs['fkey2'])"/>
    </hbox>
    <hbox>
        <textbox id="t43" value="@bind(fx.value3)"/>
        <label id="l43" value="@bind(vmsgs['fkey3'])"/>
    </hbox>
    <button id="submit" label="submit" onClick="@command('submit')"/>
</vbox>
```

> 您可以使用自定义键来显示特定消息。 （第4行）

请阅读`自定义验证消息KEY`以获取有关验证器的详细信息。

**Display Multiple Messages(显示多条消息)**

> Available for ZK: EE
>
> Since 6.0.1

验证器可以为组件或自定义键设置多个消息。 您可以从Validation Message Holder的特殊属性`texts`中获取所有消息。

> 在EL中，`vmsgs.texts`与`vmsgs ['texts']`相同，因此应避免使用`texts`作为自定义键。

**Display multiple messages of a form validator**

```xml
<div id="formdiv" form="... @validator('fooValidator')">
    ...
</div>
<grid id="msggrid" model="@bind(vmsgs.texts[formdiv])" visible="@bind(not empty vmsgs.texts[formdiv])">
    <template name="model" var="msg">
        <row>
            <label value="@bind(msg)"/>
        </row>
    </template>
</grid>
```

> 使用网格显示多个消息。 （第4行）

您还可以使用语法`@bind(vmsgs.texts)`获取验证消息持有者的所有消息，并使用语法`@bind(vmsgs.texts ['a_self_defined_key'])`获取自定义键的消息。

**Implement a Validator(实现一个验证器)**

为方便起见，您可以通过实现Validator接口或继承AbstractValidator，根据应用程序的要求创建自定义验证器

**Property Validator(属性验证器)**

**Single Property Validation(单个属性验证)**

我们通常需要一次验证一个属性，最简单的方法是继承`AbstractValidator`并重写`validate()`以实现自定义验证规则。 如果验证失败，请使用`addInvalidMessage()`存储要显示的验证消息。 您必须将验证程序绑定的组件对象作为关键字来传递，以检索此消息，就像我们在`Validation Message Holder`一节中提到的那样。

```xml
<intbox value="@save(vm.quantity) @validator(vm.rangeValidator)"/>
```

```java
public Validator getRangeValidator() {
    return new AbstractValidator() {
        public void validate(ValidationContext ctx) {
            Integer val = (Integer) ctx.getProperty().getValue();
            if (val < 10 || val > 100) {
                addInvalidMessage(ctx, "value must not < 10 or > 100, but is " + val);
            }
        }
    };
}
```

第4行：我们可以通过`ctx.getProperty().getValue()`从validator的绑定源组件中获取用户输入数据。
第6行：`addInvalidMessage()`将使用默认密钥将消息添加到验证消息持有人中

**Dependent Property Validation(依赖属性验证)**

有时我们需要另一个属性的值来验证当前属性。 我们必须保存依赖于同一命令的那些值（使用@save(vm.p,before ='command')，因此，绑定程序会将保存在同一命令上的那些属性传递给ValidationContext，我们可以检索他们以属性的KEY来处理。

**假设发货日期必须至少比创建日期晚3天。**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('eg.ValidationMessagesVM')"
        validationMessages="@id('vmsgs')">
    <grid hflex="true">
        <columns>
            <column width="120px"/>
            <column/>
        </columns>
        <rows>
            <!-- other input fields -->
            <row>Creation Date
                <hlayout>
                    <datebox id="cdBox"
                             value="@load(vm.selected.creationDate) @save(vm.selected.creationDate, before='saveOrder')
@validator(vm.creationDateValidator)"/>
                    <label value="@load(vmsgs[cdBox])" sclass="red"/>
                </hlayout>
            </row>
            <row>Shipping Date
                <hlayout>
                    <datebox id="sdBox"
                             value="@load(vm.selected.shippingDate) @save(vm.selected.shippingDate, before='saveOrder') @validator(vm.shippingDateValidator)"/>
                    <label value="@load(vmsgs[sdBox])" sclass="red"/>
                </hlayout>
            </row>
        </rows>
    </grid>
</window>
```

当我们在命令 `saveOrder`之前保存`vm.selected.creationDate`和`vm.selected.shippingDate`时，它们将被传递到验证上下文中

**我们的自定义发货日期验证器应获取创建日期以进行比较**。

```java
public class ShippingDateValidator extends AbstractValidator {
    public void validate(ValidationContext ctx) {
        Date shipping = (Date) ctx.getProperty().getValue();//the main property
        Date creation = (Date) ctx.getProperties("creationDate")[0].getValue();//the collected
        //multiple fields dependent validation, shipping date have to large than creation more than 3 days.
        if (!isDayAfter(creation, shipping, 3)) {
            addInvalidMessage(ctx, "must large than creation date at least 3 days");
        }
    }

    static public boolean isDayAfter(Date date, Date laterDay, int day) {
        if (date == null) return false;
        if (laterDay == null) return false;
        Calendar cal = Calendar.getInstance();
        Calendar lc = Calendar.getInstance();
        cal.setTime(date);
        lc.setTime(laterDay);
        int cy = cal.get(Calendar.YEAR);
        int ly = lc.get(Calendar.YEAR);
        int cd = cal.get(Calendar.DAY_OF_YEAR);
        int ld = lc.get(Calendar.DAY_OF_YEAR);
        return (ly * 365 + ld) - (cy * 365 + cd) >= day;
    }
}
```

发货日期是要验证的主要属性。 检索其他属性的方法与主要方法不同。 （第4行）

**Dependent Property Validator in Form Binding(表单绑定中的从属属性验证器)**

如果要根据表单绑定中另一个属性的值来验证属性，则必须在form属性中应用验证器，而不是绑定到中间对象属性的输入组件。 然后，您可以从验证上下文中获取所有属性。

下面的示例演示了相同的发货日期验证器，但在表单绑定中使用了它。

**Using validator in form binding**

```xml
...
<toolbar>
    ...
    <button label="Save" onClick="@command('saveOrder')" disabled="@bind(empty vm.selected)"/>
    ...
</toolbar>
<groupbox form="@id('fx') @load(vm.selected) @save(vm.selected, before='saveOrder') @validator(vm.shippingDateValidator)">
    <grid hflex="true">
        <columns>
            <column width="120px"/>
            <column/>
        </columns>
        <!-- other components -->
        <rows>
            <row>Creation Date
                <hlayout>
                    <datebox id="cdBox" value="@bind(fx.creationDate) @validator(vm.creationDateValidator)"/>
                    <label value="@load(vmsgs[cdBox])" sclass="red"/>
                </hlayout>
            </row>
            <row>Shipping Date
                <hlayout>
                    <datebox id="sdBox" value="@bind(fx.shippingDate)"/>
                    <label value="@load(vmsgs[sdBox])" sclass="red"/>
                </hlayout>
            </row>
        </rows>
    </grid>
</groupbox>
```

第9行：在表单绑定中应用验证器，而不是在绑定到中间对象属性的单个输入组件中。

**Validator for form binding**

```java
public Validator getShippingDateValidator() {
    return new AbstractValidator() {
        public void validate(ValidationContext ctx) {
            Date shipping = (Date) ctx.getProperties("shippingDate")[0].getValue();
            Date creation = (Date) ctx.getProperties("creationDate")[0].getValue();
            //dependent validation, shipping date have to later than creation date for more than 3 days.
            if (!CalendarUtil.isDayAfter(creation, shipping, 3)) {
                addInvalidMessage(ctx, "must large than creation date at least 3 days");
            }
        }
    };
}
```

第4〜5行：应用于表单绑定时，主要属性值是Form。 我们应该使用其键检索所有属性。 

从6.0.1开始，您可以通过`ValidationContext.getProperties(Object)`获取bean的值。

```java
public Validator getShippingDateValidator() {
    return new AbstractValidator() {
        public void validate(ValidationContext ctx) {
            //base of main property is the bean, you can get all properties of the bean 
            Map<String, Property> beanProps = ctx.getProperties(ctx.getProperty().getBase());
            Date shipping = (Date) beanProps.get("shippingDate").getValue();
            Date creation = (Date) beanProps.get("creationDate").getValue();
            //....
        }
    };
}
```

**Pass and Retrieve Parameters(传递和检索参数)**

我们可以通过EL表达式将一个或多个参数传递给验证器。 参数以键-值对格式写入@validator()注解中。

**Passing a constant value in a ZUL(在ZUL中传递常量)**

```xml
<textbox id="keywordBox" value="@save(vm.keyword) @validator(vm.maxLengthValidator, length=3)"/>
```

**Retrieving parameters in a validator(在验证器中检索参数)**

```java
public class MaxLengthValidator implements Validator {
    public void validate(ValidationContext ctx) {
        Number maxLength = (Number) ctx.getBindContext().getValidatorArg("length");
        if (ctx.getProperty().getValue() instanceof String) {
            String value = (String) ctx.getProperty().getValue();
            if (value.length() > maxLength.longValue()) {
                ctx.setInvalid();
            }
        } else {
            ctx.setInvalid();
        }
    }
}
```

参数名称`length`是用户定义的

**Passing object in a ZUL(在ZUL中传递对象)**

```xml
<combobox id="upperBoundBox">
    <comboitem label="90" value="90"/>
    <comboitem label="80" value="80"/>
    <comboitem label="70" value="70"/>
    <comboitem label="60" value="60"/>
    <comboitem label="50" value="50"/>
</combobox>
<intbox value="@save(vm.quantity) @validator(vm.upperBoundValidator, upper=upperBoundBox.selectedItem.value)"/>
```

> 从另一个组件的属性传递值

**Self-defined Validation Message Key(自定义验证消息Key)**

您可能需要自行设置验证消息的密钥，尤其是当您使用`单个验证器`来验证`多个字段`时。 由一个验证器为组件生成的所有验证消息都具有相同的默认密钥（组件对象本身），为了分别检索和显示验证消息，必须为每条消息设置不同的密钥。

假设您在表单绑定中仅使用一个验证器来验证所有字段。

```java
public Validator getFormValidator() {
    return new AbstractValidator() {
        public void validate(ValidationContext ctx) {
            String val = (String) ctx.getProperties("value1")[0].getValue();
            if (invalidCondition01(val)) {
                addInvalidMessage(ctx, "fkey1", "value1 error");
            }
            val = (String) ctx.getProperties("value2")[0].getValue();
            if (invalidCondition02(val)) {
                addInvalidMessage(ctx, "fkey2", "value2 error");
            }
            val = (String) ctx.getProperties("value3")[0].getValue();
            if (invalidCondition03(val)) {
                addInvalidMessage(ctx, "fkey3", "value3 error");
            }
        }
    };
}
```

> 由于验证消息存储在相同的上下文中，因此对于不同的消息应使用不同的密钥。

在每个组件旁边显示验证消息

```xml
<vbox form="@id('fx') @load(vm) @save(vm,before='submit') @validator(vm.formValidator)">
    <hbox>
        <textbox id="t41" value="@bind(fx.value1)"/>
        <label id="l41" value="@bind(vmsgs['fkey1'])"/>
    </hbox>
    <hbox>
        <textbox id="t42" value="@bind(fx.value2)"/>
        <label id="l42" value="@bind(vmsgs['fkey2'])"/>
    </hbox>
    <hbox>
        <textbox id="t43" value="@bind(fx.value3)"/>
        <label id="l43" value="@bind(vmsgs['fkey3'])"/>
    </hbox>
    <button id="submit" label="submit" onClick="@command('submit')"/>
</vbox>
```

**Validation in Non-Conditional Property Binding(非条件属性绑定中的验证)**

非条件属性绑定的执行与命令的执行分开。 如果一个事件同时触发了属性保存和命令执行，则它们不会互相阻塞。

For example:

```xml
<textbox value="@bind(vm.value) @validator(vm.myValidator)" onChange="@command('submit')"/>
```

当`onChange`事件触发时，binder将在`保存vm.value之前`执行验证。 无论验证失败与否，它将执行命令`submit`。 验证失败只会停止binder将文本框的值保存到vm.value中

再举一个相反的例子：

```xml
<textbox id="t1" value="@bind(vm.foo) " onChange="@command('submit')"/>
<textbox id="t2" value="@save(vm.bar, before='submit') @validator(vm.myValidator)"/>
```

当onChange事件触发时，如果t2的值验证失败，它将停止绑定程序执行命令`submit`。 但是，binder仍会将t1的值保存到vm.foo中。

**Register Application Level Validators(注册应用程序级别验证器)**

> Available for ZK: EE
>
> Since 6.0.1

您可以通过在zk.xml中设置library-property(org.zkoss.bind.appValidators)来注册应用程序级别验证程序[1]。

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

> 应用程序级别验证程序只有一个实例，并且在所有绑定程序之间共享。

**Use Built-in Validator(使用内置验证器)**

ZK提供了一些内置的验证器，您可以按照语法直接使用：`**@validator('validatorName')**`

ValidatorName应该替换为内置的验证者名称，我们在下面列出：

- `beanValidator`
- `formBeanValidator`

**Bean Validator**

该验证器集成了Java Bean验证[`1`]框架，该框架定义了元数据模型和API [`2`]来验证JavaBean。 开发人员可以通过Java注解指定bean属性的约束，并通过API针对bean进行验证。 通过使用ZK的bean验证器，您只需指定对bean属性的约束，然后bean验证器将调用API来为您验证。

> [1]: JSR 303: http://jcp.org/en/jsr/detail?id=303
> [2]: Bean validator api: http://jackson.codehaus.org/javadoc/bean-validatio-api/1.0/index.html

**Prepare to Use JSR 303**

**Required Jars**

一个有名的实现了JSR 303的验证器是`Hibernate Validator`。以下是pom.xml中使用Hibernate Validator的示例依赖项列表：

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>4.0.2.GA</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.6.1</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.6.1</version>
</dependency>
```

Hibernate Validator使用log4j来记录信息。 您可以使用自己喜欢的任何log4j实现。

**Setup**

在项目类路径下，您需要准备

1. log4j.properties
2. META-INF/validation.xml

Log4j.properties

```
log4j.rootLogger=info, stdout 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender log4j.appender.stdout.layout=org.apache.log4j.PatternLayout log4j.appender.stdout.layout.ConversionPattern=%5p [%t] (%F:%L) - %m%n
```

META-INF/validation.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<validation-config
        xmlns="http://jboss.org/xml/ns/javax/validation/configuration"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://jboss.org/xml/ns/javax/validation/configuration">
    <default-provider>org.hibernate.validator.HibernateValidator</default-provider>
    <message-interpolator>org.hibernate.validator.engine.ResourceBundleMessageInterpolator</message-interpolator>
    <traversable-resolver>org.hibernate.validator.engine.resolver.DefaultTraversableResolver</traversable-resolver>
    <constraint-validator-factory>
        org.hibernate.validator.engine.ConstraintValidatorFactoryImpl
    </constraint-validator-factory>
</validation-config>
```

您可以根据需要自定义设置。

用法:

```java
public static class User {
    private String _lastName = "Chen";

    @NotEmpty(message = "Last name can not be null")
    public String getLastName() {
        return _lastName;
    }

    public void setLastName(String name) {
        _lastName = name;
    }
}
```

```
@validator('beanValidator')
```

```xml
<window id="win" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init(foo.MyViewModel)"
        validationMessages="@id('vmsgs')">
    <textbox id="tb" value="@bind(vm.user.lastName) @validator('beanValidator')"/>
    <label value="@load(vmsgs[tb])"/>
</window>
```

**Validate Form's Property Loaded from a Bean(验证表单属性，加载自bean)**

> Available for ZK: EE
>
> Since 6.0.1

它还支持验证从bean [3]加载的表单对象的属性。

```xml
<grid form="@id('fx') @load(vm.user) @save(vm.user,after='save')">
    <textbox id="tb" value="@bind(fx.lastName) @validator('beanValidator')"/>
    <label value="@load(vmsgs[tb])"/>
</grid>
```

[3]：它验证表单的最后一个加载的bean，这意味着它不支持验证尚未从bean加载的表单。

**Form Bean Validator**

> Available for ZK: EE
>
> Since 6.0.1

与Bean验证程序一样，此验证程序也集成了JavaBean验证并验证Bean的所有保存属性。 有关配置和JavaBean使用情况，请参阅＃Prepare_to_Use_JSR_303。

用法：

将此验证器与formBeanValidator一起使用，并通过验证器的prefix参数设置一个unique [1]前缀键。 当Bean的任何属性无效时，它将无效消息放入带有键prefix + propertyName的验证消息所有者。

> [1]：前缀在同一binder中必须唯一。

语法：`@validator('formBeanValidator',prefix='p_')`

```xml
<window id="win" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init(foo.MyViewModel)"
        validationMessages="@id('vmsgs')">
    <grid width="600px" form="@id('fx') @load(vm.user) @save(vm.user,after='save')
@validator('formBeanValidator',prefix='p_')">
        <textbox value="@bind(fx.firstName)"/>
        <label value="@load(vmsgs['p_firstName'])"/>
    </grid>
    <!--more components-->
</window>
```

**Limitation(局限性)**

该验证器无法验证表单对象的嵌套属性，例如`fx.firstProp.secondProp`。 它只能验证`fx.firstProp`之类的属性。 如果要验证表单对象的嵌套属性，则需要使用概念-ZK 8中的`Groups`。

**Groups**

> Available for ZK: EE
>
> Since 8.0.0

约束分组的概念意味着即使约束属于不同的JavaBean，我们也可以轻松地将约束分为几组。 每个验证者调用都可以验证每个组中的那些约束。

**用法：**

使用验证组有四个主要部分：

- Group Interface
- Java Beans
- @Validator in zul file 
- ViewModel

首先，我们需要定义一个空接口：

```java
public interface GroupValidation {}
```

并且有两个JavaBean：PersonDto和NameDto

```java
public class PersonDto {
    private NameDto nameDto;
    private int age;

    @Max(value = 120, groups = GroupValidation.class)
    public int getAge() {
        return age;
    }
    // getter/setter
}
```

```java
public class NameDto {
    private String name;

    @Size(min = 4, max = 10, groups = GroupValidation.class)
    public String getName() {
        return name;
    }
    // setter
}
```

在这两个JavaBean中，我们可以在Java注解中看到属性和组的约束。

然后，我们应该在zul和viewmodel中指定组。

语法：`@validator('formBeanValidator', prefix='p_', groups=vm.validationGroups)"`

zul文件：

```xml
<div form="@id('fx') @load(vm.personDto) @save(vm.personDto, after='submit') @validator('formBeanValidator', prefix='p_', groups=vm.validationGroups)">
    <intbox value="@bind(fx.age)"/>
    <label id="err1" value="@load(vmsgs['p_age'])"/>
    <textbox value="@bind(fx.nameDto.name)"/>
    <label id="err2" value="@load(vmsgs['p_nameDto.name'])"/>
</div>
```

viewmodel文件：

```java
public class FromValidationViewModel {
    private PersonDto personDto = new PersonDto();

    public PersonDto getPersonDto() {
        return personDto;
    }

    public Class[] getValidationGroups() {
        return new Class[]{GroupValidation.class};
    }

    @Command("submit")
    public void submit() {
    }
}
```

### **Global Command Binding(全局命令绑定)**

**概述**

全局命令绑定类似于本地命令绑定，但是目标变为全局命令。 只能通过ViewModel的Root View组件及其子组件的事件来触发本地命令。 与命令绑定的主要区别在于事件不必属于ViewModel的根视图组件或其子组件。 默认情况下，我们可以将事件绑定到同一桌面上的任何ViewModel的全局命令。 当我们触发全局命令时，将执行相同作用域的所有ViewModel中所有匹配的全局命令。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162103.png" alt="image-20200607141140671" style="zoom:67%;" />

#### **Default Global Command(默认全局命令)**

> Since 6.5.1

全局命令还支持`@DefaultGlobalCommand`，以将方法标记为默认全局命令方法，仅当全局命令绑定与任何其他全局命令方法不匹配时才调用该方法。 binder收到全局命令后，便开始通过匹配其名称来查找ViewModel的命令方法。 仅当binder找不到匹配的方法时，它才会调用默认的全局命令方法。

假定下面的ViewModel中只有两个全局命令方法。 如果我们触发全局命令`exit`，则绑定器将调用默认的全局命令方法`defaultAction()`，因为它找不到名为 `exit` 的命令方法。

**ViewModel with default command method**

```java
public class OrderVM {
    @GlobalCommand
    public void newOrder() { 
        ...
    }

    @DefaultGlobalCommand
    public void defaultAction() { 
        ...
    }
}
```

用法：

#### **Notification(通知)**

假设我们在页面中有2个区域，一个是用于添加项目的主要区域，另一个是用于显示项目的列表区域。 每个区域都绑定到一个ViewModel。 两个ViewModel可以从单个数据源访问相同的项目列表，为简化起见，我们使用静态列表作为数据源。 主要问题是在我们单击`add`按钮（添加新项目）后，如何通知另一个ViewModel刷新项目列表。 我们可以通过将`add`按钮的`onClick`事件绑定到列表区域的ViewModel的全局命令来实现它，并在该全局命令中刷新项目列表并通知相关属性更改以更新View。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162132.png" alt="image-20200607141208650" style="zoom:67%;" />

具有2个ViewModels的zul:

```xml
<hlayout>
    <vbox id="mainArea" width="200px" height="300px"
          style="border:dashed 2px" visible="@bind(vm.visible)" apply="org.zkoss.bind.BindComposer"
          viewModel="@id('vm') @init('org.zkoss.mvvm.examples.globalcommand.AddViewModel')"
          validationMessages="@id('vmsgs')">
        <label value="Main Panel" style="font-size:30px"/>
        Enter a Item (at least 3 characters):
        <textbox id="iBox"
                 value="@load(vm.item)@save(vm.item, before='add') @validator(vm.itemValidator)"/>
        <label value="@load(vmsgs[iBox])" style="color:red"/>
        <button label="Add"
                onClick="@command('add') @global-command('refresh')"/>
        <separator height="20px"/>
        <label value="@load(vm.msg)"/>
    </vbox>
    <vbox id="listArea" width="400px" height="300px"
          visible="@bind(vm.visible)" apply="org.zkoss.bind.BindComposer"
          style="border:solid 2px"
          viewModel="@id('vm') @init('org.zkoss.mvvm.examples.globalcommand.ListViewModel')">
        <listbox model="@load(vm.items)">
            <listhead>
                <listheader label="Items"/>
            </listhead>
            <template name="model">
                <listitem>
                    <listcell label="@load(each)"/>
                </listitem>
            </template>
        </listbox>
        <label value="@load(vm.lastUpdate)"/>
    </vbox>
</hlayout>
```

我们将`onClick`事件绑定到`本地命令add`和`全局命令refresh`

**AddViewModel.java**

```java
public class AddViewModel {
    private String msg;

    @Command
    @NotifyChange("msg")
    public void add() {
        ItemList.add(item);
        msg = "Added " + item;
        //other code
    }
}
```

**ListViewModel.java**

```java
public class ListViewModel {
    private List<String> items;
    private Date lastUpdate;

    @GlobalCommand @NotifyChange({"items", "lastUpdate"})
    public void refresh() {
        items = ItemList.getList();
        lastUpdate = Calendar.getInstance().getTime();
    }
    //other code
}
```

> - 声明全局命令并通知更改。 （第5行）
> - 该命令方法将重新加载项目并将当前时间设置为最后更新时间。 （第9行）

**A new item "hat" is added**

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162141.png" alt="image-20200607141235392" style="zoom:50%;" />

在上面的示例中，我们将onClick事件绑定到本地和全局命令，`全局命令将始终在执行本地命令后执行`。 如果由于验证失败未执行本地命令，则将不执行全局命令。 我们有一个验证器来验证商品的名称，其长度不能小于3。当我们输入一个简短的商品名称来违反验证规则时，不会执行本地命令。 因此，也不执行全局命令。 我们可以确认此行为，因为主区域中的消息和列表区域中的最后更新没有改变。

**Validation failure blocks command execution**

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162146.png" alt="image-20200607141257158" style="zoom:50%;" />

#### **One to Many Communication(一对多通信)**

我们还可以使用全局命令绑定在一次事件触发时传达多个ViewModel。 如果事件绑定到名为`show`的全局命令，则将在所有ViewModel中执行所有名称为`who`的全局命令。 与本地命令的另一个区别是，您可以将事件绑定到全局命令而无需确保其存在。 如果您绑定的全局命令不存在，则它没有响应，也不会引发任何异常。 事件和全局命令之间的关系就像`发布者-订阅者`。 `发布者发布事件，只有订阅该事件的ViewModel才执行相应的全局命令`。

根据前面的示例，假定我们要同时隐藏和显示两个带有菜单项的区域。 我们可以将menuitem的onClick事件绑定到名为`show`的全局命令，两个ViewModels实现`show`命令以显示自身，`hide`命令也是如此。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162151.png" alt="image-20200607141319928" style="zoom:50%;" />

**One to many communication(一对多通信)**

```xml
<vlayout>
    <menubar width="600px" apply="org.zkoss.bind.BindComposer"
             viewModel="@id('vm') @init('org.zkoss.mvvm.examples.globalcommand.ControlViewModel')">
        <menuitem label="Show" onClick="@global-command('show')"></menuitem>
        <menuitem label="Hide" onClick="@global-command('hide')"></menuitem>
    </menubar>
    <!-- main area-->
    <!-- list area-->
</vlayout>
```

添加带有2个菜单项的菜单栏：显示和隐藏, 绑定到全局命令

**ViewModel implement show & hide command**

```java
public class AddViewModel {
    private boolean visible = true;

    @GlobalCommand @NotifyChange("visible")
    public void show() {
        visible = true;
    }

    @GlobalCommand @NotifyChange("visible")
    public void hide() {
        visible = false;
    }
}
```

`ListViewModel`的实现应具有相同的命令方法。

单击`hide`菜单项时，两个ViewModel的全局命令`hide`都将被执行，因此2个区域消失。

#### **Command Execution(命令执行)**

全局命令的执行非常简单。 它只是执行一个命令方法，然后重新加载@NotifyChange注解中指定的那些属性。 但是，如果您将事件同时绑定到本地和全局命令，则绑定程序始终会首先执行本地命令。 如果有任何原因阻止了本地命令的执行，例如 验证失败，它将不会执行全局命令。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162156.png" alt="image-20200607141342169" style="zoom:50%;" />

#### **Background Concept(背景概念)**

当我们将BindComposer应用于组件时，它将自动为我们创建一个binder。 由于ViewModel是POJO，因此绑定器就像是与他人进行通信的代理。 默认情况下，所有绑定程序都订阅默认的桌面作用域事件队列，因此，这是绑定程序之间的一种常见通信机制。

当我们通过事件触发全局命令时，binder将事件发布到其订阅的队列中。 订阅同一范围内相同队列的所有绑定程序（包括发布事件的绑定程序）将接收此事件，并尝试在其关联的ViewModel中查找请求的全局命令。 如果绑定程序找到匹配的全局命令，它将执行该命令。 否则，它将忽略该事件而不会引发任何异常。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162201.png" alt="image-20200607141404114" style="zoom:50%;" />

在上图中，当我们在`ViewA`中触发全局命令`show`时，`BinderA`将事件发布到事件队列中。 预订相同事件队列的`BinderB`和`BinderA`本身都将接收该事件，并尝试在其关联的ViewModels（`ViewModelA`和`ViewModelB`）中查找全局命令`show`

ZK允许您更改binder预订的事件队列的名称和范围。 请参阅数据`绑定/活页夹`。

#### **Trigger a Command Dynamically(动态触发命令)**

除了在ZUL中触发全局命令外，我们还可以通过调用API来实现。 对于上面的示例，我们可以在`本地命令add`中触发全局命令，代码如下：

```java
@Command
@NotifyChange("msg")
public void add() {
    ItemList.add(item); //add an item
    msg = "Added " + item; //update message 
    BindUtils.postGlobalCommand(null, null, "refresh", null);
}
```

> BindUtils.postGlobalCommand()的第一个参数是队列名称，第二个是队列作用域。

您可以在`Composer`中调用此方法，并且以MVC模式编写的页面可以与ViewModel通信。 示例代码段如下：

```java
public class MyComposer extends SelectorComposer { 
    //other codes
    @Listen("onClick=button#postx")
    public void postX() {
        Map<String, Object> args = new HashMap<String, Object>();
        args.put("data", "postX");
        BindUtils.postGlobalCommand("myqueue", EventQueues.DESKTOP, "cmdX", args);
    }
}
```

### **Collection and Selection(集合和选择)**

#### **Property Binding in Collection Type(集合类型中的属性绑定)**

假设我们有以下ViewModel，它具有集合类型属性：

```java
public class ItemsVM {
    private ItemService itemService = new ItemService();
    private List<Item> itemList = itemService.getAllItems();

    public List<Item> getItemList() {
        return itemList;
    }
}
```

对于ViewModel的`Collection`类型属性，我们通常将其绑定到具有 `model` 属性的那些组件，例如Listbox，Grid或Tree。 （您不应将多个属性绑定到共享集合对象（例如，静态列表），因为可能会出现并发问题。）

```xml
<grid width="400px" model="@bind(vm.itemList)">
```

如果该属性是Java集合类型对象（如List或Set）之一，则内部转换器会自动将集合对象转换为ListModel对象，以方便用户使用。 但是它有局限性，我们将在后面的部分中讨论它们。

##### **Implicit Iteration Variable(隐式迭代变量)**

将集合类型属性绑定为数据源之后，我们必须指定如何使用`<template>`渲染模型的每个对象，ZK将根据`<template>`中指定的片段迭代创建组件。 我们可以在`<template>`中使用2个隐式变量。

- `each`: `迭代对象变量`，它们引用模型的每个对象。 我们可以使用它来使用点符号来访问对象的属性，例如 `each.name`。
- `forEachStatus`，`迭代状态变量`。 它用于通过`forEachStatus.index`获取迭代索引。

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.ItemsVM')">
    <grid width="400px" model="@bind(vm.itemList)">
        <columns>
            <column label="index"/>
            <column label="name"/>
        </columns>
        <template name="model">
            <row>
                <label value="@bind(forEachStatus.index)"/>
                <label value="@bind(each.name)"/>
            </row>
        </template>
    </grid>
</window>
```

> 使用模板元素时，我们不需要将`<rows>`放在`<grid>`内。

为了提高可读性，我们还可以设置模板的`var`和`status`属性来更改隐式变量的名称。

var ="`myvar`"，设置`迭代对象变量`的名称。 如果您未同时设置`status`属性，则ZK会将`迭代状态变量`的名称设置为`myvarStatus`，就是在迭代变量名称后面附加`Status`。

status ="`mystatus`"，设置`迭代状态变量`的名称。

**The ViewModel for tree example**

```java
public class TreeVM {
    TreeModel<TreeNode<String>> itemTree;
    //omit getter and setter for brevity
}
```

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.TreeVM')">
    <tree model="@bind(vm.itemTree)" width="400px">
        <treecols>
            <treecol label="name"/>
            <treecol label="index"/>
        </treecols>
        <template name="model" var="node" status="s">
            <treeitem>
                <treerow>
                    <treecell label="@bind(node.data)"/>
                    <treecell label="@bind(s.index)"/>
                </treerow>
            </treeitem>
        </template>
    </tree>
    ...
</window>
```

使用模板元素时，我们不需要将`<treechildren>`放在`<tree>`内。

#### **Collection Property Binding with Dynamic Template(使用动态模板的集合属性绑定)**

动态模板允许开发人员指定在渲染容器组件时要在不同条件下应用的模板，例如 网格。 所有支持“模型”属性的组件都支持动态模板。 您可以使用以下语法在“模型”属性中使用此功能：

```
@template( [EL-expression] )
```

Binder将EL表达式评估为模板名称，并寻找相应的模板来渲染子组件。 如果指定的模板在当前组件中不存在，它将查找父组件的模板。 如果未指定@template，则默认使用名为model的模板。

**指定模板名称**

```xml
<grid model="@bind(vm.itemList) @template('myTemplate')">
    <template name="myTemplate">
        <!-- child components -->
    </template>
</grid>
```

使用在@template中指定名称的模板。

我们可以使用EL来决定使用哪个模板。

**Conditional(有条件的)**

```xml
<grid model="@bind(vm.itemList) @template(vm.type eq 'foo'?'template1':'template2')">
    <template name="template1">
        <!-- child components -->
    </template>
    <template name="template2"> 
        <!-- child components --> 
    </template>
</grid>
```

我们还可以在`@template()`的EL表达式中使用隐式变量`each`和`forEachStatus`。 这意味着我们可以在绑定集合中的不同对象(行)上动态应用不同的模板。

**Condition with implicit variables(带隐式变量的条件)**

```xml
<grid model="@bind(vm.itemList) @template(each.type eq 'A'?'templateA':'templateB')">
    <template name="templateA">
        <!-- child components -->
    </template>
    <template name="templateB"> 
        <!-- child components --> 
    </template>
</grid>
```

> 第1行：假定绑定集合中的对象具有属性“ type”。 它的值可以是A或B。

#### **Collection Property in Children Binding(在子绑定中使用集合属性)**

对于那些不支持`model`属性的组件，我们仍然可以使用子绑定将它们绑定到集合类型属性（请参阅`数据绑定/子绑定`以了解基本概念。）。

对于已经支持`model`属性的那些组件，我们还可以使用子级绑定来更改原始组件的渲染。 例如，Radios在RadioGroup中水平放置，我们可以使用绑定在Vlayout上的子级将Radios垂直放置。

**Children binding in radiogroup(radiogroup 使用子绑定)**

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.ItemsVM')">
    <radiogroup>
        <vlayout children="@bind(vm.itemList)">
            <template name="children">
                <radio label="@load(each.name)" value="@load(each.name)"></radio>
            </template>
        </vlayout>
    </radiogroup>
</window>
```

**Combine with Dynamic Template(与动态模板结合)**

我们可以将绑定的子代与动态模板结合起来，以便在不同的条件下渲染不同的子代组件。

这是一个动态创建菜单栏的示例。 如果菜单项没有子菜单，则创建Menuitem，否则创建Menu。

**动态菜单栏示例**

```xml
<menubar id="mbar" children="@bind(vm.menuList) @template(empty each.children?'menuitem':'menu')">
    <template name="menu" var="menu">
        <menu label="@bind(menu.name)">
            <menupopup children="@bind(menu.children) @template(empty each.children?'menuitem':'menu')"/>
        </menu>
    </template>
    <template name="menuitem" var="item">
        <menuitem label="@bind(item.name)" onClick="@command('menuClicked',node=item)"/>
    </template>
</menubar>
```

上面的示例使用树状数据结构，子模板将递归地呈现内容。

**Dynamic menu bar screenshot**

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162214.png" alt="image-20200607141452321" style="zoom: 67%;" />

#### **Selection in Collection(集合中的选择)**

列表框和树存储用户选择状态，包括单个或多个选择。 默认情况下启用单选。 启用多重选择的方式取决于您使用`model`属性绑定的对象的类型。 ZK允许我们将选择状态与ViewModel的属性绑定。

**Single Selection(单选)**

对于单选状态，ZK提供2种状态。 一个被选择的索引，另一个被选择的数据项。 我们可以在ViewModel中分别为它们创建2个属性。

**Properties for selection(选择属性)**

```java
public class SingleSelectionVM{
    private ItemService itemService = new ItemService(); 
    private List<Item> itemList = itemService.getAllItems(); 
    private int pickedIndex;
    private Item pickedItem;
    //omit getter and setter for brevity
}
```

第4行：pickedIndex是所选索引的整数。
第5行：pickedItem的类型应等于集合中对象的类型。

##### **Binding to Selected Index(绑定到选定索引)**

要保存或恢复组件的选定索引，我们应该使用`@bind`将`selectedIndex`属性绑定到ViewModel的属性。 如果我们在ViewModel中更改选择状态的属性值，则组件的选择状态也会更改。

**Listbox selectedIndex example**

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.SingleSelectionVM')">
    <listbox width="400px" model="@bind(vm.itemList)" selectedIndex="@bind(vm.pickedIndex)">
        <listhead>
            <listheader label="index"/>
            <listheader label="name"/>
        </listhead>
        <template name="model" var="item" status="s">
            <listitem>
                <listcell label="@bind(s.index)"/>
                <listcell label="@bind(item.name)"/>
            </listitem>
        </template>
    </listbox>
</window>
```

##### **Binding to Selected Item(绑定到所选项目(对象))**

要保存或恢复所选项目，我们应该将`selectedItem`绑定到其类型等于Model对象的属性。 在我们的示例中，我们应该将 `selectedItem`绑定到Item对象。

**Listbox selectedItem example**

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.SingleSelectionVM')">
    <listbox width="400px" model="@bind(vm.itemList)" selectedItem="@bind(vm.pickedItem)">
        <listhead>
            <listheader label="index"/>
            <listheader label="name"/>
        </listhead>
        <template name="model" var="item" status="s">
            <listitem>
                <listcell label="@bind(s.index)"/>
                <listcell label="@bind(item.name)"/>
            </listitem>
        </template>
    </listbox>
</window>
```

Tree的用法与此类似。 大多数支持`model`属性的组件都支持上述2个属性，但某些不支持，例如 `Tree不支持selectedIndex属性`。 （有关支持的属性，请参考每个组件的javadoc）

**ViewModel for tree example(Tree 的 ViewModel 示例)**

```java
public class TreeSelectionVM {
    private TreeModel<TreeNode<String>> itemTree;
    private String pickedItem;
    //omit getter and setter for brevity
}
```

**Tree example**

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.TreeSelectionVM')">
    <tree id="tree" model="@bind(vm.itemTree) " width="600px" selectedItem="@bind(vm.pickedItem)">
        <treecols>
            <treecol label="name"/>
            <treecol label="index"/>
        </treecols>
        <template name="model" var="node" status="s">
            <treeitem open="@bind(node.open)">
                <treerow>
                    <treecell label="@bind(node.data)"/>
                    <treecell label="@bind(s.index)"/>
                </treerow>
            </treeitem>
        </template>
    </tree>
</window>
```

**Custom layout for Radiogroup(为 Radiogroup 自定义布局)**

Radiogroup水平排列其子Radio，用户无法更改此排列，但是我们可以使用子级绑定来垂直排列Radio。 为了使`selectedItem`正常工作，我们应该在`<radiogroup>`之前放置`<radio>`。

> Since 6.5.1

我们可以将 `selectedItem`与任何类型的对象绑定，而不仅限于字符串。

```xml
<vlayout children="@load(vm.itemList)">
    <template name="children">
        <radio label="@load(each.name)" value="@load(each)" radiogroup="rg"/>
    </template>
</vlayout>
<radiogroup id="rg" selectedItem="@bind(vm.pickedItem)"/>
```

#### **Binding to Selected Item with Static UI Data(使用静态 UI 数据绑定到所选项目)**

使用静态数据时，我们仍然可以使用类似的规则来获取并保存`selectedItem`。 在下面的示例中，我们应该将`selectedItem`与String属性绑定。 因为我们将String设置为Radio的值。 它们的类型必须匹配。

**Static data model(静态数据模型)**

```xml
<radiogroup selectedItem="@bind(vm.pickedItemName)">
    <radio label="Item 0" value="Item 0"/>
    <radio label="Item 1" value="Item 1"/>
    <radio label="Item 2" value="Item 2"/>
</radiogroup>
```

第1行：`vm.pickedItemName`是String属性

在上一节中，我们已经说过，对象`selectedItem`属性绑定的类型应等于模型的对象。 使用静态数据（不使用模型）时，选择组件的值类型应等于`selectedItem`的属性类型。 因为ZK通过比较`selectedItem`属性和所选组件的`value`属性来恢复选择状态。

**Custom Single Selection(自定义单选)**

因为Tabbox不支持`model`属性，所以我们应该做一些额外的工作来保持选择状态。 我们可以使用所选项目的变量创建一个ViewModel，并创建一个命令方法来保存通过命令绑定传递的所选项目。

```java
public class CustomSingleSelectionVM {
    private ItemService itemService = new ItemService();
    private List<Item> itemList = itemService.getAllItems();
    private Item pickedItem = itemList.get(0);

    //omit setter and getter for brevity
    @NotifyChange("pickedItem")
    @Command
    public void select(@BindingParam("item") Item item) {
        pickedItem = item;
    }
}
```

首先，我们使用子级绑定动态创建Tab和Tabpanel，然后在`onSelect`上编写命令绑定以传递所选项目。 然后，我们可以通过将所选项目与每个Tab绑定的对象进行比较来设置`selected`属性。

```xml
<div apply="org.zkoss.bind.BindComposer" width="600px"
     viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.CustomSingleSelectionVM')">
    <tabbox>
        <tabs children="@load(vm.itemList)">
            <template name="children">
                <tab label="@load(each.name)" selected="@load(vm.pickedItem eq each)"
                     onSelect="@command('select',item=each)"/>
            </template>
        </tabs>
        <tabpanels children="@load(vm.itemList)">
            <template name="children">
                <tabpanel height="200px">
                    <label value="@load(each.name)"/>
                </tabpanel>
            </template>
        </tabpanels>
    </tabbox>
</div>
```

#### **Multiple Selections(多项选择)**

某些组件支持多种选择，但是在使用前需要将其启用。 启用多个选择的方式取决于您使用`model`属性绑定的属性的类型。 如果属性类型是Java Collection类型（例如List，Set或Map），则应在组件上指定`multiple ="true"`以启用多个选择。 如果属性的类型为ListModel并且实现了Selectable，则应调用`setMultiple(true)`启用它。

为了保持多个选择状态，我们应该将`selectedItems`绑定到类型为`Set`的属性。

**Selected Set**

```java
public class MultipleSelectionsVM {
    private ItemService itemService = new ItemService();
    private List<Item> itemList = itemService.getAllItems();
    private Set pickedItemSet;
    //omit getter and setter for brevity
}
```

**Multiple selections enabled Listbox**

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.MultipleSelectionsVM')">
    <listbox width="400px" model="@bind(vm.itemList)" checkmark="true" multiple="true"
             selectedItems="@bind(vm.pickedItemSet)">
        <listhead>
            <listheader label="index"/>
            <listheader label="name"/>
        </listhead>
        <template name="model" var="item" status="s">
            <listitem>
                <listcell label="@bind(s.index)"/>
                <listcell label="@bind(item.name)"/>
            </listitem>
        </template>
    </listbox>
</window>
```

第3行：因为vm.itemList是Java列表对象，所以我们可以通过将`multiple`设置为`true`来启用多重选择。

**Custom Multiple Selections(自定义多选)**

某些组件不支持 `selectedItems`属性, 如`Checkbox`，我们仍然可以通过传递参数来保持选择状态。 创建以下ViewModel，其中包含一个用于存储所选项目的集合以及一个根据传递的参数更新集合中项目的命令。

```java
public class CustomMultipleSelectionsVM {
    private ItemService itemService = new ItemService();
    private Set<Item> pickedItemSet = new HashSet<Item>();

    @Command
    @NotifyChange("pickedItemSet")
    public void pick(@BindingParam("checked") boolean isPicked, @BindingParam("picked") Item item) {
        if (isPicked) {
            pickedItemSet.add(item);
        } else {
            pickedItemSet.remove(item);
        }
    }
//omit getter and setter for brevity
}
```

第8行：有关参数注解的详细信息，请参阅“`高级/参数`”。

复选框不支持`model`属性，因此我们使用子级绑定将其呈现在商品列表上。 我们还将`onCheck`属性绑定到命令，并将选定的Item对象传递给命令方法以更新选定的集合。

**Selected items for checkboxes**

```xml
<window apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.collection.CustomMultipleSelectionsVM ')">
    <vlayout id="vlayout" children="@load(vm.itemList)">
        <template name="children">
            <checkbox label="@load(each.name)"
                      onCheck="@command('pick', checked=self.checked, picked=each)"></checkbox>
        </template>
    </vlayout>
    <label value="@bind(vm.pickedItemSet)"/>
</window>
```

第6行：在命令绑定中传递参数以更新所选项目。

#### **Choose a Component's Model Type(选择组件的模型类型)**

如前一节所述，我们可以将Java Collection type属性绑定为组件的模型。 内部转换器会自动将其转换为LIstModel。 这很方便，但代价之一是`当集合大小更改时滚动位置丢失`。 如果滚动到索引为50的项目并删除（或添加）项目。 该组件将强制滚动回第一项，因为它必须重新转换并重新呈现所有项。 一种简单的处理方法是将Listbox设置为“分页”模具。 另一种方法是使用ListModelList包装集合对象。

与性能有关的另一个问题是，当使用Java集合对象作为默认模具中的模型时，`Listbox重新渲染所有项目`[1]。 通过删除或添加项目来更改集合属性时，必须通知其更改。 这将触发列表框从头开始重新渲染。 但是，如果将`ListModelList`用作模型并通过调用其方法`add()`和`remove()`对其进行修改。 `ListModelList`将自动更新到客户端，而无需显式指定属性通知。 并且此更新`仅重新渲染受影响（添加或删除）的项目`，而不是所有项目,这样更高效。[2]

但是，如果您有大量数据，那么获取它们将花费很长时间。 您应该实现自己的模型对象，我们将在“`高级`”部分中讨论。

> [1]：如果启用ROD，则仅呈现可见项。
> [2]：如果仍为ListModelList属性指定属性通知，则Listbox呈现所有项目（如果ROD被禁用）。 这样一来，您将不会获得性能提升。

**Use Java List as a model**

```java
public class ModelTypeVM extends SingleSelectionVM {
    protected List<Item> itemList;

    @Init
    public void init() {
        pickedItem = new Item();
        itemService = new ItemService(100);
        itemList = itemService.getAllItems(); //return a java.util.List ...
    }

    @Command
    @NotifyChange({"itemList", "pickedItem"})
    public void add() {
        itemList.add(pickedItem);
        pickedItem = new Item();
    }

    @Command
    @NotifyChange({"itemList", "pickedItem"})
    public void delete() {
        int index = itemList.indexOf(pickedItem);
        if (index != -1) {
            itemList.remove(index);
            pickedItem = new Item();
        }
    }
    //omit setter and getter
}
```

第8行：`itemList`是`java.util.List`对象。
第14、23行：因为我们更改了属性`itemList`，所以我们应通知binder重新加载它。

**Use ListModelList as a model**

```java
public class ModelTypeVM extends SingleSelectionVM {
    private ListModelList<Item> itemListModel;

    @Init
    public void init() {
        pickedItem = new Item();
        itemService = new ItemService(100);
        ...
        itemListModel = new ListModelList<Item>(itemService.getAllItems());
    }

    @Command
    @NotifyChange("pickedItem")
    public void modelAdd() {
        itemListModel.add(pickedItem);
        pickedItem = new Item();
    }

    @Command
    @NotifyChange("pickedItem")
    public void modelDelete() {
        int index = itemListModel.indexOf(pickedItem);
        if (index != -1) {
            itemListModel.remove(index);
            pickedItem = new Item();
        }
    }
    //omit setter and getter
}
```

第11行：我们可以简单地通过将`java.util.List`传递给`ListModelList`的构造函数来包装它。

第15、24行：由于`ListModelList`将自动将更改更新到客户端，因此我们可以忽略以通知属性 `itemListModel`。

### **Client Binding(客户端绑定)**

> Since 8.0.0

为了与客户端库进行交互，客户端绑定可以帮助我们在本地html元素上发布ZK的数据绑定命令。 例如，您可以在视图模型中发布doClick命令，并在html Button中使用onClick事件。

**Implementation(实作)**

客户端绑定提供4种方法-客户端2种，服务器2种。 下图说明了它们的关系：

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162229.png" alt="image-20200607141550310" style="zoom:67%;" />

**在客户端**

首先，我们必须获取客户端绑定程序才能使用客户端方法。 要获取binder，只需使用：

```javascript
var binder = zkbind.$('$id');
```

有了客户端绑定程序后，我们可以使用以下两种方法将视图模型与返回到服务器的交互。

**Method 1 - command**

```javascript
binder.command(commandName, data);
```

此方法用于触发服务器上的命令。

**Parameters**

- `commandName`-服务器端的命令名称（ViewModel）。
- `data`-JavaScript对象，用于通过命令传递所需的任何信息。

注意：您还可以在数据对象中传递ZK小部件，并使用`@BindingParam`在服务器上获取相应的ZK组件。

**Method 2 - after**

```javascript
binder.after(commandName, callback);
```

在服务器上执行命令后，使用此方法在客户端上放置回调。

**Parameters**

- `commandName`-服务器端的命令名称（ViewModel）。
- `callback`-在服务器上执行命令后的回调函数。

**服务器端**

在服务器端，我们可以将以下两个注解用于客户端绑定。 它们应该放在我们的View模型的类声明的开头。

**Annotation 1 - NotifyCommand**

```java
@NotifyCommand(value="commandName", onChange="_vm_.expression")
```

NotifyCommand注解使我们可以在服务器上给定表达式发生更改时触发命令。 注意，触发的命令是我们ViewModel中的命令。 _vm_在这里表示当前ViewModel。

**Annotation 2 - ClientCommand**

```java
@ClientCommand(commandNames)
```

ClientCommand注解允许我们放置要在执行完成后通知客户端的命令。 请注意，只有我们在此注解中放置的命令才会触发我们在客户端放置在binder.after中的回调。

**Examples**

There are two examples using client binding:

- ZK Blog : ZK8: Work with Polymer Components using ZK’s new client side binding API
- ZK Small Talk :ZK8 Series: Interact with Client Side Libaries using ZK's New Client Side Binding