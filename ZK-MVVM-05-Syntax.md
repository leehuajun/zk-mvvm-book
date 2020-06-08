# Syntax(语法)

在以下各节中，我们将列出可用于实现ViewModel和应用ZK绑定注解的所有语法。

## ViewModel

我们将列出ViewModel类中使用的所有Java注解，并在以下各节中提供描述和示例。

### **@Init**

**语法**:

```java
@Init 
@Init(superclass=true)
```

**描述**: 

**目标**：method, class（从6.0.1开始）

**目的**：标记注解以标识初始方法(对方法名称没有限制)。

初始化ViewModel时，Binder使用此注解调用方法。 在ViewModel类中，最多只允许一个初始方法。 如果将注解的属性superclass设置为true，则将首先调用ViewModel的父类的初始方法，然后再调用当前类的初始方法，并且此逻辑在超类上重复。 如果一个类没有使用@Init的方法，则不会调用任何方法（包括超类的方法）[1]。

例如，在类层次结构中：A（具有@init）<-B（具有@init）<-C（没有@init）<-D（具有@ init，superclass = true）。 D是最后一个子类。

- 当binder将D初始化为viewModel时，它将调用D的初始方法。

- 当binder初始化C时，将不会调用任何方法。

- 当binder初始化B时，它将先调用A，然后再调用B。

- 当binder初始化A时，它将调用A。

我们还可以在初始方法的参数上使用与参数相关的注解，请参阅`Syntax/ViewModel/Parameters`的小节。

> [1]：如果您覆盖父类的初始方法，例如 Parent.m1()<-Child.m1()。 由于Java的限制，binder仍然调用Child.m1()，而Child.m1()将被调用两次。 为避免这种情况，应将Child.m1()的超类设置为false并在其中调用super.m1()。

示例:

```java
public class FooViewModel {
    @Init
    public void initFoo() { 
        //initializing
    }
}

public class BarViewModel extends FooViewModel {
    @Init(superclass = true)
    public void initBar() {
        //initial method of super class FooViewModel will be called first.
    }
}

//since 6.0.1
@Init(superclass = true)
public class ChildViewModel extends BarViewModel {
}
```



### **@AfterCompose**

> Since 6.0.2

语法:

```java
@AfterCompose 
@AfterCompose(superclass=true)
```

**描述**: 

**目标**：method, class

**目的:**   标记注解，用于标识将在BindComposer的`doAfterCompose()`中调用的生命周期方法。 ViewModel类中最多`只允许使用一个@AfterCompose注解方法`。 如果将注解的属性superclass设置为true，则将首先调用ViewModel的父类的`@AfterCompose`注解方法，然后再调用子类的方法，并且此逻辑在超类上重复。 如果一个类没有使用@AfterCompose的方法，则不会调用任何方法（包括超类的方法）。[1]。

例如，在类层次结构中：A（具有@AfterCompose）<-B（具有@AfterCompose）<-C（没有@AfterCompose）<-D（具有@ AfterCompose, superclass = true）。 D是最后一个子类。

- 当BindComposer在doAfterCompose()中开始处理D时，它将调用D的@AfterCompose方法。

- 当BindComposer到达C时，将不会调用任何方法。

- 当BindComposer到达B时，它将先调用As，然后调用B。

- 当BindComposer到达A时，它将调用A。

就像我们在@Init中所做的一样，我们也可以在AfterCompose方法的参数上使用与参数相关的注解，请参阅`Syntax/ViewModel/Parameters`的小节。

> [1]如果您覆盖父类的@AfterCompose注解方法，例如 Parent.m1()<-Child.m1()。 由于Java的限制，BindComposer仍然调用Child.m1()，并且Child.m1()将被调用两次。 为避免这种情况，您应该设置对于Child.m1()，superclass = false，并在其中调用super.m1()。

**示例:**

```java
public class FooViewModel {
    @AfterCompose
    public void doFoo() {
        //do while AfterCompose
    }
}

public class BarViewModel extends FooViewModel {
    @AfterCompose(superclass = true)
    public void afterComposeBar() {
        //AfterCompose method of super class FooViewModel will be called first.
    }
}

@AfterCompose(superclass = true)
public class ChildViewModel extends BarViewModel {
}
```

### **@NotifyChange**

语法:

```java
@NotifyChange("anotherProperty") 
@NotifyChange({"secondProperty","thirdProperty"}) 
@NotifyChange("*")
@NotifyChange(".")
```

**描述**: 

**目标**：method(setter or command method)

**目的:**  通知binder一个或多个属性更改。

默认情况下，binder通过setter方法设置的属性将通知属性更改，而无需使用此注解。 您可以在setter方法上使用此注解来覆盖默认的通知目标。 您还可以在命令方法上添加此注解，以通知在命令执行后更改的属性。 为避免默认通知，请仅在setter方法上使用`@NotifyChangeDisabled`。 在注解元素中提供“ `*`”表示通知ViewModel中的`所有属性`。

