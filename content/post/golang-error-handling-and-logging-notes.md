+++
title = 'Golang 错误处理和日志'
date = '2025-05-11T08:30:13+08:00'
categories = ['编程']
tags = ['code','golang']
toc = true
+++

Go语言的错误处理和日志规范。
- 大部分情况下包装error使用 %w，而不是%v，另外有了%w后pkg/errors这个包也需要了。
- 错误包装需要避免failed to这种前缀，只需要模块名： err即可。参考 [uber-golang-style](https://github.com/uber-go/guide/blob/master/style.md#error-wrapping)
- 错误包装可以添加额外的变量信息： return fmt.Errorf("get user %q: %w", id, err)
- 全局错误变量使用Err开头，例如  ErrBrokenLink = errors.New("link is broken")
- 如果是自定义错误struct类型 则使用Error结尾 type NotFoundError struct {}
- logging优先使用slog即可。
<!--more-->

## error handling

### error变量或者类型
```go
package myerrors

import "errors"
// Err前缀
var ErrNotFound = errors.New("not found")

// 或者自定义一个错误类型 实现Error()方法用于error wrap
// 一般是Error结尾
type ValidationError struct {
    Field string
    Msg   string
}

func (v *ValidationError) Error() string {
    return "validation failed: field=" + v.Field + ", msg=" + v.Msg
}
```
### error包装
```go
// 使用全局error变量
func Svc1() error {
    return myerrors.ErrNotFound
}
// 或者使用上面自定义错误类型
func Svc1() error {
    return &myerrors.ValidationError{
        Field: "email",
        Msg:   "invalid format",
    }
    
}
// wrap error
func CallSvc1() {
    err := Svc1()
    if err != nil {
        // 包装原始错误，保留错误链
        return fmt.Errorf("B failed: %w", err)
    }
    return nil
}
```
- %w 表示wrap，错误包装，99.99%的场景下面应该使用这个包装错误
- %v 表示value，等于value，适用于任意类型，%+v表示显示字段名
- %s 表示str 字符串（或 []byte 转换为字符串）
- %dboxXc 分别表示十进制 二进制 八进制 十六进制大小写和Unicode

### errors.Is
```go
import (
    "errors"
    "fmt"
    "myapp/myerrors"
)

func main() {
    err := CallSvc1()
    if err != nil {
        // 这里会一直unwrap到匹配或者最后
        if errors.Is(err, myerrors.ErrNotFound) {
            fmt.Println("handle not found error specifically")
        } else {
            fmt.Println("generic error:", err)
        }
    }
}
```
### errors.Join
```go 
func cleanup() error {
    var errs []error

    if err := closeFile(); err != nil {
        errs = append(errs, err)
    }
    if err := closeDB(); err != nil {
        errs = append(errs, err)
    }
    if err := closeNetwork(); err != nil {
        errs = append(errs, err)
    }

    return errors.Join(errs...) // 返回合并后的错误
}

// 可以使用errors.Is
func main() {
    err := doSomething()

    fmt.Println(errors.Is(err, ErrA)) // true
    fmt.Println(errors.Is(err, ErrB)) // true
}

```
### errors.As

```go
type MyError struct {
    Code int
    Msg  string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("code %d: %s", e.Code, e.Msg)
}

func doSomething() error {
    return fmt.Errorf("wrapper: %w", &MyError{Code: 404, Msg: "not found"})
}

func main() {
    err := doSomething()

    var myErr *MyError
    if errors.As(err, &myErr) {
        fmt.Println("Matched MyError, code:", myErr.Code)
    } else {
        fmt.Println("Not a MyError")
    }
}
```
`errors.As(err, &myErr)`将err转换成myErr类型并赋值给myErr，如果成功，则可以使用myErr，否则myErr为nil，执行else逻辑。

### if err styles
```go 
// 这是最常见、最推荐的写法，清晰简洁。
if err != nil {
    return err
}

// result 和 err 作用域只在 if 内部
if result, err := doSomething(); err != nil {
    return err
}
```

## logging
go 1.21有了slog之后，日志可以统一使用slog即可。下面是一个包含ctx实现链路日志的示例

```go 
// 日志初始化
import (
	"log/slog"
	"os"
)

func initLogger() *slog.Logger {
	handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
		Level: slog.LevelInfo,
	})
	logger := slog.New(handler)
	slog.SetDefault(logger)
	return logger
}

// 注入trace_id
import (
	"context"
	"github.com/google/uuid"
	"log/slog"
	"net/http"
)

func RequestContextMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		reqID := uuid.New().String()

		logger := slog.With("request_id", reqID)
		ctx := slog.NewContext(r.Context(), logger)

		// 将 context 传给下一个 handler
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}

// 在业务逻辑中使用slogger
func handleLogin(w http.ResponseWriter, r *http.Request) {
	ctx := r.Context()
	logger := slog.FromContext(ctx)

	logger.Info("user login", slog.String("user", "alice"))

	// or shorter:
	slog.InfoContext(ctx, "auth success", slog.String("user", "alice"))
}
```
其他logging资料：
- [Go official-Structured Logging with slog](https://go.dev/blog/slog)
- [Logging in Go with Slog](https://betterstack.com/community/guides/logging/logging-in-go/)