# 实验六：云上的应用开发、部署和运维

## 整体代码逻辑

整体上与TA的提供样例差异不大，故此处不再赘述。

## 对样例主要改动之处

### metrics部分（metrics/metrics.go）

实验五中我们采用CPU和内存的可申请量，同时保留有一个随机的扰动，作为打分公式的元素；此处沿用这一种思想，只是去除随机化的元素，仅保留CPU和内存的可申请量作为metrics指标。同时，因为取得相关数值的API发生了变化，将原来的可申请的资源量改为了已用的资源量。
<br/>
metrics.go原来只定义了requestCount和requestLatency，现在加上used_cpu和used_memory来表示CPU和内存的已用量。
```go
var (
	requestCount = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name:      "request_total",
			Help:      "Number of request processed by this service.",
		}, []string{},
	)

	requestLatency = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Name:      "request_latency_seconds",
			Help:      "Time spent in this service.",
			Buckets:   []float64{0.01, 0.02, 0.05, 0.1, 0.2, 0.5, 1.0, 2.0, 5.0, 10.0, 20.0, 30.0, 60.0, 120.0, 300.0},
		}, []string{},
	)

	used_cpu = prometheus.NewGauge(
		prometheus.GaugeOpts{
			Name: "used_cpu",
			Help: "get",
		},
	)

	used_memory = prometheus.NewGauge(
		prometheus.GaugeOpts{
			Name: "used_memory",
			Help: "get",
		},
	)
)
```

当然，Register()和RequestIncrease()的部分也要进行相应修改：
```go
func Register() {
	prometheus.MustRegister(requestCount)
	prometheus.MustRegister(requestLatency)
	prometheus.MustRegister(used_cpu)
	prometheus.MustRegister(used_memory)
}

func RequestIncrease() {
	requestCount.WithLabelValues().Add(1)
	for {
		cpu_temp, _ := cpu.Percent(time.Second, false)
		used_cpu.Set(cpu_temp[0])
		mem_temp, _ := mem.VirtualMemory()
		used_memory.Set(mem_temp.UsedPercent)
		time.Sleep(time.Second)
	}
}
```