采用 "." 强制重新加载注解所在的`类的实例`，而不是`实例的属性`。

**示例:**

```java
public class OrderVM { //other code...
    @NotifyChange({"selected", "messages"})
    public void setSelected(Order selected) {
        this.selected = selected;
    }

    //action command
    @NotifyChange({"selected", "orders", "messages"})
    @Command
    public void newOrder() {
        Order order = new Order();
        getOrders().add(order); //add new order to order list 
        selected = order; //select the new one
    }
}
```



### **@NotifyChangeDisabled**

语法:

```java
@NotifyChangeDisabled
```

**描述**: 

**目标**：setter method

**目的:**  标记注解可禁用默认通知行为。

当binder设置属性时禁用默认通知。

**示例:**

```java
public class OrderVM {
    //other code...
    @NotifyChangeDisabled
    public void setSelected(Order selected) {
        this.selected = selected;
    }
}
```

### **@SmartNotifyChange**

语法:

```java
@SmartNotifyChange("anotherProperty") 
@SmartNotifyChange({"secondProperty","thirdProperty"}) 
@SmartNotifyChange("*")
@SmartNotifyChange(".")
```

**描述**: 

**目标**：method (command method)

**目的:**  与@NotifyChange不同，更改后通知值更改。 通常，注解与@Command一起使用。

**示例:**

```java
public class OrderVM { 
    //other code...
    //action command
    @SmartNotifyChange({"selected", "orders", "messages"})
    @Command
    public void newOrder() {
        Order order = new Order();
        getOrders().add(order); //add new order to order list 
        selected = order;//select the new one
    }
}
```

### **@DependsOn**

语法:

```java
@DependsOn
```

**描述**: 

**目标**：getter method

**目的:**  通知属性变化的依赖性。

它具有与@NotifyChange相同的功能，但含义相反。 它用于通知binder，getter方法的target属性已更改，因为它依赖的一个或多个属性已更改。 当计算的属性取决于多个属性时，它可以消除ViewModel中的分布式@NotifyChange注解。

> 可以简单理解,当当前方法/getter 所依赖的属性发生改变时,执行该方法.

**示例:**

```java
public class FullnameViewModel {
    private String firstname;
    private String lastname;

    public String getFirstname() {
        return firstname;
    }

    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }

    public String getLastname() {
        return lastname;
    }

    public void setLastname(String lastname) {
        this.lastname = lastname;
    }

    @DependsOn({"firstname", "lastname"})
    public String getFullname() {
        return (firstname == null ? "" : firstname) + " " + (lastname == null ? "" : lastname);
    }
}
```

### **@Command**

语法:

```java
@Command()
@Command("commanName") 
@Command({"commanName1", "commandName2"})
```

**描述**: 

**目标**：method

**目的:**  标识一个Command方法。

可选注解的属性是command 名称的字符串，并且该名称在ZUL中通过事件命令绑定进行引用。 如果未提供，则默认情况下将`方法名称`设置为命令名称。

我们也可以在初始方法的参数上使用与参数相关的注解，有关更多信息，请参考`Parameters`的小节。

**示例:**

**Method name as command name**

```java
@Command
public void search() {
  items = new ListModelList<Item>();
  items.addAll(getSearchService().search(filter));
  selected = null;
}
```

**Specify command name**

```java
@Command("delete")
public void deleteOrder() {
    getService().delete(selected); //delete selected
    getOrders().remove(selected);
    selected = null; //clean the selected
}
```

### **@DefaultCommand**

> Since 6.5.1

语法:

```java
@DefaultCommand
```

**描述**: 

**目标**：method

**目的:**  标记默认的Command方法。

binder收到命令后，便开始通过匹配其名称来查找ViewModel的命令方法。 仅当binder找不到匹配的方法时，它才会调用现有的默认命令方法。。

**示例:**

**Mark method as default**

```java
@DefaultCommand
public void defaultAction(){
  ...
}
```

### **@GlobalCommand**

语法:

```java
@DefaultCommand
```

**描述**: 

**目标**：method

**目的:**  识别全局命令方法。

注解可选属性是命令名称的字符串，并且该名称在具有全局命令绑定的ZUL中被引用。 如果未提供，则默认情况下将方法名称设置为命令名称。

我们可以在命令方法的参数上使用与参数相关的注解，有关更多信息，请参阅`Parameters`小节。

**示例:**

**Method name as command name**

```java
@GlobalCommand
public void show() {
  //...
}
```

### **Specify command name**

```java
@Command("delete")
@GlobalCommand("delete")
public void deleteOrder() {
    //...
}
```

### **@DefaultGlobalCommand**

> Since 6.5.1

语法:

```java
@DefaultGlobalCommand
```

**描述**: 

**目标**：method

**目的:**  将方法标记为默认全局命令。

Binder收到全局命令后，便开始通过匹配其名称来查找ViewModel的全局命令方法。 仅当binder找不到匹配的方法时，它才会调用现有的默认全局命令方法。

