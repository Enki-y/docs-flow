## **7.复合资源**

---

在本教程中，我们将通过创建、部署和移动可组合 NFT 来学习资源如何拥有其他资源。

---

>在 Flow Playground 打开本教程的入门代码：
>
>https://play.onflow.org/fb934880-ba3a-4d0d-8350-7bea017e6138
>
>本教程将要求您采取各种措施与此代码进行交互。

>需要您采取行动的说明总是包含在这样的标注框中。
>
>这些突出显示的操作是让代码运行所需的全部操作，但阅读其余部分对于理解语言的设计是必要的。

拥有其他资源的资源是区块链和智能合约世界中的一个强大功能。为了展示此功能在 Flow 上的工作原理，本教程将通过可组合 NFT 引导您完成以下步骤：

1. 将 `Kitty` 和 `KittyHat` 定义部署到帐户 `0x01`。
2. 创建一个 `Kitty` 和两个 `KittyHats` 并将它们存储在您的帐户中。
3. 移动小猫和帽子，看看复合的 NFT 如何在 Flow 上发挥作用。

在继续本教程之前，我们建议您按照入门和 Hello, World! 中的说明进行操作。以便了解开发者平台和 Cadence。

如需更多支持，请参阅 [开发者平台手册](https://docs.onflow.org/intro/playground-manual/)。



### **拥有资源的资源**

---

在[非同质化代币](https://docs.onflow.org/cadence/tutorial/04-non-fungible-tokens)谈到的 NFT 集合是拥有其他资源的资源的例子。我们拥有一个资源，一个NFT集合，它拥有存储在其中的 NFT 资源的所有权。所有者和任何拥有引用的人都可以移动这些资源，但当它们在集合中时它们仍然属于集合，并且集合中定义的代码对资源具有最终控制权。

当集合被移动或销毁时，其中的所有 NFT 都会随之移动或销毁。

如果集合的所有者将整个收藏资源转移到另一个用户的帐户，所有代币都将移动到其他用户的帐户中。代币不会留在原始所有者的帐户中。这就像把你的钱包而不是一美元的钞票交给别人。这不是一个常见的动作，但肯定是可能的。

无法为存储在其他资源中的资源创建引用。拥有资源可以控制它，因此控制外部调用对存储资源的访问类型。



### **拥有资源的资源：一个例子**

---

NFT 集合是资源如何拥有其他资源的一个简单示例，但可以制作一个更强大的版本。

CryptoKitties（以及以太坊区块链上的应用程序）的一个重要特性是任何开发人员都可以围绕现有应用程序创造新体验。即使原始合同不包括对 CryptoKitty 配件（如帽子）的具体支持，独立开发人员仍然能够制作原始合同中 Kitties 可以使用的帽子。

以下是我们如何在 Cadence 中复制此功能的基本示例：

>打开帐户 `0x01` 选项卡，其中包含名为 `KittyVerse.cdc` 的合约。将代码部署到账户 `0x01`

`KittyVerse.cdc`

```Cadence
// The KittyVerse contract defines two types of NFTs.
// One is a KittyHat, which represents a special hat, and
// the second is the Kitty resource, which can own Kitty Hats.
//
// You can put the hats on the cats and then call a hat function
// that tips the hat and prints a fun message.
//
// This is a simple example of how Cadence supports
// extensibility for smart contracts, but the language will soon
// support even more powerful versions of this.

pub contract KittyVerse {

    // KittyHat is a special resource type that represents a hat
    pub resource KittyHat {
        pub let id: Int
        pub let name: String

        init(id: Int, name: String) {
            self.id = id
            self.name = name
        }

        // An example of a function someone might put in their hat resource
        pub fun tipHat(): String {
            if self.name == "Cowboy Hat" {
                return "Howdy Y'all"
            } else if self.name == "Top Hat" {
                return "Greetings, fellow aristocats!"
            }

            return "Hello"
        }
    }

    // Create a new hat
    pub fun createHat(id: Int, name: String): @KittyHat {
        return <-create KittyHat(id: id, name: name)
    }

    pub resource Kitty {

        pub let id: Int

        // place where the Kitty hats are stored
        pub let items: @{String: KittyHat}

        init(newID: Int) {
            self.id = newID
            self.items <- {}
        }

        destroy() {
            destroy self.items
        }
    }

    pub fun createKitty(): @Kitty {
        return <-create Kitty(newID: 1)
    }
}
```

这些定义显示了 Kitty 资源如何拥有帽子。

帽子存储在 Kitty 资源中的一个变量中。

```
// place where the Kitty hats are stored
pub let items: <-{String: KittyHat}
```

Kitty 主人可以摘下 Kitty 的帽子并单独转让。或者主人可以转让一只拥有帽子的小猫，帽子会和小猫一起去。

这是一个创建 `Kitty` 和 `KittyHat` 的交易，将帽子存储在 `Kitty` 中，然后将其存储在您的帐户存储中。

>打开`Transaction1.cdc`。
>
>选择账户 `0x01` 作为唯一的签名者，单击“发送”按钮发送交易。
>
>`Transaction1.cdc` 应包含以下代码：

`Transaction1.cdc`

```Cadence
import KittyVerse from 0x01

// This transaction creates a new kitty, creates two new hats and
// puts the hats on the cat. Then it stores the kitty in account storage.
transaction {
  prepare(acct: AuthAccount) {

    // Create the Kitty object
    let kitty <- KittyVerse.createKitty()

    // Create the KittyHat objects
    let hat1 <- KittyVerse.createHat(id: 1, name: "Cowboy Hat")
    let hat2 <- KittyVerse.createHat(id: 2, name: "Top Hat")

    // Put the hat on the cat!
    let oldCowboyHat <- kitty.items["Cowboy Hat"] <- hat1
    destroy oldCowboyHat
    let oldTopHat <- kitty.items["Top Hat"] <- hat2
    destroy oldTopHat

    log("The cat has the hats")

    // Store the Kitty in storage
    acct.save<@KittyVerse.Kitty>(<-kitty, to: /storage/Kitty)
  }
}
```

您应该会看到如下所示的输出：

```
> "The Cat has the Hats"
```

现在我们可以运行一个事务来移动小猫和它的帽子，从小猫上取下牛仔帽，然后让小猫摘掉它的帽子。

> 打开`Transaction2.cdc`。
>
> 选择账户 `0x01` 作为唯一的签名者，单击“发送”按钮发送交易。
>
> `Transaction2.cdc` 应包含以下代码：

`Transaction2.cdc`

```Cadence
import KittyVerse from 0x01

// This transaction moves a kitty out of storage,
// takes the cowboy hat off of the kitty,
// calls its tip hat function, and then moves it back into storage.
transaction {
  prepare(acct: AuthAccount) {

    // Move the Kitty out of storage, which also moves its hat along with it
    let kitty <- acct.load<@KittyVerse.Kitty>(from: /storage/Kitty)!

    // Take the cowboy hat off the Kitty
    let cowboyHat <- kitty.items.remove(key: "Cowboy Hat")!

    // Tip the cowboy hat
    log(cowboyHat.tipHat())
    destroy cowboyHat

    // Tip the top hat that is on the Kitty
    log(kitty.items["Top Hat"]?.tipHat())

    // Move the Kitty to storage, which
    // also moves its hat along with it.
    acct.save<@KittyVerse.Kitty>(<-kitty, to: /storage/Kitty)
  }
}
```

您应该看到类似这样的输出：

```Cadence
> "Howdy Y'all"
> "Greetings, fellow aristocats!"
```

每当 Kitty 被移动时，它的帽子也隐含地随之移动。这是因为帽子是小猫所有的。





### **未来是喵喵的！可扩展性时代即将到来！**

---

以上是复合资源的一个简单示例。我们必须明确地说明了Kitty猫戴着它的帽子，但是在不远的未来，Cadence将会支持更加强大的资源可扩展性，开发人员可以声明单独资源可以拥有的类型，即使拥有的资源从一开始就没有指定所有权的可能性。这是一个非常复杂的问题，需要以安全的方式解决，Flow 社区正在非常努力地为此设计解决方案，但它即将到来。

练习您在Flow开发者平台学习到的知识！

