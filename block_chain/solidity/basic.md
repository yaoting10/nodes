# Basic
uint256 max_value: 

115792089237316195423570985008687907853269984665640564039457584007913129639935



msg.sender :调用者地址

tx.origin : 交易发起者地址



方法名：4字节的bytes 0x后8位

函数签名是通过将函数的名称和参数进行hash值，然后取hash值的前4位十六进制数字：比如：

```solidity
function transfer(address _to, uint256 _amount) external {}
```

取的是"transfer(address,uint256)", 注意不要空格

```solidity
function getSelector(string calldata _func) external pure returns(byte4){
		return byte4(keccak256(bytes(_func)));
}

```

得出的值是"0xa9059cbb"



storage, memory, calldata

Storage 相当于存储读取，能够进行链上数据的改变--状态变量

memory 相当于内存读取，不能够进行链上数据改变（修改的值，函数调用结束后就消失，不会写入链上）--局部变量，只在内存中生效

calldata 和memory 类似，但是只能用于输入参数中，智能合约参数中使用calldata可以节约gas费