**示例:**

**Mark a default global command method**

```java
@DefaultGlobalCommand
public void defaultAction() {
  //...
}
```

### **@Immutable**

语法:

```java
@Immutable
```

**描述**: 

**目标**：method, class

**目的:**  标记注解，指示不可变的类或方法。

不可变类的属性将不会被跟踪，从而减少了应用程序所需的资源。

**示例:**

```java
@Immutable
public class SysConfiguration{
  
}
```

```java
public class DataObject { 
    //other code...
    @Immutable
    public String getInfo() {
        return info;
    }
}
```

### **@ImmutableElements**

语法:

```java
@ImmutableElements
```

**描述**: 

**目标**：collection

**目的:**  标记注解，指示FormProxyObject的集合的元素是不可变的类。

不可变类的属性将不会被跟踪，从而减少了应用程序所需的资源。

**示例:**

```java
public class CustomVM {
    @ImmutableElements
    private Map<String, Object> info; 
    //other code...
}
```

### **@NotifyCommand**

> Since 8.0.0

语法:

```java
@NotifyCommand(value="commandName", onChange="_vm_.expression")
```

**描述**: 

**目标**：class

**目的:**  只要给定表达式在服务器上发生更改，就会在ViewModel中触发命令。`_vm_`在这里表示当前视图模型。



**示例:**

```java
@NotifyCommand(value = "upateData", onChange = "_vm_.data")
public class VM {
    private Map<String, String> data = new HashMap<String, String>();

    // getter/setter...
    @Command
    public void upateData() {
        data.put("Status", "updated");
    }
}
```

### **@NotifyCommands**

> Since 8.0.0

语法:

```java
@NotifyCommands({
        @NotifyCommand(value = "commandName1", onChange = "_vm_.expression1"), 
        @NotifyCommand(value = "commandName2", onChange = "_vm_.expression2")
})
```

**描述**: 

**目标**：class

**目的:**  每当服务器上给定的表达式发生更改时，都会在ViewModel中触发`一系列命令`。 `_vm_`在这里表示当前视图模型。



**示例:**

```java
@NotifyCommands({
        @NotifyCommand(value = "upateData1", onChange = "_vm_.data1"), 
        @NotifyCommand(value = "upateData2", onChange = "_vm_.data2")
})
public class VM {
    private Map<String, String> data1 = new HashMap<String, String>();
    private Map<String, String> data2 = new HashMap<String, String>();

    // getter/setter...
    @Command
    public void upateData1() {
        data1.put("Status", "updated");
    }

    @Command
    public void upateData2() {
        data2.put("Status", "updated");
    }
}
```

### **@ToClientCommand**

> Since 8.0.0

语法:

```java
@ToClientCommand(commandNames)
```

**描述**: 

**目标**：class

**目的:**  在执行完成后，放入我们要通知客户端的命令。 请注意，只有我们在此注解中放置的命令才会触发我们在客户端放置在`binder.after`中的回调。

注意：如果注解的值包含“ *”值，则表示接受所有命令以通知客户端。

**示例:**

```java
@ToClientCommand("doCountChange")
public class VM {
    private int count = 0;

    // getter/setter...
    @Command
    public void doCountChange() {
        count++;
    }
}
```

```xml
<window viewModel="@id('vm') @init('org.zkoss.VM')" xmlns:n="native">
    <n:div id="display"></n:div>
    <n:script>
        zk.afterMount(function() {
        var binder = zkbind.$('$display');
        // the event handler of after 'doCountChange' from server 
        binder.after('doCountChange', function() {
           alert("after doCountChange"); });
        });
    </n:script>
    <button label="change" onClick="@command('doCountChange')"/>
</window>
```

### **@ToServerCommand**

> Since 8.0.0

语法:

```java
@ToServerCommand(commandNames)
```

**描述**: 

**目标**：class

**目的:**  放置我们要通知客户端绑定程序触发的服务器的命令。 注意，只有我们放置在此注解中的命令才会在服务器上收到。

注意：如果注解的值包含值“ *”，则表示接受所有命令以通知服务器。

**示例:**

```java
@ToServerCommand("doCountChange")
public class VM {
    private int count = 0;

    // getter/setter...
    @Command
    public void doCountChange() {
        count++;
    }
}
```

```xml
<window viewModel="@id('vm') @init('org.zkoss.VM')" xmlns:n="native">
    <n:div id="display"></n:div>
    <n:script>
        zk.afterMount(function() {
        var binder = zkbind.$('$display');
        // this will trigger the server side command 'doCountChange' to execute once.
        binder.command('doCountChange');
        });
    </n:script>
</window>
```

### **@Parameters**

您可以通过在初始方法（使用@Init的方法）和命令方法（使用@Command的方法）的各种上下文范围中检索值或隐式对象，方法是在这些方法的参数上应用与参数相关的注解。 我们将在以下各节中列出所有与参数相关的注解。

