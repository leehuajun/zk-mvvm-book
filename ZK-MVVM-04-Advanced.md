# **Advanced(高级)**

我们将讨论一些高级主题，包括如何检索和操作隐式对象或UI组件。 这些方法无疑为应用程序开发人员提供了更大的灵活性，但是大多数方法`都增加了ViewModel和View之间的耦合`。 因此，它们削弱了使用MVVM模式的长处。 因此，我们仅向经验丰富的ZK用户推荐它们。

## **Parameters(参数)**

ZK允许您通过命令绑定注解将ZUL上EL可以引用的任何对象或值传递给命令方法。 您的命令方法的签名应具有一个对应的参数，该参数应使用`@BindingParam`进行注解，并具有相同的`类型`和`键`。

语法:

```java
@command('commandName', [arbitratyKey]=[EL-expression]) 
@global-command('commandName', [arbitratyKey]=[EL-expression]) 
[arbitratyKey]=[EL-expression]
```

该表达式是可选的, 仅在要将参数传递给命令方法时使用。

**本地 Command 例子**

假设我们在网格中有数据。 通常在行尾放置一个按钮来操作它，例如删除或更新。 我们通常需要一行的索引或域对象来执行更新之类的操作。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607162246.png" alt="image-20200607141735604" style="zoom:67%;" />

**Example to pass parameters(传参示例)**

```xml
<grid id="outergrid" width="700px" model="@bind(vm.items)">
    <columns>
        <column label="index"/>
        <column label="name"/>
        <column label="action" width="300px"/>
    </columns>
    <template name="model" var="item">
        <row>
            <label value="@bind(itemStatus.index)"/>
            <label value="@bind(item.name)"/>
            <hbox>
                <button label="Index" onClick="@command('showIndex', index=itemStatus.index)"/>
                <button label="Delete" onClick="@command('delete', item=item)"/>
            </hbox>
        </row>
    </template>
</grid>
```

第12行：我们通过迭代状态变量检索行索引（`Integer类`），并将其与键`index`一起传递。
第13行：我们通过迭代变量检索域对象（`Item类`），并将其与键`item`一起传递。

**Command methods in the ViewModel(ViewModel 的 Command 方法)**

```java
@Command
public void showIndex(@BindingParam("index") Integer index) {
    message = "item index: " + index;
}

@Command
public void delete(@BindingParam("item") Item item) {
    int i = items.indexOf(item);
    items.remove(item);
    message = "remove item index " + i;
}
```

第2行：命令方法showIndex()的参数列表中应有一个Integer。 我们还必须在@BindingParam中指定键“ index”。

第7行：与delete()相同，它的参数列表中应包含带有Item类的参数。 我们还必须在@BindingParam中指定键“ item”。

**全局 Command 例子**

全局命令绑定中的传递参数可以在ViewModel之间共享数据。

以下代码将选定的项目传递给另一个ViewModel。

```xml
<button label="Submit" onClick="@command('submit') @global-command('detail', name=vm.selectedName)" />
```

全局命令方法通过@BindingParam接收参数。

```java
@GlobalCommand
public void detail(@BindingParam("name") String name) {
    //...
}
```

**Parameter Default Value(参数默认值)**

您可以为具有参数的命令方法选择不传递参数。 如果不传递该参数，则该参数将为null，但可以选择使用`@Default`为其提供默认值。 该注解可以与其他参数相关的注解一起使用。

**指定参数的默认值**

```java
//getter and setter
@Command
public void showIndex(@BindingParam("index") @Default("0") Integer index) {
    this.index = index;
}
```

我们将索引的默认值设置为0。

**Example to bind above command(绑定上面 command 的例子)**

```java
<label value="@bind(vm.index)"/>
<button label="button01" onClick="@command('showIndex', index=9)"/>
<button label="button02" onClick="@command('showIndex')"/>
```

单击button01，标签的值为9。
单击button02，标签的值为0。



您甚至可以传递UI组件。 此方法可以使您直接操作UI组件，但还可以在ViewModel和View之间添加耦合，从而削弱MVVM模式的长处。

**Example to pass a UI component(传递 UI 组件)**

