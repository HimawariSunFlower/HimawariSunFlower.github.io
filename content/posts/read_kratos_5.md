---
title: "go-kratos源码阅读之errgroup"
date: 2022-09-28T17:11:27+08:00
draft: false
categories: ["go-kratos"]
---

阅读kratos源码发时候，发现在app.Run()方法里调用了一个叫errgroup的官方库(居然和singleflight一个库)。      
决定去看一下这是个什么样的库，[golang.org/x/sync/errgroup](https://pkg.go.dev/golang.org/x/sync@v0.0.0-20220923202941-7f9b1623fab7/errgroup)。  

发现是官方对goroutines group的一个封装。  

    Package errgroup provides synchronization, error propagation, and Context cancelation for groups of goroutines working on subtasks of a common task.

errgroup的结构：  

```
    type Group struct {
        cancel func()

        wg sync.WaitGroup

        sem chan token

        errOnce sync.Once
        err     error
    }
```

errgroup提供了以下方法：

```
    func WithContext(ctx context.Context) (*Group, context.Context)
    func (g *Group) Go(f func() error)
    func (g *Group) SetLimit(n int)
    func (g *Group) TryGo(f func() error) bool
    func (g *Group) Wait() error
```

withcontext是根据context去build一个errgroup，并返回从ctx派生的context。派生的Context在传递给Go的函数第一次返回非nil错误或Wait第一次返回时被取消，以先发生的情况为准。  

go是开一个goroutine去调用func，如果setlimit设置了goroutine的数量并超出了数量，会阻塞。内置的waitgroup会在运行func的时候add，完成时done。如果组是通过调用WithContext创建的，第一个返回非nil错误的调用将取消组的上下文。该错误将由Wait返回。  

SetLimit将该组中活动的goroutine的数量限制为最多n个。负值表示不限制。对Go方法的任何后续调用都将阻塞，直到它能够添加一个活动的gor例程而不超过配置的限制。当组中的任何gor例程处于活动状态时，不能修改该限制。  

TryGo只有当组中活动gor例程的数量当前低于配置的限制时，TryGo才在新的gor例程中调用给定的函数，不回去阻塞。返回值报告goroutine是否已启动。  

Wait等待阻塞，直到所有来自Go方法的函数调用都返回，然后从它们返回第一个非nil错误(如果有的话)。

记录一下kratos的调用[app.go](https://github.com/go-kratos/kratos/blob/HEAD/app.go):
```
func (a *App) Run() error {
    ...
    eg, ctx := errgroup.WithContext(NewContext(a.ctx, a))
	wg := sync.WaitGroup{}
	for _, srv := range a.opts.servers {
		srv := srv
		eg.Go(func() error {
			<-ctx.Done() // wait for stop signal
			stopCtx, cancel := context.WithTimeout(NewContext(a.opts.ctx, a), a.opts.stopTimeout)
			defer cancel()
			return srv.Stop(stopCtx)
		})
		wg.Add(1)
		eg.Go(func() error {
			wg.Done() // here is to ensure server start has begun running before register, so defer is not needed
			return srv.Start(NewContext(a.opts.ctx, a))
		})
	}
	wg.Wait()
    ...
	c := make(chan os.Signal, 1)
	signal.Notify(c, a.opts.sigs...)
	eg.Go(func() error {
		select {
		case <-ctx.Done():
			return nil
		case <-c:
			return a.Stop()
		}
	})
	if err := eg.Wait(); err != nil && !errors.Is(err, context.Canceled) {
		return err
	}
    return nil
}
```