#### **@BindingParam**

语法:

```java
@BindingParam("keyString")
```

**描述**: 

**目标**：Command method's parameter

**目的:**  告诉binder从ZUL上的绑定参数中使用指定的键检索此参数。

注解将应用于命令方法的参数。 它声明所应用的参数应来自用指定键在ZUL上编写的绑定参数。

**示例:**

**Command binding that pass parameters**

```xml
<listbox model="@load(vm.items)" selectedItem="@bind(vm.selected)" hflex="true" height="300px">
    <listhead>
        <listheader label="Name"/>
        <listheader label="Price" align="center"/>
        <listheader label="Quantity" align="center"/>
    </listhead>
    <template name="model" var="item">
        <listitem onMouseOver="@command('popupMessage', myKey='myValue', content=item.description)">
            <listcell label="@bind(item.name)"/>
            <listcell label="@bind(item.price)"/>
            <listcell label="@bind(item.quantity)"/>
        </listitem>
    </template>
</listbox>
```

**Command method in ViewModel with binding parameter**

```java
@Command
public void popupMessage(@BindingParam("myKey") String target, @BindingParam("content") String content) {
  //...
}
```

> target的值是“ myValue”，content是对象项的description属性。

#### **@QueryParam**

语法:

```java
@QueryParam("keyString")
```

**描述**: 

**目标**：A method's parameter (initial method and command method)

**目的:**  告诉binder从HTTP请求参数中使用指定的key检索此参数。

注解将应用于初始方法（或命令方法）的参数。 它声明应用的参数应来自具有指定key的HTTP请求参数。

**示例:**

Http请求参数附加在URL后面, 如: http://localhost:8080/zkbinddemo/httpparam.zul?param1=abc

```java
public class HttpParamVM {
    String queryParam;

    @Init
    public void init(@QueryParam("param1") String parm1) {
        queryParam = parm1;
    }
}
```

在此示例中，binder将`abc`传递给`parm1`。

#### **@HeaderParam**

语法:

```java
@HeaderParam("keyString")
```

**描述**: 

**目标**：A method's parameter (applied on initial and command methods)

**目的:**  告诉binder从HTTP请求标头中使用指定的key检索此参数。

注解将应用于初始（或命令）方法的参数。 它声明应用的参数应来自具有指定key的HTTP请求标头。。

**示例:**

```java
public class HttpParamVM {
    String headerParam;

    @Init
    public void init(@HeaderParam("user-agent") String browser) {
        headerParam = browser;
    }
}
```

#### **@CookieParam**

语法:

```java
@CookieParam("keyString")
```

**描述**: 

**目标**：A method's parameter (for initial and command methods)

**目的:**  告诉binder从HTTP请求Cookie中使用指定的key检索此参数。

注解将应用于初始（或命令）方法的参数。 它声明所应用的参数应来自具有指定key的HTTP请求cookie。

**示例:**

```java
public class HttpParamVM {
    String cookieParam;

    @Init
    public void init(@CookieParam("nosuch") String guess) {
        cookieParam = guess;
    }
}
```

#### **@ExecutionParam**

语法:

```java
@ExecutionParam("keyString")
```

**描述**: 

**目标**：A method's parameter (initial method and command method)

**目的:**  告诉binder从当前执行的属性中使用指定key检索此参数。

注解将应用于初始（或命令）方法的参数。 它声明所应用的参数应来自具有指定key的当前执行的属性。

**示例:**

假设我们要通过执行属性将对象传递给使用ViewModel：ExecutionParamVM的包含ZUL。

**outer ZUL**

```xml
<window id="w2">
    <zscript>
        void doClick(){
            org.zkoss.zk.ui.Execution ex = org.zkoss.zk.ui.Executions.getCurrent(); 
            ex.setAttribute("param1","abc");
            inc.src = "executionparam-inner.zul";
        }
    </zscript>
    <button label="do include" onClick="doClick()"/>
    <include id="inc"/>
</window>
```

我们使用注解通过键“ param1”检索执行的属性。

```java
public class ExecutionParamVM {
    private String param1;

    @Init
    public void init(@ExecutionParam("param1") String param1) {
        this.param1 = param1;
    }
    //setter, getter, and others
}
```

**executionparam-inner.zul**

```xml
<vbox apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.ExecutionParamVM')">
    <label value="@load(vm.param1)"/>
</vbox>
```

> 标签将显示“ abc”

#### **@ExecutionArgParam**

语法:

```java
@ExecutionArgParam("keyString")
```

**描述**: 

**目标**：A method's parameter (for initial and command methods)

**目的:**  告诉活页夹从当前执行的参数中使用指定的key检索此参数。

注解将应用于初始（或命令）方法的参数。 它声明所应用的参数应来自具有指定key的当前执行的参数。

**示例:**

假设我们要将参数传递给使用ViewModel的包含ZUL：ExecutionParamVM。

