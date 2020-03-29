[TOC]

## 监控

### http handler stats采集

```go

// klook-product/src/service/prosrv/srv/auth.go
// KlHandleFunc 来自客路App端的调用
func KlHandleFunc(f comm.HandleFuncType) comm.HandleFuncType {
		startTime := time.Now()
		//...

		// 监控统计v2
		duration := time.Since(startTime)
		statsd.HttpHandlerStatsCollector.Collect(w, r, funcName, h.RequestID(), startTime, duration)
  	//...
}

// klook-libs/src/klook.libs/statsd/http_handler.go
// Collect
func (c *httpHandlerStatsCollector) Collect(w http.ResponseWriter, r *http.Request, handler string, requestID string, startTime time.Time, duration time.Duration) {
}	
```



## Influxdb 监控数据上报

### ReportEntity

**src/klook.framework/common/stat/realtime_stat.go**

```go
type ReportEntity struct {
	Type        string
	HeaderNames []string
	Headers     []string
	TimeStamp   int64
	KeyNames    []string
	ValueNames  []string
	StatList    Snapshot
}

type StatPair struct {
	Keys   []string
	Values []interface{}
}

type Snapshot []StatPair
```



### InfluxdbReporter

**src/klook.framework/common/stat/influxdb_reporter.go**

```go
func init() {
	rpt := &InfluxdbReporter{}
	RegisterReporter("influxdb_reporter", rpt)
}

type InfluxdbReporter struct {
}

func (rpt *InfluxdbReporter) Report(snapshot *ReportEntity) {
	if len(snapshot.StatList) == 0 {
		//Debug("StatList 没有数据,无需上报")
		return
	}
	for _, metric := range packStat(snapshot) {
		monitor.SendInfluxdbMetric(metric)
		//Infof("reporter a message tags:%v fileds:%v timestamp:%v", metric.Tags, metric.FieldsFloat64, time.Unix(0, metric.TimestampNanoSec))
	}
}

// packStat 打包上传数据,
// 由slice 转成 map
func packStat(entity *ReportEntity) []influxdbmetricpb.Metric {

	if len(entity.StatList) == 0 {
		return nil
	}
	metrics := make([]influxdbmetricpb.Metric, 0)
	for _, stat := range entity.StatList {
		metric := influxdbmetricpb.Metric{}
		metric.Measurement = getReportMeasurement(entity.Type)
		metric.TimestampNanoSec = entity.TimeStamp
		metric.Tags = make(map[string]string)
		metric.FieldsFloat64 = make(map[string]float64)
		if len(entity.Headers) != len(entity.HeaderNames) {
			Errorf("headers的值和Names数量不匹配,请及时调整,以免影响数据上报,type:%s Headers:%v Names:%v", entity.Type, entity.Headers, entity.HeaderNames)
			continue
		}
		if len(stat.Keys) != len(entity.KeyNames) {
			Errorf("Keys的值和Names数量不匹配,请及时调整,以免影响数据上报,type:%s Keys:%v Names:%v", entity.Type, stat.Keys, entity.KeyNames)
			continue
		}
		if len(stat.Values) != len(entity.ValueNames) {
			Errorf("Values的值和Names数量不匹配,请及时调整,以免影响数据上报,type:%s Values:%v Names:%v", entity.Type, stat.Values, entity.ValueNames)
			continue
		}
		for idx, header := range entity.Headers {
			metric.Tags[entity.HeaderNames[idx]] = header
		}
		for idx, key := range stat.Keys {
			metric.Tags[entity.KeyNames[idx]] = key
		}
		for idx, val := range stat.Values {
			v, ok := toInt64Float64(val).(int64)
			if ok {
				metric.FieldsFloat64[entity.ValueNames[idx]] = float64(v)
			} else {
				metric.FieldsFloat64[entity.ValueNames[idx]] = val.(float64)
			}
		}
		metrics = append(metrics, metric)
	}
	return metrics
}

//getReportMeasurement 根据上报类型,设置Measurement 即每个上报类型对应一个表
func getReportMeasurement(reporterType string) string  {
	// influnx db的 Measurement 不允许空格和逗号
	reporterType = strings.Replace(reporterType," ", "_",-1)
	reporterType = strings.Replace(reporterType,",", "_",-1)
	return fmt.Sprintf("%s_reporter", reporterType)
}
```



### Metric

**src/klook.libs/monitor/srvproto/model/influxdbmetricpb/influxdbmetric.pb.go**