```xml
<listbox model="@load(vm.items)" selectedItem="@bind(vm.selected)" hflex="true" height="300px">
    <listhead>
        <listheader label="Name"/>
        <listheader label="Price" align="center"/>
        <listheader label="Quantity" align="center"/>
    </listhead>
    <template name="model" var="item">
        <listitem onMouseOver="@command('popupMessage', target=self, content=item.description)">
            <listcell label="@bind(item.name)"/>
            <listcell label="@bind(item.price)@converter('formatedNumber', format='###,##0.00')"/>
            <listcell label="@bind(item.quantity)" sclass="@bind(item.quantity lt 3 ?'red':'')"/>
        </listitem>
    </template>
</listbox>
```

我们通过隐式对象 `self` 传递listitem。

**Command method to receive UI component(Command 方法接收 UI 组件)**

```java
@Command
public void popupMessage(@BindingParam("target") Component target, @BindingParam("content") String content) {
    //...
}
```

**Retrieve Context Object(检索上下文对象)**

通过在初始方法（使用@Init的方法）和命令方法（使用@Command的方法）中的各种上下文范围中检索值或隐式对象，可以在这些方法的参数上应用与参数相关的注解。 我们在“参数”下的部分中列出了由参数注解和相关注解相关的所有可用HTTP上下文对象。

**Retrieve HTTP Context Object(检索HTTP上下文对象)**

**Example to get browser information(获取浏览器信息示例)**

```java
public class HttpParamVM {
    String headerParam;

    @Init
    public void init(@HeaderParam("user-agent") String browser) {
        headerParam = browser;
    }
}
```

您可以在一个方法的参数上应用多个与参数相关的注解，并且绑定器将按指定顺序在多个上下文范围中检索值。 它继续在下一个上下文范围中查找，直到检索第一个非空对象。



**Multiple context scope retrieval example(多上下文范围检索示例)**

```java
@Init
public void init(@CookieParam("nosuch") @HeaderParam("user-agent") String guess) {
    cookieParam = guess;
}
```

在上面的示例中，它首先搜索HTTP请求cookie。 如果找不到非空对象，它将继续在HTTP请求标头中进行检索。



**Retrieve ZK Context Object(检索ZK上下文对象)**

您还可以通过@ContextObject接收具有各种org.zkoss.bind.annotation.ContextType的ZK上下文对象，包括Execution，Desktop，Session，BindContext，Binder等。我们在`Syntax/ViewModel/Parameters/@ContextParam`中列出了可以通过@ContextObject检索的所有可用上下文对象。

我们通过初始方法和命令方法检索当前绑定源组件和ViewModel的视图组件。

**Example to retrieve ZK context object**

```java
//getter and setter
@Init
public void init(@ContextParam(ContextType.COMPONENT) Component component, @ContextParam(ContextType.VIEW) Component view) {
    bindComponentId = component.getId();
    bindViewId = view.getId();
}

@Command
public void showId(@ContextParam(ContextType.COMPONENT) Component component, @ContextParam(ContextType.VIEW) Component view) {
    bindComponentId = component.getId();
    bindViewId = view.getId();
}
```

我们创建2个标签，它们绑定到当前绑定组件的ID和View组件的ID。

**A zul bound to above ViewModel**

```xml
<vbox id="vbox" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('eg.ContextParamVM')">
    <label id="componentId" value="@load(vm.bindComponentId)"/>
    <label id="viewId" value="@load(vm.bindViewId)"/>
    <button id="cmd" label="cmd" onClick="@command('showId')"/>
</vbox>
```

加载页面后，组件的ID和视图的ID均为`vbox`。 对于`init()`，其当前绑定源组件是`vbox`。 当我们在vbox上应用@Init时，它始终是ViewModel的视图组件。

单击按钮后，组件的ID变为“ cmd”，因为命令绑定的绑定源组件是按钮，视图的ID不变。



**Retrieve Event Object(检索事件对象)**

> Since 6.0.1 and ZK EE
>
> Since 6.5.2, it is supported in CE

有两种方法可以检索事件对象：

1. 在ZUL的`@command`参数列表中使用保留的关键字`event`。
2. 在命令方法的参数上应用注解`@ContextParam(ContextType.TRIGGER_EVENT)`。

在这里，我们有一个标签来显示用户在`textbox`中键入的内容。 用户的键入内容存储在`InputEvent`的`value`属性中，因此当将命令绑定到`onChanging`属性时，我们可以通过`event.value`传递它。

