# bips-note
个人的 [Bitcoin Improvement Proposals](https://github.com/bitcoin/bips) 阅读笔记，对除了 Withdrawn 和 Replaced 状态以外的 BIP 进行摘要和评论。

-------------

### BIP 2 - BIP 的接受流程
  * BIP 作者该如何提交 BIP 和 BIP workflow
  * 软分叉硬分叉如何处理
    * 软分叉应当投票决定是否采用，见 BIP 8
    * 硬分叉应让整个比特币生态参与（包括币的持有者和接受比特币的商人）
  * 特别指出 BIP 2 不是个严格的规定，不应该用 BIP 2 来反对或支持某个 BIP。原话：如果有某个 BIP 不遵守 BIP2 但是应该被合并那么它应该被合并
  * 看得出来 BIP 系统是个比较松散的规则，用来方便组织各种 proposal，并明确指出不是强制性的 governance, 很有比特币和开源社区的风格
  * BIP2 中对比特币经济系统的描述也很有趣：持币者、商人。指出了这二者影响币价（毕竟比特币是个支付系统） 但矿工、交易所、开发者却不包括(因为这些人是为经济系统提供服务，但非决定者)
  
  
### BIP 8 - 描述了软分叉的一些细节协议和功能部署 workflow (对 BIP 9 的替代，WTF?)
  * 规定了某个软分叉的功能部署状态的 workflow
  * 开发者设置 Block 中 nVersion 的含义，客户端选择投票(出块时标记)，最后根据投票情况更新功能的状态
  * 每 2016 个块（称作 retarget period）按照软分叉的 workflow 变更一次功能部署状态, 2016 个块是给矿工利用 block version 投票的时间
  * 要有 95% 的算力才能部署成功 即 >= 1916 个块，要求很高、可能目的是为了防矿场操纵投票结果。
  * 因为投票使用 nVersion 字段的某个 bit 来完成，比特币只能同时支持 33 个功能部署进行投票。
  * BlockTemplate 接口需要修改, 让挖矿进程可以取到软分叉的规则，名字带有 ! 的软分叉功能表明对挖矿可能有影响
  
  
### BIP 11 - M-of-N-signatures, 支持 OP_CHECKMULTISIG 操作码用来实现多签功能
  * OP_CHECKMULTISIG 操作码用来验证多签签名，BIP 中举例的两个场景比较有趣
  * Usecase 1: 用多签钱包做第三方的支付保护服务，生成 2 of 2 私钥，把一个私钥给第三方保管，当发生支付时第三方通过 App/邮件 等方式提示用户，用户确认后才能支付成功，防止用户私钥泄漏，且第三方使用单独的一个私钥无法支付成功。
  * Usecase 2: 可以用多签钱包+第三方做交易匹配功能


### BIP 13 - pay-to-script-hash(P2SH) 的地址格式
  * base58-encode: [one-byte version][20-byte hash][4-byte checksum]
  * 第一位为测试网/主网等不同网络的标记。感觉比以太坊地址要清晰，不会在不同网络共用地址
  * 最后四位用来校验，比以太坊地址的校验简单很多
  

### BIP 14 - 分离 Protocol version 和 Client version
  * 这个 BIP 之前两个字段没有区分，导致 nakamoto 客户端更新时比特币协议版本也要更新
  
  
### BIP 15 - 地址别名
  * 这个 BIP 列举了几种用别名来映射到地址的方案
  * FirstBits，即取地址前几位作为别名，缺点如下
    * 想要生成有意义的地址前缀需要浪费大量计算资源
    * 扩展性差，需要遍历 UXTO 寻找对应的地址全名
  * DNS / Https 服务器查询地址别名，附带公钥签名(BIP 中没提到，但感觉这个方案是不是过于中心化？)
  * namecoin 返回比特币地址（感觉上这个方案比较靠谱，类似的 ENS 也可以）
  
  
### BIP 16 - Pay to Script Hash(P2SH)
  * 比特币交易引入了 P2SH 的重要概念, 由获取者花费 Output 时解锁，而不是发送者来决定发给谁
  * P2SH 验证规则：
    1. 解锁 script 只能包含 push data(serialized script) 操作
    2. data(serialized script) 的 hash 和 outpoint 相等
    3. 弹出 data(serialized script) 后再作为 script 执行
  * serialized script 中包含的签名操作按照一定规则累加计入到 block 的签名数里（此 BIP 时 Bitcoin 每个块最多 20000 SigOp）
  
  
### BIP 18 - hashScriptCheck, 更新了比特币交易的格式，以支持 BIP 16 P2SH
  * 废弃交易的 scriptSig， scriptPubKey（考虑向后兼容，仍支持这两个字段）
  * 新字段 scriptCheck, dataSig, hashScriptCheck
  * scriptCheck: 一段脚本，hash 后必须符合 hashScriptCheck, 执行后返回 true 则交易通过验证
  * dataSig: 只包含 data，会预先放在 stack 上，再去执行 scriptCheck (似乎类似 CKB 中的 signed_arguments 设计？)
  * hashScriptCheck: 放在交易的 output 中，指定哪些脚本可以解锁 output


### BIP 19 - M-of-N Standard Transactions (Low SigOp) 提供两个新的 Op 支持低 SigOp 的多签操作
  * BIP 11 已经支持多签并提供了 OP_CHECKMULTISIG，这个 BIP 提供新操作码是为了降低多签操作占用的 SigOp 以在 block 包含更多多签交易
  * BIP 11 中的 OP_CHECKMULTISIG 占用 20 SigOp, 而 OP_CHECKSIG 每个签名只占用 1 SigOp
  * N-of-N: OP_CHECKSIGVERIFY: `  {pubkey} OP_CHECKSIGVERIFY {pubkey} OP_CHECKSIGVERIFY {pubkey} OP_CHECKSIGVERIFY`
  * N-of-M: OP_CHECKSIG: `  {pubkey} OP_CHECKSIG OP_SWAP {pubkey} OP_CHECKSIG OP_ADD {n} OP_EQUAL`
  * 新 Op 相比之下更加灵活可扩展
  
  
### BIP 21 - 定义了 bitcoin 的 URI schema, 允许用 URI、QR code 等发起支付
  *  `bitcoin:<address>[?amount=<amount>][?label=<label>][?message=<message>]`
  * URI 支持 `amountparam / labelparam / messageparam / otherparam / reqparam` 等参数
  * req 开头代表必须填的字段


### BIP 22 - 描述 Bitcoin 提供给 Miner 的新 JSON RPC 接口
  * `getblocktemplate` - 矿工用来获取挖矿模板
    * 请求时可以选择 mode 是 "template" / "proposal"
    * 返回 `height`, `previousblockhash`, `transactions` 等字段供矿工挖矿使用
  * `submitblock` - 用来提交 block, 内容是一个 hex-encoded 的 block
  * 可选功能 Long Polling - 支持 Long Polling 的客户端会返回 `longpollid` 给矿工，矿工用来 polling 判断当前任务是否过期（比如客户端从网络上接收了新的块）


### BIP 23 - 在 BIP 22 的基础上扩展，对矿池挖矿做了优化
  * 本 BIP 对 `get_block_template` 做了扩展以支持矿池，从功能支持上分为 level1 支持和 level2 支持
  * Level 1 支持
    * RFC 1945 - DNS over HTTPS
    * BIP 22 Long Polling
    * Basic Pool Extensions - `getblocktemplate` 接口支持 `target` 参数，获取小于某个难度的 block 模板
    * Mutation "coinbase/append" - append the provided coinbase scriptSig（没明白这样的意图，scriptSig 已在 BIP 18 废弃）
    * Submission Abbreviation "submit/coinbase" - 如果 miner 不改动 `get_block_template` 的交易列表，这个字段允许 miner 只提交 coinbase, 不再重复提交 transactions 以优化提交过程的性能
    * Mutation "time/increment" - 改变 header 中的时间到某个时间点后（没明白这样的意图）
  * Level 2 支持
    * Mutation "transactions/add" - 允许 Miner 追加已验证的交易到 `get_block_template` 中，但不保证一定加进去
    * Block Proposal - Miner 可以在任务过期前向节点提交 block proposal，节点返回*可接收*、*拒绝* 或 *修改后的 template*
  * 还有很多琐碎的扩展参数未列出
  * serverlist - 节点的功能，返回一个列表包含节点和节点的状态，miner 可以把这些节点当成一个逻辑节点来用（相当于客户端负载均衡）
  * BIP 23 中对部分参数的用途的解释 https://github.com/bitcoin/bips/blob/master/bip-0023.mediawiki#rationale
    
  
