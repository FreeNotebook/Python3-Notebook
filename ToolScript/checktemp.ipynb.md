## Python 编写的检测 CPU 温度的 ngios 插件

> 作者: 老菜   来源: [老菜园子](https://github.com/laocaiyuan)

Nagios 每次在查询一个服务的状态时，产生一个子进程，并且它使用来自该命令的输出和退出代码来确定具体的状态。退出状态代码的含义如下所示

- OK —退出代码 0—表示服务正常地工作。
- WARNING —退出代码 1—表示服务处于警告状态。
- CRITICAL —退出代码 2—表示服务处于危险状态。
- UNKNOWN —退出代码 3—表示服务处于未知状态

根据退出状态码来给 nagios 提供监控信息



```python
import re
import os
import sys
from optparse import OptionParser

tmp = '''
coretemp-isa-0005
Adapter: ISA adapter
Core 2:       +40.0°C  (high = +84.0°C, crit = +94.0°C)
coretemp-isa-0004
Adapter: ISA adapter
Core 2:       +46.0°C  (high = +84.0°C, crit = +94.0°C)
coretemp-isa-0003
Adapter: ISA adapter
Core 1:       +43.0°C  (high = +84.0°C, crit = +94.0°C)
coretemp-isa-0002
Adapter: ISA adapter
Core 1:       +49.0°C  (high = +84.0°C, crit = +94.0°C)
coretemp-isa-0001
Adapter: ISA adapter
Core 0:       +43.0°C  (high = +84.0°C, crit = +94.0°C)
coretemp-isa-0000
Adapter: ISA adapter
Core 0:       +47.0°C  (high = +84.0°C, crit = +94.0°C)
'''

def check_cputemp(wval,cval):
    def get_state(tval):
        if tval < wval:
            return 0
        if wval < tval < cval:
            return 1
        if tval > cval:
            return 2

    output_str = {
        0 : 'check OK;',
        1 : 'check Wanning;',
        2 : 'check Crit;',
        3 : 'check Unknow;'
    }

    outputs = []
    status = 0

    for line in os.popen("sensors").readlines():
    # for line in tmp.split('\n'):
        grp = re.search('(Core\s\d+):\s+\+(\d+\.*\d+)',line)
        if not grp:
            continue
        _core_no,_temp = grp.groups()
        _status =  get_state(float(_temp))
        if _status > status:
            status = _status
        _out = ','.join([_core_no,_temp,str(_status)])
        outputs.append(_out)

    if not outputs :
        raise ValueError('no sensors data')

    _outstr = ';'.join(outputs)
    return status,output_str[status] + _outstr + '|' + _outstr



if __name__ == "__main__":
    usage = "usage: %prog [options] arg"
    parser = OptionParser(usage)
    parser.add_option("-c", "--warn",dest="warn", help="warn value")
    parser.add_option("-w", "--crit", dest="crit",help="crit value")
    (options, args) = parser.parse_args()
    if not options.warn and not options.crit:
        parser.error("incorrect number of arguments")
        sys.exit(3)
    else:
        try:
            status,output =  check_cputemp(int(options.warn),int(options.crit))
            print(output)
            sys.exit(status)
        except Exception as e:
            print('Unknow check %s'%e)
            sys.exit(3)
```

保存脚本到 /usr/lib/nagios/plugins/check_temp.py

在 nagios 的 command.cfg 的增加配置

添加如下信息：

    define command{
        command_name check_temp
        command_line python /usr/lib/nagios/plugins/check_temp.py -c $ARG1$  -w $ARG2$
    }

具体使用方法，请参考 nagios 的相关文档， 这里不再详述。
