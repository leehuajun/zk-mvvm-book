# ViewModel

## ViewModel是什么

ViewModel是Model的抽象。 它从一个或多个Model类中提取要在View上显示的必要数据。 这些数据通过getter和setter方法（如JavaBean的属性）公开。

 

ViewModel也是视图的模型。 它包含视图的状态（例如，选择，启用状态），该状态在用户交互过程中可能会发生变化。 例如，如果用户选择列表框的项目时启用了按钮。 ViewModel应存储按钮的启用状态的状态，并实现启用按钮的逻辑。

 

尽管ViewModel存储视图的状态，但它不包含对UI组件的引用。 它不能直接访问任何UI组件。 因此，有一种数据绑定机制可以在View和ViewModel之间同步数据。 开发人员定义了View（UI组件）和ViewModel之间的绑定关系后，数据绑定机制将自动同步数据。 这使得ViewModel和View松散耦合。

![image-20200605154101145](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161843.png)

ViewModel就像MVC中的Controller一样，因此数据绑定机制将事件转发给ViewModel的处理程序。 处理程序是具有特定Java注解的ViewModel方法。 我们称这种方法为ViewModel的Command。 这些方法通常操纵ViewModel的数据，例如删除项目。 数据绑定机制还支持使用ViewModel的命令绑定UI组件的事件。 触发组件的事件将触发绑定命令的执行，并调用带注解的方法。

 

在执行命令期间，通常会更改ViewModel的数据。 开发人员应指定哪些属性更改将通过Java注解进行通知。

