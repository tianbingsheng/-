**## Internal包简介**

Internal包主要是封装了js的命令行界面，并且包含了命令行所调用的api信息。

**## ethapi/api包分析**

ethapi/api包主要是进入js的命令行界面后，输入的命令实现部分。<br>

js的命令实现在ethapi/api和node/api中。目前一共有三种api的命令。<br>

(1)第一种是admin相关的命令，这个是通过安全的RPC通道实现的。其结构体为PrivateAdminAPI<br>

\```

// PrivateAdminAPI is the collection of administrative API methods exposed only

// over a secure RPC channel.

type PrivateAdminAPI struct {

​    node *Node // Node interfaced by this API

}

\```

(2)第二种是personal相关的命令，主要是负责账户管理相关命令，可以lock和unlock账户。其结构体为PrivateAccountAPI<br>

\```

// PrivateAccountAPI provides an API to access accounts managed by this node.

// It offers methods to create, (un)lock en list accounts. Some methods accept

// passwords and are therefore considered private by default.

type PrivateAccountAPI struct {

​    am *accounts.Manager

​    nonceLock *AddrLocker

​    b Backend

}

\```

(3)第三种是eth相关的命令，主要是可以操作区块上的相关命令。其结构体为PublicBlockChainAPI<br>

\```

// PublicBlockChainAPI provides an API to access the Ethereum blockchain.

// It offers only methods that operate on public data that is freely available to anyone.

type PublicBlockChainAPI struct {

​    b Backend

}

\```

 

**## otto包**

以太坊的命令是通过在js虚拟机上来实现命令的。而在go语言中，有第三方的otto包，可以直接在go语言中实现js命令。而以太坊代码则使用了otto包来实现搭建js命令。<br>

在otto包中，set方法是设置变量的值，get方法是获取变量的值。

\```

// Set the property of the given name to the given value.

func (self Object) Set(name string, value interface{})

// Get the value of the property with the given name.

func (self Object) Get(name string) (Value, error)

\```

Compile是根据输入的路径对js的代码进行编译，返回变量的值。

\```

// Compile will parse the given source and return a Script value or nil and

// an error if there was a problem during compilation.

func (self *Otto) Compile(filename string, src interface{}) (*Script, error)

\```

Run方法会运行相关的js代码，如果有返回值的话会返回。

\```

// Run will run the given source (parsing it first if necessary), returning the resulting value and error (if any)

func (self Otto) Run(src interface{}) (Value, error)

\```

**## 如何编写自己的以太坊命令**

接上篇ethapi.api-analysis分析，如果我们需要在相关模块添加相关命令，首先我们需要找到相关命令所对应的api结构体。<br>

各个命令对应的结构体，包的位置如下：

\```

​     admin  PrivateAdminAPI,PublicAdminAPI  node/api

​     debug  PrivateDebugAPI eth/api

​     eth    PublicBlockChainAPI ethapi/api

​     miner  PrivateMinerAPI eth/api

​     net    PublicNetAPI    ethapi/api

​     personal   PrivateAccountAPI   ethapi/api

​     txpool PublicTxPoolAPI ethapi/api

​     rpc    所有可调用包集合

​     web3   所有命令集合

\```

假设我们需要在personal包中添加一个命令，那么我们就在PrivateAccountAPI中添加一个方法：

\```

func (s *PrivateAccountAPI) TestMethod() {

fmt.Print("TestMethod")

}

\```

接下来到internal/web3ext/web3ext.go中，找到personal命令集合，然后添加一条自己的命令：

\```

const Personal_JS = `

web3._extend(

methods: [

​        new web3._extend.Method({

​            name : 'testMethod',

​            call : 'personal_testMethod'

​        }), //our method

...

\```

最后到internal/jsre/deps/web3.js中，找到personal方法的定义：

\```

function Personal(web3) {

this._requestManager = web3._requestManager;

 

var self = this;

 

methods().forEach(function(method) {

method.attachToObject(self);

method.setRequestManager(self._requestManager);

});

 

properties().forEach(function(p) {

p.attachToObject(self);

p.setRequestManager(self._requestManager);

});

}

var methods = function () {

...

\```

然后再methods中添加你定义的方法名：

\```

var methods = function () {

var testMethod = new Method({

name : 'testMethod',

call : 'personal_testMethod'

});

...

\```

并在最后的return中添加你的方法：

\```

return [

newAccount,

testMethod, //our method

importRawKey,

unlockAccount,

ecRecover,

sign,

sendTransaction,

lockAccount

];

\```

这之后在启动命令行，我们就可以调用我们的方法了。结果如下：

\```

\> personal.testMethod()

TestMethodnull

\```