```go
// message定义区
// message 名字大写, camelCase
type Metric struct {
	Measurement          string             `protobuf:"bytes,1,opt,name=Measurement,proto3" json:"measurement"`
	Tags                 map[string]string  `protobuf:"bytes,2,rep,name=Tags,proto3" json:"tags" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
	FieldsFloat64        map[string]float64 `protobuf:"bytes,3,rep,name=FieldsFloat64,proto3" json:"fields_float64" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"fixed64,2,opt,name=value,proto3"`
	FieldsInt64          map[string]int64   `protobuf:"bytes,4,rep,name=FieldsInt64,proto3" json:"fields_int64" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"varint,2,opt,name=value,proto3"`
	FieldsString         map[string]string  `protobuf:"bytes,5,rep,name=FieldsString,proto3" json:"fields_string" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"bytes,2,opt,name=value,proto3"`
	FieldsBool           map[string]bool    `protobuf:"bytes,6,rep,name=FieldsBool,proto3" json:"fields_bool" protobuf_key:"bytes,1,opt,name=key,proto3" protobuf_val:"varint,2,opt,name=value,proto3"`
	TimestampNanoSec     int64              `protobuf:"varint,8,opt,name=TimestampNanoSec,proto3" json:"timestamp_nanosec"`
	XXX_NoUnkeyedLiteral struct{}           `json:"-"`
	XXX_sizecache        int32              `json:"-"`
}
```



### func SendInfluxdbMetric(m influxdbmetricpb.Metric)

**src/klook.libs/monitor/influxdbItem.go**

```go
var once sync.Once
var sender *InfluxdbSender

func SendInfluxdbMetric(m influxdbmetricpb.Metric) {
	once.Do(func() {
		sender = &InfluxdbSender{
			ch: 		make(chan influxdbmetricpb.Metric, 10240),
			retryCh:	make(chan influxdbmetricpb.Metric, 1024),
		}

		go sender.daemon()
		go sender.retryDaemon()
	})

	sender.SendInfluxdbMetricToCollector(m)
}

type InfluxdbSender struct {
	ch			chan influxdbmetricpb.Metric
	retryCh		chan influxdbmetricpb.Metric
}

func (s *InfluxdbSender)SendInfluxdbMetricToCollector(r influxdbmetricpb.Metric) {
	select {
	case s.ch <- r:
	default:
		logger.Warnf("InfluxdbSender's channel is full. rep:%s", krpc.ProtoMessageToBytes(r))
	}
}

func (s *InfluxdbSender)daemon() {
	for {
		if err := s.sendMetricStream(); err != nil {
			logger.Errorf("sendMetricStream error: %v", err)
			time.Sleep(time.Second)
		}
	}
}

func (s *InfluxdbSender)sendMetricStream() error {
	ctx := context.Background()
	client := influxdbmetricpb.NewSenderClient(krpc.TargetKlookMonitorMcollectorsrv)
	callOptions := func(options *krpc.CallOptions) {
		options.Timeout = time.Second * 20
	}

	stream, err := client.SendMetric(ctx, callOptions)
	if err != nil {
		logger.Errorf("failed create SendMetric stream :%v", err)
		return err
	}
	defer func() {
		_, err = stream.CloseAndRecv()
		if err != nil {
			logger.Errorf("CloseAndRecv error:%v", err)
		}
	}()

	for m := range s.ch {
		err := stream.Send(&m)
		if err != nil {
			s.retryCh <- m

			logger.Errorf("Send InfluxdbMetric to mcollectorsrv failed,error: %s ", err)
			return err
		}
	}

	return nil
}

func (s *InfluxdbSender)retryDaemon() {
	for m := range s.retryCh {
		logger.Infof("retry set into process channel metric:%s", krpc.ProtoMessageToBytes(m))
		s.ch <- m

		logger.Info("retry set into process channel finished")
	}
}
```



## Rest Filter

**src/klook.framework/protocol/rest/filter_chain.go**

```go
package rest

import (
	"container/list"
	"klook.framework/resource"
	"net/http"
)

type Filter interface {
	PreHandle(rsp http.ResponseWriter, req *http.Request, path string, kss *resource.KSession) (bool, error)
	AfterCompletion(rsp http.ResponseWriter, req *http.Request, path string, kss *resource.KSession) error
}

type FilterChain struct {
	chain list.List
}

func (fc *FilterChain) Add(filter Filter) {
	fc.chain.PushBack(filter)
}
```