![image-20200605154116873](https://gitee.com/leehuajun/imgbed/raw/master/2020/20200607161848.png)

UI有时必须以与模型中原始格式不同的格式显示数据。 转换逻辑应在ViewModel中实现，否则开发人员可以采用可重用性更高的元素转换器，我们将在后面的部分中进行讨论。

 

在将数据保存到ViewModel之前，还将执行验证。 如果验证失败，则数据将不会保存到ViewModel。

### ViewModel with Annotation

在ZK中，ViewModel可以只是一个POJO，它对View的视觉元素一无所知，这意味着它不应该包含对UI组件的任何引用。 数据绑定机制处理View和ViewModel之间的通信和同步。 开发人员必须使用ZK提供的Java注解指定ViewModel的命令和数据依赖性，然后数据绑定机制将知道如何与ViewModel进行交互。

 

创建ViewModel类似于创建POJO，并且通过setter和getter方法公开其属性，例如JavaBean。

```java
public class MyViewModel {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @NotifyChange({"selected", "orders"})
    @Command
    public void newOrder() {
        //manipulate data
    }
}
```

我们将在 [ViewModel/Notification](#_bookmark10) 中详细描述上述注解

### 在ZUL中引用ViewModel

我们可以通过设置ZK UI组件的viewModel属性将其绑定到ViewModel，然后该组件成为ViewModel的Root View组件。 此根视图组件的所有子组件都可以访问相同的ViewModel及其属性。 要绑定ViewModel，我们必须应用一个名为`org.zkoss.bind.BindComposer`的组合器，它将为ViewModel创建一个绑定器并实例化ViewModel的类。 然后使用ZK Bind注解设置viewModel属性，在@id中设置ViewModel的ID，在@init中设置ViewModel的全限定类名。 该 id 用于引用ViewModel的属性，例如 vm.name，而完全限定的类名用于实例化ViewModel对象本身。 我们将在“数据绑定”的小节中详细解释ZK Bind注解语法。

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.MyViewModel')">
    <label value="@bind(vm.name)"/>
    <button onClick="@command('newOrder')" label="New Order"/>
</window>
```

## 初始化（Initialization）

### 初始化方法

binder负责创建和初始化ViewModel实例。 如果要在ViewModel中执行自己的初始化，则可以通过使用@Init注解方法来声明自己的初始方法。 初始化ViewModel时，binder将调用此方法。 每个ViewModel`只能有一个初始方法`。

```java
public class MyViewModel {
    @Init
    public void init() {
        //initialization code
    }
}
```

该注解具有一个名为“超类”的属性，如果将其设置为“ true”，则绑定器将查找ViewModel父类的初始方法，如果存在，则首先调用它。

```java
public class ParentViewModel {
    @Init
    public void parentInit() {
        //initialization code
    }
}

public class ChildViewModel extends ParentViewModel {
    /* ParentViewModel's initial method is invoked first then ChildViewModel's. */
    @Init(superclass = true)
    public void childInit() {
        //initialization code
    }
}
```

> 请注意，子类的初始方法不应覆盖父类的初始方法，否则父类的初始方法将不会被调用，而子类的初始方法将被`调用两次`。

### Apply on Class

> Since 6.0.1
>

如果super具有初始化方法，但其ChildViewModel没有，则可以在ChildViewModel上添加@Init（superclass = true）以使用super的初始化方法。

```java
public class ParentViewModel {
    @Init
    public void init() {
    }
}

@Init(superclass = true)
public class ChildViewModel extends ParentViewModel {
}
```

初始方法可以通过在其参数上添加注解来检索各种上下文对象，请参阅 [Advanced/Parameters](#_bookmark32)

## 数据与集合

### 暴露ViewModel的属性

像JavaBean一样，ViewModel通过getter和setter方法公开其属性。 View（ZUL）希望通过数据绑定检索的任何属性都应具有相应的getter方法。 方法的返回类型可以是简单类型（int，boolean ...）或JavaBean。 如果UI组件需要将其属性值（通常是用户输入）保存回ViewModel的属性，则ViewModel应该提供相应的setter方法。 因此，通过setter和getter可以使用数据绑定机制来更改View和ViewModel的数据。

### 属性是简单类型

#### 简单类型的属性

```java
public class PrimitiveViewModel {
    private int index;
    private double price;

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }
}
```

#### 使用PrimitiveViewModel的zul

```xml
<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('foo.PrimitiveViewModel')">
    <intbox value="@bind(vm.index)"/>
    <doublebox value="@bind(vm.price)"/>
    <!-- other components -->
</window>
```

> `@id('vm')`设置ViewModel名称，然后我们可以使用vm在以下组件中引用ViewModel的属性(第一行)

### 属性是Object或者JavaBean

如果属性是JavaBean，则也可以通过EL表达式访问该JavaBean的属性

#### Object类型属性

```java
public class Address {
    private String city;
    private String street;
		//gettter & setter
}
public class ObjectViewModel {

    private Integer index;
    private String name;
    //JavaBean
    private Address address;

    public int getIndex() {
        return index;
    }

    public void setIndex(Integer index) {
        this.index = index;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Item getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }
}
```

#### 使用ObjectViewModel的zul

```xml
<intbox value="@bind(vm.index)"/>

<textbox value="@load(vm.name)"/>

<label value="@load(vm.address.street)"/>
```

> `vm.address` 是一个JavaBean, 所以我们可以使用EL表达式访问它的street属性，如 `vm.address.street` (第5行)

### 集合类型属性

如果UI组件是诸如列表框或网格之类的集合容器，则应将其绑定到类型为map或集合的子接口（例如List或Set）的属性。

#### 集合类型属性

```java
public class CollectionViewModel {
    //primitive type
    private int selectedIndex;
    //Collection
    private List itemList;

    public int getSelectedIndex() {
        return selectedIndex;
    }

    public void setSelectedIndex(int index) {
        selectedIndex = index;
    }

    /* As we can't save whole list through data binding, the ViewModel only provides getter for itemList.*/
    public List getItemList() {
        return itemList;
    }
}
```

#### 使用CollectionViewModel的zul

```xml
<label value="@load(vm.address.street)"/>
<listbox model="@load(vm.itemList)" selectedIndex="@bind(vm.selectedIndex)">
<template name="model" var="item">
    <listitem label="@load(item.name)"/>
</template>
</listbox>
```

> 第二行: Listbox的 model 属性被绑定到一个 集合对象 `vm.itemList`

## Commands

### 概述

ViewModel是View的抽象。 View负责显示信息并与用户进行交互。 该信息对应于ViewModel的属性，而交互对应于ViewModel的Command。 Command是操纵ViewModel属性的动作。 每个命令都提供了View可以在ViewModel上执行的操作。 这些操作也是用户与View进行交互的方式。 例如，一个ViewModel提供2个命令：“保存”和“删除”。 这意味着用户只能使用ViewModel在View上执行这两个操作。 他们可以通过单击按钮或菜单项来执行操作。

 

由于ViewModel的角色类似于Controller，开发人员可以通过指定Command的名称来将UI组件的事件绑定到Command，该名称类似于注册事件侦听器。 多个事件可以绑定到同一命令。 当用户与组件交互时（例如，单击按钮），该组件会触发一个事件，然后数据绑定机制将触发Command的执行。 该命令可以修改ViewModel的属性，然后显示在View上的信息会更改。

<img src="https://gitee.com/leehuajun/imgbed/raw/master/2020/20200608141744.png" alt="image-20200608141744470" style="zoom: 67%;" />

Command作为ViewModel的方法实现。 因为ViewModel是POJO，所以为了使数据绑定机制标识哪个方法表示Command，开发人员必须使用ZK提供的@command注解对方法进行注解。 在下一节中，我们将使用术语：Command方法来描述ViewModel的特殊注解方法。 这些方法通常会操纵ViewModel的属性，例如删除项目。 触发组件的事件将触发绑定命令的执行，即调用Command方法。 在执行命令期间，开发人员还必须指定要更改的属性以通过Java注解进行通知，我们将在后面的部分中进行描述。

### 声明Commands

#### 本地Command

ViewModel的Command类似于事件处理程序，我们可以将事件绑定到Command。 事件和command之间的绑定就是我们所说的“Event-Command Binding”。 建立此绑定之前，我们必须在ViewModel中声明其名称的Command。 请注意，ViewModel中的命令名称不能重复，否则会导致运行时异常。

```java
public class OrderVM {

    // create and add a new order to a list
    // command name is set to method name by default
    /* Notice that we can declare a Command without specifying its name, and its name is set to method name by default. */
    @Command
    public void newOrder() {
        Order order = new Order();
        getOrders().add(order); //add a new order to order list selected = order;//select the new one
    }

    // save an order
    // command name is specified
    /* We can also give Command's name by @Command('userDefinedName'). */
    @Command("save")
    public void saveOrder() {
        orderService.save(selected);
    }

    // delete an order
    // multiple command names
    /* We can even give multiple Command's name with array of String. */
    @Command({"delete", "deleteOrder"})
    public void deleteOrder() {
    //delete order
    }
}
```

然后，我们可以在ZUL中把组件的事件绑定到command

```xml
<toolbar>
    <button label="New" onClick="@command('newOrder')" />
    <button label="Save" onClick="@command('save')" />
</toolbar>
```

> 我们将在 [command binding here](#_bookmark16) 描述command绑定细节，绑定允许您将参数传递给Command方法，请参阅 [here](#_bookmark32).

#### 全局Command

全局Command也是ViewModel的command，可以将UI组件的事件绑定到它。 只能通过ViewModel的Root View组件及其子组件的事件来触发本地Command。 全局Command可以由任何ZUL中的组件事件触发。 全局Command与本地Command的主要区别在于事件不必属于ViewModel的根视图组件或其子组件。 默认情况下，我们可以将事件绑定到同一桌面上的任何ViewModel的全局Command。 方法既可以是本地Command，也可以是全局Command。

```java
@Command("delete")
@GlobalCommand("delete")
public void deleteOrder() {
    //...
}
```

我们可以在不同的ViewModel中声明多个具有相同名称的全局Command。 当事件触发全局命令时，将执行每个ViewModel中所有匹配的Command方法。

```java
public class MainViewModel {
    @GlobalCommand
    public void show() {
        //...
    }
}

public class ListViewModel {
    @GlobalCommand
    public void show() {
        //...
    }
}
```

> 如果触发全局Command "`show`"，则与每个ViewModel关联的每个binder将执行show 全局Command方法，但`不按任何特定顺序`执行。

### Command执行

Command执行是ZK Bind的一种机制，它在ViewModel上执行方法调用。 它绑定到组件的事件，并且当绑定事件到来时，绑定程序将遵循生命周期来完成执行。 我们将在 [Command Binding ](#_bookmark16)和 [Global Command Binding](#_bookmark27)中对此进行详细描述。

## 通知（Notification）

### 概述

binder负责在View和ViewModel之间同步数据。 通常，ViewModel是POJO，并且不引用任何UI组件。 因此，它无法将数据推送到UI组件。 开发人员必须使用ZK提供的Java注解指定ViewModel属性的依赖关系，然后binder知道何时重新加载哪个属性和更新UI视图

### Notify Change

ZK绑定提供了一组Java注解（@NotifyChange，@DependsOn，@NotifyChangeDisabled），用于指定何时将哪个属性重新加载到UI组件。 开发人员必须在设计时指定它，binder将在运行时同步数据。 binder跟踪组件的属性和ViewModel的属性之间的绑定关系。 每次调用ViewModel的方法（setter，getter或command方法）后，它都会根据注解中指定的属性寻找要重新加载的相应组件属性。

#### 在Command通知（Notify on Command）

在执行command期间，一个或多个属性会更改，因为Command方法通常包含业务或表示逻辑。 开发人员必须在Java注解的元素中指定要更改的一个或多个属性，然后数据绑定机制可以在执行Command后在运行时将它们从ViewModel重新加载到View。

通知属性更改的语法：

一个属性： 							@NotifyChange("oneProperty")

多个属性:								@NotifyChange({"property01","property02"}) 

ViewModel的所有属性: 	  @NotifyChange("\*")

例子： @NotifyChange :

```java
public class OrderVM {

    //the order list 
    List<Order> orders;
    //the selected order 
    Order selected;

    //getter and setter

    @NotifyChange("selected")
    @Command
    public void saveOrder() {
        getService().save(selected);
    }

    /* In newOrder() , we change property "orders" and "selected",
    therefore we apply @NotifyChange to notify binder to reload 2 changed properties after command execution. */
    @NotifyChange({"orders", "selected"})
    @Command
    public void newOrder() {
        Order order = new Order();
        getOrders().add(order);
        selected = order;//select the new one
    }

    /* The deleteOrder() also change the same 2 properties. Assume that this ViewModel class has only 2 properties,
    we can use "*" (asterisk sign) to notify all properties in current class. */
    @NotifyChange("*")
    @Command
    public void deleteOrder() {
        getService().delete(selected);//delete selected getOrders().remove(selected);
        selected = null; //clean the selected
    }
}
```

#### Notify on Setter

由于类似的原因，属性可能会在setter方法中更改。 将值保存到ViewModel的属性后，多个组件或其他属性可能取决于更改的属性。 我们还必须通过@NotifyChange指定此数据依赖项。

#### 默认启用（Enabled by Default）

setter方法通常仅更改一个属性，例如 setMessage()将message设置为新值。 为了节省开发人员的工作量，将自动通知通过setter方法更改的目标属性。 也就是说，除非在setter上更改了某些其他属性并且需要显式通知，否则在setter上不需要@NotifyChange注解

```java
public class MessageViewModel {
    private String message;

    public String getMessage() {
        return this.message;
    }

    public void setMessage(String msg) {
        message = msg;
    }
}
```

> 如果在setter方法中仅更改了message，则无需在MessageViewModel中的setMessage()上指定@NotifyChange

 

**使用MessageViewModel的zul**

```xml
<textbox id="msgBox" value="@bind(vm.message)"/>
<label id="msg1" value="@load(vm.message)"/>
<label id="msg2" value="@load(vm.message)"/>
```

当用户在msgBox中输入值并将该值保存回ViewModel时，数据绑定机制将自动将该值同步到msg1和msg2。

> 开发人员可以通过在setter方法上添加@NotifyChangeDisabled来禁用此默认行为。

#### 指定依赖关系（Specify Dependency）

对于setter方法，当多个属性具有依赖关系时,使用@NotifyChange,例如 一个属性的值是从其他属性计算得出的。 请注意，如果在setter方法上应用@NotifyChange，则默认通知行为将被覆盖。

**@NotifyChange on Setter**

```java
public class FullnameViewModel {
    //getter and setter 
    @NotifyChange("fullname")
    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }

    /* By default setLastname() will notify "lastname" property's change. Becuase we apply NotifyChange on it, this default notification is overridden.
    We have to notify it explicitly. */
    @NotifyChange({"lastname", "fullname"})
    public void setLastname(String lastname) {
        this.lastname = lastname;
    }

    public String getFullname() {
        return (firstname == null ? "" : firstname) + " "
                + (lastname == null ? "" : lastname);
    }
}
```

在上面的代码段中，全名由名和姓连接在一起。 通过设置程序更改名字和姓氏时，应重新加载全名以反映其更改。

如果一个属性依赖于许多属性，则@NotifyChange将分布在ViewModel类的多个位置。 为了方便起见，开发人员可以使用@DependsOn。 该注解定义了属性之间的依赖关系，可以提供与显式@NotifyChange相同的功能，但它用于getter方法。

#### @DependsOn例子

```java
public class FullnameViewModel {