**outer ZUL**

```xml
<window >
    <include arg1="foo" src="executionparam-inner.zul"/>
</window>
```

我们使用注解通过键“ arg1”检索执行的参数。

```java
public class ExecutionParamVM {
    private String arg1;

    @Init
    public void init(@ExecutionArgParam("arg1") String arg1) {
        this.arg1 = arg1;
    }
    //setter, getter, and others
}
```

**executionparam-inner.zul**

```xml
<vbox apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.ExecutionParamVM')">
    <label value="@load(vm.arg1)"/>
</vbox>
```

> 标签将显示“ foo”

#### **@ScopeParam**

语法:

```java
@ScopeParam("keyString") 
@ScopeParam(scopes=Scope.APPLICATION, value="keyString")
```

所有作用域枚举的列表：

```java
enum Scope {
  COMPONENT, SPACE, PAGE, DESKTOP, SESSION, APPLICATION, // single scope 
  AUTO //find by comp.getAttribute(name,true)
}
```

**描述**: 

**目标**：A method's parameter (for initial and command methods)

**目的:**  告诉binder检索具有指定范围的值。

默认范围：AUTO, 是指自动逐个搜索从COMPONENT到SPACE，PAGE，DESKTOP，SESSION，APPLICATION的值，直到找到一个非空值。 如果指定了scopes元素，则binder将搜索您指定的唯一范围。

**示例:**

```java
public class ScopeParamVM {
    @Init
    public void init(
     @ScopeParam(scopes = Scope.APPLICATION, value = "config") String sysConfig,
     @ScopeParam(scopes = Scope.SESSION, value = "user") String userCredential) {
    }
}
```

#### **@SelectorParam**

语法:

```java
@SelectorParam("#componentId") 
@SelectorParam("tagName") 
@SelectorParam(".className") 
@SelectorParam(":root") 
@SelectorParam("button[label='Submit']") 
@SelectorParam("window > button")
```

有关选择器语法，请参阅SelectorComposer

**描述**: 

**目标**：A method's parameter (for initial and command methods)

**目的:**  为了识别应该从binder的视图组件中检索方法的参数。

值元素是用于查找组件的选择器。 它使用`Selectors`来选择组件。 选择器的基本组件是binder的视图组件，即使用ViewModel的组件。

如果参数类型是Collection，则binder直接传递结果。 否则，它将传递第一个结果；如果没有结果，则为null。

**示例:**

```xml
<vbox apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.SelectorParamVM')">
    <hbox><label id="message" /></hbox> <hbox><label /></hbox>
    <hbox><label /></hbox>
    <hbox><label /></hbox>
    <button id="cmd" label="cmd" onClick="@command('cmd')" /> 
</vbox>
```

通过选择器传递组件的示例

```java
public class SelectorParamVM {
    @Command
    public void cmd(@SelectorParam("label") LinkedList<Label> labels, @SelectorParam("#message") Label msg) {
        for (int i = 0; i < labels.size(); i++) {
            labels.get(i).setValue("Command " + i);
        }
        msg.setValue("msg in command");
    }
}
```

#### **@ContextParam**

语法:

```java
@ContextParam(ContextType.PAGE)
```

枚举所有上下文:

```java
enum ContextType {
    BIND_CONTEXT,       //BindContext instance
    BINDER,             //Binder instance
    TRIGGER_EVENT,      //Event that trigger the command (since 6.0.1)
    COMMAND_NAME,       //Command name (since 6.0.1)
    EXECUTION,          //Execution instance
    COMPONENT,          //Component instance of current binding
    SPACE_OWNER,        //IdSpance instance of spaceOwner of current component
    VIEW,               //the view component of binder
    PAGE,               //Page instance of current component
    DESKTOP,            //Desktop instance of current component
    SESSION,            //Session instance
    APPLICATION         //Application instance
}
```

**描述**: 

**目标**：A method's parameter (for initial and command methods)

**目的:**  告诉binder传递具有指定类型的上下文对象。

注解将应用于初始（或命令）方法的参数。 方法可以通过对参数应用注解来获取各种ZK上下文对象，例如：Page或Desktop。

**示例:**

在ViewModel中检索各种上下文对象

```java
@Init
public void init(
        @ContextParam(ContextType.EXECUTION) Execution execution,
        @ContextParam(ContextType.COMPONENT) Component component,
        @ContextParam(ContextType.VIEW) Component view,
        @ContextParam(ContextType.SPACE_OWNER) IdSpace spaceOwner,
        @ContextParam(ContextType.PAGE) Page page,
        @ContextParam(ContextType.DESKTOP) Desktop desktop,
        @ContextParam(ContextType.SESSION) Session session,
        @ContextParam(ContextType.APPLICATION) WebApp application,
        @ContextParam(ContextType.BIND_CONTEXT) BindContext bindContext,
        @ContextParam(ContextType.BINDER) Binder binder) {
    //...
}
```

