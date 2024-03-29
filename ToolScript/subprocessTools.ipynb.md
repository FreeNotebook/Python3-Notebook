## 一个 subprocess 命令执行工具封装

> 作者: 老菜   来源: [老菜园子](https://github.com/laocaiyuan)

简单执行 os.system 当然方便， 但是我们要控制完整的交互过程，打印日志， 判断是否执行成功， 获取完整的执行结果等， 那么我们就必须多做点功课了。

> 打印日志我么还做了点颜色区分， 不过在 windows 无法支持


```python
import sys
import os
import subprocess
import time

class CmdError(Exception):
    def __init__(self, message):
        self.message = message

class CmdShell(object):
    
    def __init__(self,logfile=None,debug=False):
        self.logfile = logfile
        self.is_debug = debug
        self.is_win32 = sys.platform in ['win32']
    # error
    def inred(self,s):
        return self.is_win32 and s or "%s[31;2m%s%s[0m"%(chr(27),s, chr(27))
    
    # success    
    def ingreen(self,s):
        return self.is_win32 and s or "%s[32;2m%s%s[0m"%(chr(27),s, chr(27))
    
    # operate
    def inblue(self,s):
        return self.is_win32 and s or "%s[34;2m%s%s[0m"%(chr(27),s, chr(27))

    # info
    def incblue(self,s):
        return self.is_win32 and s or "%s[36;2m%s%s[0m"%(chr(27),s, chr(27))

    # warning Magenta
    def inwarn(self,s):
        return self.is_win32 and s or "%s[35;2m%s%s[0m"%(chr(27),s, chr(27))

    def log(self,msg,_font=None,logfile=None):
        print(_font(msg)) 
        if self.logfile:
            with open(self.logfile,'ab') as fs:
                fs.write(msg)
                fs.write('\n')
                
    def info(self,msg):
        self.log('[INFO] - %s'%msg,_font=self.incblue)
        
    def debug(self,msg):
        self.log('[DEBUG] - %s'%msg,_font=self.inblue)
    
    def succ(self,msg):
        self.log('[SUCC] - %s'%msg,_font=self.ingreen)
    
    def err(self,msg):
        self.log('[ERROR] - %s'%msg,_font=self.inred)
        
    def warn(self,msg):
        self.log('[WARN] - %s'%msg,_font=self.inwarn)
        
    def read(self,ask):
        result = input(self.incblue('[INPUT] - %s'%ask))
        if self.is_debug:
            self.debug('<question - %s | answer - %s>'%(ask,result))
        return result
    
    def wait(self,sec=0):
        if not sec:return
        sec = int(sec)
        _range = range(1,sec+1)
        _range.reverse()
        for i in _range:
            self.debug(str(i))
            time.sleep(1.0)
    
    def run(self,command, raise_on_fail=False, shell=True, env=None,wait=0):
        self.info(">> run command : %s"%command)
        _result = dict(code=0)
        run_env = os.environ.copy()
        if env:run_env.update(env)
        if wait > 0:
            subprocess.Popen(command, shell=True)
            self.wait(wait)
        else:    
            proc = subprocess.Popen(command,shell=shell,
                                    stdout=subprocess.PIPE,stderr=subprocess.PIPE,
                                    env=run_env)
            stdout, stderr = proc.communicate('through stdin to stdout')
            result = proc.returncode, stdout, stderr
            if proc.returncode > 0 and raise_on_fail:
                error_string = "# Could not run command (return code= %s)\n" % proc.returncode
                error_string += "# Error was:\n%s\n" % (stderr.strip())
                error_string += "# Command was:\n%s\n" % command
                error_string += "# Output was:\n%s\n" % (stdout.strip())
                if proc.returncode == 127:  # File not found, lets print path
                    path = os.getenv("PATH")
                    error_string += "# Check if y/our path is correct: %s" % path
                self.err(error_string)
                raise CmdError(error_string)
            else:
                if self.is_debug:
                    if stdout.strip():
                        self.debug(stdout)
                    if stderr.strip():
                        self.err(stderr)
                if proc.returncode == 0:
                    self.succ(">> run command : %s success!"%command)
                else:
                    self.err(">> run command : %s failure!"%command)
                return result    
                
shell = CmdShell()
shell.run("dir")

```

    [INFO] - >> run command : dir
    [SUCC] - >> run command : dir success!
    




    (0,
     b' \xc7\xfd\xb6\xaf\xc6\xf7 E \xd6\xd0\xb5\xc4\xbe\xed\xca\xc7 \xbf\xaa\xb7\xa2\xb9\xa4\xd7\xf7\xc7\xf8\r\n \xbe\xed\xb5\xc4\xd0\xf2\xc1\xd0\xba\xc5\xca\xc7 C404-E5B3\r\n\r\n E:\\github\\notebook\\notebooks\\ToolScript \xb5\xc4\xc4\xbf\xc2\xbc\r\n\r\n2019/10/13  10:20    <DIR>          .\r\n2019/10/13  10:20    <DIR>          ..\r\n2019/10/13  00:41             3,595 aescipher.ipynb\r\n2019/10/13  00:41             2,160 aescipher.ipynb.md\r\n2019/10/12  22:58             2,080 hashfilepath.ipynb\r\n2019/10/13  00:41               792 hashfilepath.ipynb.md\r\n2019/10/13  10:20             5,811 subprocessTools.ipynb\r\n2019/10/12  22:28             3,653 timefuncs.ipynb\r\n2019/10/13  00:41             1,484 timefuncs.ipynb.md\r\n               7 \xb8\xf6\xce\xc4\xbc\xfe         19,575 \xd7\xd6\xbd\xda\r\n               2 \xb8\xf6\xc4\xbf\xc2\xbc 90,429,157,376 \xbf\xc9\xd3\xc3\xd7\xd6\xbd\xda\r\n',
     b'')