```xml
<vbox>
    <label id="msg" value="@load(vm.message)"/>
    <textbox onChanging="@command('showTyping',v=event.value)"/>
</vbox>
```

**ViewModel for the zul above**

```java
public class EventViewModel {
    //getter for "message"
    @Command
    @NotifyChange("message")
    public void showTyping(@BindingParam("v") String value, @ContextParam(ContextType.TRIGGER_EVENT) InputEvent event) {
        typingMessage = value;
        //typingMessage = event.getValue(); same effect
    }
}
```

第一个参数从ZUL接收值。 第二个由绑定器传递，因此我们不需要在ZUL中指定它。

## **Binding in Special Attribute(绑定特殊属性)**

**特殊属性问题**

ZK Bind是在组件创建后对它们进行的后处理工作，它可以控制大多数属性以更改组件的状态。 但是，有一些特殊属性，例如`if`和`forEach`，ZK Bind无法使用，因为在创建组件时确定并固定了这些属性的值。 因此，绑定这些特殊属性不会对组件产生任何影响，但是您可能希望使用这些特殊属性提供的功能，在此我们演示MVVM方法中的替代方法。

**The "if" Versus the "visible"(if 与 visible)**

假定您只想向具有管理权限的用户显示“删除”按钮。 有几种用法：

**specialAttribte.zul**

```xml
<!-- wrong usage, no effect -->
<button label="Delete " if="@load(vm.currentUser.admin)"/>
<!-- determined at the beginning -->
<button label="Delete (EL)" if="${vm.currentUser.admin}"/>
<!-- can change during user interaction -->
<button label="Delete (visible)" visible="@load(vm.currentUser.admin)"/>
<button label="Delete (disabled)" disabled="@load(not vm.currentUser.admin)"/>
<checkbox label="Is Admin" checked="@bind(vm.currentUser.admin)"/>
```

第2行：用法错误； 删除按钮将永远不会被创建。
第4行：按钮的创建是在用户访问页面时确定的，除非用户成为管理员并再次访问该页面，否则该按钮不会显示。
第6,7行：在大多数情况下，我们建议这样做。 您可以在`visible`上进行绑定，这几乎可以为您带来与`if`效果相同的效果。 `disabled`也有类似的作用。

**The "forEach" Versus Children Binding(forEach 与 子绑定)**

forEach属性也有相同的问题。

**specialAttribute.zul**

```xml
<!-- "forEach" versus children binding -->
<!-- wrong usage, no effect -->
<checkbox label="@load(each.firstName)" forEach="@load(vm.personList)"/>
<!-- determined at the beginning -->
<checkbox label="${each.firstName}" forEach="${vm.personList}"/>
<!-- children binding -->
<div children="@load(vm.personList)">
    <template name="children">
        <checkbox label="@load(each.firstName)"/>
    </template>
</div>
```

第3行：用法错误； 它不会创建多个复选框。

第5行：复选框是在用户访问页面时开始创建的，即使我们更改了vm.personLIst也不会更改。

第7行：要动态创建和销毁组件时，将使用子级绑定。

## **Wire Variables(装配变量)**

我们可以使用`@WireVariable`来装配ViewModel中隐式对象或已注册变量解析器中的变量，就像我们在作曲家中所做的一样（请参阅 `ZK Developer's Reference/MVC/Controller/Wire Variables`。因为BindComposer在调用初始方法之前先为我们装配了这些变量。 但是，它不会像Composer那样自动装配`component`和`listener`，要实现此目的，请参阅`Advance/Wire Components` 和`Advance/Wire Event Listeners.`

**Wire from Implicit Objects(装配隐式对象)**

使用@WireVariable装配隐式对象等同于调用Components.getImplicit(org.zkoss.zk.ui.Page，java.lang.String)。 如果不存在该名称，并且字段或方法参数的类型为Execution，Page，Desktop，Session或WebApp，则仍将其连接到正确的隐式对象。 但是，在其他情况下，将引发异常。

```java
public class FooViewModel {
    @WireVariable
    private Page _page;
    @WireVariable
    private Desktop _desktop;
    @WireVariable
    private Session _sess;
    @WireVariable
    private WebApp _wapp;
    @WireVariable("desktopScope")
    private Map<String, Object> _desktopScope;
}
```

**Wire from Variable Resolver(装备变量解析器)**

