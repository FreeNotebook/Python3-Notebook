## importlib 动态加载模块的例子

> 作者: 老菜   来源: [老菜园子](https://github.com/laocaiyuan)

这是一个 importlib 的例子, importlib 可以动态的导入模块， 使得我们的应用程序实现丰富的动态特性， 减少很多手工编码，让程序效率变得更高。

假设我们使用 [apscheduler](https://github.com/agronholm/apscheduler) 开发一个定时任务程序， 一般我们可以这么实现



```python
from datetime import datetime
import time
import os

from apscheduler.schedulers.background import BackgroundScheduler

def tick():
    print('Tick! The time is: %s' % datetime.now())

if __name__ == '__main__':
    scheduler = BackgroundScheduler()
    scheduler.add_job(tick, 'interval', seconds=3)
    scheduler.start()
    try:
        for i in range(3):
            time.sleep(1)
    except (KeyboardInterrupt, SystemExit):
        scheduler.shutdown()

```

    Tick! The time is: 2019-10-12 21:52:05.004903
    Tick! The time is: 2019-10-12 21:52:07.591508
    

如果我们的 job 很少， 我们只需要简单的加入即可，可是如果有很多呢，比如我们有这样一个模块目录， 可能有很多，而且后期会继续增加

    ├── jobs
        ├── backup_job.py
        ├── any_job.py
        ├── xxx_job.py

这个时候，我们就可以请 importlib 来帮忙了， 我们来重新改造我们的代码, 重新实现我们的任务调度器。

在下面的代码中我们实现了一个动态加载 job 模块的方法，他的作用就是动态加载 jobs 目录下的所有模块



```python
# sched.py
import os
import importlib
from apscheduler.schedulers.background import BackgroundScheduler

class Scheduler(object):

    def __init__(self):
        self.sched = BackgroundScheduler()
        self.sched.start()
        self.load_jobs(os.path.join(os.path.dirname(__file__),"jobs"), package_prefix="jobs")
        
    def load_jobs(self, jobdir, package_prefix="jobs"):
        """
        遍历目录， 动态加入 job 模块
        :param jobdir: job 模块目录
        :param package_prefix: 包前缀，根据项目实际配置， 方便准确导入模块
        :return: 
        """
        _excludes = ['__init__']
        jobs = set(os.path.splitext(it)[0] for it in os.listdir(jobdir))
        jobs = [it for it in jobs if it not in _excludes]
        for _job in jobs:
            try:
                job = "{0}.{1}".format(package_prefix,_job)
                jobmodule = importlib.import_module(job)
                if hasattr(jobmodule, 'jobclass'):
                    jobobj = jobmodule.jobclass(self)
                    self.sched.add_job(jobobj.execute, **jobobj.cron_params)
            except Exception:
                print("skip module file %s" % _job)
                continue
```

下面是我们的 job 模块， 每个 job 模块我们都有一个固定的属性名， 它被指向一个具体的对象类， 并且需要一个参数，这个参数就是我们的 sched 的父对象。

我们所有的 job 模块都采用这种方式编写, 在 job 模块定义中，我们顺便把 job 相关的参数也定义在一个字典中， 在导入 job 时一并传入



```python
# jobs/backup_job.py

class BackupDatabase():
    
    def __init__(self, sched):
        self.sched = sched
        self.cron_params = dict(id="backup_data", minute="30", hour="1", day="*")
        
    def execute(self):
        print("start backup data")

jobclass = BackupDatabase

```
