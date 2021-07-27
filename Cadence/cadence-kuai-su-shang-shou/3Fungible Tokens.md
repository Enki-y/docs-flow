## **3.同质化代币（Fungible Tokens）**

---

在这篇教程中，我们将部署、存储和转移同质化代币。

---

>在Flow开发者平台中打开本教程的入门代码：
>
>https://play.onflow.org/7c60f681-40c7-4a18-82de-b81b3b6c7915
>
>本教程将指导您采取各种操作来与此代码进行交互。

>需要您采取行动的说明总是包含在这样的标注框中。
>
>这些突出显示的操作是让代码运行所需的全部操作，但阅读其余部分对于理解语言的设计是必要的。

当今区块链上一些最受欢迎的合约类别是同质化代币。这些合约创建了同质化代币，可以转移给其他用户并作为货币消费（例如，以太坊上的 ERC-20）。

通常，中央分布式账本会跟踪用户的代币余额。通过 Cadence，我们使用新的资源范式来实现同质化的代币并避免使用中央分布式账本。



### **Flow网络代币**

---

在Flow，本地网络代币将使用类似于本教程中的智能合约作为普通的同质化代币智能合约来实现。将有特殊的合约和挂钩允许它用于交易执行支付和抵押，但除此之外，开发人员和用户将能够像对待网络中的任何其他代币一样对待和使用它！

我们将带您完成以下步骤，以熟悉同质化代币：

1. 将同质化代币合约部署到账户 `0x01`
2. 创建一个同质化代币对象并将其存储在您的帐户存储中。
3. 创建对您的代币的引用，其他人可以使用它来向您发送代币。
4. 以同样的方式设置另一个帐户。
5. 将代币从一个帐户转移到另一个帐户。
6. 使用脚本读取账户余额。