以下是另一个示例。

```xml
<vbox id="vbox" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('eg.ContextParamVM')">
    <button id="cmd" label="cmd" onClick="@command('cmd')"/>
</vbox>
```

**A ViewModel used by above zul**

```java
public class ContextParamVM {
    @Command
    public void cmd(
            @ContextParam(ContextType.COMPONENT) Component component, 
            @ContextParam(ContextType.VIEW) Component view) {
    }
}
```

在上面的示例中，component是Button对象，而view是Vbox对象。

#### **@Default**

语法:

```java
@Default("defaultValue")
```

**描述**: 

**目标**：A method's parameter (for initial and command methods)

**目的:**  如果为空，请指定绑定参数的默认值。

您使用字符串给注解元素的值，ZK将转换为参数的相应类型。 您可以在其他与参数相关的注解（例如@BindingParam）之后应用此注解。 如果第一个注解检索的参数为null，则它将使用此注解中指定的默认值。

**示例:**

**Pass parameter from a zul**

```xml
<button id="first" onClick="@command('cmd', arg2=100)" /> 
<button id="second" onClick="@command('cmd')" />
```

**Example to assign default value**

```java
@Command
public void cmd(
  @Default("false") Boolean arg1,
  @BindingParam("arg2") @Default("3") Integer arg2) {
  //...
}
```

根据上面的示例，如果我们单击第一个按钮，则绑定器会将100传递给arg2。 如果单击第二个按钮，则arg2将为3。

## **Data Binding**

在以下各节中，我们将介绍ZUL上使用的ZK Bind的注解。 它们都用在组件的属性中，并具有ZUML的注解中描述的常规格式：

```
@AnnotationName(attr-value1, attr-name2=attr-value2) 

@AnnotationName(attr-name1={attr-value1-1, attr-value1-2}, attr-name2=attr-value2)
```

基本上，语法由带逗号分隔键/值对的注解名称组成。 绑定器将注解的属性值视为EL表达式，并将不带属性名的EL表达式设置为默认属性名“ value”。 所有注解都必须定义默认属性的值。

对于每个ZK Bind支持的注解，binder评估EL表达式的方式略有不同。 某些注解的EL表达式（如@id和@init）仅被评估一次。 一些注解保留了属性名称，“ before”和“ after”，例如@load和@save。

### **@id**

语法:

```java
@id( [EvaluateOnce EL-expression] )
```

**描述**: 

**目标**：viewModel, form, validationMessages

**目的:**  为当前绑定目标提供一个ID，该ID可用于在子组件的绑定注解中引用其属性。

我们建议您在EL表达式中使用字符串文字。 因为binder只对该注解的EL表达式求值一次才能确定ViewModel的ID，并且此EL表达式也用于其他子组件的ZK绑定注解中。 如果不是固定值，将导致错误的评估结果。

当我们在“ validationMessages”属性中使用它时，它会提供一个ID来引用验证消息所有者。

当我们在“ form”属性中使用它时，它将为引用表单的中间对象提供一个ID。 表单状态变量的命名为：[middleObjectId]状态

**示例:**

**Usage in viewModel attribute**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.ChildrenMenuVM')" validationMessages="@id('vmsgs')">
</window>
```

**Usage in form attribute**

```xml
<grid form="@id('fx') @load(vm.user) @save(vm.user,before='register')"> </grid>
```

### **@Init**

语法:

```java
@init( [EvaluateOnce EL-expression], [arbitraryKey]=[EL-expression] )
```

**描述**: 

**目标**：any

**目的:**  初始化属性的值，并且在用户交互期间不重新加载。

绑定程序解析该注解中的EL表达式时，该表达式仅计算一次。 在“ viewModel”属性上使用它时，binder将尝试将EL表达式解析为一个类并创建它的实例。

在“ form”属性中，评估EL表达式的结果应为Form对象。

对于其他属性，活页夹使用EL评估结果初始化其值。

在“ viewModel”属性上使用时，我们可以用逗号分隔的键值对形式传递参数。 我们可以通过`@BindingParam`在ViewModel的初始方法中获取参数。

**[arbitraryKey]=[EL-expression]**

基本上是键值对。 您可以使用不同的键名编写多个键值对。

不带键的EL表达式会隐式设置为名为**`value`**的默认键。

由于每个注解具有不同的功能，因此某些注解可能会忽略默认值（例如@id）以外的键值对表达式。

**[arbitraryKey]**

它可以是任何名称，它用作ViewModel中与参数相关的Java注解的键。

**示例:**

**Usage example in viewModel attribute**

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.ChildrenMenuVM')" >
</window>
```

**Usage example in form attribute**

```xml
<vbox form="@id('fx') @init(vm.myForm) @load(vm.person) @save(vm.person, before='save')">
</vbox>
```

**Pass arguments**

