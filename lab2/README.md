# Submit Lab2 report here!
## How to reproduce the result
1 tile and 1 L2XBar:  
Modify the following code in configs/archlab/lab2/board/single_tile_test.py
```python
#one hiracracy
#system.cache = Lab447PrivateL1CacheHierarchy(l1d_size="64kB", l1i_size="64kB")
#1 tile 2 L2XBar
#system.cache = Lab447PrivateL1PrivateL2CacheHierarchy(l1i_size="64kB",l1d_size="64kB",l2_size="128kB")
#1 tile 1 L2XBar
system.cache = Lab447PrivateL1SharedL2CacheHierarchy(l1i_size="64kB",l1d_size="64kB",l2_size="128kB")
```
Enter the following command in the terminal.
```bash
./build/NULL/gem5.opt configs/archlab/lab2/board/multi_tile_test.py
```
---
1 tile and 2 L2XBar:  
Modify the following code in configs/archlab/lab2/board/single_tile_test.py
```python
#one hiracracy
#system.cache = Lab447PrivateL1CacheHierarchy(l1d_size="64kB", l1i_size="64kB")
#1 tile 2 L2XBar
system.cache = Lab447PrivateL1PrivateL2CacheHierarchy(l1i_size="64kB",l1d_size="64kB",l2_size="128kB")
#1 tile 1 L2XBar
#system.cache = Lab447PrivateL1SharedL2CacheHierarchy(l1i_size="64kB",l1d_size="64kB",l2_size="128kB")
```
Enter the following command in the terminal.
```bash
./build/NULL/gem5.opt configs/archlab/lab2/board/multi_tile_test.py
```
---
2 tile and 1 L2XBar:  
Modify the following code in configs/archlab/lab2/board/multi_tile_test.py
```python
#one hiracracy
#system.cache = [Lab447PrivateL1CacheHierarchy(
#        l1d_size="64kB",
#        l1i_size="64kB")
#        for i in range(numTiles)]
#2 tiles 2 L2XBar
#system.cache = [Lab447PrivateL1PrivateL2CacheHierarchy(
#        l1i_size="64kB",
#        l1d_size="64kB",
#        l2_size="128kB")
#        for i in range(numTiles)]
#2 tiles 1 L2XBar
system.cache = [Lab447PrivateL1SharedL2CacheHierarchy(
        l1i_size="64kB",
        l1d_size="64kB",
        l2_size="128kB")
        for i in range(numTiles)]
```
Enter the following command in the terminal.
```bash
./build/NULL/gem5.opt configs/archlab/lab2/board/multi_tile_test.py
```
---
2 tile and 2 L2XBar:  
Modify the following code in configs/archlab/lab2/board/multi_tile_test.py
```python
#one hiracracy
#system.cache = [Lab447PrivateL1CacheHierarchy(
#        l1d_size="64kB",
#        l1i_size="64kB")
#        for i in range(numTiles)]
#2 tiles 2 L2XBar
system.cache = [Lab447PrivateL1PrivateL2CacheHierarchy(
        l1i_size="64kB",
        l1d_size="64kB",
        l2_size="128kB")
        for i in range(numTiles)]
#2 tiles 1 L2XBar
#system.cache = [Lab447PrivateL1SharedL2CacheHierarchy(
#        l1i_size="64kB",
#        l1d_size="64kB",
#        l2_size="128kB")
#        for i in range(numTiles)]
```
Enter the following command in the terminal.
```bash
./build/NULL/gem5.opt configs/archlab/lab2/board/multi_tile_test.py
```
## Screenshots of hardware configuration result
1 tile and 1 L2XBar:  
<img src="https://i.imgur.com/2XeGywa.png">
1 tile and 4 L2XBar:  
<img src="https://i.imgur.com/eXNSUrB.png">
2 tile and 1 L2XBar:  
<img src="https://i.imgur.com/LqAK6qd.png">
2 tile and 2 L2XBar:  
<img src="https://i.imgur.com/z4bS4Us.png">
You can find those diagrams in ./block_diagram .

## Screenshots of stats, anything that could prove it works 
1 tile and 1 L2XBar:  
<img src="https://i.imgur.com/jIo2Lw0.png">
1 tile and 4 L2XBar:  
<img src="https://i.imgur.com/LW5LbXB.png">
2 tile and 1 L2XBar:  
<img src="https://i.imgur.com/WUfDMpf.png">
2 tile and 2 L2XBar:  
<img src="https://i.imgur.com/6152kyc.png">

## Brief description of your implementation of function

