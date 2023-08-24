# Arch Gem5-I lab

## Scenario1 - Hit on another cache, not dirty
- How to simulate  
如果不想發生dirty，就是在write之前就read好。
- How to reproduce the result  
```bash
build/NULL/gem5.opt -d ../lab1/m5out/scene1 --debug-flags SimplePE,CacheAll,XBar,DRAM configs/archlab/lab1/arch_config.py > ../lab1/log/scene1/config.log
``` 
- config script  
```python
#comment
#system.simplePEs[1].addTraffic(20000, 1) 
#system.simplePEs[0].addTraffic(20000, 2)
system.simplePEs[0].addTraffic(20000, 0)
```
- Highlight the request & resopnse flow from log  
<img src="https://i.imgur.com/0NXaQqw.png">
The detail of the log can be found at ./log/scene1/config.log  

## Scenario2 - Hit on another cache, dirty
- How to simulate  
如果想讀到dirty，就是其中一個PE先write，另一個PE後read。
- How to reproduce the result  
```bash
build/NULL/gem5.opt -d ../lab1/m5out/scene2 --debug-flags SimplePE,CacheAll,XBar,DRAM configs/archlab/lab1/arch_config.py > ../lab1/log/scene2/config.log
``` 
- config script  
```python
#decomment
system.simplePEs[1].addTraffic(20000, 1) 
system.simplePEs[0].addTraffic(20000, 2)
```
- Highlight the request & resopnse flow from log  
<img src="https://i.imgur.com/TL1DjVL.png">
The detail of the log can be found at ./log/scene2/config.log

## Other observation
我在做這份作業時，發現如果要從客製library.py中引用.cc的function，可以像我下面這樣寫，算是一個小發現吧，畢竟gem5教學網站也沒講。
```python
class SimplePE(ClockedObject):
    #...
    # Add function in corresponding cxx_class
    cxx_exports = [
        PyBindMethod("start"),
        PyBindMethod("addTraffic"),
    ]
    
    @cxxMethod
    def start(self):
        self.start()

    @cxxMethod
    def addTraffic(self, addr, type):
        self.addTraffic(addr, type)
    #...
```
另外，如果想進一步學好gem5，網路上的教學值得仔細一讀，寫的非常詳盡。  
https://www.gem5.org/documentation/learning_gem5/part1/building/