```xml
<window width="400px" apply="org.zkoss.bind.BindComposer"
        viewModel="@id('vm') @init('org.zkoss.reference.developer.mvvm.databinding.InitVM', arg1='myValue')">
    ...
</window>
```

```java
public class InitVM {
    @Init
    public void init(@BindingParam("arg1") String arg1) {
        System.out.println("arg1: " + arg1);
    }
}
```

请注意，键（例如：`arg1`）具有保留字- `value`。

### **@load**

语法:

```java
@load( [EL-expression], [conditionKeyword]=[EvaluateOnce EL-expression] )
```

**描述**: 

**目标**：any (except viewModel, validationMessages)

**目的:**  限制binder仅从ViewModel加载数据，而不保存回去。

对于某些不能将数据保存回ViewModel的属性（例如列表框的模型或标签的值），您还可以编写@bind或@load。

**[conditionKeyword]=[EvaluateOnce EL-expression]**

该表达式是可选的，除非您要保存或加载命令

**[conditionKeyword]**

可能是[before |after]中的一个

**[EvaluateOnce EL-expression]**

评估结果必须是一个或多个命令名称。

命令名称必须与ViewModel中Java注解@Command中指定的名称相对应。

**示例:**

```xml
<label value="@load(vm.user.id)"/>
<label value="@load(vm.user.permission, after='showPermission')"/>
<label value="@load(vm.user.permission, after={'showPermission', 'showAll'})"/> 
<label value="@load(vm.user.action, before='process')"/>
```

### **@save**

语法:

```java
@save( [EL-expression], [conditionKeyword]=[EvaluateOnce EL-expression] )
```

**描述**: 

**目标**：any save-allowed attributes (except viewModel, validationMessages)

**目的:**  限制binder仅将数据保存到ViewModel，不加载。

当您要在不同条件下保存和加载数据时，通常使用此语法，应在属性中同时写入@save和@load。 您必须在表单绑定中使用它才能保存命令。

**[conditionKeyword]=[EvaluateOnce EL-expression]**

除非您要保存或加载命令，否则此表达式是可选的。

**[conditionKeyword]**

可能是[before |after]中的一个

**[EvaluateOnce EL-expression]**

评估结果必须是一个或多个命令名称。

命令名称必须与ViewModel中Java注解@Command中指定的名称相对应。

**示例:**

**Basic usage**

```xml
<textbox value="@load(vm.person.name) @save(vm.person.name, before='save')"/>
<textbox value="@load(vm.person.name) @save(vm.person.name, before={'save', 'update'})"/>
```

**Saving and loading form attribute**

```xml
<textbox value="@save(vm.number) @load(vm.number, after='cmd')" />
```

### **@bind**

语法:

```java
@bind( [EL-expression] )
```

**描述**: 

**目标**：all (except viewModel, validationMessages, form)

**目的:**  指定binder可以保存和加载数据。

就像是将@save和@load结合在一起的快捷方式注解。 当您的保存和加载不依赖于其他条件时，建议使用此注解。

**示例:**

```xml
<textbox id="t2" value="@bind(vm.user.account)" />
<doublebox id="pbox" value="@bind(vm.selected.price)"/> 
<datebox id="sdbox" value="@bind(vm.selected.shippingDate)"/>
```

### **@ref**

> Available for ZK: PE, EE 
>
> Since6.0.1

语法:

```java
@ref( [EL-expression] )
```

**描述**: 

**目标**：自定义属性

**目的:**  创建对可以在另一个EL表达式中使用的EL表达式的引用。

在组件的自定义属性上创建此绑定，该属性的名称可以在另一个EL表达式中引用和使用。 引用引用的EL表达式必须是包含引用绑定的组件的子代。 通过在另一个表达式中引用它可以缩短EL表达式。

**示例:**

```xml
<div p="@ref(vm.selectedPerson)"> 
  <label value="@load(p.id)" /> 
  <label value="@load(p.name)" />
</div>
```

### **@command**

语法:

```java
@command( [EL-expression], [arbitraryKey]=[EL-expression] )
```

**描述**: 

**目标**：event attributes (e.g. onClick, onOK)

**目的:**  指定事件触发时执行的命令。

您可以在键值对中以逗号分隔传递任意参数。

请注意，`value`是保留字，应避免将其用作任意键。

**[arbitraryKey]=[EL-expression]**

基本上是键值对。 您可以使用不同的键名编写多个键值对。

不带键的EL表达式会隐式设置为名为“`value`”的默认键。

由于每个注解具有不同的功能，因此某些注解可能会忽略默认值（例如@id）以外的键值对表达式。

**[arbitraryKey]**

它可以是任何名称，它用作ViewModel中与参数相关的Java注解的键。

**示例:**

```xml
<button label="Save" onClick="@command('saveOrder')" />
<button label="Delete" onClick="@command(empty vm.selected.id?'deleteOrder':'confirmDelete')" /> 
<button label="Index" onClick="@command('showIndex', index=10, keyword='myKeyword')"/>
```

### **@global-command**