**在继续本教程之前，**我们建议您按照[Getting Started](https://docs.onflow.org/cadence/tutorial/01-first-steps/)和 [Hello, World!](https://docs.onflow.org/cadence/tutorial/02-hello-world/) 中的说明进行操作。以便学会这门语言的基础和了解开发者平台。



### **在Flow模拟器中的同质化代币**

---

>首先，您需要按照此链接打开带有预加载的同质化合约、交易和脚本的开发者平台：
>
>https://play.onflow.org/7c60f681-40c7-4a18-82de-b81b3b6c7915?type=account&id=0

>打开账户`0x01`标签页名为`ExampleToken.cdc`的文件。
>
>`ExampleToken.cdc`应该包含完整的代码和同质化代币，这个代码提供了在您的帐户中存储同质化代币
>
>以及转移和接受其他用户的代币的核心功能。

在 Cadence 中实现同质化代币所涉及的概念一开始可能很难理解。有关此功能和代码的深入说明，请继续阅读下一节。

或者，如果您想立即部署它并在开发者平台中使用它，您可以跳到本教程的[与同质化代币交互](https://docs.onflow.org/cadence/tutorial/03-fungible-tokens/#interacting-with-the-fungible-token-in-the-flow-playground)部分。



### **深入探索同质化代币**

---

Flow 实现同质化代币的方式与其他编程语言不同：

+ 所有权是去中心化的，不依赖于中央账本
+ 漏洞和漏洞为用户带来的风险更小，攻击者的机会更少
+ 没有整数下溢或上溢的风险
+ 资产不可复制，极难丢失、被盗、毁坏
+ 代码可以组合
+ 规则可以是不变的
+ 代码不会未经授权账户允许公开



### **去中心化所有权**

---

与中心分布式记账系统不同，对于资产所有权来说，通过一种新的模式Flow把所有权与每个账户相关联。下面的示例展示了 Solidity 如何实现同质化代币，为简洁起见，仅显示了用于存储和传输代币的代码。

`ERC20.sol`

```
contract ERC20 {
    mapping (address => uint256) private _balances;

    function _transfer(address sender, address recipient, uint256 amount) {
        // ensure the sender has a valid balance
        require(_balances[sender] >= amount);

        // subtract the amount from the senders ledger balance
        _balances[sender] = _balances[sender] - amount;

        // add the amount to the recipient’s ledger balance
        _balances[recipient] = _balances[recipient] + amount
    }
}
```

如您所见，Solidity 使用中央分类帐系统去操作同质化代币。有一个管理代币状态的合约，每次用户想要用他们的代币做任何事情时，他们都必须与中央 ERC20 合约进行交互，调用其函数来更新他们的余额。该合约处理所有功能的访问控制，所有正确性检查都调用自己，并为所有用户实施这些规则。

与中心分布式记账不同，Flow 利用一些不同的概念为智能合约开发人员和用户提供更好的安全性、保密性和可读性。在本节中，我们将通过一个同质化代币示例展示如何使用 Flow 的资源、接口和其他功能。



### **直观的资源所有权**

---

**资源**（**Resources**）在Cadence是一个很重要的概念，这代表了线性模型。一个资源是一个复合类型，它有自己定义的字段和函数，类似于结构体。区别在于资源对象有特殊的规则来防止它们被复制或丢失。资源是资产所有权的新范式。每个账户在其账户存储中拥有一个资源对象，记录他们拥有的代币数量，而不是在中央账本智能合约中代表代币所有权。这样，当用户想要相互交易时，他们可以进行点对点交易，而无需与中央代币合约进行交互。为了相互转移代币，他们在自己的资源对象和其他用户的资源上调用`transfer`转移函数（或等效的东西），而不是中央`transfer`转移函数。

这种方法简化了访问控制，因为中央合约不必检查函数调用的发送者，大多数函数调用发生在存储在账户的资源对象中，并且每个用户控制谁能够调用其帐户中资源的函数。这个概念称为基于功能定位符（Capability-based security）的安全性，将在后面的部分中详细解释。

这种方法也防止了潜在的bug。在具有中央合约包含的所有逻辑的 Solidity 中，一个漏洞可能会影响所有参与合约的用户。但在Cadence中，如果一个资源有逻辑上的bug，攻击者必须单独利用每个代币持有者帐户中的漏洞，这比中央分布式记账系统要复杂和耗时得多。

下面是同质化代币资源示例。拥有这些代币的每个用户都会将此资源存储在他们的帐户中。请务必记住，每个帐户仅存储 Vault 资源的副本，而不是整个 ExampleToken 合约的副本。 ExampleToken 合约只需要存储在管理代币定义的初始帐户中。

`Token.cdc`

```Cadence
pub resource Vault: Provider, Receiver {

    // Balance of a user's Vault
    // we use unsigned integers for balances because they do not require the
    // concept of a negative number
    pub var balance: UFix64

    init(balance: UFix64) {
        self.balance = balance
    }

    pub fun withdraw(amount: UFix64): @Vault {
        self.balance = self.balance - amount
        return <-create Vault(balance: amount)
    }

    pub fun deposit(from: @Vault) {
        self.balance = self.balance + from.balance
        destroy from
    }
}
```

这段代码很容易理解并且很有代表性。因为它展示了一个代币资源的工作原理。每一个代币资源对象都有其余额并且和相应的函数绑定（例如：`deposit`, `withdraw`, 等等）。当用户想要使用这些代币时，系统会帮他们在账户存储中实例化出来一个零余额的资源副本。Cadence要求只运行一次的初始化函数 `init` 必须初始化所有成员变量。

```Candece
// Balance of a user's Vault
// we use unsigned fixed-point integers for balances because they do not require the
// concept of a negative number and allow for more clear precision
pub var balance: UFix64

init(balance: UFix64) {
    self.balance = balance
}
```

然后，存款（deposit）函数可用于任何账户转移代币。

```Cadence
pub fun deposit(from: @Vault) {
    self.balance = self.balance + from.balance
    destroy from
}
```

当账户想要发送给不同账户一些代币，发送方首先会调用他们自己的提款（withdraw）函数，这会从他们的资源余额中减去代币，并临时创建一个新的资源对象来保持这个余额。然后发送方调用接收方的存款（deposit）函数，这也符合表面上将资源实例移动到另一个帐户，将其添加到他们的余额中，然后销毁使用的资源。资源需要销毁，因为 Cadence 对资源交互执行了严格的限制。资源永远不能挂在一段代码中。它要么需要明确销毁，要么存储在帐户的存储中。

与资源交互时，您使用`@ `符号和特殊的“移动运算符”`<-`。

```Cadence
pub fun withdraw(amount: UInt64): @Vault {
```

`@`符号需要为字段、参数或返回值指定资源类型。移动运算符 `<-` 明确地表明，当在赋值、参数或返回值中使用资源时，它被移动到新位置，旧位置无效。这确保了同一时间资源只能存在在一处位置。

如果资源从账户的存储中移出，则需要将其移动到账户的存储中或明确销毁。

```Cadence
destroy from
```

此规则可确保通常代表实际价值的资源不会因编码错误而丢失。

您会注意到算术运算并未明确防止上溢或下溢。

```Cadence
self.balance = self.balance - amount
```

在 Solidity 中，这可能存在整数上溢或下溢的风险，但 Cadence 具有内置的上溢和下溢保护，因此不存在风险。在这个例子中我们也使用了无符号整数，所以钱包的余额不能低于 0。

此外，帐户存储中包含代币资源类型副本的限制可确保资金不会因发送到错误地址而丢失。如果地址没有导入正确的资源类型，交易将恢复，确保发送到错误地址的交易不会丢失。

在提款（withdraw）函数里这行创建了一个新的`Vault`并且在调用过程中指定了参数名`balance`。

```Cadence
return <-create Vault(balance: amount)
```

另一个作用是Cadence必须提高代码的可读性。所有函数调用都需要指定它们发送的参数的名称，除非开发人员明确覆盖了函数声明中的要求。



### **确保公有安全性：功能定位符的安全性**

---

Cadence 的另一个重要特性是它对功能运算符安全性的支持。这一特性确保，虽然提款功能是公开的，但除了预期用户和他们批准的人之外，没有人可以从他们的金库中提取代币。

Cadence的安全性模型确保了在账户存储中的对象只能被所有者授权的账户才能访问。如果一个用户想让另一个用户访问他们存储的对象，他们可以链接一个公有功能定位符，这就像一个“API”，允许其他人调用他们对象上的指定函数。

一个帐户只有在拥有明确允许他们访问这些字段和方法的对象的功能定位符时，才有权访问不同帐户中对象的字段和方法。只有对象的所有者才能为其创建引用。因此，当用户在其帐户中创建 Vault 时，他们仅发布对存款功能和余额的引用。提款函数可以保持隐藏状态，作为只有所有者才能调用的函数。

正如您看到的，这消除了出于访问控制目的检查 `msg.sender` 的需要，因为此功能由协议和类型检查器处理。如果您不是对象的所有者或没有所有者创建的对它的有效引用，您根本无法访问该对象！



### **使用接口来保护实现**

---

下一个在Cadence中重要的概念是智能合约的设计，使用前置条件和后置条件来记录并以编程方式断言由程序段引起的状态更改。这些条件在接口中指定，这些接口强制执行有关如何定义类型和行为的规则。它们可以以不可变的方式存储在链上，以便某些代码可以导入和实现它们以确保它们符合某些标准。

这是我们上面定义的 `Vault` 资源接口的示例。

`Interfaces.cdc`

```Cadence
// Interface that enforces the requirements for withdrawing
// tokens from the implementing type
//
pub resource interface Provider {
    pub fun withdraw(amount: UFix64): @Vault {
        post {
            result.balance == amount:
                "Withdrawal amount must be the same as the balance of the withdrawn Vault"
        }
    }
}
// Interface that enforces the requirements for depositing
// tokens into the implementing type
//
pub resource interface Receiver {

    // There aren't any meaningful requirements for only a deposit function
    // but this still shows that the deposit function is required in an implementation.
    pub fun deposit(from: @Vault)
}
```

在我们的例子中，`Vault`资源将会实现这两个接口。这些接口确保资源实现中存在特定字段和函数，并且这些功能在执行之前和/或之后满足某些条件。这些接口可以存储在链上并导入到其他合约或资源中，以便这些要求由不易受人为错误影响的不可变事实来源强制执行。

还可以看到，函数和字段的定义都有关键字`pub`。我们需要显式地定义这些字段或函数为公有，因为在Cadence中所有字段和函数都默认是私有的，这也就意味着只有本地范围能访问它们。用户必须明确公开他们拥有的一部分类型。这有助于防止类型无意中公开代码。



### **在Flow开发者平台上与同质化代币交互**

---

现在您已经了解了同质化代币的工作原理，我们可以将其部署到您的帐户并发送一些交易以与之交互。

>按照本页顶部的链接，确保您已在开发者平台中打开了同质化代币的模板。您应该打开帐户 
>
>`0x01`，并且应该看到下面的代码。

`ExampleToken.cdc`

```Cadence
pub contract ExampleToken {

    // Total supply of all tokens in existence.
    pub var totalSupply: UFix64

    // Provider
    //
    // Interface that enforces the requirements for withdrawing
    // tokens from the implementing type.
    //
    // We don't enforce requirements on self.balance here because
    // it leaves open the possibility of creating custom providers
    // that don't necessarily need their own balance.
    //
    pub resource interface Provider {

        // withdraw
        //
        // Function that subtracts tokens from the owner's Vault
        // and returns a Vault resource (@Vault) with the removed tokens.
        //
        // The function's access level is public, but this isn't a problem
        // because even the public functions are not fully public at first.
        // anyone in the network can call them, but only if the owner grants
        // them access by publishing a resource that exposes the withdraw
        // function.
        //
        pub fun withdraw(amount: UFix64): @Vault {
            post {
                // `result` refers to the return value of the function
                result.balance == UFix64(amount):
                    "Withdrawal amount must be the same as the balance of the withdrawn Vault"
            }
        }
    }

    // Receiver
    //
    // Interface that enforces the requirements for depositing
    // tokens into the implementing type.
    //
    // We don't include a condition that checks the balance because
    // we want to give users the ability to make custom Receivers that
    // can do custom things with the tokens, like split them up and
    // send them to different places.
    //
    pub resource interface Receiver {
        // deposit
        //
        // Function that can be called to deposit tokens
        // into the implementing resource type
        //
        pub fun deposit(from: @Vault)
    }

    // Balance
    //
    // Interface that specifies a public `balance` field for the vault
    //
    pub resource interface Balance {
        pub var balance: UFix64
    }

    // Vault
    //
    // Each user stores an instance of only the Vault in their storage
    // The functions in the Vault and governed by the pre and post conditions
    // in the interfaces when they are called.
    // The checks happen at runtime whenever a function is called.
    //
    // Resources can only be created in the context of the contract that they
    // are defined in, so there is no way for a malicious user to create Vaults
    // out of thin air. A special Minter resource needs to be defined to mint
    // new tokens.
    //
    pub resource Vault: Provider, Receiver, Balance {

        // keeps track of the total balance of the account's tokens
        pub var balance: UFix64

        // initialize the balance at resource creation time
        init(balance: UFix64) {
            self.balance = balance
        }

        // withdraw
        //
        // Function that takes an integer amount as an argument
        // and withdraws that amount from the Vault.
        //
        // It creates a new temporary Vault that is used to hold
        // the money that is being transferred. It returns the newly
        // created Vault to the context that called so it can be deposited
        // elsewhere.
        //
        pub fun withdraw(amount: UFix64): @Vault {
            self.balance = self.balance - amount
            return <-create Vault(balance: amount)
        }

        // deposit
        //
        // Function that takes a Vault object as an argument and adds
        // its balance to the balance of the owners Vault.
        //
        // It is allowed to destroy the sent Vault because the Vault
        // was a temporary holder of the tokens. The Vault's balance has
        // been consumed and therefore can be destroyed.
        pub fun deposit(from: @Vault) {
            self.balance = self.balance + from.balance
            destroy from
        }
    }

    // createEmptyVault
    //
    // Function that creates a new Vault with a balance of zero
    // and returns it to the calling context. A user must call this function
    // and store the returned Vault in their storage in order to allow their
    // account to be able to receive deposits of this token type.
    //
    pub fun createEmptyVault(): @Vault {
        return <-create Vault(balance: UFix64(0))
    }

    // VaultMinter
    //
    // Resource object that an admin can control to mint new tokens
    pub resource VaultMinter {

        // Function that mints new tokens and deposits into an account's vault
        // using their `Receiver` reference.
        // We say `&AnyResource{Receiver}` to say that the recipient can be any resource
        // as long as it implements the Receiver interface
        pub fun mintTokens(amount: UFix64, recipient: &AnyResource{Receiver}) {
            ExampleToken.totalSupply = ExampleToken.totalSupply + UFix64(amount)
            recipient.deposit(from: <-create Vault(balance: amount))
        }
    }

    // The init function for the contract. All fields in the contract must
    // be initialized at deployment. This is just an example of what
    // an implementation could do in the init function. The numbers are arbitrary.
    init() {
        self.totalSupply = UFix64(30)

        // create the Vault with the initial balance and put it in storage
        // account.save saves an object to the specified `to` path
        // The path is a literal path that consists of a domain and identifier
        // The domain must be `storage`, `private`, or `public`
        // the identifier can be any name
        self.account.save(<-create Vault(balance: UFix64(30)), to: /storage/MainVault)

        // Create a new VaultMinter resource and store it in account storage
        self.account.save(<-create VaultMinter(), to: /storage/MainMinter)
    }
}

```

> 单击编辑器右下角的 `Deploy` 按钮部署代码。

此部署将同质化代币合约存储在您的活动账户（账户 `0x01`）中，以便可以将其导入到交易中。

合约的 `init` 函数在合约创建时运行，之后不再运行。在我们的示例中，此函数存储初始余额为 30 的 `Vault` 对象实例，并存储可用于铸造新代币的 `VaultMinter` 对象。

```Cadence
// create the Vault with the initial balance and put it in storage
// account.save saves an object to the specified `to` path
// The path is a literal path that consists of a domain and identifier
// The domain must be `storage`, `private`, or `public`
// the identifier can be any name
self.account.save(<-create Vault(balance: UFix64(30)), to: /storage/MainVault)
```

此行将新的 `@Vault` 对象保存到存储中。帐户存储是由路径索引，路径由域名和标识符组成。 `/domain/identifier`。路径只允许三个域：

+ `storage`：存放所有对象的地方。只能由帐户所有者访问。
+ `private`：将存储链接（也称为功能定位符）绑定存储的对象。只能由帐户所有者访问。
+ `public`：存储链接指向存储的对象：网络中的任何人都可以访问。

合约可以使用 `self.account` 访问其部署到的帐户的私有 `AuthAccount` 对象。该对象具有可以通过多种方式修改存储的方法。有关它可以调用的所有方法的列表，请参阅[帐户](https://docs.onflow.org/cadence/language/accounts)文档。

在这一行中，我们调用 `save` 方法将对象存储在 storage 中。第一个参数是要存储的值，第二个参数是存储值的路径。`sava`路径必须在 `/storage` 域名中。

我们还以相同的方式将 `VaultMinter` 对象存储到下一行的 `/storage/` 中：

```Cadence
self.account.save(<-create VaultMinter(), to: /storage/MainMinter)
```

您还应该看到 `ExampleToken.Vault` 和 `ExampleToken.VaultMinter` 资源对象存储在帐户存储中。这将显示在屏幕底部的资源框中。

您现在已准备好运行使用同质化代币的交易！



### **创建、存储和公开对 Vault 的功能定位符和引用**

---

功能定位符（Capabilities）就像其他语言中的指针。它们是帐户存储中对象的链接，可用于读取字段或调用它们引用的对象上的函数。他们不能直接移动或修改对象。

面对许多不同的场景你可以对你的同质化代币库存（Vault）去创建功能定位符。你可能想要一个简单的方式从交易中的任何位置调用你的`Vault`方法。您还可以发送一个仅在您的 `Vault` 中公开提款功能的功能定位符，以便其他人可以为您转移代币。你也可以有一个只公开 Balance 接口，以便其他人可以检查你拥有多少代币。也可以有一个函数将 `Vault` 的功能定位符作为参数，借用对该功能的引用，对该引用进行单个函数调用，然后完成并销毁该引用。

我们已经在 `mintTokens` 函数的 `Vault` 资源中使用了这种模式，如下所示：

```Cadence
// Function that mints new tokens and deposits into an account's vault
// using their `Receiver` capability.
// We say `&AnyResource{Receiver}` to say that the recipient can be any resource
// as long as it implements the Receiver interface
pub fun mintTokens(amount: UFix64, recipient: Capability<&AnyResource{Receiver}>) {
    let recipientRef = recipient.borrow()
        ?? panic("Could not borrow a receiver reference to the vault")

    ExampleToken.totalSupply = ExampleToken.totalSupply + UFix64(amount)
    recipientRef.deposit(from: <-create Vault(balance: amount))
}
```

该函数获取对任何实现 `Receiver` 接口的资源的功能定位符，从中借用一个引用，并使用该引用调用该资源的存款函数。

让我们为您的 `Vault` 创建功能定位符，以便单独的帐户可以向您发送代币。

> 打开名为 `Create Link` 的交易。
>
> `Create Link` 应包含以下代码，用于创建对存储的 Vault 的引用：

`Create`

```Cadence
import ExampleToken from 0x01

// This transaction creates a capability
// that is linked to the account's token vault.
// The capability is restricted to the fields in the `Receiver` interface,
// so it can only be used to deposit funds into the account.
transaction {
  prepare(acct: AuthAccount) {

    // Create a link to the Vault in storage that is restricted to the
    // fields and functions in `Receiver` and `Balance` interfaces,
    // this only exposes the balance field
    // and deposit function of the underlying vault.
    //
    acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>(/public/MainReceiver, target: /storage/MainVault)

    log("Public Receiver reference created!")
  }

  post {
    // Check that the capabilities were created correctly
    // by getting the public capability and checking
    // that it points to a valid `Vault` object
    // that implements the `Receiver` interface
    getAccount(0x01).getCapability<&ExampleToken.Vault{ExampleToken.Receiver}>(/public/MainReceiver)
                    .check():
                    "Vault Receiver Reference was not created correctly"
    }
}
```

为了使用功能定位符，我们必须创建一个链接绑定存储里的对象。然后可以从功能定位符创建引用，并且不能存储引用。这些引用需要在交易执行结束时丢弃。此限制是为了防止重入攻击，即恶意用户在原始执行完成之前一遍又一遍地调用同一函数的攻击。一次只允许一个对象的一个引用可以防止对存储的对象进行攻击。

为了创建功能定位符，我们使用`link`函数。

```Cadence
// Create a link to the Vault in storage that is restricted to the
// fields and functions in `Receiver` and `Balance` interfaces,
// this only exposes the balance field
// and deposit function of the underlying vault.
//
acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>.
(/public/MainReceiver, target: /storage/MainVault)
```

`link`函数创建了一个新的功能定位符其路径保存在第一个参数里，绑定的目标在第二个`target`参数里。链接中的类型显式用`<>`去进行类型检查。我们使用`&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}`去表达这个链接可以是任何资源，只要它实现并被转换为 Receiver 接口。这是描述引用的通用格式。首先有一个符号`&`后面紧跟具体类型，然后是大括号里的接口，以确保它是实现该接口的引用，并且只包含该接口中指定的字段。

我们把功能定位符放到`/public/MainReceiver`路径下，因为我们想要这个路径公有可读。网络中的任何人都可以通过帐户的 `PublicAccount` 对象访问帐户的`public`域名，该对象是使用 `getAccount(address)` 函数获取的。

接下来是交易的后置条件`post`阶段。

```Cadence
post {
// Check that the capabilities were created correctly
// by getting the public capability and checking
// that it points to a valid `Vault` object
// that implements the `Receiver` interface
getAccount(0x01).getCapability(/public/MainReceiver)
                .check<&ExampleToken.Vault{ExampleToken.Receiver}>():
                "Vault Receiver Reference was not created correctly"
}
```

后置条件`post`阶段是为了确保在交易执行后满足某些条件。在这里，我们从其公有路径获取功能定位符并调用其`check`函数以确保该功能定位符包含指向存储中指定类型的有效链接绑定的有效对象。

>选择帐户 `0x01` 作为唯一的签名者。
>
>点击`Send`发送按钮提交交易。
>
>此交易会创建一个对您的 `Vault` 的新公有引用，并检查它是否已正确创建。



### **代币转移给其他用户**

---

现在，我们想要运行一笔交易去发送给`0x02`账户10个代币。我们将通过调用账户 `0x01` 的 Vault 上的提款`withdraw`函数来做到这一点，它创建一个临时的 Vault 对象来移动代币，然后通过调用他们的存款`deposit`函数将这些代币存入账户 `0x02` 的账户。

在这里，我们遇到了 Cadence 的另一个安全特性。我们拥有的代币需要一个在你账户中的`Vault`对象来存储，所以如果任何人想要发送给并没有想要准备接收代币的人发送代币，这笔交易就会失败。以这种方式，如果用户在发送代币时不小心输入了错误的帐户地址，Cadence 会取消这笔交易来保护用户。

帐户 `0x02` 尚未设置为接收代币，因此我们现在将这样做：

> 打开交易`Setup Account`。
>
> 选择帐户` 0x02` 作为唯一的签名者。
>
> 单击`Send`按钮设置帐户 `0x02`，以便它可以接收代币。

`Setup`

```Cadence
import ExampleToken from 0x01

// This transaction configures an account to store and receive tokens defined by
// the ExampleToken contract.
transaction {
  prepare(acct: AuthAccount) {
    // Create a new empty Vault object
    let vaultA <- ExampleToken.createEmptyVault()

    // Store the vault in the account storage
    acct.save<@ExampleToken.Vault>(<-vaultA, to: /storage/MainVault)

    log("Empty Vault stored")

    // Create a public Receiver capability to the Vault
    let ReceiverRef = acct.link<&ExampleToken.Vault{ExampleToken.Receiver, ExampleToken.Balance}>(/public/MainReceiver, target: /storage/MainVault)

    log("References created")
  }

  post {
    // Check that the capabilities were created correctly
    getAccount(0x02).getCapability(/public/MainReceiver)
                    .check<&ExampleToken.Vault{ExampleToken.Receiver}>():
                    "Vault Receiver Reference was not created correctly"
  }
}
```

在这里，我们执行帐户 `0x01` 为设置其 `Vault` 所做的相同操作，但都是在一笔交易中完成的。账户 `0x02` 已准备好开始赚钱！如您所见，当我们为账户 `0x02` 创建 `Vault` 时，我们必须通过调用 `createEmptyVault()` 函数创建一个余额为零的账户。资源的创建仅限于定义它的合约，因此通过这种方式，同质化代币智能合约可以确保没有人能够凭空创建新的代币。

作为 ExampleToken 合约初始部署过程的一部分，账户 `0x01` 创建了一个 `VaultMinter` 对象。通过使用这个对象，拥有它的账户可以铸造新的代币。现在，帐户 `0x01` 拥有它，因此它拥有铸造新代币的唯一权力。我们本可以在合约中定义一个 `mintTokens` 函数，但是我们必须检查函数调用的发送者以确保他们被授权，这不是执行访问控制的推荐方式。

正如我们之前所解释的，资源模型加上功能定位符的安全性为我们处理这种访问控制，作为一种内置的语言结构，而不是必须在代码中定义。如果账户 `0x01` 想要授权另一个账户铸造代币，他们可以将 `VaultMinter` 对象移动到另一个账户，或者给另一个账户一个私有功能给单个 `VaultMinter`。或者，如果他们不希望在部署后进行铸造，他们会在合约初始化时简单地铸造所有代币，甚至不将 `VaultMinter` 包含在合约中。

在下一笔交易中，账户 `0x01` 将铸造 30 个新代币并将它们存入账户 `0x02` 新创建的 Vault对象中。

>仅选择帐户 `0x01` 作为签名者，并发送 `Mint Tokens`为帐户 `0x02` 铸造 30 个代币。
>
>`Mint Tokens` 应包含以下代码。

`Mint`

```Cadence
import ExampleToken from 0x01

// This transaction mints tokens and deposits them into account 2's vault
transaction {

  // Local variable for storing the reference to the minter resource
  let mintingRef: &ExampleToken.VaultMinter

  // Local variable for storing the reference to the Vault of
  // the account that will receive the newly minted tokens
  var receiverRef: &ExampleToken.Vault{ExampleToken.Receiver}

  prepare(acct: AuthAccount) {
    // Borrow a reference to the stored, private minter resource
    self.mintingRef = acct.borrow<&ExampleToken.VaultMinter>(from: /storage/MainMinter)
        ?? panic("Could not borrow a reference to the minter")

    // Get the public account object for account 0x02
    let recipient = getAccount(0x02)

    // Get the public receiver capability
    let cap = recipient.getCapability(/public/MainReceiver)

    // Borrow a reference from the capability
    self.receiverRef = cap.borrow<&ExampleToken.Vault{ExampleToken.Receiver}>()
        ?? panic("Could not borrow a reference to the receiver")
}

  execute {
      // Mint 30 tokens and deposit them into the recipient's Vault
      self.mintingRef.mintTokens(amount: UFix64(30), recipient: self.receiverRef)

      log("30 tokens minted and deposited to account 0x02")
  }
}
```

在这笔交易中，我们利用跨越不同阶段的本地变量来完成交易，这在我们的示例中尚属首次。我们在准备阶段之外声明 `mintingRef` 和`receiverRef` 变量，但必须在准备阶段初始化它们。然后我们可以在交易的后置条件阶段使用它们。

除了从功能定位符中借用引用之外，您还将在此交易中看到，您还可以直接从存储中的对象借用引用。

```Cadence
// Borrow a reference to the stored, private minter resource
self.mintingRef = acct.borrow<&ExampleToken.VaultMinter>(from: /storage/MainMinter)
    ?? panic("Could not borrow a reference to the minter")
```

在这里，我们将借用指定为 `VaultMinter` 引用，并将引用指向 `/storage/MainMinter`。他的引用是作为备选项借用的，因此我们使用 nil 合并运算符 (`??`) 来确保该值不是 nil。如果值为nil，交易将执行`??`。代码就是错误的，所以它将返回状态并打印错误消息。

您可以使用 `getAccount()` 内置函数来获取任何帐户的公有帐户对象。公共帐户对象允许您从存储公有功能定位符的帐户的`public`域名中获取功能定位符。

我们使用 `getCapability` 函数从公共路径获取公共能力，然后使用功能定位符上的`borrow`借用函数从中获取引用，类型为 `ExampleToken.Vault{ExampleToken.Receiver}`。

```Cadence
// Get the public receiver capability
let cap = recipient.getCapability(/public/MainReceiver)

// Borrow a reference from the capability
self.receiverRef = cap.borrow<&ExampleToken.Vault{ExampleToken.Receiver}>()
        ?? panic("Could not borrow a reference to the receiver")
```

在执行阶段，我们简单地使用引用来铸造 30 个代币并将它们存入账户 `0x02` 的 `Vault` 中。



### **检查帐户余额**

---

现在，帐户 `0x01` 和帐户 `0x02` 的存储中都应该有一个 `Vault` 对象，该对象的余额为 30 个代币。他们都应该在他们的 `/public/` 域名中存储一个接收`Receiver`功能，链接到他们存储中的 `Vault`对象。

![img](https://storage.googleapis.com/flow-resources/documentation-assets/cadence-tuts/account-balances.png)

除非专门配置过接受这些代币，否则帐户无法接收任何代币类型。因此，很难意外地将代币发送到错误的地址。但是，如果您在新帐户中设置 `Vault` 时出错，您将无法向其发送代币。

让我们运行一个脚本来确保我们正确设置了库存(vaults)。

您可以使用脚本访问帐户的公开状态。脚本未由任何帐户签名且无法修改状态。

在这个例子中，我们将查询每个账户的库存(vaults)余额。下面将在模拟器中打印出每个帐户的余额。

>在脚本栏下打开名为 `Script1.cdc` 的脚本
>
>`Script1.cdc` 应包含以下代码：

`Script1.cdc`

```Cadence
import ExampleToken from 0x01

// This script reads the Vault balances of two accounts.
pub fun main() {
    // Get the accounts' public account objects
    let acct1 = getAccount(0x01)
    let acct2 = getAccount(0x02)

    // Get references to the account's receivers
    // by getting their public capability
    // and borrowing a reference from the capability
    let acct1ReceiverRef = acct1.getCapability<&ExampleToken.Vault{ExampleToken.Balance}>(/public/MainReceiver)
        .borrow()
        ?? panic("Could not borrow a reference to the acct1 receiver")

    let acct2ReceiverRef = acct2.getCapability<&ExampleToken.Vault{ExampleToken.Balance}>(/public/MainReceiver)
        .borrow()
        ?? panic("Could not borrow a reference to the acct2 receiver")

    // Read and log balance fields
    log("Account 1 Balance")
    log(acct1ReceiverRef.balance)
    log("Account 2 Balance")
    log(acct2ReceiverRef.balance)
}
```

>单击执行按钮执行 `Script1.cdc`。
>
>执行后保证了以下几点：

+ 账户 `0x01` 的余额为 30
+ 账户 `0x02` 的余额为 30

如果正确，您应该看到以下几行：

```
"Account 1 Balance"
30
"Account 2 Balance"
30
Result > "void"
```

如果出现错误，这可能意味着您之前错过了一步，可能需要从头开始。

要重新启动开发者平台，请关闭当前会话并打开教程顶部的链接。

现在我们有两个账户，每个账户都有一个 `Vault`，我们可以看看他们是如何相互转移代币的！

>打开名为 `Transfer Tokens` 的交易。
>
>选择账户 `0x02` 作为签名者并发送交易。
>
>`Transfer Tokens` 应包含以下用于向另一个用户发送令牌的代码：

`Transfer`

```Cadence
import ExampleToken from 0x01

// This transaction is a template for a transaction that
// could be used by anyone to send tokens to another account
// that owns a Vault
transaction {

  // Temporary Vault object that holds the balance that is being transferred
  var temporaryVault: @ExampleToken.Vault

  prepare(acct: AuthAccount) {
    // withdraw tokens from your vault by borrowing a reference to it
    // and calling the withdraw function with that reference
    let vaultRef = acct.borrow<&ExampleToken.Vault>(from: /storage/MainVault)
        ?? panic("Could not borrow a reference to the owner's vault")

    self.temporaryVault <- vaultRef.withdraw(amount: UFix64(10))
  }

  execute {
    // get the recipient's public account object
    let recipient = getAccount(0x01)

    // get the recipient's Receiver reference to their Vault
    // by borrowing the reference from the public capability
    let receiverRef = recipient.getCapability(/public/MainReceiver)
                      .borrow<&ExampleToken.Vault{ExampleToken.Receiver}>()
                      ?? panic("Could not borrow a reference to the receiver")

    // deposit your tokens to their Vault
    receiverRef.deposit(from: <-self.temporaryVault)

    log("Transfer succeeded!")
  }
}
```

在这个例子中，签名者从他们的 `Vault` 中提取代币，这会创建并返回一个 `balance=10` 的临时 `Vault` 资源对象，用于转移代币。在执行阶段。交易使用他们的存款`deposit`函数将该资源移动到另一个用户的`Vault`。临时`Vault`在其余额添加到接收者的`Vault`后被销毁。

您可能想知道为什么我们必须使用两个函数调用来完成一次代币传输，而不是一次完成。因为这是资源在 Cadence 中的工作原理。在基于分布式账本的模型中，您只需调用 transfer，它只会更新分布式账本，但在 Cadence 中，代币的位置很重要，因此大多数代币转移情况不仅仅是直接账户到账户的转账。大多数情况下，代币将首先用于不同的目的，例如购买东西，这需要在存入帐户存储之前单独发送和验证 `Vault`。将两者分开还使我们能够利用静态验证帐户的哪些部分可以在交易的准备部分进行修改，这将帮助用户在从应用程序获取馈送交易时高枕无忧。

> 再次执行 `Script1.cdc` 。

如果正确，您应该看到以下几行，表明账户 `0x01` 的余额为 40，账户 `0x02` 的余额为 20：

```
"Account 1 Balance"
40
"Account 2 Balance"
20
Result > "void"
```

您现在知道如何在 Cadence 和 Flow 中使用基本的同质化代币了！

从这里开始，您可以尝试通过以下方式扩展同质化代币的功能：

+ 生成代币的源头
+ 可以存入的托管（但只有在余额达到某个点时才能提取）
+ 用于铸造新代币资源的函数！



### **非同质化代币**

---

既然您已经了解了同质化代币如何在 Flow 上工作，您就可以开始使用非同质化代币了！

