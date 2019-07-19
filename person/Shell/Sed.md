# Sed

## 快速语法

### 替换

```shell
sed -i "s/oldstring/newstring/g" filename   # 替换每行所有出现的oldstring字符串【常用】
sed -i "s/oldstring/newstring/" filename    # 替换每行第一次出现的oldstring字符串【少用】
```

### 插入

```bash
sed -i '/oldstring/i newLine' filename      # 在匹配内容上面插入一条新行
sed -i '/oldstring/a newLine' filename      # 特定字符串的行后插入新行
sed -i "3i\newLine" filename                # 在第三行的上方添加一行字符串
sed -i "3a\newLine" filename                # 在第三行的下方添加一行字符串
```

### 删除

```bash
sed -i '8d' filename                # 删除第8行
sed -i '2,5d' filename              # 删除第2到5行
sed -i '2,$d' filename              # 删除第2到最后行
sed -i '/oldstring/d' filename      # 删除包含oldstring关键字的行
```

---

## 命令格式

```bash
sed [options] 'command' file(s)
sed [-nefr] [动作]
```

> 选项与参数：

- `-n` 使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过 sed 特殊处理的那一行(或者动作)才会被列出来。
- `-e` 直接在命令列模式上进行 sed 的动作编辑；
- `-f` 直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；
- `-r` sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
- `-i` 直接修改读取的文件内容，而不是由屏幕输出。

> 动作说明：  [n1[,n2]]function

- `n1, n2` ：不见得会存在，一般代表『选择进行动作的行数』，举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则『 10,20[动作行为] 』

> function 有底下这些咚咚：

- `a`  **新增** `a` 的后面可以接字串，而这些字串会在新的一行出现(下一行)～
- `c`  **取代** `c` 的后面可以接字串，这些字串可以取代 `n1,n2` 之间的行！
- `d`  **删除** 因为是删除啊，所以 `d` 后面通常不跟参数
- `i`  **插入** `i` 的后面可以接字串，而这些字串会在新的一行出现(上一行)；
- `p`  **列印** 亦即将某个选择的数据印出。通常 p 会与参数 `sed -n` 一起运行～
- `s`  **取代** 可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 `1,20s/old/new/g`

---

## 增加

### 指定行号添加内容

```bash
sed -i "3i\test123" aa.txt  # 在第三行的上方添加一行字符串

sed -i "3a\ceshi456" aa.txt  # 在第三行的下方添加一行字符串
```

### 插入行：通过行号插入

```bash
nl /etc/passwd | sed '2,5c 2-5行被吃了'  #2-5行替换成指定的内容

nl /etc/passwd | sed "2i it's second line"  #第二行前面插入内容；参数i

nl /etc/passwd | sed '2a  The third line'  #第二行下面插入内容；参数a
```

### 插入行：匹配关键字前后插入

```bash
sed  -i  "/rm/i\alias vi='vim'"  ~/.bashrc  #在匹配的rm内容上面插入一条vim配置别名的行

grep vi ~/.bashrc || sed  -i  "/mv/a\alias vi='vim'"  ~/.bashrc  #先判断vi内容是否存在，如果不存在则匹配到mv内容在下面插入一行；
```

### 匹配行之后在其上方/下方添加内容

```bash
sed -ne 's/aaa/HELLO&/p' test  #在aaa字符前面插入内容;输出结果：HELLOaaa

sed -ne 's/aaa/&HELLO/p' test #在aaa字符后面插入内容;输出结果：aaaHELLO

sed 's/^/HEAD&/g' file  #在每行的头添加字符，比如"HEAD"

sed 's/$/&TAIL/g' file  #在每行的行尾添加字符，比如“TAIL”
```

### 插入行： 行首、行某插入

```bash
sed '1istart'   /root/.bashrc  #首行添加start字符串

sed '$a end.'   /root/.bashrc  #结尾添加end.内容
```

>
- `a` 代表apend，是在匹配行追加的意思。字母前面跟行号或匹配的内容
- `i` 代表insert，是在匹配行插入的意思。字母前面跟行号或匹配的内容
- `\n` 换行，可通过该参数插入多行内容  
- `\` 转义符

### 换行、空格

```bash
nl /etc/passwd |sed '10a 1第十行后面开始插入三行\n2\n3 斜杠n是换行\tt大空格'  #换行\n 空格\t,空格键小空格

```

### 整行操作

搜索显示

```bash
nl /etc/passwd | sed -n '2p'   #打印第二行,类似于awk NR==2

nl /etc/passwd | sed -n '5,7p'  #打印5-7行

sed -n '/root/p' /etc/passwd   #只显示包含root的行;参数-n只打印处理的行

sed '/nologin/d' /etc/passwd   #删除包含nologin的行，其他输出;d 参数删除
```
