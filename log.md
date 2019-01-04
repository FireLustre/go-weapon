# 日志库
## logrus

### 安装

> go get github.com/sirupsen/logrus


### 使用

#### 简单使用

```
Log = logrus.New()
Log.WithFields(logrus.Fields{
	"ip":   ip,
	"data": data,
}).Info()


```

#### 进阶使用

使用logrus的hook来加载 github.com/lestrrat/go-file-rotatelogs 模块. 每次当我们写入日志的时候，logrus都会调用 go-file-rotatelogs 来判断日志是否要进行切分…  
go-file-rotatelogs  可以实现linux logratate的大多数功能

```
var Log *logrus.Logger

func NewLogger() *logrus.Logger {
	if Log != nil {
		return Log
	}

	Log = logrus.New()

	path := GetCurrPath() + "/data/data.log"

	writer, _ := rotatelogs.New(
		path+".%Y%m%d",
        rotatelogs.WithLinkName(path),             // 生成软链，指向最新日志文件
        rotatelogs.WithMaxAge(7*24*time.Hour),     // 文件最大保存时间
        rotatelogs.WithRotationTime(24*time.Hour), // 日志切割时间间隔
	)

	Log.Hooks.Add(lfshook.NewHook(
		lfshook.WriterMap{
			logrus.InfoLevel:  writer,
			logrus.ErrorLevel: writer,
		},
		&logrus.JSONFormatter{},
	))

	return Log
}

func init() {

	Log = NewLogger()
}

// 获取当前路径
func GetCurrPath() string {
	file, _ := exec.LookPath(os.Args[0])
	path, _ := filepath.Abs(file)
	index := strings.LastIndex(path, string(os.PathSeparator))
	ret := path[:index]
	return ret
}
```

#### 总结
logrus本身比较倾向于把日志输出到第三方，如logstash、filebeat等，所以对日志的切分功能做的不是很到位。