1. lab447_private_l1_private_l2_cache_hierarchy.py   
You could find the following code in the ../gem5/src/python/gem5/components/lab447/lab447_private_l1_shared_l2_cache_hierarchy.py  
```python
class Lab447PrivateL1SharedL2CacheHierarchy(
    AbstractClassicCacheHierarchy, AbstractTwoLevelCacheHierarchy
):
    def __init__(
        #.....
    )
        # We should store the initilization information in the initializer.
        self._l1i_size=l1i_size
        self._l1d_size=l1d_size
        self._l2_size=l2_size
        self._l1d_assoc=l1d_assoc
        self._l1i_assoc=l1i_assoc
        #....
    def incorporate_tile(self, sys_membus, pe) -> None:
        # TODO
        #Establish devices in the hiracracy structure.
        self.l1icaches = [
            L1ICache(size=self._l1i_size, assoc = self._l1i_assoc) for i in range(pe.get_num_cores())
        ]
        self.l1dcaches = [
            L1DCache(size=self._l1d_size, assoc = self._l1d_assoc) for i in range(pe.get_num_cores())
        ]
        self.l2bus = L2XBar()#A bar
        self.l2cache = L2Cache(size=self._l2_size, assoc = self._l2_assoc)
        self.iptw_caches = [
            MMUCache(size="8KiB") for _ in range(pe.get_num_cores())
        ]
        self.dptw_caches = [
            MMUCache(size="8KiB") for _ in range(pe.get_num_cores())
        ]

        for i, cpu in enumerate(pe.get_cores()):
            #connect cpu to l1cache
            cpu.connect_icache(self.l1icaches[i].cpu_side)
            cpu.connect_dcache(self.l1dcaches[i].cpu_side)
            #connect l1cache to l2bus
            self.l1icaches[i].mem_side = self.l2bus.cpu_side_ports
            self.l1dcaches[i].mem_side = self.l2bus.cpu_side_ports

            self.iptw_caches[i].mem_side = self.l2bus.cpu_side_ports
            self.dptw_caches[i].mem_side = self.l2bus.cpu_side_ports

            #connect cpu to l1cache
            cpu.connect_walker_ports(
                self.iptw_caches[i].cpu_side, self.dptw_caches[i].cpu_side
            )

            cpu.connect_interrupt()
        #connect l2bus to l2 cache
        self.l2bus.mem_side_ports = self.l2cache.cpu_side
        #connect l2cache to memory bus
        self.l2cache.mem_side = sys_membus.cpu_side_ports
```

2. lab447_private_l1_private_l2_cache_hierarchy.py   
You could find the following code in the ../gem5/src/python/gem5/components/lab447/lab447_private_l1_private_l2_cache_hierarchy.py  
```python
class Lab447PrivateL1PrivateL2CacheHierarchy(
    AbstractClassicCacheHierarchy, AbstractTwoLevelCacheHierarchy
):
    def __init__(
        #.....
    )
        # We should store the initilization information in the initializer.
        self._l1i_size=l1i_size
        self._l1d_size=l1d_size
        self._l2_size=l2_size
        #....
    def incorporate_tile(self, sys_membus, pe) -> None:

        # TODO
        #Establish devices in the hiracracy structure.
        self.l1icaches = [
            L1ICache(size=self._l1i_size) for i in range(pe.get_num_cores())
        ]
        self.l1dcaches = [
            L1DCache(size=self._l1d_size) for i in range(pe.get_num_cores())
        ]
        self.l2buses = [
            L2XBar() for i in range(pe.get_num_cores())
        ]#Multiple bars
        self.l2caches = [
            L2Cache(size=self._l2_size) for i in range(pe.get_num_cores())
        ]
        self.iptw_caches = [
            MMUCache(size="8KiB") for _ in range(pe.get_num_cores())
        ]
        self.dptw_caches = [
            MMUCache(size="8KiB") for _ in range(pe.get_num_cores())
        ]

        for i, cpu in enumerate(pe.get_cores()):
            #connect cpu to l1cache
            cpu.connect_icache(self.l1icaches[i].cpu_side)
            cpu.connect_dcache(self.l1dcaches[i].cpu_side)
            #connect l1cache to l2bus
            self.l1icaches[i].mem_side = self.l2buses[i].cpu_side_ports
            self.l1dcaches[i].mem_side = self.l2buses[i].cpu_side_ports

            self.iptw_caches[i].mem_side = self.l2buses[i].cpu_side_ports
            self.dptw_caches[i].mem_side = self.l2buses[i].cpu_side_ports
            #connect l2bus to l2 cache
            self.l2buses[i].mem_side_ports = self.l2caches[i].cpu_side
            #connect l2cache to memory bus
            self.l2caches[i].mem_side = sys_membus.cpu_side_ports
            #connect cpu to l1cache
            cpu.connect_walker_ports(
                self.iptw_caches[i].cpu_side, self.dptw_caches[i].cpu_side
            )

            cpu.connect_interrupt()
```

## Anything what I learned
盡量在library內的新變數命名時，可以在前面加底線，像是```self.l1i_size```可以改成```_self.l1i_size```，才不會到處遇到重複的變數名稱而模擬失敗。