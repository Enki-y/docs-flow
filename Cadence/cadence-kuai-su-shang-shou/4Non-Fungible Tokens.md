## **4.非同质化代币（Non-Fungible Tokens）**

---

在本教程中，我们将部署、存储和传输**非同质化代币 (NFT)**。

---

>在 Flow开发者平台中打开本教程的入门代码：
>
>https://play.onflow.org/b60aa97d-d030-476e-9490-24f3a6f90161
>
>本教程将要求您采取操作来与此代码进行交互。

>需要您采取行动的说明总是包含在这样的标注框中。
>
>这些突出显示的操作是让代码运行所需的全部操作，但阅读其余部分对于理解语言的设计是必要的。

NFT 是区块链技术不可或缺的一部分。我们需要 NFT 来表示独特且不可分割的资产（例如 CryptoKitties！或 Top Shot Moments！）。

Cadence 不像大多数智能合约语言那样在中央分布式帐本中表示，而是将每个 NFT 表示为用户存储在帐户中的资源对象。

我们将带您完成以下步骤以熟悉 NFT：

1. 部署 NFT 合约和类型定义。
2. 创建 NFT 对象并将其存储在您的帐户存储中。
3. 创建一个 NFT 集合对象以在您的账户中存储多个 NFT。
4. 创建一个 `NFTMinter` 并使用它来创建 NFT。
5. 创建对您的集合的引用，其他人可以使用它来向您发送代币。
6. 以同样的方式设置另一个帐户。
7. 将 NFT 从一个账户转移到另一个账户。
8. 使用脚本查看每个帐户的集合中存储了哪些 NFT。

**在继续本教程之前，**我们强烈建议您按照入门、Hello, World! 和 Fungible Tokens 中的说明学习如何使用 开发者平台工具并学习 Cadence 的基础知识。我们将在这里再次介绍一些概念，但不是全部。



### **在Flow模拟器上的非同质化代币**

---

在 Cadence 中，每个 NFT 都由一个带有整数 ID 的资源表示。资源是表示 NFT 的完美类型，因为资源具有所有权规则，并且由类型系统执行。它们只能有一个所有者，不能被复制，也不能被意外或恶意丢失或复制。这些保护措施确保所有者知道他们的 NFT 是安全的，并且可以代表具有实际价值的资产。