首先，您应该注册变量解析器。 注册变量解析器有两种方法：VariableResolver注解或variable-resolver指令。 这是使用注解注册变量解析器的示例。

```java
@VariableResolver({foo1.MyResolver.class, foo2.AnotherResolver.class})
public class FooViewModel {
    ....
}
```

然后使用WireVariable注解对字段或方法进行注解。 例如，

```java
@VariableResolver({foo1.MyResolver.class, foo2.AnotherResolver.class})
public class FooViewModel {
    @WireVariable
    Department department;

    @WireVariable
    public void setManagers(Collection<Manager> managers) {
        //...
    }
}
```

**Wire Spring-managed Beans(装配Spring托管的Beans)**

如果您想装配Spring管理的bean，通常会注册Spring变量解析器DelegatingVariableResolver。 然后，您可以注解@WireVariable来装配Spring托管的bean。 例如，

```java
@VariableResolver(org.zkoss.zkplus.spring.DelegatingVariableResolver.class)
public class PasswordViewModel {
    @WireVariable
    private User user;
}
```

DelegatingVariableResolver是一个变量解析器，用于检索Spring管理的bean，因此该变量将由Spring检索和实例化。

注意，在实例化组件及其子组件之前已对变量进行了装配，因此可以在EL表达式中使用它们。 例如，假设我们有一个ViewModel，如下所示。

```java
@VariableResolver(org.zkoss.zkplus.spring.DelegatingVariableResolver.class)
public class UserViewModel {
    @WireVariable
    private List<User> users;

    public ListModel<User> getUsers() {
        return new ListModelList<User>(users);
    }
}
```

然后，您可以将其绑定到ZUL文件中。 例如，

```xml
<grid model="@load(vm.users)">
```

您可能将ViewModel制成Spring Bean，然后ViewModel使用Spring注入需要使用的其他Bean，但不建议这样做

请参阅: ZK Developer's Reference/MVC/Controller/Wire Variables#Warning:_Not_a_good_idea_to_have_Spring_managing_the_composer.

**Wire CDI-managed Beans(装配 CDI 托管的 Beans)**

使用CDI的方法类似于Spring的方法，除了您应该为CDI注册另一个变量解析器：DelegatingVariableResolver。

**Wiring Sequence(装配顺序)**

装配变量时, 查找变量解析器的预定义顺序如下:

1. ZUML文档中定义的变量解析器。
2. 在类中使用VariableResolver注解注册的变量解析器。
3. 如果以上均未找到，它将查找隐式对象，例如会话和页面。

## **Wire Components(装配组件)**

> Since 6.0.2

尽管MVVM模式的设计原理是ViewModel不应对UI组件进行任何引用，但是ZK仍提供了两种方法来检索ViewModel中的UI组件。 但是，我们不建议这种用法，因为它失去了ViewModel的一个重要优点：与View的松散耦合。 请注意，binder还可以操纵UI组件，因此您对UI组件的操作可能会影响binder的工作。 使用时请小心。

获取组件的一种方法是将组件作为参数传递给我们之前讨论过的命令绑定。 另一种方法是调用Selectors.wireComponents(),这种方法使您可以像在SelectorComposer中一样使用@Wire来连接组件。 您应该在具有@AfterCompose的方法中调用Selectors.wireComponents()，如下所示：

**Example to wire components in a ViewModel**

```java
public class SearchAutowireVM {
    //UI component
    @Wire("#msgPopup")
    Popup popup;
    @Wire("#msg")
    Label msg;

    @AfterCompose
    public void afterCompose(@ContextParam(ContextType.VIEW) Component view) {
        Selectors.wireComponents(view, this, false);
    }
}
```

> Selectors.wireComponents()的第一个参数是Root View Component，可以通过@ContextParam检索。

## **Wire Event Listeners(装配事件监听器)**

> since 6.0.2

在 ViewModel 中wire(装配)事件监听器, 如`ZK Developer's Reference/MVC/Controller/Wire Event Listeners`介绍的那样，我们必须在@AfterCompose方法中调用`Selectors.wireEventListeners()`。 然后，我们可以使用`@Listen`将方法声明为事件侦听器。 我们不推荐这种用法，因为它失去了ViewModel的一个重要优点：与View的松散耦合。 请在使用前评估权衡。

**Wire event listener in a ViewModel**

