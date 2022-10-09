# 零知识snark电路

**本文引自[simmel_](https://blog.csdn.net/simmel_92/article/details/119514688)。在实验过程中没有出现任何报错。**



**前言**：本文我们使用两个公开库circom、snarkjs尝试构建一个简单的零知识证明。其中，[circom库](https://github.com/iden3/circom)用于构建和编译代数电路，[snarkjs库](https://github.com/iden3/snarkjs)是zk-snarks协议的独立实现。本文使用的方法主要参考的是[snarkjs库0.4.6官方教程（英文）](https://www.npmjs.com/package/snarkjs )和[博文circom与snarkjs经典教程](https://learnblockchain.cn/article/1078 )。

本文使用windos 的DOS命令



## 准备：安装Node.js（不多介绍） 

## 1. 安装circom&snarkjs

在终端输入如下安装命令：

```
npm install -g circom
npm install -g snarkjs
```

```
C:\Users\userli>npm install -g circom
npm WARN deprecated circom@0.5.46: Package no longer supported. Contact Support at https://www.npmjs.com/support for more info.

added 133 packages in 11s

C:\Users\userli>npm install -g snarkjs

added 38 packages in 10s
```



## 2. 可信设置

### 2.1 创建一个新文件夹，用于包含后续所有文件：

我是手动创建

```
mkdir factor
cd factor
```

### 2.2 执行new命令：

```
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
```

* 解释：

[new]：用于开启一个新的 powers of tau ceremony （该叫法摘自[snarkjs库0.4.6官方教程](https://www.npmjs.com/package/snarkjs)）

[bn128]：选择bn128曲线（可选的还有bls12-381曲线，bn和bls的简介见后面“补充”部分）

[12]：规定了我们接下来要构建的电路的两个限制。限制1：电路中constraints（稍后会解释）的数量不能多于个；限制2：电路中parameter的数量不能多于
$$
2^{28} \approx 268 \times 10^6
$$
个。


* 效果：

此条命令会创建一个pot12_0000.ptau文件，执行命令后会打印以下字段：

```
E:\factor>snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
[DEBUG] snarkJS: Calculating First Challenge Hash
[DEBUG] snarkJS: Calculate Initial Hash: tauG1
[DEBUG] snarkJS: Calculate Initial Hash: tauG2
[DEBUG] snarkJS: Calculate Initial Hash: alphaTauG1
[DEBUG] snarkJS: Calculate Initial Hash: betaTauG1
[DEBUG] snarkJS: Blank Contribution Hash:
                786a02f7 42015903 c6c6fd85 2552d272
                912f4740 e1584761 8a86e217 f71f5419
                d25e1031 afee5853 13896444 934eb04b
                903a685b 1448b755 d56f701a fe9be2ce
[INFO]  snarkJS: First Contribution Hash:
                9e63a5f6 2b96538d aaed2372 481920d1
                a40b9195 9ea38ef9 f5f6a303 3b886516
                0710d067 c09d0961 5f928ea5 17bcdf49
                ad75abd2 c8340b40 0e3b18e9 68b4ffef
```



* 补充

参考博文[zk-SNARK零知识证明曲线选择](https://blog.csdn.net/mutourend/article/details/92784689)及[Electric Coin Co. 官网](https://electriccoin.co/blog/new-snark-curve/)的相关内容

Barreto-Naehrig (BN) 和Barreto-Lynn-Scott (BLS)都是pairing-friendly椭圆曲线。不同之处在于，BN曲线若想要达到128-bit的安全性，需要q≈2384，相应的BN曲线的阶数r也会提高到2384量级，r值的增大会影响multi-exponentiation、FFT等运算性能，从而影响zk-SNARK以及安全多方计算的执行效率，同时也会影响key文件不必要的增大。 而对于BLS曲线，当q≈2384 且 embedding degree k=12时，具有128-bit的安全性，而相应的阶数r≈2256，远小于BN曲线的2384量级。


### 2.3 执行contribute命令

```
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
```

* 解释：

[contribute] 该命令将上一步创建的pot12_0000.ptau文件作为输入，并输出一个新的ptau文件pot12_0001.ptau。ptau文件包含所有已经进行过的challenge和response计算。

[name] 这一字段内你可以任意填入一些内容。在后面执行验证（第6步）的时候这个内容会被打印出来。

* 效果：

执行此条命令，会被要求输入一些随机内容，输入后打印如下内容：

```
E:\factor>snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
Enter a random text. (Entropy): First
[DEBUG] snarkJS: Calculating First Challenge Hash
[DEBUG] snarkJS: Calculate Initial Hash: tauG1
[DEBUG] snarkJS: Calculate Initial Hash: tauG2
[DEBUG] snarkJS: Calculate Initial Hash: alphaTauG1
[DEBUG] snarkJS: Calculate Initial Hash: betaTauG1
[DEBUG] snarkJS: processing: tauG1: 0/8191
[DEBUG] snarkJS: processing: tauG2: 0/4096
[DEBUG] snarkJS: processing: alphaTauG1: 0/4096
[DEBUG] snarkJS: processing: betaTauG1: 0/4096
[DEBUG] snarkJS: processing: betaTauG2: 0/1
[INFO]  snarkJS: Contribution Response Hash imported:
                eab27788 a2e94ffc 295e677b a617a2fd
                6674cbc5 fd544518 5ff0b91f 64dd3182
                2e6a33b6 2881095d 85b29cfc 063149e3
                403d755d e770cf92 46519672 011cc44e
[INFO]  snarkJS: Next Challenge Hash:
                eb4b4650 6c6f3c7a 6329dd30 0895cb15
                24f59d72 374acab9 a8e881d0 7fa87482
                ca363d68 dc912eb7 5707e77b 83f5fb9d
                439f15a1 f3bdd1e3 2b857299 cf7aa30c
```

### 2.4 第二次contribute

```
snarkjs powersoftau contribute pot12_0001.ptau pot12_0002.ptau --name="Second contribution" -v -e="some random text"
```

* 解释：

此条命令与上一条功能相同，不同点在于，加入了[-e]命令，直接输入了一些随机值（上一步是交互式地输入随机值，这一步直接将随机值写入了命令中）。

* 效果：

执行此条命令，打印如下内容：

```
E:\factor>snarkjs powersoftau contribute pot12_0001.ptau pot12_0002.ptau --name="Second contribution" -v -e="some random text"
[DEBUG] snarkJS: processing: tauG1: 0/8191
[DEBUG] snarkJS: processing: tauG2: 0/4096
[DEBUG] snarkJS: processing: alphaTauG1: 0/4096
[DEBUG] snarkJS: processing: betaTauG1: 0/4096
[DEBUG] snarkJS: processing: betaTauG2: 0/1
[INFO]  snarkJS: Contribution Response Hash imported:
                22c8eca3 49242792 61096622 2e185193
                eda695b1 38539544 a54d256a bae2e71b
                2d13762d c2fa9deb 1ad4bae4 7a7f2e5f
                7dd726f9 42ef159d a8be5d42 ee4ce593
[INFO]  snarkJS: Next Challenge Hash:
                b0dd0df9 3ce8ada1 5660f253 4c796104
                c088e07d f6189496 f84e2475 26923c33
                7bc2b247 edd83239 2b7d462a 741b209c
                f9be1b7a 2589c1f7 947d9b53 4991ff77
```



### 2.5 第三次contribute（第三方contribute）

```
snarkjs powersoftau export challenge pot12_0002.ptau challenge_0003
snarkjs powersoftau challenge contribute bn128 challenge_0003 response_0003 -e="some random text"
snarkjs powersoftau import response pot12_0002.ptau response_0003 pot12_0003.ptau -n="Third contribution name"
```

不太确定这步跟上两步的异同，但是执行完成后会生成pot12_0003.ptau文件和相关文件。



### 2.6 验证

```
snarkjs powersoftau verify pot12_0003.ptau
```

如果验证通过，在打印信息的第一行会显示“[INFO] snarkJS: Powers Of tau file OK!” ：

```
E:\factor>snarkjs powersoftau verify pot12_0003.ptau
[INFO]  snarkJS: Powers Of tau file OK!
[INFO]  snarkJS: Next challenge hash:
                6aa4d9a2 6a59e90e 266539ae c5ec75b3
                12bff2da 03962e69 2f4efe0f d2ce10a8
                1a4af897 29e20524 08390f5d 4f088eb0
                24ce1add 5099c6cd e283c665 e92695cd
[INFO]  snarkJS: -----------------------------------------------------
[INFO]  snarkJS: Contribution #3: Third contribution name
[INFO]  snarkJS: Next Challenge:
                6aa4d9a2 6a59e90e 266539ae c5ec75b3
                12bff2da 03962e69 2f4efe0f d2ce10a8
                1a4af897 29e20524 08390f5d 4f088eb0
                24ce1add 5099c6cd e283c665 e92695cd
[INFO]  snarkJS: Response Hash:
                79262b40 03d4f337 cf059d4b 2f274deb
                839d33f5 c671874c cb8612fd e6da1ab1
                11949a3e 20c22fc4 55ce2efc 43cff8ad
                008361fe 2a997dab 29d4c66f 6206ee74
[INFO]  snarkJS: Response Hash:
                b0dd0df9 3ce8ada1 5660f253 4c796104
                c088e07d f6189496 f84e2475 26923c33
                7bc2b247 edd83239 2b7d462a 741b209c
                f9be1b7a 2589c1f7 947d9b53 4991ff77
[INFO]  snarkJS: Powers Of tau file OK!
[INFO]  snarkJS: -----------------------------------------------------
[INFO]  snarkJS: Contribution #2: Second contribution
[INFO]  snarkJS: Next Challenge:
                b0dd0df9 3ce8ada1 5660f253 4c796104
                c088e07d f6189496 f84e2475 26923c33
                7bc2b247 edd83239 2b7d462a 741b209c
                f9be1b7a 2589c1f7 947d9b53 4991ff77
[INFO]  snarkJS: Response Hash:
                22c8eca3 49242792 61096622 2e185193
                eda695b1 38539544 a54d256a bae2e71b
                2d13762d c2fa9deb 1ad4bae4 7a7f2e5f
                7dd726f9 42ef159d a8be5d42 ee4ce593
[INFO]  snarkJS: Response Hash:
                eb4b4650 6c6f3c7a 6329dd30 0895cb15
                24f59d72 374acab9 a8e881d0 7fa87482
                ca363d68 dc912eb7 5707e77b 83f5fb9d
                439f15a1 f3bdd1e3 2b857299 cf7aa30c
[INFO]  snarkJS: Powers Of tau file OK!
[INFO]  snarkJS: -----------------------------------------------------
[INFO]  snarkJS: Contribution #1: First contribution
[INFO]  snarkJS: Next Challenge:
                eb4b4650 6c6f3c7a 6329dd30 0895cb15
                24f59d72 374acab9 a8e881d0 7fa87482
                ca363d68 dc912eb7 5707e77b 83f5fb9d
                439f15a1 f3bdd1e3 2b857299 cf7aa30c
[INFO]  snarkJS: Response Hash:
                eab27788 a2e94ffc 295e677b a617a2fd
                6674cbc5 fd544518 5ff0b91f 64dd3182
                2e6a33b6 2881095d 85b29cfc 063149e3
                403d755d e770cf92 46519672 011cc44e
[INFO]  snarkJS: Response Hash:
                9e63a5f6 2b96538d aaed2372 481920d1
                a40b9195 9ea38ef9 f5f6a303 3b886516
                0710d067 c09d0961 5f928ea5 17bcdf49
                ad75abd2 c8340b40 0e3b18e9 68b4ffef
[INFO]  snarkJS: -----------------------------------------------------
[WARN]  snarkJS: this file does not contain phase2 precalculated values. Please run:
   snarkjs "powersoftau preparephase2" to prepare this file to be used in the phase2 ceremony.
[INFO]  snarkJS: Powers of Tau Ok!
```



### 2.7 引入random beacon

```
snarkjs powersoftau beacon pot12_0003.ptau pot12_beacon.ptau 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon"
```

 此条命令会生成一个pot12_beacon.ptau文件。

打印结果如下：

```
E:\factor>snarkjs powersoftau beacon pot12_0003.ptau pot12_beacon.ptau 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon"
[INFO]  snarkJS: Contribution Response Hash imported:
                4cc6917c 58dc4455 540bcf9f 997e047d
                86b120e6 a322ba30 6882f66f b703d264
                57c590ac 10dbbb55 485c65e9 98118c07
                6766a61b 1f71aa8e 153295f0 f117a70a
[INFO]  snarkJS: Next Challenge Hash:
                8407ca57 0876c7b0 bd5ba1b8 31d0e034
                7b7418ac ac22dee5 859bd8f5 7046c543
                eee73375 9a4e2104 ea4f9a20 c7568390
                0b656544 303c379a 6eea888a 7a372bb0
```



### 2.8 生成公开信息

```
snarkjs powersoftau prepare phase2 pot12_beacon.ptau pot12_final.ptau -v
```

该过程需要一点时间



* 解释：

[prepare phase2]：生成多项式在点
$$
\tau
$$
、点
$$
\alpha \cdot \tau
$$
、点
$$
\beta \cdot \tau
$$
的值的密文。

此命令将pot12_beacon.ptau文件作为输入，生成一个新文件pot12_final.ptau。

部分打印结果如下：

```
E:\factor>snarkjs powersoftau prepare phase2 pot12_beacon.ptau pot12_final.ptau -v
[DEBUG] snarkJS: Starting section: tauG1
[DEBUG] snarkJS: tauG1: fft 0 mix start: 0/1
[DEBUG] snarkJS: tauG1: fft 0 mix end: 0/1
[DEBUG] snarkJS: tauG1: fft 1 mix start: 0/1
[DEBUG] snarkJS: tauG1: fft 1 mix end: 0/1
[DEBUG] snarkJS: tauG1: fft 2 mix start: 0/1
[DEBUG] snarkJS: tauG1: fft 2 mix end: 0/1
[DEBUG] snarkJS: tauG1: fft 3 mix start: 0/1
[DEBUG] snarkJS: tauG1: fft 3 mix end: 0/1
[DEBUG] snarkJS: tauG1: fft 4 mix start: 0/2
[DEBUG] snarkJS: tauG1: fft 4 mix start: 1/2
[DEBUG] snarkJS: tauG1: fft 4 mix end: 0/2
[DEBUG] snarkJS: tauG1: fft 4 mix end: 1/2
[DEBUG] snarkJS: tauG1: fft  4  join: 4/4
```



### 2.9 最终验证

```
snarkjs powersoftau verify pot12_final.ptau
```

* 解释：

在开始下一步（构建电路）之前，执行一个最终验证。

* 效果：

执行此条命令，部分打印信息如下：

```
E:\factor>snarkjs powersoftau verify pot12_final.ptau
[INFO]  snarkJS: Powers Of tau file OK!
[INFO]  snarkJS: Next challenge hash:
                8407ca57 0876c7b0 bd5ba1b8 31d0e034
                7b7418ac ac22dee5 859bd8f5 7046c543
                eee73375 9a4e2104 ea4f9a20 c7568390
                0b656544 303c379a 6eea888a 7a372bb0
[INFO]  snarkJS: -----------------------------------------------------
[INFO]  snarkJS: Contribution #4: Final Beacon
[INFO]  snarkJS: Next Challenge:
                8407ca57 0876c7b0 bd5ba1b8 31d0e034
                7b7418ac ac22dee5 859bd8f5 7046c543
                eee73375 9a4e2104 ea4f9a20 c7568390
                0b656544 303c379a 6eea888a 7a372bb0
[INFO]  snarkJS: Response Hash:
                4cc6917c 58dc4455 540bcf9f 997e047d
                86b120e6 a322ba30 6882f66f b703d264
                57c590ac 10dbbb55 485c65e9 98118c07
                6766a61b 1f71aa8e 153295f0 f117a70a
[INFO]  snarkJS: Response Hash:
                6aa4d9a2 6a59e90e 266539ae c5ec75b3
                12bff2da 03962e69 2f4efe0f d2ce10a8
                1a4af897 29e20524 08390f5d 4f088eb0
                24ce1add 5099c6cd e283c665 e92695cd
```



# 3.  构建电路 

我们准备证明的问题是：我们知道数字a和b（秘密），它们相乘得到c（公开值）。

### 3.1 创建一个circom文件：

我是手动创建

```
vim circuit.circom
```

内容为如下：

```
template Multiplier() {
   signal private input a;
   signal private input b;
   signal output c;
   c <== a*b;
}

component main = Multiplier();
```

* 解释：

2-4行：这个电路有两个private输入信号，即a和b；有一个输出信号，即c。

第5行：输入和输出使用<==运算符进行关联。 在circom中，<==运算符做两件事。 首先是连接信号。 第二个是施加约束。在本例中，我们使用<==将c连接到a和b，同时将c约束为a * b的值，即电路做的事情是让强制信号 c 为 a*b的值。

第8行：在声明 Multiplier 模板之后, 我们使用名为main的组件实例化它。编译电路时，必须始终有一个名为main的组件。


（对第5行和第8行的解释摘自博文[circom与snarkjs经典教程](https://learnblockchain.cn/article/1078)）



### 3.2 编译电路

```
circom circuit.circom --r1cs --wasm --sym -v
```

打印内容如下：

```
E:\factor>circom circuit.circom --r1cs --wasm --sym -v
NConstraints Before: 1
NSignals Before: 4
Reduce Constants
reducing constants:  0
Reduce Constraints
indexing constraints: 0/1
reducing constraints: 0/1   reduced: 0
Removed: 0 TotalConstraints: 1
reordering constraints: 0/1
NConstraints After: 1
Classify Signals
marking as internal: 0/1
classify signals: 0/4
generate witness (counting):  0
seting id:  0
writing constraint:  0
writing wire2label map: 0/4
Generating wasm...
buildHeader...
buildEntryTables...
buildEntryTables component: 0/1
buildCode...
buildCode component: 0/1
buildComponentsArray...
buildComponentsArray component: 0/1
buildMapIsInput...
buildMapIsInput signal: 0/4
buildWit2Sig...
buildWit2Sig signal: 0/4
Symbols saved: 0
constructionPhase: .009
reduceConstants: .004
reduceConstraints: .006
classifySignals: .000
generateWitnessNames: .000
generateR1cs: .015
generateWasm: .348
generateSyms: .001
```



### 3.3 用snarkjs查看编译后的电路

```
snarkjs r1cs info circuit.r1cs
```

打印内容如下：

```
E:\factor>snarkjs r1cs info circuit.r1cs
[INFO]  snarkJS: Curve: bn-128
[INFO]  snarkJS: # of Wires: 4
[INFO]  snarkJS: # of Constraints: 1
[INFO]  snarkJS: # of Private Inputs: 2
[INFO]  snarkJS: # of Public Inputs: 0
[INFO]  snarkJS: # of Labels: 4
[INFO]  snarkJS: # of Outputs: 1
```



### 3.4 打印constraints

```
snarkjs r1cs print circuit.r1cs circuit.sym
```

打印内容如下：

```
E:\factor>snarkjs r1cs print circuit.r1cs circuit.sym
[INFO]  snarkJS: [ 21888242871839275222246405745257275088548364400416034343698204186575808495616main.a ] * [ main.b ] - [ 21888242871839275222246405745257275088548364400416034343698204186575808495616main.c ] = 0
```



### 3.5 将电路输出成json格式

```
snarkjs r1cs export json circuit.r1cs circuit.r1cs.json
cat circuit.r1cs.json
```

执行这两条语句后会打印json字符串。

输出内容如下：

```
E:\factor>snarkjs r1cs export json circuit.r1cs circuit.r1cs.json
[INFO]  snarkJS: undefined: Loading constraints: 0/1
[INFO]  snarkJS: undefined: Loading map: 0/4

E:\factor>cat circuit.r1cs.json
'cat' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
```

由于我用dos命令，无法执行cat命令，不过它生成一个json文件，我们可以打开看看，内容如下：

```json
{
 "n8": 32,
 "prime": "21888242871839275222246405745257275088548364400416034343698204186575808495617",
 "nVars": 4,
 "nOutputs": 1,
 "nPubInputs": 0,
 "nPrvInputs": 2,
 "nLabels": 4,
 "nConstraints": 1,
 "useCustomGates": false,
 "constraints": [
  [
   {
    "2": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
   },
   {
    "3": "1"
   },
   {
    "1": "21888242871839275222246405745257275088548364400416034343698204186575808495616"
   }
  ]
 ],
 "map": [
  0,
  3,
  1,
  2
 ],
 "customGates": [
 ],
 "customGatesUses": [
 ]
}
```



# 4. 生成groth16算法密钥 

** 步骤4.2--4.7类似于上文步骤2.3--2.7, 2.9 **



### 4.1 设置

```
snarkjs groth16 setup circuit.r1cs pot12_final.ptau circuit_0000.zkey
```

* 解释：

此命令生成zkey，即零知识证明密钥，包含证明密钥（proving key）和验证密钥（verification key）。每个电路都单独生成一个zkey，我们可以验证一个zkey是否属于某个电路。



输出内容如下：

```
E:\factor>snarkjs groth16 setup circuit.r1cs pot12_final.ptau circuit_0000.zkey
[INFO]  snarkJS: Reading r1cs
[INFO]  snarkJS: Reading tauG1
[INFO]  snarkJS: Reading tauG2
[INFO]  snarkJS: Reading alphatauG1
[INFO]  snarkJS: Reading betatauG1
[INFO]  snarkJS: Circuit hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
```





### 4.2 zkey Contribute

```
snarkjs zkey contribute circuit_0000.zkey circuit_0001.zkey --name="1st Contributor Name" -v
```

此命令会生成一个新的circuit_0001.zkey文件，里面包含了一个contribution。这一步会要求输入一些随机内容，如下：

```
E:\factor>snarkjs zkey contribute circuit_0000.zkey circuit_0001.zkey --name="1st Contributor Name" -v
Enter a random text. (Entropy): First
[DEBUG] snarkJS: Applying key: L Section: 0/2
[DEBUG] snarkJS: Applying key: H Section: 0/4
[INFO]  snarkJS: Circuit Hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
[INFO]  snarkJS: Contribution Hash:
                5b7c735e 7c57a114 603789f2 db365836
                3ca87da8 a1700847 785cc886 5135d651
                143646fc 9f367c17 843dcd23 034fc46e
                587aed4d 6a39c23e 02efa78c af8e83e0
```



### 4.3 第二次contribute

```
snarkjs zkey contribute circuit_0001.zkey circuit_0002.zkey --name="Second contribution Name" -v -e="Another random entropy"
```

输出内容如下：

```
E:\factor>snarkjs zkey contribute circuit_0001.zkey circuit_0002.zkey --name="Second contribution Name" -v -e="Another random entropy"
[DEBUG] snarkJS: Applying key: L Section: 0/2
[DEBUG] snarkJS: Applying key: H Section: 0/4
[INFO]  snarkJS: Circuit Hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
[INFO]  snarkJS: Contribution Hash:
                65910ed6 56ef3f26 2258193b d9fc8277
                50613eb5 0e0ae596 b58f97c2 93cf3473
                83ae381c 5b914dc2 a2db0b40 3aa297cf
                d47552b0 29108a7b b7923c61 9dab0031	
```



### 4.4 第三次contribution（第三方contribute）

```
snarkjs zkey export bellman circuit_0002.zkey  challenge_phase2_0003
snarkjs zkey bellman contribute bn128 challenge_phase2_0003 response_phase2_0003 -e="some random text"
snarkjs zkey import bellman circuit_0002.zkey response_phase2_0003 circuit_0003.zkey -n="Third contribution name"
```



### 4.5 验证

```
snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_0003.zkey
```

输出内容如下：

```
E:\factor>snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_0003.zkey
[INFO]  snarkJS: Reading r1cs
[INFO]  snarkJS: Reading tauG1
[INFO]  snarkJS: Reading tauG2
[INFO]  snarkJS: Reading alphatauG1
[INFO]  snarkJS: Reading betatauG1
[INFO]  snarkJS: Circuit hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
[INFO]  snarkJS: Circuit Hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #3 Third contribution name:
                49834ba0 2b0f1904 5ac2c6d7 60b0eb24
                6bc8faf6 ef754b0d a1eea80a 156b3d1f
                31ffa938 a7bac3d7 aab32c38 613aba00
                38304a57 98478803 fccc762b f2ff87c8
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #2 Second contribution Name:
                65910ed6 56ef3f26 2258193b d9fc8277
                50613eb5 0e0ae596 b58f97c2 93cf3473
                83ae381c 5b914dc2 a2db0b40 3aa297cf
                d47552b0 29108a7b b7923c61 9dab0031
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #1 1st Contributor Name:
                5b7c735e 7c57a114 603789f2 db365836
                3ca87da8 a1700847 785cc886 5135d651
                143646fc 9f367c17 843dcd23 034fc46e
                587aed4d 6a39c23e 02efa78c af8e83e0
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: ZKey Ok!
```



### 4.6 引入random beacon

```
snarkjs zkey beacon circuit_0003.zkey circuit_final.zkey 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon phase2"
```

输出内容如下：

```
E:\factor>snarkjs zkey beacon circuit_0003.zkey circuit_final.zkey 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f 10 -n="Final Beacon phase2"
[INFO]  snarkJS: Contribution Hash:
                b4f4e176 047799f8 5a5ca33b 2deb1b66
                fb0f52fb 4e1298e9 5d486e72 ff93a063
                6356d9ff d07b8794 f5a00367 47672066
                1a451b8a f7a18209 973c9998 13891485
```



### 4.7 验证最终zkey

```
snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_final.zkey
```

输出内容如下：

```
E:\factor>snarkjs zkey verify circuit.r1cs pot12_final.ptau circuit_final.zkey
[INFO]  snarkJS: Reading r1cs
[INFO]  snarkJS: Reading tauG1
[INFO]  snarkJS: Reading tauG2
[INFO]  snarkJS: Reading alphatauG1
[INFO]  snarkJS: Reading betatauG1
[INFO]  snarkJS: Circuit hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
[INFO]  snarkJS: Circuit Hash:
                f83cc2c3 3f176240 69847be7 92fb3bbf
                4cfd3199 60ebd337 f70f94f8 94c7241c
                2045fbdd fa6fdd58 cf4c42ff 23e9d6e3
                5ccdb3b7 740c155f a2a03d60 d2ce94de
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #4 Final Beacon phase2:
                b4f4e176 047799f8 5a5ca33b 2deb1b66
                fb0f52fb 4e1298e9 5d486e72 ff93a063
                6356d9ff d07b8794 f5a00367 47672066
                1a451b8a f7a18209 973c9998 13891485
[INFO]  snarkJS: Beacon generator: 0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f
[INFO]  snarkJS: Beacon iterations Exp: 10
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #3 Third contribution name:
                49834ba0 2b0f1904 5ac2c6d7 60b0eb24
                6bc8faf6 ef754b0d a1eea80a 156b3d1f
                31ffa938 a7bac3d7 aab32c38 613aba00
                38304a57 98478803 fccc762b f2ff87c8
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #2 Second contribution Name:
                65910ed6 56ef3f26 2258193b d9fc8277
                50613eb5 0e0ae596 b58f97c2 93cf3473
                83ae381c 5b914dc2 a2db0b40 3aa297cf
                d47552b0 29108a7b b7923c61 9dab0031
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: contribution #1 1st Contributor Name:
                5b7c735e 7c57a114 603789f2 db365836
                3ca87da8 a1700847 785cc886 5135d651
                143646fc 9f367c17 843dcd23 034fc46e
                587aed4d 6a39c23e 02efa78c af8e83e0
[INFO]  snarkJS: -------------------------
[INFO]  snarkJS: ZKey Ok!
```



### 4.8 输出验证密钥

```
snarkjs zkey export verificationkey circuit_final.zkey verification_key.json
```

它生成一个verification_key.json文件，我们可以打开看看，内容如下：

```
{
 "protocol": "groth16",
 "curve": "bn128",
 "nPublic": 1,
 "vk_alpha_1": [
  "14307847838448837795045810475061986504553509081614953114476858712850383484856",
  "14397350453820464332153016551648509942758452718080193709935093645324545685527",
  "1"
 ],
 "vk_beta_2": [
  [
   "16030077851365913589678456150106175177430552653688674551594118856213195035003",
   "3404532281697646418626793199959763417273544462603652136362105780844753431558"
  ],
  [
   "19124466857712679107014297995594956562782725602714494550271088050395887469588",
   "9844174518424602023011839524648797909751775208079518310859648986295054482129"
  ],
  [
   "1",
   "0"
  ]
 ],
 "vk_gamma_2": [
  [
   "10857046999023057135944570762232829481370756359578518086990519993285655852781",
   "11559732032986387107991004021392285783925812861821192530917403151452391805634"
  ],
  [
   "8495653923123431417604973247489272438418190587263600148770280649306958101930",
   "4082367875863433681332203403145435568316851327593401208105741076214120093531"
  ],
  [
   "1",
   "0"
  ]
 ],
 "vk_delta_2": [
  [
   "10113197364502701822513954479568648288291692883581942979080642768873065643831",
   "11175515265368712315132910457117027139422582204527334913723306160996274553120"
  ],
  [
   "15051946769612120764329402607627420160258615710779168077702721273012743113432",
   "4873115357887694599733887830039362963182322006895073463008390511862763231838"
  ],
  [
   "1",
   "0"
  ]
 ],
 "vk_alphabeta_12": [
  [
   [
    "7044056860817777654154509959660674355015050831320610057443065068322910565895",
    "10331625462092170687640274745982378335372748458872831306252323452126814554131"
   ],
   [
    "18040503605850356525188459549959222258687874239992141206912024646060897474317",
    "18827187281144666860613305385540784866960018135708380822998402712655947973564"
   ],
   [
    "11274378994751172006468526902468597534765251990849217320327040581320759012660",
    "19274558546592248969003452180476475614917525690465556662049817190926800629144"
   ]
  ],
  [
   [
    "9579292205767277785037267514941301295326092324814566495476405245670217705444",
    "8189968080643184307415548548147525387352724344814096244185751084846761700503"
   ],
   [
    "13462879787812569224049377655370051392827547945190233465440516223457141218238",
    "9231269590310307265622085951916792582849815072877368008082912498798765970911"
   ],
   [
    "3885746965337606777418900043385187742854223068345383890852606753920650561905",
    "5812591705543262214135984565509356725932745303661857366540167099077764681654"
   ]
  ]
 ],
 "IC": [
  [
   "11994672791283902276192880120921491507552417245216260234350788916775631529054",
   "2115307076939231963712007551115185680030078618410449316262149755029753574980",
   "1"
  ],
  [
   "17532675471561472612487375934612816062261862756871282335618043817123285063744",
   "349604453999632063807404304322605636452464906050035288629813788746930833504",
   "1"
  ]
 ]
}
```





# 5. 计算见证（witness）

witness中包含与已知电路所有constraints相匹配的信号，可以理解为问题的答案。在我们的例子中，一个见证包含a、b、c。我们先生成witness，然后再用witness生成零知识证明。

### 5.1 创建input文件

创建一个json文件：

我是手动创建

```
vim input.json
```

在json文件中写入如下内容并保存：

```
{"a": 3, "b": 11}
```



### 5.2 运行计算命令

```
snarkjs wtns calculate circuit.wasm input.json witness.wtns
```



### 5.3 检查witness计算有无错误

```
snarkjs wtns debug circuit.wasm input.json witness.wtns circuit.sym --trigger --get --set
```

* 效果：

此命令会打印我们输入的a、b以及相应的输出c

输出内容如下：

```
E:\factor>snarkjs wtns debug circuit.wasm input.json witness.wtns circuit.sym --trigger --get --set
[INFO]  snarkJS: SET main.a <-- 3
[INFO]  snarkJS: SET main.b <-- 11
[INFO]  snarkJS: START: main
[INFO]  snarkJS: GET main.a --> 3
[INFO]  snarkJS: GET main.b --> 11
[INFO]  snarkJS: SET main.c <-- 33
[INFO]  snarkJS: FINISH: main
```



# 6. 证明与验证

### 6.1 构建零知识证明

```
snarkjs groth16 prove circuit_final.zkey witness.wtns proof.json public.json
```

* 解释：

这一命令生成文件proof.json和public.json，前者包含证明，后者包含公开输入和公开输出。

* 注：

第5步（计算witness）和第6.1步（生成证明）可以合并为一条语句：

```
snarkjs groth16 fullprove input.json circuit.wasm circuit_final.zkey proof.json public.json
```



### 6.2 验证

```
snarkjs groth16 verify verification_key.json public.json proof.json
```

输出内容如下：

```
E:\factor>snarkjs groth16 verify verification_key.json public.json proof.json
[INFO]  snarkJS: OK!
```

谢谢
