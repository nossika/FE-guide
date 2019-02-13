# 事件循环

node中的事件循环通过libuv库实现：

- timers：这个阶段执行timer（setTimeout、setInterval）的回调
- I/O callbacks：执行一些系统调用错误，比如网络通信的错误回调
- idle, prepare：仅node内部使用
- poll：获取新的I/O事件, 适当的条件下node将阻塞在这里
- check：执行 setImmediate() 的回调
- close callbacks：执行 socket 的 close 事件回调

每个阶段间隙都会检查并执行microtask。

    setTimeout(() => {
      console.log('timeout 1');
      Promise.resolve().then(() => console.log('promise 1'));
    });
    setTimeout(() => {
      console.log('timeout 2');
      Promise.resolve().then(() => console.log('promise 2'));
    });


以上例子在浏览器中运行结果：timeout 1 / promise 1 / timeout 2 / promise 2，在node中运行结果：timeout 1 / timeout 2 / promise 1 / promise 2

node会在timers阶段统一检查当前是否有到期的定时器任务，有的话会把它们放在同一次task中执行（不同于浏览器，一个定时器任务算一个task，在task间隙执行microtask）