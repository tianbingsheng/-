# [以太坊系列之一: 以太坊RLP用法-以太坊源码学习](https://www.cnblogs.com/baizx/p/6928622.html)

[RLP](https://github.com/ethereum/wiki/wiki/%5B%E4%B8%AD%E6%96%87%5D-RLP) (递归长度前缀)提供了一种适用于任意二进制数据数组的编码，RLP已经成为以太坊中对对象进行序列化的主要编码方式。RLP的唯一目标就是解决结构体的编码问题；对原子数据类型（比如，字符串，整数型，浮点型）的编码则交给更高层的协议；以太坊中要求数字必须是一个大端字节序的、没有零占位的存储的格式（也就是说，一个整数0和一个空数组是等同的）。

```
如果想学习go语言中的反射用法,这个包里面倒是有比较完善的学习示例,感兴趣的可以看看.
下面是我写的一个使用示例,演示如何使用rlp这个包.
/*
rlp包用法
rlp目的是可以将常用的数据结构,uint,string,[]byte,struct,slice,array,big.int等序列化以及反序列化.
要注意的是rlp特别不支持有符号数的序列化
具体用法见下
*/
//编码
type TestRlpStruct struct {
    A      uint
    B      string
    C      []byte
    BigInt *big.Int
}

//rlp用法
func TestRlp(t *testing.T) {
    //1.将一个整数数组序列化
    arrdata, err := rlp.EncodeToBytes([]uint{32, 28})
    fmt.Printf("unuse err:%v\n", err)
    //fmt.Sprintf("data=%s,err=%v", hex.EncodeToString(arrdata), err)
    //2.将数组反序列化
    var intarray []uint
    err = rlp.DecodeBytes(arrdata, &intarray)
    //intarray 应为{32,28}
    fmt.Printf("intarray=%v\n", intarray)

    //3.将一个布尔变量序列化到一个writer中
    writer := new(bytes.Buffer)
    err = rlp.Encode(writer, true)
    //fmt.Sprintf("data=%s,err=%v",hex.EncodeToString(writer.Bytes()),err)
    //4.将一个布尔变量反序列化
    var b bool
    err = rlp.DecodeBytes(writer.Bytes(), &b)
    //b:true
    fmt.Printf("b=%v\n", b)

    //5.将任意一个struct序列化
    //将一个struct序列化到reader中
    _, r, err := rlp.EncodeToReader(TestRlpStruct{3, "44", []byte{0x12, 0x32}, big.NewInt(32)})
    var teststruct TestRlpStruct
    err = rlp.Decode(r, &teststruct)
    //{A:0x3, B:"44", C:[]uint8{0x12, 0x32}, BigInt:32}
    fmt.Printf("teststruct=%#v\n", teststruct)

}
```