## **2.Hello World**

---

### **让我们写一个并部署我们第一个智能合约**

>在Flow开发者平台打开本教程的入门代码：
>
>https://play.onflow.org/83315e10-8a9d-40d1-9f4a-877ca93eb93c
>
>本教程将要求您采取多种操作来与此代码进行交互。

>需要您采取行动的说明总是包含在这样的标注框中。
>
>这些突出显示的操作是让代码运行所需的全部操作，但阅读其余部分对于理解语言的设计是必要的。

智能合约是一段可验证和可执行并且无需可信任第三方的程序。在区块链上运行的程序通常被称为智能合约，因为它们可以调解重要的功能（例如货币）而无需依赖中央机构（例如银行）。

在开始在Flow上编程前，我们需要知道账户和交易是如何创建起来的。

### **账户和交易**

---

像许多其他区块链一样，Flow的设计模式也是以账户和交易为中心的。所有持久状态全都存储在账户中。接口（交互状态的方式）也存储在账户中。所有代码的运行都发生在交易中，交易是外部用户提交的代码块，用于与持久状态交互，包括直接修改帐户存储的状态。

每个账户都会被一个或多个私钥绑定。这意味着协议本身默认就支持了[多重签名](https://www.coindesk.com/what-is-a-multisignature-crypto-wallet)绑定账户/钱包。

一个账户分为两个主要的模块：

1. 第一个模块是[合同模块](https://docs.onflow.org/cadence/language/accounts/)。这个模块存放了所有的智能合约，包括合约的类型定义，作用域，还有与之相关具有相同功能的函数。这个模块也拥有合约的接口，这些接口基本上是其他合约导入和实现的程序指南。合同模块不能直接从交易中读取到，否则交易仅仅是读取到部署在账户上面的代码。账户的所有者可以直接添加、删除或更新/覆盖存储在其中的合约。
2. 第二个模块是账户的文件系统。这个模块是账户存储他们拥有的对象和控制如何访问这些对象功能的地方。

对象存放在[文件系统下的路径](https://docs.onflow.org/cadence/language/accounts/#paths)。路径由域名和标识符构成。

路径起始字符是 `/`，然后紧跟域名，路径分隔符用`/`，最后是标识符。举个栗子，路径`/storage/test`域名为`storage`标识符为`test`。

只有三个有效的域名分别代表了账户文件系统三个不同的模块：`storage`, `private` ,  `public`。

标识符是自定义的，可以是您想要使用的任何名称，以指示该路径中存储的内容。

+ `storage`域名是一个存储所有对象（例如代表着tokens和NFTs的结构体和资源对象）的路径。它仅能被账户所有者直接访问。
+ `private`域名就像一个私有API。您可以选择在此处将[功能定位符](https://docs.onflow.org/cadence/language/capability-based-access-control/)（capabilities）存储到您存储的任何资产中。仅有所有者或者给予授权的人才可以使用这个接口去调用定义在你的私人资产里的函数。
+ `public`域名就像一个账户公有API。所有者可以在此处链接功能定位符，网络中的任何其他人都可以访问这些功能定位符，以便与存储在帐户中的私有资产进行交互。

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/accounts-diagram.png)

在Flow中，一笔[交易(transaction)](https://docs.onflow.org/cadence/language/transactions/)被定义成一个具有任意大小，可以被一个或多个账户签名的代码块。交易可以访问那些已经签名过的账户的`/storage`和 `/private`域名，并且可以读写这些域名，同时还可以在读和调用这些公有合约的函数和其他用户公用域名。



### **创建一个智能合约**

---

我们从写一个返回`"Hello World!"`公有函数来创建我们的第一个智能合约。

>首先，您需要按照此链接打开一个带有预加载的 Hello World 合约、交易和脚本的平台会话：{" "}
>
>https://play.onflow.org/83315e10-8a9d-40d1-9f4a-877ca93eb93c

>打开账户 `0x01` 的标签页可以看到名为 `HelloWorld.cdc`的文件。
>
>`HelloWorld.cdc` 应该包含如下代码

`HelloWorld.cdc`

```Cadence
pub contract HelloWorld {

    // Declare a public field of type String.
    //
    // All fields must be initialized in the init() function.
    pub let greeting: String

    // The init() function is required if the contract contains any fields.
    init() {
        self.greeting = "Hello, World!"
    }

    // Public function that returns our friendly greeting!
    pub fun hello(): String {
        return self.greeting
    }
}
```

在Flow中，一个合约就是代码（函数）和数据（状态）的集合。合约的作用域只在一个账户下的合约模块中。账户可以有一个或者多个合约和合约接口，并且这些合约及合约接口可以由账户所有者免费地添加，修改和删除。

`pub let greeting: String `这行声明了一个公有（`pub`）状态常量（`let`）叫做`greeting`字符串类型。如果我们想要声明一个变量可以使用`var`。

`pub` 关键字是一个访问控制格式的例子，这意味着在它可以在所有作用域下访问，但是并不能在所有作用域下写入。如果你更喜欢带有描述性的关键字，你也可以使用`access(all)`来替换`pub`

请参阅[语言参考：访问控制部分](https://docs.onflow.org/cadence/language/access-control/)，了解有关 Cadence 中允许的不同级别的访问控制的更多信息。

`init()`这部分称为构造函数，这个函数只有当合约创建时运行并且之后不会在运行。这个 栗子中，构造函数设置了`greeting`值为`"Hello, World!"`。

下一个声明的公有函数返回了一个`String`类型的值。导入这个合约的任何人都可以访问这个公有域，使用公有类型，然后调用公有合约函数；这就是声明`pub`或者`access(all)` 的作用。

现在我们可以将此合约部署到您的帐户并在交易中调用其函数。



### **部署代码**

---

现在您已经有一些可以运行的Cadence代码，你可以部署它到您的账户上。

>确认账户页`0x01`存在并且已经编辑完成`HelloWorld.cdc`文件
>
>现在让我们点击部署按钮把编辑器里的内容部署到账户`0x01`上。

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/playground-deploy.png)

您应该会在输出区域中看到一条日志，表明部署成功。 （如果交易号或区块不同，请不要担心。）

```Cadence
11:18:34 Deployment > [1] > Deployed Contract To: 0x01
```

你也可以看到以该名字命名的合约已经出现在所选账户的下面。这表示`HelloWorld` 合约已经成功部署到该账户。你也可以点进这个账户标签页去确认哪些合约在哪些账户中，但每个账户只能有一个合约。



### **创建一个交易**

---

>打开名为`Say Hello`的交易
>
>`Say Hello`应该包含如下代码

`Say`

```Cadence
import HelloWorld from 0x01

transaction {

    // No need to do anything in prepare because we are not working with
    // account storage.
    prepare(acct: AuthAccount) {}

    // In execute, we simply call the hello function
    // of the HelloWorld contract and log the returned String.
    execute {
        log(HelloWorld.hello())
    }
}
```

这就是一个Cadence的**交易**。一个交易可以包含导入来自其他账户，与账户存储交互，与其他账户交互等的任何代码。

为了与智能合约交互，这个交易首先导入通过从存储它的地址检索其定义来导入该智能合约。这个导入包含接口的定义，资源定义，和公有函数，以便交易可以使用它们与合约本身或使用该合约的其他账户进行交互。

为了导入来自其他账户的智能合约，你可以写入这一行：

```Cadence
import {合约名称} from {账户地址}
```

交易可以分为两个阶段：`prepare`（准备阶段）和`execute`（执行阶段）。

1. `prepare`准备阶段是唯一可以访问签名帐户的私有 `AuthAccount`（授权账户） 对象的地方。`AuthAccount`（授权账户）有特殊的方法，允许从`/storage`读出和写入，创建`/private`和`/public`链接到`/storage`中的对象。（称之为[功能定位符](https://docs.onflow.org/cadence/language/capability-based-access-control)（capabilities），之后会介绍）
2. `execute`执行阶段无法访问`AuthAccount`，因此只能修改在准备阶段删除的对象并调用外部合约和对象上的函数。

通过在准备阶段不允许访问账户存储，我们可以静态地验证对于指定的交易哪些资产和认证的存储区域可以修改。对于用户而言，在浏览器钱包和应用程序中提交的交易可以使用这种方法来显示交易可能改变的内容，并且用户可以更有信心地相信他们不会通过应用程序生成的交易获得恶意攻击。

您也可以在平台上通过单击多个帐户头像来拥有多个交易签名者，但是需要交易的准备的代码块参数数量需要与签名者的数量相同。如果没有，这将导致错误。

在这个交易中，我们从它部署到的地址导入合约并调用它的 `hello` 函数。

>在编辑器右上角的框中，选择账户 `0x01` 作为交易签名者。 点击发送`Send`按钮提交交易

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/playground-signers.png)

您应该会看到如下内容：

```
"Hello, World!"
```

恭喜您，您刚才运行了你的第一个Cadence交易！:100:



### **创建一个资源**

---

接下来，我们将要练习使用[资源](https://docs.onflow.org/cadence/language/composite-types/#resources)，这是Cadence定义的新特性之一。一个资源包含多个基本类型就像一个结构体或者类，但还有些特别的规则。

>打开账户页`0x22`可以看到名为`HelloWorldResource.cdc`的文件
>
>`HelloWorldResource.cdc`应该包含如下的代码：

`HelloWorldResource.cdc`

```Cadence
pub contract HelloWorld {

  // Declare a resource that only includes one function.
  pub resource HelloAsset {
    // A transaction can call this function to get the "Hello, World!"
    // message from the resource.
    pub fun hello(): String {
      return "Hello, World!"
    }
  }

  init() {
    // Use the create built-in function to create a new instance
    // of the HelloAsset resource
    let newHello <- create HelloAsset()

    // We can do anything in the init function, including accessing
    // the storage of the account that this contract is deployed to.
    //
    // Here we are storing the newly created HelloAsset resource
    // in the private account storage
    // by specifying a custom path to the resource
    self.account.save(<-newHello, to: /storage/Hello)

    log("HelloAsset created and stored")
  }
}
```

>使用`Deploy按钮`把代码部署到账户`0x02`上

这是合约的另一个用处。Cadence 可以在部署的合约中声明类型定义。任何帐户都可以导入这些定义并使用它们与这些类型的对象进行交互。该合约声明了 `HelloAsset` 资源的定义。

让我们来看看这个合约：

```
pub resource HelloAsset {
    pub fun hello(): String {
        return "Hello, World!"
    }
}
```

资源是复合类型就像结构体和类一样，因为它们可以包含任意数量的字段或函数。唯一的区别就是代码如何与它们交互。当直接在所有者创建的时候它们都是很有用的。但事实上每一个资源的实例都只能存在一份并且不能被复制。在它们被访问的时候，必须显式地明确资源被修改或者移动，只有这样才能确保避免发生意外丢失。

其他传统编程语言的结构对于支持所有权可能不是一个理想的方式，因为他们可以被复制。这意味着一个编程上的错误会导致把一个相同的资产创建了很多的副本，这会打破了资产具有真正价值所需的稀缺性。我们必须要思考避免发生一个房子，一辆车或者一个具有百万美元的银行账户，甚至一匹马的损失和盗窃事件发生。因此，资源（Resource），这个数据结构，通过显式表达资产的创建，销毁和移动来解决这个问题。

```Cadence
init(){
// ...
```

之前的例子已经看到，声明了`init()`构造函数。所有的复合类型像合约（contracts），资源（resources）和结构体（structs），都可以有一个可选的 `init()` 函数，该函数仅在最初创建对象时运行。

合约还可以通过使用内置的 `self.account` 对象将它们部署到的帐户的存储进行读写访问。这是一个`AuthAccount`（授权账户）对象，可以使他们访问许多不同的功能，以与帐户的私有存储进行交互。

在合约的`init`构造函数中，合约使用`create`关键字去创建一个`HelloAsset`类型的实例并且将其保存到局部变量。为了创建一个新的资源对象，我们使用`create`关键字，然后使用任何 `init()` 参数调用资源命名。资源只能在定义它的范围内创建。这样可以防止任何人能够创建其他人定义的任意数量的资源对象。

```Cadence
let newHello <- create HelloAsset()
```

在这里我们使用运算符`<-`。代表移动操作。在涉及资源操作中，移动运算符取代赋值运算符`=`。为了明确分配资源，移动运算符`<-`当且仅当如下的情况才使用：

+ 初始化时，资源是常量或变量。
+ 资源在分配中移动到不同的变量。
+ 资源作为参数移动到函数。
+ 资源从函数中返回。

在资源移动后，之前的存放地址已经失效，对象移动到新的存放地址的上下文中。不允许常规的任务分配，因为常规的任务分配只会复制值。资源在同一时间仅能存在于一个位置，所以移动资源务必要显式的展示在代码中。

它使用 `AuthAccount.save` 函数将其存储在帐户存储中。

```Cadence
self.account.save(<-newHello, to: /storage/Hello)
```

合约可以使用关键字 `self` 引用其成员函数和字段。所有合约都可以访问部署它们的帐户的存储内容，并且可以使用 `self.account` 访问该 `AuthAccount` 对象。

`AuthAccount`（授权对象）有许多不同的方法用来和帐户的存储内容交互。您可以在语言参考-存储部分或词汇表中查看所有这些的文档。`save`方法保存一个对象到账户的存储中去。对象类型的类型参数包含在 `<>` 中以指示存储的对象是什么类型。这也可以推断出来参数是什么类型。

第一个参数是正在存储的对象，参数`to`代表了该对象应该存放在哪个路径下。路径应该是一个存储路径，也就是说，只有路径的域名为stroage才可以出现在`to`参数中。

如果给定路径下已经存储了一个对象，则程序中止。要记住，Cadence类型系统确保了资源不会发生意外的丢失。把资源移动到字段、数组、字典或存储时，存储位置可能已经包含该资源。Cadence要求开发人员处理这种已经存在资源的这种情况，以免因覆盖而意外丢失。这也是为什么我们在`newHello`中什么也不做也不能让函数执行完毕。并且解释了如果资源已经存在在指定的路径下，`save`会报错。

在这种情况下，这是我们使用所选帐户运行的第一笔交易，因此我们知道`/storage/Hello `处的存储位置是空的。在实际应用中，我们可能会对存储的位置执行必要的检查和操作，以确保我们不会因为意外覆盖而中止交易。

现在您已将资源存储在帐户中，您应该会看到该资源显示在编辑器下方的资源框中。此框指示所选帐户中存储了哪些资源，以及这些资源中字段的值。现在，您应该看到 `HelloAsset` 资源存储在帐户 `0x02` 的账户存储中，并且没有字段。

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/playground-resources.png)



### **与资源交互**

---

>打开名为`Load Hello `的交易
>
>`Load Hello`应该包含如下代码

```Cadence
import HelloWorld from 0x02

// This transaction calls the "hello" method on the HelloAsset object
// that is stored in the account's storage by removing that object
// from storage, calling the method, and then putting it back in storage

transaction {

    prepare(acct: AuthAccount) {

        // load the resource from storage, specifying the type to load it as
        // and the path where it is stored
        let helloResource <- acct.load<@HelloWorld.HelloAsset>(from: /storage/Hello)

        // We use optional chaining (?) because the value in storage
        // may or may not exist, and thus is considered optional.
        log(helloResource?.hello())

        // Put the resource back in storage at the same spot
        // We use the force-unwrap operator `!` to get the value
        // out of the optional. It aborts if the optional is nil
        acct.save(<-helloResource!, to: /storage/Hello)
    }
}
```

这笔交易从我们刚才部署的账户上导入了`HelloWorld`的定义，从账户存储中移走了`HelloAsset`对象，并且调用了存储在`HelloAsset`资源中的`hello()`函数。

为了把账户存储中的对象拿走，我们使用了`load`方法。

```Cadence
let helloResource <- acct.load<@HelloWorld.HelloAsset>(from: /storage/Hello)
```

如果没有对象存储在给定的路径下，函数会返回nil。当函数返回后，该路径下的存储里不会有该对象。

要加载的对象类型的类型参数包含在 `<>` 中。类型参数必须要显式表达出来，就像这里`@HelloWorld.HelloAsset`。（注意`@`表明这是一个资源）

路径`path`一定是一个账户存储路径，也就是只允许域名为` /storage/`。

接下来，我们调用 `hello` 函数并记录输出。

```Cadence
log(helloResource?.hello())
```

我们用 `?`因为存储中的值作为[备选值(optionals)](https://docs.onflow.org/cadence/language/values-and-types/#optionals)返回。备选值可以表示空值。备选值有两种情况：要么有指定类型的值，要么什么都没有（`nil`）。可选类型是使用 `?`后缀。

```Cadence
let newResource: HelloAsset?  // could either have a value of type `HelloAsset`
                              // or it could have a value of `nil`
```

备选值允许开发者更优雅地处理值为`nil`的情况。在这里，我们必须明确地考虑到我们通过 `load` 获得的 `helloResource` 对象为`nil`的可能性。在调用`hello`函数之前，使用`?`"扩展"备选值使用范围，但前提是该值不为零。如果值为 `nil`， `?`返回零。

因为在调用`hello`函数时`?`已经使用了，如果存储值不为`nil`函数调用才会发生。在这个例子中，`hello` 函数的结果将作为备选值返回。但是，如果存储的值为 `nil`，则不会发生函数调用并且结果为 `nil`。

接下来，我们再次使用 `save` 将对象放回存储中的同一位置：

```Cadence
acct.save(<-helloResource!, to: /storage/Hello)
```

请记住，`helloResource`为一个备选值，所以我们必须要处理值为`nil`的情况。我们使用强制解析（force-unwrap operator）(`!`)。如果它包含一个值，则此运算符获取备选值中的值，如果对象为 `nil`，则中止整个交易。这是一种处理备选值的风险更大的方式，但是如果您的程序曾经处于值为`nil`会破坏整个交易，那么强制解析（force-unwrap） 运算符是处理这种情况的不错选择。

可以参阅[Cadence备选值(Optionals In Cadence)](https://docs.onflow.org/cadence/language/values-and-types/#optionals)]来学习到更多用法。

>选择帐户 `0x02` 作为唯一的签名者。点击`Send`按钮提交交易。

您应该会看到如下内容：

```
"Hello, World!"
```



### **创建功能定位符和对存储资源的引用**

---

在这个栗子中，我们会创建一个链接去引用`HelloAsset`资源对象，然后使用这个引用去调用`hello`函数。此交易中发生的事情的详细说明位于交易代码下方，因此如果您感到费解，请继续阅读！

>打开名为`Create Link`的交易
>
>`Create Link`应该包含如下代码

`Create`

```Cadence
import HelloWorld from 0x02

// This transaction creates a new capability
// for the HelloAsset resource in storage
// and adds it to the account's public area.
//
// Other accounts and scripts can use this capability
// to create a reference to the private object to be able to
// access its fields and call its methods.

transaction {
  prepare(account: AuthAccount) {

    // Create a public capability by linking the capability to
    // a `target` object in account storage.
    // The capability allows access to the object through an
    // interface defined by the owner.
    // This does not check if the link is valid or if the target exists.
    // It just creates the capability.
    // The capability is created and stored at /public/Hello, and is
    // also returned from the function.
    let capability = account.link<&HelloWorld.HelloAsset>(/public/Hello, target: /storage/Hello)

    // Use the capability's borrow method to create a new reference
    // to the object that the capability links to
    // We use optional chaining "??" to get the value because
    // result of the borrow could fail, so it is an optional. 
    // If the optional is nil,
    // the panic will happen with a descriptive error message
    let helloReference = capability!.borrow()
      ?? panic("Could not borrow a reference to the hello capability")

    // Call the hello function using the reference 
    // to the HelloAsset resource.
    //
    log(helloReference.hello())
  }
}
```

>确保账户`0x02`仍然为账户签名者
>
>点击`Send`按钮来执行这笔交易

您应该可以看到`"Hello, World"`又一次出现在终端上。这是因为我们用**功能定位符(capability)**绑定了`HelloAsset`这个对象，存储在功能定位符中，路径为`/public/Hello`，并借用了一个引用，使用我们的引用来调用对象的 `hello` 方法。

功能定位符（Capabilities）有点类似于其他语言的指针。账户存储域名`/storage/`是私有的，但是用户仍然希望允许其他用户访问他们的私有对象的功能，而不能删除它们。***译注：这句话概述了功能定位符的作用***

在我们的栗子中，`HelloAsset`的所有者仍然希望想要让其他人去调用`hello`方法。这就是功能定位符做的事情。它们代表帐户存储中对象的链接，该对象具有创建链接时指定的类型。

要注意功能定位符只能允许访问字段和方法。但是并不允许复制，移动，或者直接修改源对象。

让我们分解一下这笔交易中发生的事情。

首先，我们在创建了一个功能定位符指向了`/stroage/`路径下的`HelloAsset`私有对象。

```Cadence
let capability = account.link<&HelloWorld.HelloAsset>(/public/Hello, target: /storage/Hello)
```

`HelloAsset`对象存储在`/stroage/Hello`，这个路径下只有账户所有者才能访问。他们想要互联网上的任何用户都可以调用`hello`方法，因此在`/public/Hello`创建了一个功能定位符。

为了创建一个功能定位符，我们使用了`AuthAccount.link`方法来用一个新的功能定位符来绑定指定对象。`<>` 中包含的类型是功能定位符代表的受限引用类型。该功能表示，从该功能定位符中借用引用的人只能访问 `<>` 中类型指定的字段和方法。

用符号`&`代表引用。在这里，功能定位符引用了`HelloAsset`对象，而且我们明确了限定`<&HelloWorld.HelloAsset>`该类型，这授予对 `HelloAsset` 对象中所有内容的访问权限。

`link`这个函数的第一个参数是您要存储功能的路径，`target`这个参数是要链接到的存储中对象的路径。

为了从功能定位符借用对象的引用，我们使用功能定位符的借用方法。

```Cadence
let helloReference = capability!.borrow()
    ?? panic("Could not borrow a reference to the hello capability")
```

我们在`link`函数中明确了这个方法在创建引用时候只能使用`<>`中的类型。在这里我们使用强制解析运算符(`!`)因为功能定位符是一个备选值(optional)。如果功能运算符为`nil`交易就会终止。在借用引用时，我们使用备选值来确保返回值，因为引用的借用可能会失败。如果目标存储槽是空的、已经被借用，或者如果请求的类型超出了功能运算符允许的范围，则引用可能为`nil`。我们用`panic`来包装错误消息，以便调用者可以更好地知道出了什么问题。

我们将此过程分为功能定位符和引用的原因是为了防止恶意行为者多次调用对象的重入bug。这些bug已经困扰其他的智能合约语言。一次只能存在一个对象引用，因此存储中的对象不可能存在这种类型的漏洞。

此外，对象的所有者可以通过移动功能定位符绑定的对象或使用 `unlink` 方法破坏链接来有效地撤销他们创建的功能定位符。如果被引用对象被移动或者链接被销毁，之前创建的功能定位符就变成无效的链接了。

最后，我们使用借用的引用调用 `hello()` 方法：

```Cadence
// Call the hello function using the reference to the HelloAsset resource
log(helloReference.hello())
```



### **执行脚本**

---

在Cadence中，脚本是非常简单的合约类型，它不能对区块链执行任何写入操作，只能读取帐户的状态。运行脚本无需任何账户授权。

为了运行脚本，您需写一个叫做`pub fun main()`的函数。你可以点击执行脚本按钮来去运行脚本。执行的结果会打印在终端。

>打开文件`Script1.cdc`
>
>`Script1.cdc`应该为如下代码：

`Script1.cdc`

```Cadence
import HelloWorld from 0x02

pub fun main() {

    // Cadence code can get an account's public account object
    // by using the getAccount() built-in function.
    let helloAccount = getAccount(0x02)

    // Get the public capability from the public path of the owner's account
    let helloCapability = helloAccount.getCapability<&HelloWorld.HelloAsset>(/public/Hello)

    // borrow a reference for the capability
    let helloReference = helloCapability.borrow()
        ?? panic("Could not borrow a reference to the hello capability")

    // The log built-in function logs its argument to stdout.
    //
    // Here we are using optional chaining to call the "hello"
    // method on the HelloAsset resource that is referenced
    // in the published area of the account.
    log(helloReference.hello())
}
```

该脚本用`getAccount`方法获取到了一个账户对象`PublicAccount`。

```Cadence
let helloAccount = getAccount(0x02)
```

公有账户对象`PublicAccount`是在互联网上对任何人都可以使用的对象，但只能访问一小部分功能，这些功能只能从帐户中的` /public/` 域名中读取。在此处查看有关帐户的更多信息：

https://docs.onflow.org/cadence/language/accounts/

然后，`hellAccount`这个对象获取功能定位符中绑定链接`Create Link` 。

```Cadence
// Get the public capability from the public path of the owner's account
let helloCapability = helloAccount.getCapability(/public/Hello)
```

为了获得存储在账户的功能运算符，我们使用`account.getCapability`函数。这个函数可以在`AuthAccount`和`PublicAccount`中使用。它在指定的路径上返回一个功能定位符，也具有指定的类型。它不检查目标是否存在，但如果能力无效，借用将失败。

脚本从功能定位符中借用了一个引用。

```Cadence
let helloReference = helloCapability.borrow()
```

然后，脚本使用引用调用 `hello` 函数并打印结果。

让我们执行脚本以查看它是否正确运行。

> 在开发者平台点击`Execute`按钮

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/playground-execute.png)

您应该看到类似这样的打印：

```Cadence
> "Hello, World"
> Result > "void"
```

做得好！您已经部署了您的第一个 Cadence 智能合约，并使用交易和脚本与之交互！

如果您仍然需要一些说明，这里有一些关于开发者平台的某些方面的提示。



### **账户**

---

当您打开开发者平台时，平台已经自动为您初始化了一定数量的可配置的默认账户。

在这个平台下，您可以通过选择屏幕左侧部分中该帐户的选项卡来选择帐户以编辑要部署的合同。该帐户对应的合约将显示在编辑器中，您可以在其中编辑并将其部署到区块链。

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/playground-accounts.png)

升级已经部署的合约是非常危险的，所以要小心！



### **交易**

---

智能合约部署后，你可以提交交易来与之进行交互。在屏幕左侧的交易选择部分，您可以选择不同的交易进行编辑和发送。在交易开启时，您可以选择一个或多个账户来签署交易。这是因为在Flow中，多个账户可以签署相同的合约，从而可以访问其私人存储。如果选择多个账户作为签名者，这需要在交易的签名中体现，以显示多个签名者：

```Cadence
// One signer

transaction {
    prepare(acct1: AuthAccount) {}
}

// Two signers

transaction {
    prepare(acct1: AuthAccount, acct2: AuthAccount) {}
}
```

如果您想要更多练习，您可以在新帐户上运行一些以前的交易，用来查找一些不同的交互和潜在的错误消息。



### **Flow 上的同质化代币**

---

现在您已经在 Flow 上编写并启动了您的第一个智能合约，您已经为更复杂的事情做好了准备！

