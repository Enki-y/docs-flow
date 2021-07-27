## **6.市场**

---

在本教程中，我们将创建一个市场，使用我们在之前的教程中学到的同质化和非同质化的代币 (NFT) 合约。

---

> 在 Flow开发者平台中打开本教程的入门代码：
>
> https://play.onflow.org/46ad1d6d-3ee2-40d4-adef-bfbad33f9846
>
> 本教程将要求您采取各种操作来与此代码进行交互。

> 需要您采取行动的说明总是包含在这样的标注框中。
>
> 这些突出显示的操作是让代码运行所需的全部操作，但阅读其余部分对于理解语言的设计是必要的。

市场是区块链技术和智能合约最活跃的应用。市面有NFTs存在时，用户希望能够使用他们的同质化代币买卖。

现在有了同质化和非同质化代币的标准，我们可以建立一个同时使用两者的市场。这被称为**模块化**：开发人员能够利用共享资源（例如代码或用户库）并将它们用作新应用程序的构建块。Flow 旨在实现模块化，因为它使开发人员能够事半功倍，从而实现快速创新。

为了创建一个市场，我们需要将同质化和非同质化代币的功能整合到一个单一的合约中，让用户可以控制他们的资金和资产。为了实现这一点，我们将带您完成以下步骤来创建一个模块化的智能合约并应用到市场：

1. 确保您的可替代代币和不可替代代币合约已正确部署和设置。
2. 将市场类型声明部署到账户 `0x03`。
3. 创建一个市场对象并将其存储在您的帐户中，将 NFT 出售并为您的售卖发布公有功能定位符。
4. 使用不同的帐户从销售中购买 NFT。
5. 运行脚本以验证是否已购买 NFT。

