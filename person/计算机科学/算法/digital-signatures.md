### 数字签名（Digital Signatures）架构解析

数字签名是密码学核心机制，用于验证数字文档或消息的真实性、完整性和不可否认性。其基于公钥密码学，允许签名者用私钥生成签名，验证者用公钥验证签名。以下从架构角度系统阐述原理、算法和实现。

#### 1. 核心原理与安全属性
数字签名方案必须满足三个核心安全属性：
1. **真实性（Authenticity）**：签名可证明消息来自声称的发送者
2. **完整性（Integrity）**：签名确保消息未被篡改
3. **不可否认性（Non-repudiation）**：签名者事后不能否认签名行为

数学模型上，数字签名方案定义为三元组 \((Gen, Sign, Verify)\)：
- \(Gen(1^\lambda) \rightarrow (pk, sk)\)：生成公钥/私钥对的安全参数化算法
- \(Sign(sk, m) \rightarrow \sigma\)：用私钥对消息 \(m\) 生成签名 \(\sigma\)
- \(Verify(pk, m, \sigma) \rightarrow \{0,1\}\)：用公钥验证签名，返回接受/拒绝

安全性要求满足**存在不可伪造性（EUF-CMA）**：即使攻击者可获取多个消息的签名，也无法伪造新消息的有效签名。

#### 2. 主要算法分类

##### 2.1 基于数论难题的签名方案
###### RSA签名方案
基于大整数分解难题，密钥生成：
1. 选择大素数 \(p,q\)，计算 \(n = p \times q\)
2. 选择 \(e\) 满足 \(\gcd(e, \phi(n)) = 1\)，其中 \(\phi(n) = (p-1)(q-1)\)
3. 计算 \(d \equiv e^{-1} \pmod{\phi(n)}\)

签名生成：
\[
\sigma = H(m)^d \mod n
\]
签名验证：
\[
Verify: H(m) \stackrel{?}{=} \sigma^e \mod n
\]
其中 \(H\) 是密码学哈希函数（如SHA-256）。

###### DSA（Digital Signature Algorithm）
基于离散对数难题，参数：
- 大素数 \(p\)（模数），\(q\)（子群阶，\(q|p-1\)）
- 生成元 \(g \in \mathbb{Z}_p^*\) 满足 \(g^q \equiv 1 \pmod{p}\)

签名生成：
1. 选择随机 \(k \in [1, q-1]\)
2. 计算 \(r = (g^k \mod p) \mod q\)
3. 计算 \(s = k^{-1}(H(m) + x \cdot r) \mod q\)
签名： \((r, s)\)

验证：
1. 计算 \(w = s^{-1} \mod q\)
2. 计算 \(u_1 = H(m) \cdot w \mod q\), \(u_2 = r \cdot w \mod q\)
3. 计算 \(v = (g^{u_1} y^{u_2} \mod p) \mod q\)
4. 接受当且仅当 \(v = r\)

##### 2.2 基于椭圆曲线的签名方案
###### ECDSA（Elliptic Curve DSA）
将DSA移植到椭圆曲线群，安全性基于ECDLP（椭圆曲线离散对数问题）。参数：
- 椭圆曲线 \(E\) 定义在有限域 \(GF(p)\) 或 \(GF(2^m)\)
- 基点 \(G\) 的阶为 \(n\)

签名生成类似DSA，但运算在椭圆曲线群进行：
\[
R = k \cdot G,\quad r = x_R \mod n,\quad s = k^{-1}(H(m) + d \cdot r) \mod n
\]
其中 \(d\) 为私钥，公钥 \(Q = d \cdot G\)。

##### 2.3 基于哈希的签名方案
###### Lamport一次性签名
基于密码学哈希函数，每个密钥对只能签名一次：
- 私钥：两组随机串 \(\{x_{i,0}, x_{i,1}\}_{i=1}^n\)
- 公钥：对应的哈希值 \(\{H(x_{i,0}), H(x_{i,1})\}_{i=1}^n\)
- 对消息 \(m\) 的每个比特 \(b_i\)，释放 \(x_{i,b_i}\) 作为签名

###### Merkle签名方案（MSS）
通过Merkle树将多个Lamport密钥聚合，实现多次签名。

##### 2.4 后量子签名方案
抵抗量子计算攻击的新型方案：
- **基于格的签名**：如BLISS、Dilithium
- **基于多变量的签名**：如Rainbow
- **基于哈希的签名**：SPHINCS+

#### 3. 性能与安全权衡
| 方案类型 | 签名大小 | 计算开销 | 安全假设 | 适用场景 |
|---------|---------|---------|---------|---------|
| RSA | 大（2048+比特） | 高（模幂） | 分解难题 | 通用应用 |
| ECDSA | 小（256-512比特） | 中 | ECDLP | 区块链、移动设备 |
| 基于哈希 | 极大 | 低 | 哈希函数 | 后量子过渡 |
| 基于格 | 中等 | 中高 | LWE/SIS | 后量子标准 |

#### 4. 实现架构
##### 4.1 密钥生成模块
```python
def key_gen(algorithm, params):
    if algorithm == "RSA":
        p, q = generate_large_primes(params.key_size)
        n = p * q
        phi = (p-1)*(q-1)
        e = choose_public_exponent(phi)
        d = mod_inverse(e, phi)
        return (e, n), (d, n)
    elif algorithm == "ECDSA":
        curve = select_curve(params.curve_name)
        private_key = random_int(1, curve.n-1)
        public_key = scalar_mult(private_key, curve.G)
        return public_key, private_key
```

##### 4.2 签名生成引擎
通常包含：
- 哈希计算模块（SHA-2/SHA-3）
- 随机数生成器（CSPRNG）
- 核心签名运算单元（模幂/椭圆曲线点乘）

##### 4.3 验证引擎
实现并行验证和批量验证优化，特别是对于区块链等需要验证大量签名的场景。

#### 5. 标准化与应用
- **PKCS#1**：RSA签名标准
- **FIPS 186-5**：DSA和ECDSA标准
- **RFC 8032**：EdDSA标准（Ed25519）

应用场景：
- SSL/TLS证书验证
- 代码签名（软件分发）
- 区块链交易授权
- 文档电子签名（符合eIDAS法规）

#### 6. 安全考虑
1. **随机数质量**：DSA/ECDSA中 \(k\) 的重用或偏差会导致私钥泄露
2. **侧信道攻击**：防护定时攻击、功耗分析
3. **量子威胁**：RSA和ECC面临Shor算法威胁，需向后量子密码迁移

数字签名是现代数字信任体系的基石，架构设计需平衡安全性、性能和标准化要求。