NFT 通常也由某种元数据表示，例如名称或图片。从历史上看，这些元数据的大部分都存储在链外，链上代币仅包含指向链外元数据的 URL 或类似内容。在 Flow 中，这是可能的，与代币相关的所有元数据都可以存储在链上。这超出了该教程涉及的范围。这类标准仍然逐步改善并且您也可以查看在flow NFT repository[相关问题](https://github.com/onflow/flow-nft/issues/9)参加讨论。

当flow用户想要与他人进行交易，他们可以调用在每个账户下资源定义的方法，无需与中央NFT合约交互，去实现点对点的交易。



### **添加 NFT 到您的帐户**

---

>首先，您需要按照此链接打开带有预加载的非同质化代币合约、交易和脚本的开发者平台：
>
>https://play.onflow.org/b60aa97d-d030-476e-9490-24f3a6f90161

>打开账户 `0x01` 以查看 `NFTv1.cdc`。 `NFTv1.cdc` 应包含以下代码：

`NFTv1.cdc`

```Cadence
pub contract ExampleNFT {

    // Declare the NFT resource type
    pub resource NFT {
        // The unique ID that differentiates each NFT
        pub let id: UInt64

        // String mapping to hold metadata
        pub var metadata: {String: String}

        // Initialize both fields in the init function
        init(initID: UInt64) {
            self.id = initID
            self.metadata = {}
        }
    }

    // Create a single new NFT and save it to account storage
    init() {
        self.account.save<@NFT>(<-create NFT(initID: 1), to: /storage/NFT1)
    }
}
```

在这个合约中，NFT 是一个具有整数 ID 和元数据字段的资源。

这与同质化代币不同，因为同质化代币只是一个金库，当代币被提取或存入时，其余额可能会发生变化。每个 NFT 资源都有唯一的 ID，因此它们不能组合或复制，除非智能合约允许。

这种设计的另一个独特之处是每个 NFT 都可以包含自己的元数据。在这个例子中，我们使用了一个简单的字符串（`String`）到字符串（`String`）的映射，但您可以想象一个更丰富的版本，它可以允许存储复杂的文件格式和其他此类数据。

一个 NFT 甚至可以拥有其他 NFT！此示例在后面的教程中显示。

在合约的 `init` 函数中，我们创建一个新的 NFT 对象并将其移动到帐户存储中。

```
// put it in storage
self.account.save<@NFT>(<-create NFT(initID: 1), to: /storage/NFT1)
```

在这里，我们合约部署到的帐户上的 `AuthAccount` 对象并调用其 `save` 方法，指定 `@NFT` 作为其保存的类型。我们还在同一行中创建了 NFT，并将其作为第一个参数传递给 `save`。我们将它保存到 `/storage` 域名中，在那里存储对象。

> 单击编辑器右下角的部署按钮部署 `NFTv1`。

您现在应该在您的帐户中有一个 NFT。让我们运行一个交易去检查。

> 打开 `NFT Exists` 交易，选择账户 `0x01` 作为唯一签名者，并发送交易。
>
> `NFT Exists` 应如下所示：

`NFT`

```
import ExampleNFT from 0x01

// This transaction checks if an NFT exists in the storage of the given account
// by trying to borrow from it
transaction {
    prepare(acct: AuthAccount) {
        if acct.borrow<&ExampleNFT.NFT>(from: /storage/NFT1) != nil {
            log("The token exists!")
        } else {
            log("No token found!")
        }
    }
}
```

在这里，我们试图直接从存储中的 NFT 借用引用。如果对象存在，则借用成功，备选引用不会为 `nil`，但如果借用失败，则可选引用为 `nil`。

您应该会看到`“The token exists!”`的内容。

干得好！您的账户中有第一个 NFT。



### **在一个集合中存储多个 NFT**

---

我们可以将我们的 NFT 放在存储的顶层，但是如果您有很多 NFT，那么组织所有 NFT 可能会开始变得混乱。这种方法不具有可扩展性，但我们可以通过使用可以容纳任意数量 NFT 的数据结构来克服这个问题。我们可以通过数组或字典来实现这一点，但这些类型相对不透明。相反，我们可以使用资源作为我们的 NFT 集合，以启用更复杂的方式与我们的 NFT 交互。

>打开账户`0x22`可以看到`NFTv2.cdc`。
>
>单击编辑器右下角的 Deploy 按钮部署合约。
>
>`NFTv2.cdc` 应包含以下代码。它包含 `NFTv1.cdc` 中已有的内容以及合约正文中的其他资源声明。

`NFTv2.cdc`

```Cadence
// NFTv2.cdc
//
// This is a complete version of the ExampleNFT contract
// that includes withdraw and deposit functionality, as well as a
// collection resource that can be used to bundle NFTs together.
//
// It also includes a definition for the Minter resource,
// which can be used by admins to mint new NFTs.
//
// Learn more about non-fungible tokens in this tutorial: https://docs.onflow.org/docs/non-fungible-tokens

pub contract ExampleNFT {

    // Declare the NFT resource type
    pub resource NFT {
        // The unique ID that differentiates each NFT
        pub let id: UInt64

        // Initialize the field in the init function
        init(initID: UInt64) {
            self.id = initID
        }
    }

    // We define this interface purely as a way to allow users
    // to create public, restricted references to their NFT Collection.
    // They would use this to only expose the deposit, getIDs,
    // and idExists fields in their Collection
    pub resource interface NFTReceiver {

        pub fun deposit(token: @NFT)

        pub fun getIDs(): [UInt64]

        pub fun idExists(id: UInt64): Bool
    }

    // The definition of the Collection resource that
    // holds the NFTs that a user owns
    pub resource Collection: NFTReceiver {
        // dictionary of NFT conforming tokens
        // NFT is a resource type with an `UInt64` ID field
        pub var ownedNFTs: @{UInt64: NFT}

        // Initialize the NFTs field to an empty collection
        init () {
            self.ownedNFTs <- {}
        }

        // withdraw 
        //
        // Function that removes an NFT from the collection 
        // and moves it to the calling context
        pub fun withdraw(withdrawID: UInt64): @NFT {
            // If the NFT isn't found, the transaction panics and reverts
            let token <- self.ownedNFTs.remove(key: withdrawID)!

            return <-token
        }

        // deposit 
        //
        // Function that takes a NFT as an argument and 
        // adds it to the collections dictionary
        pub fun deposit(token: @NFT) {
            // add the new token to the dictionary with a force assignment
            // if there is already a value at that key, it will fail and revert
            self.ownedNFTs[token.id] <-! token
        }

        // idExists checks to see if a NFT 
        // with the given ID exists in the collection
        pub fun idExists(id: UInt64): Bool {
            return self.ownedNFTs[id] != nil
        }

        // getIDs returns an array of the IDs that are in the collection
        pub fun getIDs(): [UInt64] {
            return self.ownedNFTs.keys
        }

        destroy() {
            destroy self.ownedNFTs
        }
    }

    // creates a new empty Collection resource and returns it 
    pub fun createEmptyCollection(): @Collection {
        return <- create Collection()
    }

    // NFTMinter
    //
    // Resource that would be owned by an admin or by a smart contract 
    // that allows them to mint new NFTs when needed
    pub resource NFTMinter {

        // the ID that is used to mint NFTs
        // it is only incremented so that NFT ids remain
        // unique. It also keeps track of the total number of NFTs
        // in existence
        pub var idCount: UInt64

        init() {
            self.idCount = 1
        }

        // mintNFT 
        //
        // Function that mints a new NFT with a new ID
        // and returns it to the caller
        pub fun mintNFT(): @NFT {

            // create a new NFT
            var newNFT <- create NFT(initID: self.idCount)

            // change the id so that each ID is unique
            self.idCount = self.idCount + 1 as UInt64
            
            return <-newNFT
        }
    }

	init() {
		// store an empty NFT Collection in account storage
        self.account.save(<-self.createEmptyCollection(), to: /storage/NFTCollection)

        // publish a reference to the Collection in storage
        self.account.link<&{NFTReceiver}>(/public/NFTReceiver, target: /storage/NFTCollection)

        // store a minter resource in account storage
        self.account.save(<-create NFTMinter(), to: /storage/NFTMinter)
	}
}
```

拥有一个或多个 `ExampleNFT` 的任何用户都将在其帐户中存储 `ExampleNFT.Collection` 资源的实例。该集合将所有 `NFT` 存储在一个字典中，该字典将整数 ID 映射到 NFT，类似于 `Vault` 资源在 `balance` 字段中存储所有代币的方式。另一个相似之处是每个`collection`都有存款`withdraw`和取款`deposit`函数。这些功能允许用户遵循同一个模式：即先从`collection`取走代币，然后存到其他的账户的`collection`。

当用户想要在他们账户存储NFTs，他们会在`ExampleNFT`智能合约中调用`createEmptyCollection`函数实例化一个空的`Collection`。这将返回一个空的 `Collection` 对象，他们可以将其存储在他们的帐户存储中。

我们在此示例中使用了一些新功能，因此让我们逐一介绍它们。



### **字典**

---

该资源使用了[字典：一个可变的、无序的键值关联集合](https://docs.onflow.org/cadence/language/values-and-types/#dictionaries)。

```
pub var ownedNFTs: @{Int: NFT}
```

在字典中，所有键必须具有相同的类型，并且所有值必须具有相同的类型。在这种情况下，我们将整数 ID 映射到 `NFT` 资源对象。字典定义中通常不用 `@`符号来进行类型检查，是因为`ownedNFTs`映射存储资源，整个字段也必须成为资源类型，这就是为什么该字段有 `@` 符号表示它是一种资源类型。这意味着适用于资源的所有规则都适用于这种类型。

如果使用 `destroy` 命令销毁 NFT 集合(collection)资源，它需要知道如何处理它存储在字典中的资源。这也解释了在调用`destroy`函数时，要销毁的该资源内部必须包含一个`destroy`函数以供运行。这个`destroy`函数必须显式销毁资源或将它们移动到其他地方。在这个例子中，我们销毁它们。

```Cadence
destroy() {
    destroy self.ownedNFTs
}
```

当集合被创建时，`init`构造函数运行并且显式初始化所有成员变量。这有助于防止某些智能合约中出现未初始化字段可能导致错误的问题。在此之后，`init`构造函数将永远无法再次运行。在这里，我们使用空字典将字典初始化为资源类型。

```
init () {
  self.ownedNFTs <- {}
}
```

字典的另一个特性是能够使用内置的`keys`键函数获取字典键的数组。

```Cadence
// getIDs returns an array of the IDs that are in the collection
pub fun getIDs(): [UInt64] {
    return self.ownedNFTs.keys
}
```

这用于遍历字典或仅查看存储内容的列表。如您所见，可变长度数组类型是通过将成员类型括在方括号中来声明的。



### **资源可以拥有其他资源**

---

`NFTv2.cdc` 中的这个 NFT Collection 示例说明了一个重要特性：资源可以拥有其他资源。

在示例中，用户可以将一个 NFT 转移给另一个用户。此外，由于集合`Collection`明确拥有其中的 NFT，所有者只需转移单个集合`Collection`即可一次转移所有 NFT。

这是一个很重要的特性，因为它支持许多其他用例。除了允许轻松的批量传输之外，这意味着如果一个独特的 NFT 想要拥有另一个独特的 NFT，例如拥有帽子配件的 CryptoKitty，Kitty 会将帽子存储在自己的存储中并有效地拥有它。帽子属于存放它的 CryptoKitty，帽子可以单独转让，也可以与拥有它的 CryptoKitty 一起转让。

无法为存储在其他资源中的资源创建功能定位符，但可以为引用创建功能定位符。属于自己的资源可以控制功能定位符，因此也能控制外部调用对存储资源的访问类型。



### **限制对 NFT 集合的访问**

---

在NFT集合中，所有的函数和字段都是公有的，但是我们又不想让每一个人在网上都可以调用我们的`withdraw`函数。这也是加入Cadence的双层访问控制的来由。Cadence使用功能定位符提供的安全性，这意味着任何给定的对象，如果用户满足以下任一条件，则允许访问该对象的字段或方法：

+ 是对象的所有者
+ 拥有对该字段或方法的有效引用（注意，引用只能从功能定位符创建，功能定位符只能由对象的所有者创建）

当用户想要在账户存储他们的NFT`Collection`时，默认情况下不是对外开放的。除了所有者之外，任何人都无法访问用户的帐户存储对象。为了给外部账户对`deposit`存款函数、`getIDs` 函数和 `idExists` 函数的读取访问权限，所有者将使用仅包含这些字段的接口创建引用：

```Cadence
pub resource interface NFTReceiver {

    pub fun deposit(token: @NFT)

    pub fun getIDs(): [UInt64]

    pub fun idExists(id: UInt64): Bool
}
```

然后，使用该接口，他们将创建一个指向存储中对象的链接，指定该链接仅包含 NFTReceiver 接口中的函数。这个链接就是功能定位符。从功能定位符那里，所有者可以使用该功能做任何他们想做的事情：他们可以将其作为参数传递给一次性使用的函数，或者他们可以将其放入其帐户的 `/public/` 域名中，以便任何人都可以访问它。如果用户尝试使用此功能来调用`withdraw`提款函数，它将无法工作，因为它不存在于接口或引用中。

在 `init`构造函数中 `NFTv2.cdc` 的第 132 行中可以看到链接和功能定位符的创建：

```Cadence
// publish a reference to the Collection in storage
self.account.link<&{NFTReceiver}>(/public/NFTReceiver, target: /storage/NFTCollection)
```

上面的`link`函数指定了功能定位符类型为`&AnyResource{NFTReceiver}`只对公开这些字段和函数。然后链接存储在任何人都可以访问的 `/public/` 中。该链接针对我们之前创建的 `/storage/NFTCollection`。

现在，用户在他们的帐户`/storage`中拥有一个 NFT 集合，以及其他人可以使用它来查看他们拥有哪些 NFT 并向他们发送 NFT 。

让我们通过运行脚本来确认这是真的！



### **运行脚本**

---

Cadence 中的脚本是简单的交易，无需任何帐户权限即可运行，仅从区块链读取信息。

> 打开名为 `Print 0x02 NFTs` 的脚本文件。`Print 0x02 NFT` 应包含以下代码：

```Cadence
import ExampleNFT from 0x02

// Print the NFTs owned by account 0x02.
pub fun main() {
    // Get the public account object for account 0x02
    let nftOwner = getAccount(0x02)

    // Find the public Receiver capability for their Collection
    let capability = nftOwner.getCapability<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver)

    // borrow a reference from the capability
    let receiverRef = capability.borrow()
            ?? panic("Could not borrow receiver reference")

    // Log the NFTs that they own as an array of IDs
    log("Account 2 NFTs")
    log(receiverRef.getIDs())
}
```

>单击编辑器框右下角的“执行”按钮，执行打印 `0x02 NFT`。
>
>此脚本打印帐户 `0x02` 拥有的 NFT 列表。

因为账户 `0x02` 当前在其集合中没有任何NFT集合，它只会打印一个空数组：

```
"Account 2 NFTs"
[]
Result > "void"
```

如果脚本无法执行，则可能意味着 NFT 集合未正确存储在帐户 `0x02` 中。如果您遇到问题，请确保您选择帐户 `0x02` 作为签名者帐户，并且您正确执行了前面的步骤。



### **以管理员身份铸造和分发代币**

---

创建 NFT 的一种方法是让管理员铸造新代币并将其发送给用户。大多数人会通过拥有 NFT Minter 资源来实现这一点。此资源的所有者可以铸造代币，或者如果他们想赋予其他用户和合约铸造代币的能力，则所有者可以提供仅公开 `mintNFT` 功能以利用功能定位符安全性。无需像基于分布式帐本模型那样明确检查交易的发送者！

让我们使用 NFT Minter 来铸造一些代币。

如果你回顾 `NFTv2.cdc` 中的 `ExampleNFT` 合约，你会看到它定义了另一个资源，`NFTMinter`。这是一个简单的示例，说明具有铸造权限的管理员将拥有铸造新 NFT 的资产。这只是一个单一的函数来创建 NFT 并且只包含一个递增的整数字段，用于为 NFT 分配唯一的 ID。

您应该看到在帐户 `0x02` 的帐户存储中存储了一个 `ExampleNFT.NFTMinter` 资源。这显示在编辑器下方的`Resources`框中。

现在我们可以使用我们存储的 `NFTMinter` 来铸造一个新的 NFT 并将其存入账户 `0x02` 的集合中。

>打开名为 `Mint NFT` 的文件。选择账户 `0x02` 作为唯一的签名者并发送交易。
>
>该交易将铸造的 NFT 存入账户所有者的 NFT 集合中：

`Mint`

```Cadence
import ExampleNFT from 0x02

// This transaction allows the Minter account to mint an NFT
// and deposit it into its collection.

transaction {

    // The reference to the collection that will be receiving the NFT
    let receiverRef: &{ExampleNFT.NFTReceiver}

    // The reference to the Minter resource stored in account storage
    let minterRef: &ExampleNFT.NFTMinter

    prepare(acct: AuthAccount) {
        // Get the owner's collection capability and borrow a reference
        self.receiverRef = acct.getCapability<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver)
            .borrow()
            ?? panic("Could not borrow receiver reference")

        // Borrow a capability for the NFTMinter in storage
        self.minterRef = acct.borrow<&ExampleNFT.NFTMinter>(from: /storage/NFTMinter)
            ?? panic("Could not borrow minter reference")
    }

    execute {
        // Use the minter reference to mint an NFT, which deposits
        // the NFT into the collection that is sent as a parameter.
        let newNFT <- self.minterRef.mintNFT()

        self.receiverRef.deposit(token: <-newNFT)

        log("NFT Minted and deposited to Account 2's Collection")
    }
}
```

>重新打开 `Print 0x02 NFT` 并执行脚本。这将打印帐户 `0x02` 拥有的 NFT 列表。

`Print`

```Cadence
import ExampleNFT from 0x02

// Print the NFTs owned by account 0x02.
pub fun main() {
    // Get the public account object for account 0x02
    let nftOwner = getAccount(0x02)

    // Find the public Receiver capability for their Collection
    let capability = nftOwner.getCapability<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver)

    // borrow a reference from the capability
    let receiverRef = capability.borrow()
            ?? panic("Could not borrow the receiver reference")

    // Log the NFTs that they own as an array of IDs
    log("Account 2 NFTs")
    log(receiverRef.getIDs())
}

```

您应该看到帐户 `0x02` 拥有 `id=1` 的 NFT

```
"Account 2 NFTs"
[1]
```



### **转移 NFT**

---

在我们能够将 NFT 转移到另一个帐户之前，我们需要使用他们自己的 NFTCollection 设置该帐户，以便他们能够接收 NFT。

> 打开名为 `Setup Account` 的文件并提交交易，使用账户 `0x01` 作为唯一的签名者。

`Setup`

```Cadence
import ExampleNFT from 0x02

// This transaction configures a user's account
// to use the NFT contract by creating a new empty collection,
// storing it in their account storage, and publishing a capability
transaction {
  prepare(acct: AuthAccount) {

    // Create a new empty collection
    let collection <- ExampleNFT.createEmptyCollection()

    // store the empty NFT Collection in account storage
    acct.save<@ExampleNFT.Collection>(<-collection, to: /storage/NFTCollection)

    log("Collection created for account 1")

    // create a public capability for the Collection
    acct.link<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver, target: /storage/NFTCollection)

    log("Capability created")
  }
}
```

帐户 `0x01` 现在应该有一个空的`Collection`集合资源存储在其帐户存储中。它还在其 `/public/` 域名中创建并存储了集合的功能定位符。

>打开名为 `Transfer` 的文件，选择账户 `0x02` 作为唯一签名者，然后发送交易。
>
>此交易将代币从帐户 `0x02` 转移到帐户 `0x01`。

`Transfer.cdc`

```Cadence
import ExampleNFT from 0x02

// This transaction transfers an NFT from one user's collection
// to another user's collection.
transaction {

    // The field that will hold the NFT as it is being
    // transferred to the other account
    let transferToken: @ExampleNFT.NFT

    prepare(acct: AuthAccount) {

        // Borrow a reference from the stored collection
        let collectionRef = acct.borrow<&ExampleNFT.Collection>(from: /storage/NFTCollection)
            ?? panic("Could not borrow a reference to the owner's collection")

        // Call the withdraw function on the sender's Collection
        // to move the NFT out of the collection
        self.transferToken <- collectionRef.withdraw(withdrawID: 1)
    }

    execute {
        // Get the recipient's public account object
        let recipient = getAccount(0x01)

        // Get the Collection reference for the receiver
        // getting the public capability and borrowing a reference from it
        let receiverRef = recipient.getCapability<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver)
            .borrow()
            ?? panic("Could not borrow receiver reference")

        // Deposit the NFT in the receivers collection
        receiverRef.deposit(token: <-self.transferToken)

        log("NFT ID 1 transferred from account 2 to account 1")
    }
}
```

现在我们可以检查两个帐户的集合以确保帐户 `0x01` 拥有令牌而帐户 `0x02` 没有任何东西。

> 执行脚本`Print all NFTs`以查看每个帐户中的代币：

`Script2.cdc`

```Cadence
import ExampleNFT from 0x02

// Print the NFTs owned by accounts 0x01 and 0x02.
pub fun main() {

    // Get both public account objects
    let account1 = getAccount(0x01)
      let account2 = getAccount(0x02)

    // Find the public Receiver capability for their Collections
    let acct1Capability = account1.getCapability<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver)
    let acct2Capability = account2.getCapability<&{ExampleNFT.NFTReceiver}>(/public/NFTReceiver)

    // borrow references from the capabilities
    let receiver1Ref = acct1Capability.borrow()
        ?? panic("Could not borrow account 1 receiver reference")
    let receiver2Ref = acct2Capability.borrow()
        ?? panic("Could not borrow account 2 receiver reference")

    // Print both collections as arrays of IDs
    log("Account 1 NFTs")
    log(receiver1Ref.getIDs())

    log("Account 2 NFTs")
    log(receiver2Ref.getIDs())
}
```

您应该在输出中看到类似的内容：

```
"Account 1 NFTs"
[1]
"Account 2 NFTs"
[]
```

账户 `0x01` 有一个 `ID=1` 的 NFT，账户 `0x02` 没有。这表明 NFT 从账户 `0x02` 转移到账户 `0x01`。

恭喜，您现在拥有一个有效的 NFT！



### **把NFT都放在一起**

---

这只是 NFT 如何在 Flow 上工作的一个基本示例。有关官方 [Flow NFT 标准](https://github.com/onflow/flow-nft)及其示例实现的信息，请参阅 Flow NFT 标准存储库。



### **创建Flow市场**

---

现在您有了一个可用的 NFT，您可以尝试自行扩展其功能，或者您可以学习如何创建一个同时使用可替代代币和 NFT 的市场。