    //getter and setter

    public void setFirstname(String firstname) {
        this.firstname = firstname;
    }

    public void setLastname(String lastname) {
        this.lastname = lastname;
    }

    /* The example is functional equivalent to previous one, but written in
    reversed meaning. It means when any one of 2 properties (firstname, lastname) change, fullname should be reloaded. */
    @DependsOn({"firstname", "lastname"})
    public String getFullname() {
        return (firstname == null ? "" : firstname) + " "
                + (lastname == null ? "" : lastname);
    }
}
```

### 通知Bean变更（Notify Bean Change）

如果将组件的属性绑定到Bean，并希望在Bean的属性更改后重新加载它，则应在目标属性的setter上应用以下语法：

#### @NotifyChange(".")

请注意，点(.)与星号(\*)有点不同，因为星号@NotifyChange("\*")仅通知binder重新加载绑定到bean属性的那些组件，例如 @bind(vm.person.firstname)，@bind(vm.person.lastname)和@bind(vm.person.fullname)。 与Bean本身绑定的component属性（即vm.person）将不会使用@NotifyChange("\*")重新加载。 在这种情况下，您必须使用@NotifyChange(".")来告诉binder该bean本身已经更改，并且绑定到该bean的component属性必须重新加载。

 

对于哪些用例，您将需要此功能？ 大多数时候，我们将一个组件属性绑定到一个bean属性。 但是，在某些情况下，您可能希望将component属性绑定到多个Bean属性的计算结果。 但是您不想将计算逻辑放入bean中。 然后，您可以将component属性绑定到整个bean，并具有一个转换器，用于根据bean的多个属性来转换数据。

 

例如，假设旅游网站中有一个网页。 客户必须填写旅行的休假日和返回日，并且旅行时间将自动计算并在用户填写两个字段后显示在屏幕上。 在这里，我们选择使用转换器来进行持续时间计算，而不是将Trip类中的逻辑实现为属性。 在这种情况下，您将需要在LeaveDate和returnDate属性设置器上添加@NotifyChange(".")注解，以便当LeaveDate或returnDate属性更改时，可以更新duration属性。

#### 绑定到对象示例

```xml
<hlayout>
    Leave Date:<datebox value="@save(vm.trip.leaveDate)"/>