```java
public class SearchAutowireVM {
    @AfterCompose
    public void afterCompose(@ContextParam(ContextType.VIEW) Component view) {
        Selectors.wireEventListeners(view, this);
    }

    @Listen("onClick=#mybutton")
    public void submit(MouseEvent event) {
        //handle events
    }
}
```

> Selectors.wireEventListeners()的第一个参数是Root View Component，可以通过@ContextParam检索到。

## **Avoid Tracking(避免跟踪)**

当我们创建一个绑定到ViewModel属性的属性时，`跟踪器`将创建相应的跟踪记录以维护此绑定关系。 因此，当binder从注解`@NotifyChange`读取属性时，它知道要在跟踪记录上重新加载哪些属性。 此跟踪任务会消耗时间和内存，并影响应用程序性能。 如果我们有一个对象，其属性在整个应用程序运行期间不会改变，则无需跟踪此不可变对象的属性。 我们可以在此不可变对象上应用`@Immutable`注解，以降低跟踪成本。 如果我们将属性绑定到`不可变对象`的属性，则跟踪器不会为其创建相应的跟踪记录。

**Immutable object(不可变对象)**

```java
@Immutable
public class SysDefaultConfig {
    ...
}
```

**Reference an immutable object(引用不可变对象)**

```xml
<label value="@init(vm.sysDefaultConfig.size)"/>
```

## **Communication between ViewModel and Composer(ViewModel 与 Composer 的通信)**

为了深入理解以下各段，建议您在阅读本文之前先阅读并理解全局命令和全局命令绑定的概念。

全局命令绑定用法部分演示了如何在多个ViewModel之间进行通信。 我们还可以使用相同的机制在Composer和ViewModel之间执行通信，但是当然用法有所不同。

**Posting a Command from a Composer to a ViewModel(将命令从Composer发布到ViewModel)**

我们知道，全局命令是由绑定程序将事件发送到事件队列中触发的，因此，绑定程序附带的ViewModels可以通过全局命令相互通信。 但是，Composer没有Binder来发送全局命令，因此，我们提供了一个实用程序类BindUtils，可以完成此任务。 您可以在Composer中使用它。

例如，添加产品后，您要告诉ShoppingCartViewModel刷新购物车中的商品。 假设我们不更改默认设置，以使ShoppingCartViewModel的绑定器（接收器）订阅默认事件队列。

**Send a global command in a composer (Sender)**

```java
public class MyComposer extends SelectorComposer {
    @Listen("onAddProductOrder=#PrdoDiv #prodGrid row productOrder")
    public void addProduct(Event fe) {
        //business logic of adding a product
        BindUtils.postGlobalCommand(null, null, "updateShoppingCart", null);
    }
}
```

- 我们使用BindUtils.postGlobalCommand(String queueName,String queueScope,String cmdName,Map <java.lang.String,java.lang.Object> args)发布命令。 我们将前两个参数保留为“ null”，以使用默认队列名称和默认范围（桌面）。 第三个参数是全局命令的名称。 您可以通过第四个参数通过Map发送额外的参数。
- 然后，已在同（桌面）范围内订阅事件队列的所有ViewModel绑定程序将执行指定的全局命令。

在ShoppingCartViewModel中，我们应该声明一个名为“ updateShoppingCart”的全局命令方法，以接收此命令请求并刷新购物车项目。 下面的代码段显示了这一点。

**Global command in a ViewModel (Receiver)**

```java
public class ShoppingCartViewModel { 
    ...

    @GlobalCommand
    @NotifyChange("cartItems")
    public void updateShoppingCart() {
        //update shopping cart
    }
}
```

- 由于绑定器默认情况下预订桌面作用域事件队列，因此我们只需要声明一个全局命令。
- 要使用全局命令接收参数，请参阅Advanced / Parameters＃A_Global_Command_Example。



**Posting a Command from a ViewModel to a Composer**

由于ViewModel附加了绑定程序，因此触发全局命令不需要BindUtils，我们可以简单地使用全局命令绑定。

假设我们要通知Composer更新购物车中的商品。

**Bind global command in a ZUL (Sender)**

```xml
<window apply="org.zkoss.bind.BindComposer" binder="@init(queueName='myqueue')"
        viewModel="@id('vm') @init('example.MyViewModel')">
    <button id="addProduct" label="Add" onClick="@global-command('updateShoppingCart')"/>
</window>
```

