+++
categories = ["golang"]
date = "2015-05-16T12:04:30+08:00"
description = ""
keywords = []
title = "error handling in go"

+++
最近在学习用Go的过程中，发现自己的代码中出现了大量的如下代码片段

```go
if err != nil {
	....
}
```
然后找到了在Go的官方Blog中找到了一篇Rob Pike写的[erors are values](https://blog.golang.org/errors-are-values)。
他的例子是这样，有一个这样的函数

```go
func write(buf []byte) err {
	w io.Writer
	err := w.Write(buf)
    if err != nil {
        return err
    }
    return nil
}
```
使用的时候就会有这样的情况出现

```go
, err = fd.Write(p0[a:b])
if err != nil {
    return err
}
_, err = fd.Write(p1[c:d])
if err != nil {
    return err
}
_, err = fd.Write(p2[e:f])
if err != nil {
    return err
}
```
文章中提供了一种解决办法。大概的意思就是在Go语言中Error也是一种值，发挥你的想象力来解决吧。其中一种解决办法就是在struct中定义一个err的变量用来存储Error

```go
type errWriter struct {
	w io.Writer
	err error
}
```
然后这个类的成员方法就把过程中的error都保存到err中,比如

```go
func (ew *errWriter) write(buf []byte) {
    if ew.err != nil {
        return
    }
    _, ew.err = ew.w.Write(buf)
}
```
这样使用起来的好处就是

```go
ew := &errWriter{w: fd}
ew.write(p0[a:b])
ew.write(p1[c:d])
ew.write(p2[e:f])
// and so on
if ew.err != nil {
    return ew.err
}
```

`代码精炼了很多！`但是文章最后也指出了这种解决办法有它的不好的地方，就是`你无法知道整个过程中出错了多少次`。那么对于这种情况我们可以采用error数组的方法，而Gorilla的Sessions[源码](https://github.com/gorilla/sessions/blob/master/sessions.go)有一个很好的例子

```go
type MultiError []error

func (m MultiError) Error() string {
	s, n := "", 0
	for _, e := range m {
		if e != nil {
			if n == 0 {
				s = e.Error()
			}
			n++
		}
	}
	switch n {
	case 0:
		return "(0 errors)"
	case 1:
		return s
	case 2:
		return s + " (and 1 other error)"
	}
	return fmt.Sprintf("%s (and %d other errors)", s, n-1)
}
```

我们可以看见他定义了一个`[]error`的类型，然后实现了`error interface`中的`Error方法`，那么`MultiError`也是一种error。通过这种方法可以将整个过程中的出现的错误给详细的记录下来。

```go
func (s *Registry) Save(w http.ResponseWriter) error {
	var errMulti MultiError
	for name, info := range s.sessions {
		session := info.s
		if session.store == nil {
			errMulti = append(errMulti, fmt.Errorf(
				"sessions: missing store for session %q", name))
		} else if err := session.store.Save(s.request, w, session); err != nil {
			errMulti = append(errMulti, fmt.Errorf(
				"sessions: error saving session %q -- %v", name, err))
		}
	}
	if errMulti != nil {
		return errMulti
	}
	return nil
}
```

以上代码一开始不理解的地方就是`var errMulti MultiError`语句没有初始化`errMulti`，为什么在里面可以直接使用`append`，最后在Go的官方博客的一篇叫做[Go Slices: usage and internals](http://blog.golang.org/go-slices-usage-and-internals)给出了答案
>Since the zero value of a slice (nil) acts like a zero-length slice, you can declare a slice variable and then append to it in a loop:

意思就是nil slice其实就是一种0长度的slice，不需要初始化也可以对它进行append操作。