</hlayout>
<hlayout>
    Return Date:<datebox value="@save(vm.trip.returnDate)"/>
</hlayout>
Duration including leave and return(day): <label value="@load(vm.trip) @converter(vm.durationConverter)"/>
```

> 因为durationConverter需要访问2个属性（leaveDate，returnDate），所以我们必须将标签的值绑定到旅行bean本身，而不是bean的单个属性。

#### @NotifyChange(".") 例子

```java
public class DurationViewModel {
    private Trip trip = new Trip();

    public class Trip {
        private Date leaveDate;
        private Date returnDate;

        public Date getLeaveDate() {
            return leaveDate;
        }

        @NotifyChange(".")
        public void setLeaveDate(Date leaveDate) {
            this.leaveDate = leaveDate;
        }

        public Date getReturnDate() {
            return returnDate;
        }

        @NotifyChange(".")
        public void setReturnDate(Date returnDate) {
            this.returnDate = returnDate;
        }
    }

    public Converter getDurationConverter() {
        return new Converter() {
            public Object coerceToUi(Object val, Component component, BindContext ctx) {
                if (val instanceof Trip) {
                    Trip trip = (Trip) val;
                    Date leaveDate = trip.getLeaveDate();
                    Date returnDate = trip.getReturnDate();
                    if (null != leaveDate && null != returnDate) {
                        if (returnDate.compareTo(leaveDate) == 0) {
                            return 1;

                        } else if (returnDate.compareTo(leaveDate) > 0) {
                            return ((returnDate.getTime() - leaveDate.getTime()) / 1000 / 60 / 60 / 24) + 1;
                        }
                        return null;
                    }
                }
                return null;
            }

            public Object coerceToBean(Object val, Component component, BindContext ctx) {
                return null;
            }
        };
    }
}
```

> 关于如何实现转换器，请参阅  [Data Binding/Converter](#_bookmark23)

### 以编程方式通知（Notify Programmatically）

有时，我们要通知的已更改属性取决于运行时的值，因此我们无法在设计时确定属性的名称。 在这种情况下，我们可以使用 `BindUtils.postNotifyChange(String queueName, String queueScope, Object bean, String property))` 动态通知更改。 此通知的基本机制是我们在[the binder section](#_bookmark14)部分讨论的binder订阅事件队列。 默认情况下，它使用`desktop scope`事件队列。

#### 动态通知（Dynamic Notification）

```java
    String data;
    // setter & getter

    @Command
    public void cmd() {
        if (data.equal("A")) {
            //other codes... 
            BindUtils.postNotifyChange(null,null,this,"value1");
        } else {
            //other codes... 
            BindUtils.postNotifyChange(null,null,this,"value2");
        }
    }
```

postNotifyChange()的第一个参数是队列名称，第二个参数是队列作用域。 您可以将其保留为空，并且将使用默认队列名称和范围（桌面）。