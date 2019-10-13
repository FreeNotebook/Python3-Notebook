## hash 风格的文件存取路径

> 作者: 老菜   来源: [老菜园子](https://github.com/laocaiyuan)

如果我们要存取成千上万的文件，如果这时候采取手工分类的话那是要累死人的， 有没有一种更好的办法呢。

当然有， 那就是 hash 风格的文件路径算法， 简单的说，我们直接从一个固定的文件名求出 hash， 然后通过固定位置的字符来计算目录。

这种方式可以有效地把大量文件自动分散到多个目录存储。
 



```python
import os
from hashlib import md5

filename = "notebook.pdf"
hashvalue = md5(filename.encode()).hexdigest()
filepath = os.path.join("/root","{0}/{1}/{2}".format(hashvalue[0:2], hashvalue[2:4], filename))

print(filepath)


```

    /root\9b/91/notebook.pdf
    

根据文件名， 我们就可以算出存哪里，从哪力读取


