# Sed

## 批量替换字符串
```shell
sed -i "s/oldstring/newstring/g" `grep oldstring -rl path`
```
> 其中，oldstring是待被替换的字符串，newstring是待替换oldstring的新字符串，grep操作主要是按照所给的路径查找oldstring，path是所替换文件的路径  
-i选项是直接在文件中替换，不在终端输出  
-r选项是所给的path中的目录递归查找  
-l选项是输出所有匹配到oldstring的文件 

## 替换单个文件里的内容
```shell
sed -i "s/oldstring/newstring/g" filename
```