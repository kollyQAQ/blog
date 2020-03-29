[TOC]

# klook.libs

### comm.HandlerInfo 

可以通过 `rest.GetCtxParams(r *http.Request)` 获取

```go
type HandlerInfo struct {
	RequestID, Lang, Currency, Token, Version							string
  KlookAppVersion, ClientIP 														string
	FuncName, Event                                       string
	XPlatform                                             string
	Account                                               *AccountProfile
	AgentInfo                                             *AgentProfile
	MerchantInfo                                          *MerchantProfile
	RequestTTL                                            int
	RequestTime                                           string
	TraceID                                               string
	PT, DeviceID, XDeviceID, Gateway, GAID, UserAgent     string
}
```



### HandleFuncType

```go
// net/http 包
type HandlerFunc func(ResponseWriter, *Request)

// comm 包 （klook.lib）
type HandleFuncType = http.HandlerFunc
```



# Klook.framework

### SessionUser

```go
type SessionUser struct {
   UserId int64 `json:"user_id"`
}

func (su SessionUser) ID() int64 {
   return int64(su.UserId)
}

func (su SessionUser) IsLogin() bool {
   return su.UserId > 0
}
```



### KSession

```go
const (
  //----------- 身份相关(框架使用)
	// 用户token字符串
	VAR_TOKEN = "kss.token"
	// 当前登陆的用户信息
	VAR_SESSION_USER = "kss.session_user"
)

type KSession struct {
	traceID string
	//context
	context context.Context
	//session期内共享的变量
	vars map[string]interface{}
}

//用户登录token
func (ss *KSession) GetToken() string {
	return ss.GetStringVar(VAR_TOKEN, "")
}

func (ss *KSession) GetSessionUser() SessionUser {
	u := ss.GetVar(VAR_SESSION_USER)
	if u != nil {
		sessionUser := u.(SessionUser)
		return sessionUser
	} else { //session中不存在，调用接口去生成
		newSS := &KSession{traceID: ss.GetTraceID(), context: context.Background()}
		rs := Acquire(userGetterType, newSS)
		defer Release(rs)
		sg := rs.(SessionUserGetter)
		su, err := sg.GetSessionUser(ss.GetToken())
		if err != nil { //就算抛异常也会有一个默认的session user返回
			return notLoginUser
		}
		ss.SetSessionUser(su)
		return su
	}
	return notLoginUser
}

func (ss *KSession) SetSessionUser(su SessionUser) {
	ss.SetVar(VAR_SESSION_USER, su)
}
```



### Resource

```go
var ISessionResourceType = reflect.TypeOf(new(ISessionResource)).Elem()

type iResource interface {
   Init(args ...interface{})
}

type iSessionAble interface {
   BindSession(kss *KSession)
   GetSession() *KSession
   Rest()
}

//service/DB/sql_mapper/redis/mq/logger
type ISessionResource interface {
	iResource
	iSessionAble
}

type SessionResource struct {
	kss *KSession
}

func (sr *SessionResource) Init(args ...interface{}) {
}

func (sr *SessionResource) BindSession(kss *KSession) {
	//debug.PrintStack()
	//fmt.Printf("------------bindsession SessionResource, type:%+v\n", sr)
	if sr.kss != nil{
		sr.kss.Copy(kss)
	}else {
		sr.kss = kss
	}
}

func (sr *SessionResource) GetSession() *KSession {
	return sr.kss
}

func (sr *SessionResource) Rest() {
	//fmt.Printf("------------reset in SessionResource, type:%v\n", reflect.TypeOf(*sr))
	if sr.kss != nil{
		sr.kss.Rest()
	}
}
```



### KSQLMapper

```go
func init() {
	RegisterResource(struct {
		ISQLMapper
	}{new(KSQLMapper)})
}

type QueryParam map[string]interface{}

type ISQLMapper interface {
	SelectOne(name string, destPtr interface{}, p interface{}) (bool, error)
	Select(name string, destPtr interface{}, p interface{}) error

	Insert(name string, p interface{}) (int64, error)
	InsertGetLastID(name string, p interface{}) (int64, int64, error)
	Update(name string, p interface{}) (int64, error)
	Delete(name string, p interface{}) (int64, error)
}

type KSQLMapper struct {
   SessionResource
   namespace string
}

func (mapper *KSQLMapper) Init(args ...interface{}) {
}

func (mapper *KSQLMapper) SelectOne(name string, destPtr interface{}, p interface{}) (exist bool, err error) {
}

func (mapper *KSQLMapper) Select(name string, destPtr interface{}, p interface{}) (err error) {
	bt := time.Now()
	defer func() {
    ......
		//收集统计数据
		collect(mapper.GetContext(), p, mapper.namespace, "select", name, err, time.Since(bt).Nanoseconds())
	}()
	rs, err := doSelect(mapper, name, p)

	if err != nil {
		return err
	}
	defer rs.Close()

	return sqlx.StructScan(rs, destPtr)
}

func (mapper *KSQLMapper) Insert(name string, p interface{}) (affectedRows int64, err error{
}

func (mapper *KSQLMapper) InsertGetLastID(name string, p interface{}) (lastID int64, affectedRows int64, err error) {
}

func (mapper *KSQLMapper) Update(name string, p interface{}) (affectedRows int64, err error{
}

func (mapper *KSQLMapper) Delete(name string, p interface{}) (affectedRows int64, err error{
}
```



### KStatement

```go
type KStatement struct {
   mapper *KSQLMapper
   name   string
}
```