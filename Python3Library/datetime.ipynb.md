## datetime 模块

> 作者: 老菜   来源: [老菜园子](https://github.com/laocaiyuan)


```python
import datetime

# 当前时间转换为格式化日期字符串

datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```




    '2019-10-12 22:16:08'




```python
# 字符串日期时间转换为 datatime 对象

datetime.datetime.strptime('2019-10-12 22:09:01', "%Y-%m-%d %H:%M:%S")
```




    datetime.datetime(2019, 10, 12, 22, 9, 1)




```python
# 增加一天

datetime.datetime.now() + datetime.timedelta(days=1)
```




    datetime.datetime(2019, 10, 13, 22, 16, 8, 41435)




```python
# 增加一小时

datetime.datetime.now() + datetime.timedelta(hours=1)
```




    datetime.datetime(2019, 10, 12, 23, 16, 8, 49455)




```python
# 增加一分钟

datetime.datetime.now() + datetime.timedelta(minutes=1)
```




    datetime.datetime(2019, 10, 12, 22, 17, 8, 57979)



如果要准确增加一个月怎么办，datetime.timedelta 是没有的， 这里就需要使用 calendar 模块了



```python
import calendar
def add_months(dt, months, days=0):
    month = dt.month - 1 + months
    year = dt.year + int(month / 12)
    month = int( month % 12 + 1)
    day = min(dt.day, calendar.monthrange(year, month)[1])
    dt = dt.replace(year=year, month=month, day=day)
    return dt + datetime.timedelta(days=days)

# 增加一个月再加10天
add_months(datetime.datetime.now(), 1, 10)

```




    datetime.datetime(2019, 11, 22, 22, 20, 57, 338181)


