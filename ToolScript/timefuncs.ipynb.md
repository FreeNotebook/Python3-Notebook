## 几个时间小函数

> 作者: 老菜   来源: [老菜园子](https://github.com/laocaiyuan)


```python
import datetime

def fmt_second(time_total):
    def _ck(t):
        return t < 10 and "0%s" % t or t

    times = int(time_total)
    h = int(times / 3600)
    m = int(times % 3600 / 60)
    s = int(times % 3600 % 60)
    return "%s:%s:%s" % (_ck(h), _ck(m), _ck(s))

# 把秒换成时分秒格式
fmt_second(322)
```




    '00:05:22'




```python
def is_expire(dstr):
    if not dstr:
        return False
    try:
        expire_date = datetime.datetime.strptime("%s 23:59:59" % dstr, "%Y-%m-%d %H:%M:%S")
        now = datetime.datetime.now()
        return expire_date < now
    except:
        import traceback
        traceback.print_exc()
        return False

# 一个过期判断函数
is_expire("2018-10-10")
```




    True




```python
def format_time(times):
    if times <= 60:
        return u"%s秒" % times

    d = int(times / (3600 * 24))
    h = int(times % (3600 * 24) / 3600)
    m = int(times % (3600 * 24) % 3600 / 60)
    s = int(times % (3600 * 24) % 3600 % 60)

    if int(d) > 0:
        return u"%s天%s小时%s分钟%s秒" % (int(d), int(h), int(m), int(s))
    elif int(d) == 0 and int(h) > 0:
        return u"%s小时%s分钟%s秒" % (int(h), int(m), int(s))
    elif int(d) == 0 and int(h) == 0 and int(m) > 0:
        return u"%s分钟%s秒" % (int(m), int(s))

# 把已使用的秒数转换为友好的显示， 比如 linux 的运行时长

format_time(861000)

```




    '9天23小时10分钟0秒'


