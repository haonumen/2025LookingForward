#Jmeter 执行顺序 
- 配置元素
- 预处理器
- 计时器
- 采样器
- 后处理器（除非 SampleResult 为 null）
- 断言（除非 SampleResult 为 null）
- 侦听器（除非 SampleResult 为 null）