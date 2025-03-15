# doc

## terminology

Vitalik Buterin | the founder of Ethereum | 2014

Merkle Trees

##  RTFM

[Ethereum Whitepaper](https://ethereum.org/en/whitepaper/)

[Ethereum development documentation](https://ethereum.org/en/developers/docs/)

# Node

make geth

./build/bin/geth --authrpc.jwtsecret=~/.ethereum/jwtsecret --http --http.api "engine,eth,net" --syncmode "snap"

./target/release/lighthouse beacon_node --network mainnet --execution-endpoint http://127.0.0.1:8551 --jwt-secrets ~/.ethereum/jwtsecret   --checkpoint-sync-url https://mainnet.checkpoint.sigp.io

## buy a cheap and hign volume disk

## find the first block information

## do a transction

## 



make geth
./build/bin/geth account new  123456
Public address of the key:   0xf501A605F558b48246F5c4B3c54C30F8F6ab8c6c
Path of the secret key file: /home/xunchan/.ethereum/keystore/UTC--2025-03-15T14-40-34.359221619Z--f501a605f558b48246f5c4b3c54c30f8f6ab8c6c

如果你想使用 **Geth 编译的钱包** 完成一笔以太坊交易，流程大致如下：

---

## **💡 总流程**
1️⃣ **启动 Geth 并解锁钱包**  
2️⃣ **获取账户余额，确保有 ETH**  
3️⃣ **构造交易并发送**  
4️⃣ **等待交易确认**  

---

## **🔧 详细步骤**
### **📌 步骤 1：启动 Geth**
你需要运行 Geth 连接以太坊网络，建议使用轻节点（light sync）模式：
```bash
./build/bin/geth --syncmode "light"
```
如果你想使用本地私有链（测试环境），可以用：
```bash
./build/bin/geth --dev --http --http.api "eth,web3,personal"
```
这将启动一个**测试环境**，无需真实的 ETH。

---

### **📌 步骤 2：解锁钱包**
先查看钱包账户：
```bash
./build/bin/geth account list
```
会返回：
```
Account #0: {0xYourEthereumAddress} keystore://path/to/keystore/file
```
然后解锁账户：
```bash
./build/bin/geth --unlock 0xYourEthereumAddress --password password.txt --syncmode "light"
```
🔹 **注意**：
- `password.txt` 文件里存放你的钱包密码（也可以手动输入）。
- 你需要保证有足够的 ETH 来支付矿工费。

---

### **📌 步骤 3：检查余额**
查询钱包的 ETH 余额：
```bash
./build/bin/geth attach --exec "eth.getBalance('0xYourEthereumAddress')"
```
返回的余额是 **wei**（1 ETH = 10¹⁸ wei），如果余额不足，你需要先充值。

---

### **📌 步骤 4：发送交易**
使用 `personal.sendTransaction` 发送交易：
```bash
./build/bin/geth attach --exec "personal.sendTransaction({from: '0xYourEthereumAddress', to: '0xReceiverAddress', value: web3.toWei(0.01, 'ether')}, 'YourPassword')"
```
🔹 这里：
- `from` 是你的钱包地址
- `to` 是接收方的钱包地址
- `value` 是转账金额（这里是 **0.01 ETH**）
- `web3.toWei(0.01, 'ether')` 将 ETH 转换为 wei
- **交易完成后返回 Transaction Hash**（例如 `0xabc123...`）

---

### **📌 步骤 5：查询交易状态**
交易需要矿工打包到区块里，你可以通过交易哈希查询状态：
```bash
./build/bin/geth attach --exec "eth.getTransactionReceipt('0xTransactionHash')"
```
如果 `status: 1`，说明交易成功 ✅。

---

## **🎯 其他方式**
如果你更喜欢用 Python，可以用 `web3.py` 库发送交易：
```python
from web3 import Web3

# 连接以太坊节点
w3 = Web3(Web3.HTTPProvider('http://localhost:8545'))

# 账户信息
sender = '0xYourEthereumAddress'
receiver = '0xReceiverAddress'
private_key = 'YourPrivateKey'  # 真实环境请不要明文存储私钥！

# 构造交易
nonce = w3.eth.get_transaction_count(sender)
tx = {
    'nonce': nonce,
    'to': receiver,
    'value': w3.to_wei(0.01, 'ether'),
    'gas': 21000,
    'gasPrice': w3.to_wei(20, 'gwei')
}

# 签名并发送交易
signed_tx = w3.eth.account.sign_transaction(tx, private_key)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)

print(f"Transaction Hash: {w3.to_hex(tx_hash)}")
```
✅ 这种方式适合做自动化交易，比如 **智能合约交互、批量转账等**。

---

## **📌 总结**
- **使用 Geth CLI** 可以手动发送 ETH 交易（适合命令行操作）。
- **使用 Python + Web3** 可以更灵活地控制交易（适合程序化交易）。
- **一定要检查余额和 gas 费**，否则交易可能失败。

🎯 你想通过 **CLI** 还是 **Python** 来完成交易？

### **🚨 Lighthouse 报错解析**
```
ERRO Error updating deposit contract cache   
error: Failed to get remote head and new block ranges: EndpointError(FarBehind), 
retry_millis: 60000, service: beacon
```
🔹 **错误原因**：
- Lighthouse 的 **信标链（Beacon Chain）** 客户端 **未能获取最新的区块头信息**，因为它的 **执行层（Execution Layer，即 Geth） 同步落后**。
- `FarBehind` 表示 **Geth（或 Lighthouse 本身）比主网落后太多**，导致无法正常同步。

---

## **🚀 解决方案**
### ✅ **1. 确保 Geth 正在同步**
你需要确认 **Geth 是否在同步** 并且它的区块高度接近主网：
```bash
geth attach --exec "eth.syncing"
```
📌 **可能的返回结果**：
1️⃣ **如果返回 `false`**：
   ```json
   false
   ```
   - 说明 Geth 已经同步完成，问题可能在 Lighthouse 配置上。
   
2️⃣ **如果返回一个对象**：
   ```json
   {
       startingBlock: 1234567,
       currentBlock: 1238765,
       highestBlock: 1239000
   }
   ```
   - 说明 Geth 仍在同步。
   - **解决办法**：等待 Geth 同步完成，或者重启 Geth 用更快的同步方式（见下方）。

---

### ✅ **2. 重新启动 Geth 并使用 Snap Sync（快照同步）**
如果 Geth **同步太慢**，建议**用快照同步**（Snap Sync）模式重启：
```bash
geth --syncmode "snap" --authrpc.jwtsecret ~/.ethereum/jwtsecret --http --http.api "engine,eth,net"
```
🔹 **为什么这样做？**
- `--syncmode "snap"`：使用快照同步，比传统方式快 10 倍以上。
- `--authrpc.jwtsecret ~/.ethereum/jwtsecret`：确保 Geth 能与 Lighthouse 正常通信。

📌 **等待几分钟后，重新检查 Geth 的同步状态**：
```bash
geth attach --exec "eth.syncing"
```

---

### ✅ **3. 确保 Lighthouse 连接了正确的 Geth**
你的 Lighthouse 需要连接到 **最新同步的 Geth**，尝试 **重新启动 Lighthouse**：
```bash
./target/release/lighthouse beacon_node \
  --network mainnet \
  --execution-endpoint http://127.0.0.1:8551 \
  --jwt-secret ~/.ethereum/jwtsecret \
  --checkpoint-sync-url https://mainnet.checkpoint.sigp.io
```
🔹 这里 `--checkpoint-sync-url` 允许 Lighthouse **从最新状态同步**，避免因 Geth 落后导致的问题。

---

### ✅ **4. 等待 Lighthouse 重新同步**
如果 Geth **区块高度** 仍然落后，Lighthouse **可能还需要一些时间** 来同步。你可以检查它的最新状态：
```bash
curl -X POST http://localhost:5052/lighthouse/health
```
如果返回：
```json
{"status": "ready"}
```
🎉 **说明 Lighthouse 已经恢复正常同步！**

---

## **🎯 总结**
✅ **错误 `FarBehind` 说明你的 Geth 落后太多，导致 Lighthouse 无法同步最新数据。**  
✅ **检查 Geth 状态 (`eth.syncing`)，确保它在同步或者已经完成同步。**  
✅ **如果 Geth 进度太慢，重启 Geth 用 `--syncmode snap` 进行快照同步**，可以更快完成同步。  
✅ **Lighthouse 需要连接同步完成的 Geth，建议使用 `--checkpoint-sync-url` 加快同步。**  

🚀 **先检查 Geth 的同步状态，看看它是否已经赶上主网！** 🎉



# how many ethereum coin in beggining

# chain

# 找到账本

# 找到交易代码

# 找到币的地址