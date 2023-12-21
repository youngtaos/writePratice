# Promise

## Promise.race()

如果请求可以在 200ms 内完成，则不显示 loading，如果要超过 200ms，则至少显示 200ms 的 loading。
.

```
let getUserInfo = function (user) {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve('姓名：杨涛森'), Math.floor(400 * Math.random()));
    });
}

let showUserInfo = function (user) {
    return getUserInfo().then(info => {
        console.log('用户信息', info);
        return true;
    });
}

let timeout = function (delay, result) {
    return new Promise(resolve => {
        setTimeout(() => resolve(result), delay);
    });
}

// loading时间显示需要
let time = 0;
let showToast = function () {
    time = +new Date()
    console.log('show loading...');
}
let hideToast = function () {;
    console.log('hide loading' + (+new Date() - time));
}
// 执行代码示意
let promiseUserInfo = showUserInfo();
Promise.race([promiseUserInfo, timeout(200)]).then((display) => {
    if (!display) {
        showToast();

        Promise.all([promiseUserInfo, timeout(200)]).then(() => {
            hideToast();
        });
    }
});
```
