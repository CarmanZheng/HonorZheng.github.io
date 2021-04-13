---
title: Python基操-datetime 模块
layout: post
tags: Python基操
---

参考一下：https://www.pythonf.cn/read/148711

datetime是一个关于时间的库，主要包含的类有：

* datetime.date：表示日期的类。常用的属性有year, month, day；
* datetime.time：表示时间的类。常用的属性有hour, minute, second, microsecond；
* datetime.datetime：表示日期时间。
* datetime.timedelta：表示时间间隔，即两个时间点之间的长度。
* datetime.tzinfo：与时区有关的相关信息。（这里不详细充分讨论该类，感兴趣的童鞋可以参考python手册）
注：上面这些类型的对象都是不可变（immutable）的。

### 1.date模块
### datatime.date
##### 获取当前日期 
```
tday= date.today()

```

实例：tday中包含了年月日等属性，使用 tday.year month day 可以知道年、月、日等
```
from datetime import date
date = datetime.date(2018, 8, 23)
print(date)
2018-8-23

```
##### 时间戮转换->date

```
date.fromtimestamp(timestamp)
```
实例
```
import time
from datetime import date

print (time.time())
print (date.fromtimestamp(time.time()))
 
# 1429587111.21
# 2015-04-21

```

##### 生成时间
```
now = date( 2010 , 04 , 06 ) 
```
实例
```
now = date( 2010 , 04 , 06 ) 
tomorrow = now.replace(day = 07 ) 
print  ('now:' , now, ', tomorrow:' , tomorrow )
print  ('timetuple():' , now.timetuple())
print  ('weekday():' , now.weekday() )
print  ('isoweekday():' , now.isoweekday() )
print  ('isocalendar():' , now.isocalendar() )
print  ('isoformat():' , now.isoformat())
  
# # ---- 结果 ----  
# now: 2010-04-06 , tomorrow: 2010-04-07  
# timetuple(): (2010, 4, 6, 0, 0, 0, 1, 96, -1)  
# weekday(): 1  
# isoweekday(): 2  
# isocalendar(): (2010, 14, 2)  
# isoformat(): 2010-04-06
```

### 2.time模块
### datatime.time
time类表示时间，由时、分、秒以及微秒组成。time类的构造函数如下：
```
 class datetime.time(hour[ , minute[ , second[ , microsecond[ , tzinfo] ] ] ] ) 
```
各参数的意义不作解释，这里留意一下参数tzinfo，它表示时区信息。
* 注意一下各参数的取值范围：
```
hour的范围为[0, 24)，
minute的范围为[0, 60)，
second的范围为[0, 60)，
microsecond的范围为[0, 1000000)
```

#### time类定义的类属性：
```
time.min、time.max：
time类所能表示的最小、最大时间。
其中，time.min = time(0, 0, 0, 0)， time.max = time(23, 59, 59, 999999)；
time.resolution：时间的最小单位，这里是1微秒；
```

* time类提供的实例方法和属性：
```
time.hour、time.minute、time.second、time.microsecond：时、分、秒、微秒；
time.tzinfo：时区信息；
time.replace([ hour[ , minute[ , second[ , microsecond[ , tzinfo] ] ] ] ] )：创建一个新的时间对象，用参数指定的时、分、秒、微秒代替原有对象中的属性（原有对象仍保持不变）；
time.isoformat()：返回型如"HH:MM:SS"格式的字符串表示；
time.strftime(fmt)：返回自定义格式化字符串。在下面详细介绍
```
实例
```
from datetime import time
tm = time(23 , 46 , 10 ) 
print ( 'tm:' , tm ）
print  'hour: %d, minute: %d, second: %d, microsecond: %d' 
    % (tm.hour, tm.minute, tm.second, tm.microsecond) 
tm1 = tm.replace(hour = 20 ) 
print  ('tm1:' , tm1 )
print  ('isoformat():' , tm.isoformat())
>>> 
tm: 23:46:10  
hour: 23, minute: 46, second: 10, microsecond: 0  
tm1: 20:46:10  
isoformat(): 23:46:10
```

### 3.datatime模块
### datatime.time
datetime是date与time的结合体，包括date与time的所有信息。它的构造函数如下：
```
datetime.datetime (year, month, day[ , hour[ , minute[ , second[ , microsecond[ , tzinfo] ] ] ] ] )
```
各参数的含义与date、time的构造函数中的一样，要注意参数值的范围。

**datetime类定义的类属性与方法：**