我们将绑定程序发布到的队列名称设置为“ myqueue”。

如前所述，全局命令是由事件队列发送的，Composer（接收者）应订阅同作用域事件队列以接收此全局命令。

**To subscribe global command (Receiver) to the event queue**

```java
public class MyComposesr extends SelectorComposer<Component> {
    @Subscribe("myqueue")
    public void updateShoppingCart(Event evt) {
        if (evt instanceof GlobalCommandEvent) {
            if ("updateShoppingCart".equals(((GlobalCommandEvent) evt).getCommand())) {
                //update shopping cart's items
            }
        }
    }
}
```

- 订阅一个名为“ myqueue”的队列，因为先前的活页夹已发布到该队列。
- 对于@Subscribe，请参考 `ZK Developer's Reference/MVC/Controller/Subscribe to EventQueues`。
- 要通过方法调用订阅事件队列，请参阅`ZK Developer's Reference/Event Handling/Event Queues#Subscribe_to_an_Event_Queue`

## **Displaying Huge Amount of Data(显示大量数据)**

**大量数据问题**

当显示带有数据组件（列表框或网格）的数据时，我们可以创建一个模型对象，其中包含该数据组件的所有数据。 但是有时候从数据库查询所有数据的时间太长了。 在这种情况下，我们无法一次将所有数据放入模型对象。 每次模型建模对象应查询一小部分数据，这些数据是组件呈现所必需的。 现在，我们演示一个自定义的ListModel实现，该实现一次只查询少量数据，而不是本节中的所有数据。

**Cache One Page Size of Data(缓存一页数据)**

假设我们将在列表框中显示大量个人信息。 由于它们太多，因此无法一次显示它们。 一种方法是设置`rows`以限制可见行，另一种方法是使用`paging`模具。

```xml
<window width="400px" apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.advance.HugeDataVM')">
    Use custom ListModel:
    <listbox model="@bind(vm.personListModel)" selectedItem="@bind(vm.selectedPerson)" rows="10">
        <listhead>
            <listheader label="ID"/>
            <listheader label="First Name"/>
            <listheader label="Last Name"/>
            <listheader label="Age"/>
        </listhead>
        <template name="model">
            <listitem>
                <listcell label="@bind(each.id)"/>
                <listcell label="@bind(each.firstName)"/>
                <listcell label="@bind(each.lastName)"/>
                <listcell label="@bind(each.age)"/>
            </listitem>
        </template>
    </listbox>
    ...
</window>
```

由于查询所有数据非常耗时，因此我们需要为列表框实现自定义ListModel。 此自定义ListModel开始不包含任何数据，直到每次Listbox向其请求数据,它会查询数据库中的数据。 由于可见行数有限，因此当列表框呈现其Listitems时，它仅获取这些可见行的部分数据，而不是所有数据。 在一次执行期间，Listbox将多次调用ListModel.getElememtAt(int index)以获取其呈现所需的数据范围。 为了避免对每个getElementAt()调用执行一次查询，我们查询数据的页面大小并将其存储为第一个getElementAt()调用的执行属性。 在随后的调用中，我们只是从缓存中获取项目，而无需查询数据库。 该实现大大减少了查询时间，用户仍然可以浏览所有数据。 因此，此实现解决了该问题，并且在Listbox的页面调度或默认模式下可以正常工作。

**Live cached ListModel implementation(实时缓存的ListModel实现)**

```java
public class LivePersonListModel extends AbstractListModel<Person> {
    private PersonDao personDao;
    private int pageSize = 10;
    private final static String CACHE_KEY = LivePersonListModel.class + "_cache";

    public LivePersonListModel(PersonDao personDao) {
        this.personDao = personDao;
    }

    public Person getElementAt(int index) {
        Map<Integer, Person> cache = getCache();
        Person targetPerson = cache.get(index);
        if (targetPerson == null) {
        //if cache doesn't contain target object, query a page size of data starting from the index
            List<Person> pageResult = personDao.findAll(index, pageSize);
            int indexKey = index;
            for (Person o : pageResult) {
                cache.put(indexKey, o);
                indexKey++;
            }
        } else {
            return targetPerson;
        }
        //get the target after query from database
        targetPerson = cache.get(index);
        if (targetPerson == null) {
        //if we still cannot find the target object from database, there is inconsistency between memory and the database
            throw new RuntimeException("Element at index " + index + " cannot be found in the database.");
        } else {
            return targetPerson;
        }
    }

    private Map<Integer, Person> getCache() {
        Execution execution = Executions.getCurrent();
        //we only reuse this cache in one execution to avoid accessing detached objects. //our filter opens a session during a HTTP request
        Map<Integer, Person> cache = (Map) execution.getAttribute(CACHE_KEY);
        if (cache == null) {
            cache = new HashMap<Integer, Person>();
            execution.setAttribute(CACHE_KEY, cache);
        }
        return cache;
    }

    public int getSize() {
        return personDao.findAllSize();
    }
    ...
}
```