语法:

```java
@global-command( [EL-expression], [arbitraryKey]=[EL-expression] )
```

**描述**: 

**目标**：event attributes (e.g. onClick, onOK)

**目的:**  指定事件触发时执行的全局命令。

如果将此绑定与本地命令绑定一起使用，请记住`始终先执行本地命令`。

您可以在键值对中以逗号分隔传递任意参数。

**[arbitraryKey]=[EL-expression]**

基本上是键值对。 您可以使用不同的键名编写多个键值对。

不带键的EL表达式会隐式设置为名为“`value`”的默认键。

由于每个注解具有不同的功能，因此某些注解可能会忽略默认值（例如@id）以外的键值对表达式。

**[arbitraryKey]**

它可以是任何名称，它用作ViewModel中与参数相关的Java注解的键。

**示例:**

```xml
<button label="Save" onClick="@command('saveOrder') @global-command('refresh')" />
<button label="ShowAll" onClick="@global-command('show')" />
<button label="Index" onClick="@command('showIndex') @global-command('showIndex', index=10, keyword='myKeyword')"/>
```

### **@converter**

语法:

```java
@converter( [EL-expression], [arbitraryKey]=[EL-expression] )
```

**描述**: 

**目标**：any (except viewModel, validationMessages, form, and event attributes)

**目的:**  它应该与@bind，@load，@save一起使用。 它应用转换器在UI组件和ViewModel之间转换期间转换数据。

EL表达式的评估结果应为Converter对象。 您可以在键值对中附加任意参数，并用逗号分隔以将其传递给Converter对象。 字符串文字引用内置Converter作为其名称。

**[arbitraryKey]=[EL-expression]**

基本上是键值对。 您可以使用不同的键名编写多个键值对。

不带键的EL表达式会隐式设置为名为“`value`”的默认键。

由于每个注解具有不同的功能，因此某些注解可能会忽略默认值（例如@id）以外的键值对表达式。

**[arbitraryKey]**

它可以是任何名称，它用作ViewModel中与参数相关的Java注解的键。

**示例:**

**Use built-in converter named formatedNumber**

```xml
<label value="@load(item.price) @converter('formatedNumber', format='###,##0.00')"/>
```

**Use custom converter**

```xml
<label value="@load(vm.selected.totalPrice) @converter(vm.totalPriceConverter)"/>
```

### **@validator**

语法:

```java
@validator( [EL-expression], [arbitraryKey]=[EL-expression] )
```

**描述**: 

**目标**： any (except viewModel, validationMessages, form, and event attributes)

**目的:**  它应该与@bind，@load，@save一起使用。 保存到ViewModel时，它将应用验证器来验证数据。

EL表达式的评估结果应为Validator对象。 您可以在键值对中附加任意参数，并用逗号分隔以将其传递给Validator对象。 内置验证器由字符串文字引用作为其名称。。

**[arbitraryKey]=[EL-expression]**

基本上是键值对。 您可以使用不同的键名编写多个键值对。

不带键的EL表达式会隐式设置为名为“`value`”的默认键。

由于每个注解具有不同的功能，因此某些注解可能会忽略默认值（例如@id）以外的键值对表达式。

**[arbitraryKey]**

它可以是任何名称，它用作ViewModel中与参数相关的Java注解的键。

**示例:**

**Use built-in validator named beanValidator**

```xml
<window id="win" apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init(foo.MyViewModel)"> <textbox value="@bind(vm.user.lastName) @validator('beanValidator')" />
</window>
```

**Use custom validator**

```xml
<datebox id="cdbox" value="@bind(fx.creationDate) @validator(vm.creationDateValidator)"/>
```

### **@template**

语法:

```java
@template( [EL-expression] )
```

**描述**: 

**目标**： model, children

**目的:**  它应该与@bind，@load一起使用。 它确定要用于呈现子组件的模板。

EL表达式的评估结果应为用于渲染子组件的模板的名称。 结果模板的名称可以是在相同ID空间中定义的任何模板。

**示例:**

**Dynamic template upon iteration status variable**

```xml
<combobox
        model="@bind(item.options ) 
        @template(forEachStatus.index eq 0 or forEachStatus.index eq 2?'model1':'model2')">
    <template name="model1" var="option">
        <comboitem label="@bind(optionStatus)" description="@bind(option)"/>
    </template>
    <template name="model2" var="option">
        <comboitem label="@bind(optionStatus)" description="@bind(option)"/>
    </template>
</combobox>
```

**Recursive usage**

```xml
<vlayout id="vlayout" children="@load(vm.nodes) @template('greenBox')">
    <template name="greenBox" var="node">
        <vlayout style="padding-left:10px; border:2px solid green;">
            <label value="@bind(node.name)"/>
            <vlayout
                    children="@load(node.children) @template('greenBox')"/>
        </vlayout>
    </template>
</vlayout>
```

vm.nodes具有树状结构，而node.children是节点的集合。