```
datetime.min、datetime.max：datetime所能表示的最小值与最大值；
datetime.resolution：datetime最小单位；
datetime.today()：返回一个表示当前本地时间的datetime对象；
datetime.now([tz])：返回一个表示当前本地时间的datetime对象，如果提供了参数tz，则获取tz参数所指时区的本地时间；
datetime.utcnow()：返回一个当前utc时间的datetime对象；
datetime.fromtimestamp(timestamp[, tz])：根据时间戮创建一个datetime对象，参数tz指定时区信息；
datetime.utcfromtimestamp(timestamp)：根据时间戮创建一个datetime对象；
datetime.combine(date, time)：根据date和time，创建一个datetime对象；
datetime.strptime(date_string, format)：将格式字符串转换为datetime对象；
```

实例

```
dt = datetime.now() 
print  ((%Y-%m-%d %H:%M:%S %f): ' , dt.strftime( '%Y-%m-%d %H:%M:%S %f' ))
print  ((%Y-%m-%d %H:%M:%S %p): ' , dt.strftime( '%y-%m-%d %I:%M:%S %p' )) 
print  （%%a: %s ' % dt.strftime( '%a' ) ）
print  (%%A: %s ' % dt.strftime( '%A' ) )
print  (%%b: %s ' % dt.strftime( '%b' )) 
print  (%%B: %s ' % dt.strftime( '%B' ) )
print  (日期时间%%c: %s ' % dt.strftime( '%c' ) )
print  (日期%%x：%s ' % dt.strftime( '%x' ) )
print  (时间%%X：%s ' % dt.strftime( '%X' ) )
print (今天是这周的第%s天 ' % dt.strftime( '%w' ) )
print  (今天是今年的第%s天 ' % dt.strftime( '%j' ) )
print  (今周是今年的第%s周 ' % dt.strftime( '%U' )) 
  
# # ---- 结果 ----  
# (%Y-%m-%d %H:%M:%S %f): 2010-04-07 10:52:18 937000  
# (%Y-%m-%d %H:%M:%S %p): 10-04-07 10:52:18 AM  
# %a: Wed  
# %A: Wednesday  
# %b: Apr  
# %B: April  
# 日期时间%c: 04/07/10 10:52:18  
# 日期%x：04/07/10  
# 时间%X：10:52:18  
# 今天是这周的第3天  
# 今天是今年的第097天  
# 今周是今年的第14周   

```
#### python之time,datetime,string转换

###### 把datetime转成字符串 
```
def datetime_toString(dt): 
  return dt.strftime("%Y-%m-%d-%H") 
```
###### 把字符串转成datetime 
```
def string_toDatetime(string): 
  return datetime.strptime(string, "%Y-%m-%d-%H") 
```
###### 把字符串转成时间戳形式
``` 
def string_toTimestamp(strTime): 
  return time.mktime(string_toDatetime(strTime).timetuple()) 
```
###### 把时间戳转成字符串形式 
```
def timestamp_toString(stamp): 
  return time.strftime("%Y-%m-%d-%H", tiem.localtime(stamp)) 
```
###### 把datetime类型转外时间戳形式 
```
def datetime_toTimestamp(dateTim): 
  return time.mktime(dateTim.timetuple()) 

```
## 附表

格式字符及意义
===
```
%a   星期的简写。如 星期三为Web|
%A   星期的全写。如 星期三为Wednesday|
%b   月份的简写。如4月份为Apr
%B   月份的全写。如4月份为April
%c:  日期时间的字符串表示。（如： 04/07/10 10:43:39）
%d:  日在这个月中的天数（是这个月的第几天）
%f:  微秒（范围[0,999999]）
%H:  小时（24小时制，[0, 23]）
%I:  小时（12小时制，[0, 11]）
%j:  日在年中的天数 [001,366]（是当年的第几天）
%m:  月份（[01,12]）
%M:  分钟（[00,59]）
%p:  AM或者PM
%S:  秒（范围为[00,61]，为什么不是[00, 59]，参考python手册~_~）
%U:  周在当年的周数当年的第几周），星期天作为周的第一天
%w:  今天在这周的天数，范围为[0, 6]，6表示星期天
%W:  周在当年的周数（是当年的第几周），星期一作为周的第一天
%x:  日期字符串（如：04/07/10）
%X:  时间字符串（如：10:43:39）
%y:  2个数字表示的年份
%Y:  4个数字表示的年份
%z:  与utc时间的间隔 （如果是本地时间，返回空字符串）
%Z:  时区名称（如果是本地时间，返回空字符串）
%%:  %% => %
```