第1行：在大多数情况下，我们可以基于AbstractListModel创建我们的ListModel类，而无需从头开始实现。

第2行：personDao是一个数据访问对象，它从数据库查询数据。

第15-20行：查询一个页面大小的数据并将其放入缓存对象。

第37行：此方法从执行的属性获取缓存对象。 如果不存在，请放一个新的。

## **Binding Annotation for a Custom Component(自定义组件的绑定注解)**

当我们将数据绑定表达式应用于组件的属性时，ZK会自动将值保存回ViewModel并从ViewModel加载值。 ZK如何知道何时保存或加载组件的哪个属性？ 它在ZK组件的metainfo中声明。 ZK读取组件的元信息，并且知道如何处理组件上的数据绑定表达式。

在为新的自定义组件声明数据绑定注解之前，only-load绑定将起作用。 我们需要明确声明以save绑定工作。 本节介绍如何声明它。

**Create a Custom Component**

为了解释如何声明数据绑定定义，我们以一个自定义类为例创建一个宏组件。宏组件被命名为“ EditableLabel”。 这是带有in-place编辑器的标签。 该组件首先显示标签。 当我们双击它时，它会切换到文本框进行编辑并更改标签的值。

```java
public class EditableLabel extends HtmlMacroComponent {
    private static final long serialVersionUID = 1L;
    @Wire
    Textbox textbox;
    @Wire
    Label label;

    public EditableLabel() {
        setMacroURI("/WEB-INF/component/editablelabel.zul");
    }

    public String getValue() {
        return textbox.getValue();
    }

    public void setValue(String value) {
        textbox.setValue(value);
        label.setValue(value);
    }

    @Listen("onDoubleClick=#label")
    public void doEditing() {
        textbox.setVisible(true);
        label.setVisible(false);
        textbox.focus();
    }

    @Listen("onBlur=#textbox")
    public void doEdited() {
        label.setValue(textbox.getValue());
        textbox.setVisible(false);
        label.setVisible(true);
        Events.postEvent("onEdited", this, null);
    }
}
```

第33行：此组件发送自定义事件，以通知Label的值已更改。

**editablelabel.zul**

```xml
<zk>
    <label id="label" />
    <textbox id="textbox" visible="false"/>
</zk>
```

**Declare Data Binding in Language Addon XML(在语言附加XML中声明数据绑定)**

为了让BindComposer知道如何处理自定义组件的数据绑定表达式，我们应该声明数据绑定定义。 有关完整的XML元素和属性，请参阅`Data Binding`和 `Language Definition.`。 在这里，我们仅涵盖最常用的属性。

对于我们的示例，关键是声明何时将组件的 `value` 属性保存到ViewModel的成员变量中。 保存时间由事件名称指定。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<language-addon>
    <addon-name>myaddon</addon-name>
    <language-name>xul/html</language-name>
    <component>
        <component-name>editableLabel</component-name>
        <component-class>org.zkoss.reference.developer.mvvm.advance.EditableLabel</component-class>
        <annotation>
            <annotation-name>ZKBIND</annotation-name>
            <property-name>value</property-name>
            <attribute>
                <attribute-name>ACCESS</attribute-name>
                <attribute-value>both</attribute-value>
            </attribute>
            <attribute>
                <attribute-name>SAVE_EVENT</attribute-name>
                <attribute-value>onEdited</attribute-value>
            </attribute>
        </annotation>
    </component>