**在继续本教程之前**，您需要完成 [同质化代币](https://docs.onflow.org/cadence/tutorial/03-fungible-tokens/)和 [非同质化代币](https://docs.onflow.org/cadence/tutorial/04-non-fungible-tokens/) 教程以了解此智能合约的代码块是如何工作的。



### **设计市场**

---

实现市场的一种方法是拥有一个中心智能合约，用户将他们的 NFT 和价格存入其中，并且让任何人过来并能够以该价格购买NFT。这种方法是合理的，它使过程集中。我们希望用户在尝试出售 NFT 的同时能够保持对他们出售的 NFT 的所有权。

与采取中心化方法不同，每个用户可以在他们自己账户上售卖物品。然后，用户可以提供一个引用挂在中心化应用市场去售卖自己的物品，或者如果他们希望整个交易保持在链上，则到智能合约中心市场（central sale aggregator smart contract）。通过这种方式，代币的所有者在其代币出售时保留其代币的保管权。

>在开始之前，我们需要确认您的帐户状态。
>
>如果您还没有，请执行上一教程中的步骤，以确保 Fungible Token 和 Non-Fungible Token 合约部署到账户 1 和 2 并拥有一些代币。您的帐户应如下所示：

>您可以运行 `CheckSetupScript.cdc` 脚本以确保您的帐户设置正确：

`CheckSetupScript.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02

// This script checks that the accounts are set up correctly for the marketplace tutorial.
//
// Account 0x01: Vault Balance = 40, NFT.id = 1
// Account 0x02: Vault Balance = 20, No NFTs
pub fun main() {
  // Get the accounts' public account objects
  let acct1 = getAccount(0x01)
  let acct2 = getAccount(0x02)

  // Get references to the account's receivers
  // by getting their public capability
  // and borrowing a reference from the capability
  let acct1ReceiverRef = acct1.getCapability<&FungibleToken.Vault{FungibleToken.Balance}>(/public/MainReceiver)
    .borrow()
    ?? panic("Could not borrow a reference to acc1 vault receiver")

  let acct2ReceiverRef = acct2.getCapability<&FungibleToken.Vault{FungibleToken.Balance}>(/public/MainReceiver)
    .borrow()
    ?? panic("Could not borrow a reference to acc2 vault receiver")

  // Log the Vault balance of both accounts and ensure they are
  // the correct numbers.
  // Account 0x01 should have 40.
  // Account 0x02 should have 20.
  log("Account 1 Balance")
  log(acct1ReceiverRef.balance)
  log("Account 2 Balance")
  log(acct2ReceiverRef.balance)

  // verify that the balances are correct
  if acct1ReceiverRef.balance != 40.0 || acct2ReceiverRef.balance != 40.0 {
      panic("Wrong balances!")
  }

  // Find the public Receiver capability for their Collections
  let acct1Capability = acct1.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)
  let acct2Capability = acct2.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)

  // borrow references from the capabilities
  let nft1Ref = acct1Capability.borrow()
        ?? panic("Could not borrow a reference to acc1 nft collection receiver")

  let nft2Ref = acct2Capability.borrow()
        ?? panic("Could not borrow a reference to acc2 nft collection receiver")

  // Print both collections as arrays of IDs
  log("Account 1 NFTs")
  log(nft1Ref.getIDs())

  log("Account 2 NFTs")
  log(nft2Ref.getIDs())

  // verify that the collections are correct
  if nft1Ref.getIDs()[0] != UInt64(1) || nft2Ref.getIDs().length != 0 {
      panic("Wrong Collections!")
  }
}
```

如果您的帐户设置正确，您应该会看到与此输出类似的内容。如果您连续遵循 Fungible Tokens 和 Non-Fungible Tokens 教程，它们将处于相同的状态：

```
"Account 1 Vault Balance"
40
"Account 2 Vault Balance"
20
"Account 1 NFTs"
[1]
"Account 2 NFTs"
[]
```

现在您的账户处于正确状态，我们可以建立一个市场，使账户之间的 NFT 售卖成为可能。





### **建立 NFT 市场**

---

每个想要出售 NFT 的用户都会在他们的帐户存储中存储一个 `SaleCollection` 资源的实例。

>切换到帐户 `0x03`。打开`Marketplace.cdc`
>
>打开 `Marketplace.cdc`，单击出现在编辑器右下角的 `Deploy` 按钮。 `Marketplace.cdc` 应包含以下合约定义：

`Marketplace.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02

// The Marketplace contract is a sample implementation of an NFT Marketplace on Flow.
//
// This contract allows users to put their NFTs up for sale. Other users
// can purchase these NFTs with fungible tokens.

pub contract Marketplace {

  // Event that is emitted when a new NFT is put up for sale
  pub event ForSale(id: UInt64, price: UFix64)

  // Event that is emitted when the price of an NFT changes
  pub event PriceChanged(id: UInt64, newPrice: UFix64)

  // Event that is emitted when a token is purchased
  pub event TokenPurchased(id: UInt64, price: UFix64)

  // Event that is emitted when a seller withdraws their NFT from the sale
  pub event SaleWithdrawn(id: UInt64)

  // Interface that users will publish for their Sale collection
  // that only exposes the methods that are supposed to be public
  //
  pub resource interface SalePublic {
    pub fun purchase(tokenID: UInt64, recipient: &AnyResource{NonFungibleToken.NFTReceiver}, buyTokens: @FungibleToken.Vault)
    pub fun idPrice(tokenID: UInt64): UFix64?
    pub fun getIDs(): [UInt64]
  }

  // SaleCollection
  //
  // NFT Collection object that allows a user to put their NFT up for sale
  // where others can send fungible tokens to purchase it
  //
  pub resource SaleCollection: SalePublic {

    // Dictionary of the NFTs that the user is putting up for sale
    pub var forSale: @{UInt64: NonFungibleToken.NFT}

    // Dictionary of the prices for each NFT by ID
    pub var prices: {UInt64: UFix64}

    // The fungible token vault of the owner of this sale.
    // When someone buys a token, this resource can deposit
    // tokens into their account.
    access(account) let ownerVault: Capability<&AnyResource{FungibleToken.Receiver}>

    init (vault: Capability<&AnyResource{FungibleToken.Receiver}>) {
        self.forSale <- {}
        self.ownerVault = vault
        self.prices = {}
    }

    // withdraw gives the owner the opportunity to remove a sale from the collection
    pub fun withdraw(tokenID: UInt64): @NonFungibleToken.NFT {
        // remove the price
        self.prices.remove(key: tokenID)
        // remove and return the token
        let token <- self.forSale.remove(key: tokenID) ?? panic("missing NFT")
        return <-token
    }

    // listForSale lists an NFT for sale in this collection
    pub fun listForSale(token: @NonFungibleToken.NFT, price: UFix64) {
        let id = token.id

        // store the price in the price array
        self.prices[id] = price

        // put the NFT into the the forSale dictionary
        let oldToken <- self.forSale[id] <- token
        destroy oldToken

        emit ForSale(id: id, price: price)
    }

    // changePrice changes the price of a token that is currently for sale
    pub fun changePrice(tokenID: UInt64, newPrice: UFix64) {
        self.prices[tokenID] = newPrice

        emit PriceChanged(id: tokenID, newPrice: newPrice)
    }

    // purchase lets a user send tokens to purchase an NFT that is for sale
    pub fun purchase(tokenID: UInt64, recipient: &AnyResource{NonFungibleToken.NFTReceiver}, buyTokens: @FungibleToken.Vault) {
        pre {
            self.forSale[tokenID] != nil && self.prices[tokenID] != nil:
                "No token matching this ID for sale!"
            buyTokens.balance >= (self.prices[tokenID] ?? 0.0):
                "Not enough tokens to by the NFT!"
        }

        // get the value out of the optional
        let price = self.prices[tokenID]!

        self.prices[tokenID] = nil

        let vaultRef = self.ownerVault.borrow()
            ?? panic("Could not borrow reference to owner token vault")
        
        // deposit the purchasing tokens into the owners vault
        vaultRef.deposit(from: <-buyTokens)

        // deposit the NFT into the buyers collection
        recipient.deposit(token: <-self.withdraw(tokenID: tokenID))

        emit TokenPurchased(id: tokenID, price: price)
    }

    // idPrice returns the price of a specific token in the sale
    pub fun idPrice(tokenID: UInt64): UFix64? {
        return self.prices[tokenID]
    }

    // getIDs returns an array of token IDs that are for sale
    pub fun getIDs(): [UInt64] {
        return self.forSale.keys
    }

    destroy() {
        destroy self.forSale
    }
  }

  // createCollection returns a new collection resource to the caller
  pub fun createSaleCollection(ownerVault: Capability<&AnyResource{FungibleToken.Receiver}>): @SaleCollection {
    return <- create SaleCollection(vault: ownerVault)
  }
}
```

该市场合约的资源功能与非同质化代币中解释的 NFT 集合`Collection`类似，但有一些差异和补充：

+ 该市场合约具有添加和删除 NFT 的方法，但这些功能还涉及设置和删除价格。当用户想要出售他们的 NFT 时，他们会使用 `listForSale` 函数将其存入收藏中。然后，另一个用户可以调用购买`purchase`方法，发送包含他们用于购买的货币的 `Vault`。买家还包括对他们的 NFT 收藏`Collection`的引用，以便购买的代币可以在购买时立即存入他们的收藏。

+ 这个市场合约存储了一个能力:

  `pub let ownerVault: Capability<&AnyResource{FungibleToken.Receiver}>`。售卖的所有者在销售中将功能定位符保存到他们的 `Fungible Token Receiver`。这使得售卖的资源能够在购买时立即将用于购买 NFT 的货币存入所有者钱包`Vault`。

+ 市场合同包括事件函数。 Cadence 支持在合约中定义事件，当重要动作发生时可以调用这些事件函数。

```Cadence
pub event ForSale(id: UInt64, price: UFix64)
pub event PriceChanged(id: UInt64, newPrice: UFix64)
pub event TokenPurchased(id: UInt64, price: UFix64)
pub event SaleWithdrawn(id: UInt64)
```

事件是通过访问级别(`pub`)、事件关键字(`event`)以及事件的名称和参数来声明的，就像函数声明一样。事件不能修改状态；它们表示智能合约中何时发生重要操作。事件是用`emit` 关键字发出的，然后是调用事件，就好像它是一个函数调用一样。外部应用程序可以监视区块链以在发出某些事件时采取行动。

此时，我们应该在两个帐户的存储中都有一个同质化代币钱包`Vault`和一个 NFT 集合`Collection`。帐户 `0x01` 的集合中应该有 NFT。

您可以按照以下步骤创建一个 `SaleCollection` 并列出账户 `0x01` 的代币进行售卖：

>打开`Transaction1.cdc`
>
>选择账户 `0x01` 作为唯一的签名者，然后点击发送`Send`按钮提交交易。

`Transaction1.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02
import Marketplace from 0x03

// This transaction creates a new Sale Collection object,
// lists an NFT for sale, puts it in account storage,
// and creates a public capability to the sale so that others can buy the token.
transaction {

  prepare(acct: AuthAccount) {

    // Borrow a reference to the stored Vault
    let receiver = acct.getCapability<&{FungibleToken.Receiver}>(/public/MainReceiver)

    // Create a new Sale object,
    // initializing it with the reference to the owner's vault
    let sale <- Marketplace.createSaleCollection(ownerVault: receiver)

    // borrow a reference to the NFTCollection in storage
    let collectionRef = acct.borrow<&NonFungibleToken.Collection>(from: /storage/NFTCollection)
            ?? panic("Could not borrow a reference to the owner's nft collection")

    // Withdraw the NFT from the collection that you want to sell
    // and move it into the transaction's context
    let token <- collectionRef.withdraw(withdrawID: 1)

    // List the token for sale by moving it into the sale object
    sale.listForSale(token: <-token, price: 10.0)

    // Store the sale object in the account storage
    acct.save<@Marketplace.SaleCollection>(<-sale, to: /storage/NFTSale)

    // Create a public capability to the sale so that others can call its methods
    acct.link<&Marketplace.SaleCollection{Marketplace.SalePublic}>(/public/NFTSale, target: /storage/NFTSale)

    log("Sale Created for account 1. Selling NFT 1 for 10 tokens")
  }
}
```

这份交易进行了如下步骤：

1. 获取所有者 `Vault` 的功能定位符。
2. 创建 `SaleCollection`，用于存储他们的 `Vault` 引用。
3. 从他们的钱包中提取所有者的代币。
4. 列出要出售的代币并设定其价格。
5. 存储在他们的帐户中，并公开一个功能定位符，允许其他人购买任何 NFT 进行售卖。

让我们运行一个脚本来确保正确创建了售卖。

>打开`Script2.cdc`
>
>点击 `Execute` 按钮，打印账户 `0x01` 所拥有的 NFT 的 ID 和价格sale.s

`Script2.cdc`

```Cadnece
import FungibleToken from 0x01
import NonFungibleToken from 0x02
import Marketplace from 0x03

// This script prints the NFTs that account 0x01 has for sale.
pub fun main() {
  // Get the public account object for account 0x01
  let account1 = getAccount(0x01)

  // Find the public Sale reference to their Collection
  let acct1saleRef = account1.getCapability<&AnyResource{Marketplace.SalePublic}>(/public/NFTSale)
        .borrow()
        ?? panic("Could not borrow a reference to the sale")

  // Los the NFTs that are for sale
  log("Account 1 NFTs for sale")
  log(acct1saleRef.getIDs())
  log("Price")
  log(acct1saleRef.idPrice(tokenID: 1))
}
```

此脚本执行完毕后并打印如下内容：

```
"Account 1 NFTs for sale"
[1]
"Price"
10
```





### **购买 NFT**

---

买家现在可以使用`Transaction2.cdc`交易来购买卖家提供的NFT。

>打开`Transaction2.cdc`文件
>
>选择帐户 `0x02` 作为唯一的签名者，然后单击`Send`按钮。

`Transaction2.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02
import Marketplace from 0x03

// This transaction uses the signer's Vault tokens to purchase an NFT
// from the Sale collection of account 0x01.
transaction {

  // reference to the buyer's NFT collection where they
  // will store the bought NFT
  let collectionRef: &AnyResource{NonFungibleToken.NFTReceiver}

  // Vault that will hold the tokens that will be used to
  // but the NFT
  let temporaryVault: @FungibleToken.Vault

  prepare(acct: AuthAccount) {

    // get the references to the buyer's fungible token Vault
    // and NFT Collection Receiver
    self.collectionRef = acct.borrow<&AnyResource{NonFungibleToken.NFTReceiver}>(from: /storage/NFTCollection)
        ?? panic("Could not borrow reference to the signer's nft collection")

    let vaultRef = acct.borrow<&FungibleToken.Vault>(from: /storage/MainVault)
        ?? panic("Could not borrow reference to the signer's vault")

    // withdraw tokens from the buyers Vault
    self.temporaryVault <- vaultRef.withdraw(amount: 10.0)
  }

  execute {
    // get the read-only account storage of the seller
    let seller = getAccount(0x01)

    // get the reference to the seller's sale
    let saleRef = seller.getCapability<&AnyResource{Marketplace.SalePublic}>(/public/NFTSale)
        .borrow()
        ?? panic("could not borrow reference to the seller's sale")

    // purchase the NFT the the seller is selling, giving them the reference
    // to your NFT collection and giving them the tokens to buy it
    saleRef.purchase(tokenID: 1,
        recipient: self.collectionRef,
        buyTokens: <-self.temporaryVault)

    log("Token 1 has been bought by account 2!")
  }
}

```

这笔交易：

1. 获取账户 `0x01` 的公有账户对象。
2. 获取对买家资源的引用。
3. 取出买方将用于购买 NFT 的代币。
4. 获取对卖家公开售卖的引用。
5. 调用`purchase`购买函数，传入代币和`Collection`收藏的引用。然后执行购买将买入的 NFT 直接存入买家的`Collection`收藏中。



### **验证 NFT交易是否正确执行**

---

您现在可以运行脚本来验证是否正确购买了 NFT，因为：

+ 账户 `0x01` 有 50 个代币，并且在他们的收藏和账户中没有任何 NFT 出售.
+ 账户 `0x02` 有 10 个代币和一个 id=1 的 NFT

要运行验证 NFT 购买是否正确的脚本，请执行以下步骤：

>打开“Script3.cdc”文件。
>
>点击`Execute`按钮`Script3.cdc`应该包含以下代码：

`Script3.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02
import Marketplace from 0x03

// This script checks that the Vault balances and NFT collections are correct
// for both accounts.
//
// Account 1: Vault balance = 50, No NFTs
// Account 2: Vault balance = 10, NFT ID=1
pub fun main() {
  // Get the accounts' public account objects
  let acct1 = getAccount(0x01)
  let acct2 = getAccount(0x02)

  // Get references to the account's receivers
  // by getting their public capability
  // and borrowing a reference from the capability
  let acct1ReceiverRef = acct1.getCapability<&FungibleToken.Vault{FungibleToken.Balance}>(/public/MainReceiver)
        .borrow()
        ?? panic("Could not borrow reference to acct1 vault")

  let acct2ReceiverRef = acct2.getCapability<&FungibleToken.Vault{FungibleToken.Balance}>(/public/MainReceiver)
        .borrow()
        ?? panic("Could not borrow reference to acct2 vault")

  // Log the Vault balance of both accounts and ensure they are
  // the correct numbers.
  // Account 0x01 should have 50.
  // Account 0x02 should have 10.
  log("Account 1 Balance")
  log(acct1ReceiverRef.balance)
  log("Account 2 Balance")
  log(acct2ReceiverRef.balance)

  // verify that the balances are correct
  if acct1ReceiverRef.balance != 50.0
     || acct2ReceiverRef.balance != 10.0
  {
      panic("Wrong balances!")
  }

  // Find the public Receiver capability for their Collections
  let acct1Capability = acct1.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)
  let acct2Capability = acct2.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)

  // borrow references from the capabilities
  let nft1Ref = acct1Capability.borrow()
    ?? panic("Could not borrow reference to acct1 nft collection")

  let nft2Ref = acct2Capability.borrow()
    ?? panic("Could not borrow reference to acct2 nft collection")

  // Print both collections as arrays of IDs
  log("Account 1 NFTs")
  log(nft1Ref.getIDs())

  log("Account 2 NFTs")
  log(nft2Ref.getIDs())

  // verify that the collections are correct
  if nft2Ref.getIDs()[0] != 1 as UInt64 || nft1Ref.getIDs().length != 0 {
      panic("Wrong Collections!")
  }

  // Get the public sale reference for Account 0x01
  let acct1SaleRef = acct1.getCapability<&AnyResource{Marketplace.SalePublic}>(/public/NFTSale)
        .borrow()
        ?? panic("Could not borrow a reference to the sale")

  // Print the NFTs that account 0x01 has for sale
  log("Account 1 NFTs for sale")
  log(acct1SaleRef.getIDs())
  if acct1SaleRef.getIDs().length != 0 { panic("Sale should be empty!") }
}
```

如果你做的一切都正确，交易应该会成功，它应该打印类似这样的东西：

```
"Account 1 Vault Balance"
50
"Account 2 Vault Balance"
10
"Account 1 NFTs"
[]
"Account 2 NFTs"
[1]
"Account 1 NFTs for Sale"
[]
```

恭喜您，您已经成功地在 Cadence 中实现了一个简单的市场，并使用它来允许一个账户从另一个账户购买 NFT！



### **扩展市场**

---

用户可以使用这些资源和交易在他们的帐户中进行售卖。对于中心市场的支持可以使用户相对便利地去操作和在原有的基础上建立服务。如果我们想在链上建立一个中央市场，我们可以使用如下所示的合约：

`CentralMarketplace.cdc`

```Cadence
// Marketplace would be the central contract where people can post their sale
// references so that anyone can access them
pub contract Marketplace {
    // Data structure to store active sales
    pub var tokensForSale: [Capability<&SaleCollection>]

    // listSaleCollection lists a users sale reference in the array
    // and returns the index of the sale so that users can know
    // how to remove it from the marketplace
    pub fun listSaleCollection(collection: Capability<&SaleCollection>): Int {
        self.tokensForSale.append(collection)
        return (self.tokensForSale.length - 1)
    }

    // removeSaleCollection removes a user's sale from the array
    // of sale references
    pub fun removeSaleCollection(index: Int) {
        self.tokensForSale.remove(at: index)
    }

}
```

该合约并不意味着是一个工作或生产环境下的合约，但它可以通过以下方式扩展为一个完整的中央市场：

+ 卖方在本合同中为其 `SaleCollection` 列出了一项功能定位符。
+ 买家可以调用其他功能函数以获取有关所有不同售卖的信息并进行购买。

链下应用程序中的中央市场更容易实现，因为：

+ 该应用程序可以托管市场，用户只需登录该应用程序并为该应用程序提供其帐户地址即可。
+ 该应用程序可以读取用户的公有存储并找到他们的售卖引用。
+ 通过售卖引用，该应用程序可以获得有关如何在其网站上显示销售情况的所有信息。
+ 任何买家都可以在应用程序中发现售卖信息并使用他们的帐户登录，这使应用程序可以访问他们的公开引用。
+ 当买家想要购买特定的 NFT 时，应用程序会自动生成正确的交易以从卖家那里购买 NFT。



### **为任何通用 NFT 创建市场**

---

前面的例子展示了如何为特定类别的 NFT 创建一个简单的市场。但是，用户希望有一个市场，无论其类型如何，他们都可以在其中买卖任何他们想要的 NFT。对此功能的支持正在开发中，很快就会分享。



### **Flow上的复合资源**

---

现在您已经了解了可组合智能合约和市场如何在 Flow 上工作，您已准备好使用复合资源！

