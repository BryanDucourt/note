任务3：基于功耗和性能模型的Governor（调频算法，参考论文2）（禹辰）
为什么要协同调频：高通现有方案的能效比低，突出表现在：
数据采集复杂；

模型过于简单，ping-pong等问题；
	现有的方法通常采用线性或近似的简单模型，这在低复杂度的负载下可能是有效的，但在面对多样化和动态变化的工作负载时，这些模型往往显得过于简单。由于模型精度不足，系统在调频时可能会出现频繁的频率波动，即“ping-pong”效应。频繁的频率切换不仅对能效没有实质性提升，反而会增加系统的不稳定性和功耗。
 与CPU功耗协同不足
	目前的调频策略往往是独立地调节CPU和内存的频率，而没有充分考虑两者之间的相互影响。实际上，CPU和内存之间存在复杂的依赖关系。例如，当CPU频率提高时，内存的访问延迟可能会成为性能瓶颈；而受到边际效应和任务类型的影响，简单地调高内存频率可能导致不必要的功耗增加。因此，缺乏协同考虑的调频方案无法实现全局能效最优，容易导致局部优化和整体能效的下降。
给出一个协同调频的初步设计

定义输入，输出，步骤和主要算法。（启发式即可）
输入：PMU（CPU、OC、MEM），频率（CPU、OC、MEM）、性能指标（IPS、CPI、Stall）
输出：CPU、OC、MEM的频率组合
步骤：
主要算法：