</language-addon>
```

第3行：您应该给一个唯一的名称。

第6-7行：这两个元素是必需的。

第10行：指定ZK应该保存的此组件的哪个属性。

第16-17行：指定在组件发送事件名称时ZK应该保存指定属性的事件名称。

**Declare Data Binding by Java Annotation(通过Java注解声明数据绑定)**

我们还可以通过Java注解ComponentAnnotation声明数据绑定定义。 它是XML文件的替代方法。 为了使自定义组件可用于所有页面，我们仍然需要在上面提到的lang-addon.xml中对其进行声明。

```xml
<language-addon>
    <addon-name>myaddon</addon-name>
    <language-name>xul/html</language-name>
    <component>
        <component-name>editableLabel</component-name>
        <component-class>org.zkoss.reference.developer.mvvm.advance.EditableLabel</component-class>
    </component>
</language-addon>
```

以下注解与上一节的XML文件具有相同的作用。

```java
@ComponentAnnotation("value:@ZKBIND(ACCESS=both,SAVE_EVENT=onEdited)") 
public class EditableLabel extends HtmlMacroComponent{
   ...
}
```

我们可以在两个目标上应用`@ComponentAnnotation`。 一个是getter（或setter）方法，另一个是class。 该注解只有一个类型为String的元素。

在getter上应用注解意味着我们对getter方法获取的属性进行了注解。例如，您可以按如下所示对 `value` 属性进行应用：



```java
@ComponentAnnotation("@ZKBIND(ACCESS=both, SAVE_EVENT=onChange)")
public String getValue() { 
  //...
}
```

注解元素的语法与`ZUML's annotations`相同。

```
@ZKBIND( [ATTRIBUTE_NAME1]=[ATTRIBUTE_VALUE1], ...)
```

如果组件的Java类没有给定属性的getter或setter方法，则可以在类级别通过使用属性名称和冒号作为前缀来指定注解。
例如，

```java
@ComponentAnnotation({
  "selectedItem: @ZKBIND(ACCESS=both, SAVE_EVENT=onSelect)", 
  "openedItem: @ZKBIND(ACCESS=load, LOAD_EVENT=onOpen)")
public class Foo extends AbstractComponent { 
   ....
}
```

## **Pass Arguments to Include Component(给 Include 组件传递参数)**

使用`Executions.createComponents("mypage.zul",args)`或`<include>`加载ZUL页面并传递参数时。 由于生命周期问题，ZK绑定注解EL表达式无法直接引用这些参数。 Binder所做的是在创建组件后进行的后处理动作。 在进行后期处理时，它无法获取参数的值。 最简单的解决方案是添加一个自定义属性以保存参数以供以后参考，或使用`@ExecutionArgParam`在ViewModel的初始方法中进行检索。 让我们来看一个例子。

**outer.zul**

```xml
<include type="outerPageLiteralValue" src="inner.zul" />
```

在这里，我们将名为`type`的参数传递给被包含进来的ZUL文件(inner.zul)。

**ViewModel for included zul**

```java
public class InnerVM {
    private String typeFromOuter;

    @Init
    public void init(@ExecutionArgParam("type") String type) {
        typeFromOuter = type;
    }
    ...
}
```

第5行：使用键`type`检索传递的参数。

**inner.zul**

```xml
<div apply="org.zkoss.bind.BindComposer"
     viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.advance.InnerVM')">
    <groupbox width="400px">
        Argument passed from outer page:
        <label value="@load(vm.typeFromOuter)"/>
    </groupbox>
</div>
```

**Arguments from ViewModel(从 ViewModel 传递的参数)**

如果参数来自ViewModel的属性，则 `src`属性`也必须加载数据绑定`，并在生命周期问题的`最后一个属性`中指定。 原因与之前的情况类似，如果您为 `src`属性指定静态值，则会首先创建包含的页面的组件，但此时尚未确定参数的值。 除非参数和`src`属性都与具有数据绑定的ViewModel绑定在一起，它们可以在同一阶段进行处理, 然后，创建包含的页面可以正确访问参数。 另外，ZK在组件创建时只解析一次参数的EL，因此，将来ViewModel的属性更改不会反映在所包含的组件上。

```xml
<include type="@load(vm.myArgument)" src="@load('inner.zul')"/>
```

1. 必须在最后一个属性中指定`src`属性。
2. ZK在组件创建时只解析一次参数的EL，因此`将来ViewModel的属性更改不会反映在所包含的组件`上。