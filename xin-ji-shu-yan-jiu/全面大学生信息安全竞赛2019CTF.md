题目地址：0x455541c3e9179a6cd8C418142855d894e11A288c@ropsten

1.仍然先从给的源码看有用信息。

![image-20200403112433336](image-20200403112433336.png)

又是balanceOf>=10000，要很多的钱。

![image-20200403112503789](image-20200403112503789.png)

0x00为balanceOf，0x01为gift，0x02为owner地址。此时源码没有更多有价值信息，开始审计逆向出来的代码。接下来分析每一个函数。

2.func_01DC()

![image-20200403113351408](image-20200403113351408.png)

未解析出名字的函数，逻辑就是gift=0时，则balanceOf+1，且gift置为1。即空投函数。

3.profit()

![image-20200403134757106](image-20200403134757106.png)

profit函数，逻辑就是balanceOf和gift都必须为1，则balanceOf+1，且gift置为2。

4.transfer(var arg0, var arg1)

![image-20200403140503970](image-20200403140503970.png)

从函数名就能看出来是转账函数，且易知arg0为收款账户，arg1位转账金额。大致逻辑如下。首先要求arg1不能<=1，balanceOf也不能<=1，arg1要<=balanceof，然后就是标准转账函数，当前账户-arg1，收款账户+arg1。

5.transfer2(var arg0, var arg1)

![image-20200403140916019](image-20200403140916019.png)

最后一个函数，看函数名也是一个转账函数，也且易知arg0为收款账户，arg1位转账金额。但肯定有所区别。大致逻辑如下。首先要求arg1不能<=2，balanceOf不能<=2，**balanceOf-arg1不能<=0**，然后就是标准转账函数，当前账户-arg1，收款账户+arg1。这个transfer2函数与上一个函数的重要区别就在于**balanceOf-arg1不能<=0**这一步判断不同。其实二者在**storage[temp0] = storage[temp0] - temp1;**这一步的当前账户减arg1都有整数下溢漏洞。而transfer的判断是arg1和balanceOf直接比大小，而transfer2里是用二者相减再比大小那么如果设置arg1比较大，又由于balanceOf为**uint**型，所以减完balanceOf-arg1会是一个非常大的数，显然会大于0。这里即漏洞利用点。虽然transfer2还有余额大于2的要求，但是通过上面的transfer就可以实现

6.漏洞利用过程

- 所以先准备两个钱包地址（Addr1和Addr2）
- 然后两个地址都执行一遍func_01DC()和profit()，这样两个地址的balanceOf都为2，且gift也为2
- 用Addr1调用transfer函数向Addr2转账2，此时Addr1余额为0，Addr2余额为4
- 之后Addr2调用transfer2函数向Addr1转一个非常大的金额即可。此时两个地址的balanceOf都会非常大，Addr2是因为余额减法下溢造成，Addr1是因为转账数额下溢造成。

