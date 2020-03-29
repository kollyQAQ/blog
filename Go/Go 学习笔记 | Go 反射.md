[TOC]

## TypeOf 和 ValueOf

在 Go 的反射定义中，任何接口都会由两部分组成的，一个是接口的具体类型，一个是具体类型对应的值。比如 var i int = 3 ，因为 interface{} 可以表示任何类型，所以变量 i 可以转为 interface{}，所以可以把变量 i 当成一个接口，那么这个变量在 Go 反射中的表示就是 <Value,Type>，其中 Value 为变量的值 3,Type 变量的为类型 int。

在 Go 反射中，标准库为我们提供两种类型来分别表示他们 reflect.Value 和 reflect.Type，并且提供了两个函数来获取任意对象的 Value 和 Type。

```go
func main() {
   i := 1
   t := reflect.TypeOf(i)
   v := reflect.ValueOf(i)
   fmt.Println(t, v) // int 1

   s := "china"
   t = reflect.TypeOf(s)
   v = reflect.ValueOf(s)
   fmt.Println(t, v) // string china

   u := User{"张三", 20}
   t = reflect.TypeOf(u)
   v = reflect.ValueOf(u)
   fmt.Println(t, v) // main.User {张三 20}

   u = User{"张三", 20}
   t = reflect.TypeOf(&u)
   v = reflect.ValueOf(&u)
   fmt.Println(t, v)            // *main.User &{张三 20}

   fmt.Printf("%[1]T %[1]v", u) // main.User {张三 20}
}


type User struct {
   Name string
   Age  int
}
```

## reflect.Value 转原始类型

上面的例子我们可以通过 reflect.ValueOf 函数把任意类型的对象转为一个 reflect.Value，那我们如果我们想逆向转过回来呢，其实也是可以的，reflect.Value 为我们提供了 Inteface 方法来帮我们做这个事情。继续接上面的例子：

```go
u1:=v.Interface().(User)
fmt.Println(u1) // {张三 20}
```

这样我们就又还原为原来的 User 对象了，通过打印的输出就可以验证。这里可以还原的原因是因为在 Go 的反射中，把任意一个对象分为 reflect.Value 和 reflect.Type，而 reflect.Value 又同时持有一个对象的 reflect.Value 和 reflect.Type, 所以我们可以通过 reflect.Value 的 Interface 方法实现还原。现在我们看看如何从一个 reflect.Value 获取对应的 reflect.Type。

```go
t1:=v.Type()
fmt.Println(t1) // main.User
```

如上例中，通过 reflect.Value 的 Type 方法就可以获得对应的 reflect.Type。

## 获取类型底层类型

底层的类型是什么意思呢？其实对应的主要是基础类型，接口、结构体、指针这些，因为我们可以通过 type 关键字声明很多新的类型，比如上面的例子，对象 u 的实际类型是 User，但是对应的底层类型是 struct 这个结构体类型，我们来验证下。

```go
i := 1
t := reflect.TypeOf(i)
v := reflect.ValueOf(i)
fmt.Println(t, v, t.Kind()) // int 1 int
s := "china"
t = reflect.TypeOf(s)
v = reflect.ValueOf(s)
fmt.Println(t, v, t.Kind()) // string china string
u := User{"张三", 20}
t = reflect.TypeOf(u)
v = reflect.ValueOf(u)
fmt.Println(t, v, t.Kind()) // main.User {张三 20} struct
u = User{"张三", 20}
t = reflect.TypeOf(&u)
v = reflect.ValueOf(&u)
fmt.Println(t, v, t.Kind())  // *main.User &{张三 20} ptr
```

通过 Kind 方法即可获取，非常简单，当然我们也可以使用 Value 对象的 Kind 方法，他们是等价的。

Go 语言提供了以下这些最底层的类型，可以看到，都是最基本的。

```
const (
        Invalid Kind = iota
        Bool
        Int
        Int8
        Int16
        Int32
        Int64
        Uint
        Uint8
        Uint16
        Uint32
        Uint64
        Uintptr
        Float32
        Float64
        Complex64
        Complex128
        Array
        Chan
        Func
        Interface
        Map
        Ptr
        Slice
        String
        Struct
        UnsafePointer
)
```

## 例子：万能函数

```go
package flexible_reflect


import (
   "errors"
   "reflect"
   "testing"
)


func TestDeepEqual(t *testing.T) {
   a := map[int]string{1: "one", 2: "two", 3: "three"}
   b := map[int]string{1: "one", 2: "two", 3: "three"}
   //t.Log(a == b)
   t.Log(reflect.DeepEqual(a, b))


   s1 := []int{1, 2, 3}
   s2 := []int{1, 2, 3}
   s3 := []int{2, 3, 1}
   t.Log("s1 == s2?", reflect.DeepEqual(s1, s2))
   t.Log("s1 == s3?", reflect.DeepEqual(s1, s3))


}


type Employee struct {
   EmployeeID string
   Name       string `format:"normal"`
   Age        int
}


func (e *Employee) UpdateAge(newVal int) {
   e.Age = newVal
}


type Customer struct {
   CookieID string
   Name     string
   Age      int
}


func fillBySettings(st interface{}, settings map[string]interface{}) error {


   // func (v Value) Elem() Value
   // Elem returns the value that the interface v contains or that the pointer v points to.
   // It panics if v's Kind is not Interface or Ptr.
   // It returns the zero Value if v is nil.


   if reflect.TypeOf(st).Kind() != reflect.Ptr {
      return errors.New("the first param should be a pointer to the struct type.")
   }
   // Elem() 获取指针指向的值
   if reflect.TypeOf(st).Elem().Kind() != reflect.Struct {
      return errors.New("the first param should be a pointer to the struct type.")
   }


   if settings == nil {
      return errors.New("settings is nil.")
   }


   var (
      field reflect.StructField
      ok    bool
   )


   for k, v := range settings {
      if field, ok = (reflect.ValueOf(st)).Elem().Type().FieldByName(k); !ok {
         continue
      }
      if field.Type == reflect.TypeOf(v) {
         vstr := reflect.ValueOf(st)
         vstr = vstr.Elem()
         vstr.FieldByName(k).Set(reflect.ValueOf(v))
      }


   }
   return nil
}


func TestFillNameAndAge(t *testing.T) {
   settings := map[string]interface{}{"Name": "Mike", "Age": 30}
   e := Employee{}
   if err := fillBySettings(&e, settings); err != nil {
      t.Fatal(err)
   }
   t.Log(e)
   c := new(Customer)
   if err := fillBySettings(c, settings); err != nil {
      t.Fatal(err)
   }
   t.Log(*c)
}
```

