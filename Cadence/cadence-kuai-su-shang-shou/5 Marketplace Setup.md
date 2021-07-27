## **5.安装市场**

---

>在 Flow Playground 中打开本教程的入门代码：
>
>https://play.onflow.org/46ad1d6d-3ee2-40d4-adef-bfbad33f9846
>
>本教程将要求您采取各种操作来与此代码进行交互。

如果您已经完成了 Marketplace 教程，请转到[复合资源：Kitty Hats](https://docs.onflow.org/cadence/tutorial/07-resources-compose/)。

本指南将帮助您快速将开发者平台设置为完成（Marketplace）市场教程所需的知识。市场教程使用同质化代币和非同质化代币合约来允许用户使用同质化代币买卖 NFT。

帐户的操作与您在开发者平台中完成同质化代币和非同质化代币教程一样。需要您继续跟着[复合智能合约：市场](https://docs.onflow.org/cadence/tutorial/06-marketplace-compose/)来学习。

1. 打开账户 `0x01`。确保同质化代币教程中 `FungibleToken.cdc` 中的同质化代币定义在此帐户中。
2. 将 Fungible Token代码部署到账户 `0x01`。
3. 通过从帐户选择菜单中选择帐户 `0x02` 切换到帐户 `0x02`。
4. 确保您在帐户 `0x02` 中的非同质化代币教程中的 `NFTv2.cdc` 中有 NFT 定义。
5. 将 NFT 代码部署到账户 `0x02`。
6. 在事务标签栏运行事务 3。这是名为 `SetupAccount1Transaction.cdc` 文件。使用帐户 `0x01` 作为设置帐户 `0x01` 帐户的唯一签名者。

`SetupAccount1Transaction.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02

// This transaction sets up account 0x01 for the marketplace tutorial
// by publishing a Vault reference and creating an empty NFT Collection.
transaction {
  prepare(acct: AuthAccount) {
    // Create a public Receiver capability to the Vault
    acct.link<&FungibleToken.Vault{FungibleToken.Receiver, FungibleToken.Balance}>
             (/public/MainReceiver, target: /storage/MainVault)

    log("Created Vault references")

    // store an empty NFT Collection in account storage
    acct.save<@NonFungibleToken.Collection>(<-NonFungibleToken.createEmptyCollection(), to: /storage/NFTCollection)

    // publish a capability to the Collection in storage
    acct.link<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver, target: /storage/NFTCollection)

    log("Created a new empty collection and published a reference")
  }
}
```

7. 在事务标签栏运行事务4 。这是 `SetupAccount2Transaction.cdc` 文件。使用帐户 `0x02` 作为设置帐户 `0x02` 帐户的唯一签名者。

`SetupAccount2Transaction.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02

// This transaction adds an empty Vault to account 0x02
// and mints an NFT with id=1 that is deposited into
// the NFT collection on account 0x01.
transaction {

  // Private reference to this account's minter resource
  let minterRef: &NonFungibleToken.NFTMinter

  prepare(acct: AuthAccount) {
    // create a new vault instance with an initial balance of 30
    let vaultA <- FungibleToken.createEmptyVault()

    // Store the vault in the account storage
    acct.save<@FungibleToken.Vault>(<-vaultA, to: /storage/MainVault)

    // Create a public Receiver capability to the Vault
    let ReceiverRef = acct.link<&FungibleToken.Vault{FungibleToken.Receiver, FungibleToken.Balance}>(/public/MainReceiver, target: /storage/MainVault)

    log("Created a Vault and published a reference")

    // Borrow a reference for the NFTMinter in storage
    self.minterRef = acct.borrow<&NonFungibleToken.NFTMinter>(from: /storage/NFTMinter)
            ?? panic("Could not borrow an nft minter reference")
  }
  execute {
    // Get the recipient's public account object
    let recipient = getAccount(0x01)

    // Get the Collection reference for the receiver
    // getting the public capability and borrowing a reference from it
    let receiverRef = recipient.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)
        .borrow()
        ?? panic("Could not borrow an nft receiver reference")

    // Mint an NFT and deposit it into account 0x01's collection
    self.minterRef.mintNFT(recipient: receiverRef)

    log("New NFT minted for account 1")
  }
}
```

8. 在事务标签栏运行事务 5 。这是名为 `SetupAccount1TransactionMinting.cdc` 文件。使用账户 `0x01` 作为唯一的签名者为账户 1 和 2 铸造同质化代币。

`SetupAccount1TransactionMinting.cdc`

```Cadence
import FungibleToken from 0x01
import NonFungibleToken from 0x02

// This transaction mints tokens for both accounts using
// the minter stored on account 0x01.
transaction {

    // Public Vault Receiver References for both accounts
    let acct1Ref: &AnyResource{FungibleToken.Receiver}
    let acct2Ref: &AnyResource{FungibleToken.Receiver}

    // Private minter references for this account to mint tokens
    let minterRef: &FungibleToken.VaultMinter

    prepare(acct: AuthAccount) {
        // Get the public object for account 0x02
        let account2 = getAccount(0x02)

        // Retrieve public Vault Receiver references for both accounts
        self.acct1Ref = acct.getCapability<&FungibleToken.Vault{FungibleToken.Receiver}>(/public/MainReceiver)
            .borrow()
            ?? panic("Could not borrow acct1 vault receiver reference")
        self.acct2Ref = account2.getCapability<&FungibleToken.Vault{FungibleToken.Receiver}>(/public/MainReceiver)
            .borrow()
            ?? panic("Could not borrow acct2 vault receiver reference")

        // Get the stored Minter reference for account 0x01
        self.minterRef = acct.borrow<&FungibleToken.VaultMinter>(from: /storage/MainMinter)
            ?? panic("Could not borrow vault minter reference")
    }

    execute {
        // Mint tokens for both accounts
        self.minterRef.mintTokens(amount: 20.0, recipient: self.acct2Ref)
        self.minterRef.mintTokens(amount: 10.0, recipient: self.acct1Ref)

        log("Minted new fungible tokens for account 1 and 2")
    }
}
```

9. 运行脚本 1 中的脚本 `CheckSetupScript.cdc` 文件以确保一切都已设置。

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
        .borrow<&FungibleToken.Vault{FungibleToken.Balance}>()
        ?? panic("Could not borrow acct1 vault receiver reference")
    let acct2ReceiverRef = acct2.getCapability<&FungibleToken.Vault{FungibleToken.Balance}>(/public/MainReceiver)
        .borrow()
        ?? panic("Could not borrow acct2 vault receiver reference")

    // Log the Vault balance of both accounts and ensure they are
    // the correct numbers.
    // Account 0x01 should have 40.
    // Account 0x02 should have 20.
    log("Account 1 Balance")
    log(acct1ReceiverRef.balance)
    log("Account 2 Balance")
    log(acct2ReceiverRef.balance)

    // verify that the balances are correct
    if acct1ReceiverRef.balance != 40.0 || acct2ReceiverRef.balance != 20.0 {
        panic("Wrong balances!")
    }

    // Find the public Receiver capability for their Collections
    let acct1Capability = acct1.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)
    let acct2Capability = acct2.getCapability<&{NonFungibleToken.NFTReceiver}>(/public/NFTReceiver)

    // borrow references from the capabilities
    let nft1Ref = acct1Capability.borrow()
        ?? panic("Could not borrow acct1 nft receiver reference")
    let nft2Ref = acct2Capability.borrow()
        ?? panic("Could not borrow acct2 nft receiver reference")

    // Print both collections as arrays of IDs
    log("Account 1 NFTs")
    log(nft1Ref.getIDs())

    log("Account 2 NFTs")
    log(nft2Ref.getIDs())

    // verify that the collections are correct
    if nft1Ref.getIDs()[0] != 1 as UInt64 || nft2Ref.getIDs().length != 0 {
        panic("Wrong Collections!")
    }
}
```

10. 如果没有脚本抛出panic异常，你应该看到类似这样的输出

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

现在您的开发者平台处于正确状态，您就可以继续学习下一个教程了。

您无需为市场教程打开新的开发者平台。你可以继续使用这个。

