### Promise介绍

- JS的ES6规范中一门新的技术，是JS进行异步编程的新解决方案（旧方案为单纯使用回调函数）
- 常见的异步操作：
   - fs文件操作 `require('fs').readFile('./index.html',(err,data)=>{})`
   - 数据库操作
   - AJAX `$.get('/server',(data)=>{})`
   - 定时器 `setTimeout(()=>{},2000)`
- 旧的回调函数会出现回调地狱问题：回调函数的嵌套调用，外部回调函数异步执行的结果是嵌套的回调执行的条件。缺点是**不利于阅读、不便于异常处理**，并且回调函数需要在异步任务启动前指定
```javascript
asyncFunc1(opt, (...args1) => {
  asyncFunc2(opt, (...args2) => {
    asyncFunc3(opt, (...args3) => {
      asyncFunc4(opt, (...args4) => {
        // operation
      }
    }
  }
}
```

- Promise可以通过**链式调用**解决回调地狱的情况，通过返回的promise对象，可以给绑定指定一个或多个回调函数
### Promise基本使用

- promise基础使用
```javascript
<button class="btn btn-primary" id="btn">点击抽奖</button>
    <script>
        //生成随机数
        function rand(m,n){
            return Math.ceil(Math.random() * (n-m+1)) + m-1;
        }
        const btn = document.querySelector('#btn');
        btn.addEventListener('click', function(){
            // resolve 解决  函数类型的数据
            // reject  拒绝  函数类型的数据
            const p = new Promise((resolve, reject) => {
                setTimeout(() => {
                    let n = rand(1, 100);
                    if(n <= 30){
                        resolve(n); // 将 promise 对象的状态设置为 『成功』
                    }else{
                        reject(n); // 将 promise 对象的状态设置为 『失败』
                    }
                }, 1000);
            });
            console.log(p);
            p.then((value) => { // value 值
                alert('恭喜恭喜, 奖品为 10万 RMB 劳斯莱斯优惠券, 您的中奖数字为 ' + value);
            }, (reason) => { // reason 理由
                alert('再接再厉, 您的号码为 ' + reason);
            });
        });
    </script>
```

- fs模块封装
```javascript
const fs = require('fs')

// 回调函数形式
fs.readFile('./resource/content.txt', (err, data) => {
  if (err) { throw err; }
  console.log(data.toString());
}

// Promise形式
let p = new Promise((resolve, reject) => {
  fs.readFile('./resource/content.txt', (err, data) => {
  if(err) { reject(err); }
  resolve(data);   
  }
}
// Promise调用then
p.then((value) => {
    console.log(value.toString());
  }, (reason) => {
    console.log(reason);
  });

// 封装，返回promise对象，通过then进行链式调用
function mineReadFile(path){
  return new Promise((resolve, reject) => {
    require('fs').readFile(path, (err, data) => {
      if(err) { reject(err); }
      resolve(data); 
    }
  }
}
mineReadFile('./resource/content.txt').then(value=>{console.log(value.toString());}, 
                                            reason=>{console.log(reason);});          
```

- ajax请求封装
```javascript
function sendAJAX(url){
	return new Promise((resolve, reject) => {
		const xhr = new XMLHttpRequest();
		xhr.responseType = 'json';
		xhr.open("GET", url);
		xhr.send();
		//处理结果
		xhr.onreadystatechange = function(){
			if(xhr.readyState === 4){
				//判断成功
				if(xhr.status >= 200 && xhr.status < 300){
					//成功的结果
					resolve(xhr.response);
				}else{
					reject(xhr.status);
				}
			}
		}
	});
}
sendAJAX('https://api.apiopen.top/getJok')
.then(value => {
	console.log(value);
}, reason => {
	console.warn(reason);
});
```

- node中util.promisify()方法：符合Node.js中回调风格的函数都可以通过该方法转换，即
   - 最后一个参数是函数
   - 回调函数的参数为(err, data)，前面为可能的错误，后面为正常的结果
```javascript
const util = require('util');
const fs = require('fs');
//返回一个新的函数
let mineReadFile = util.promisify(fs.readFile);
mineReadFile('./resource/content.txt').then(value=>{
    console.log(value.toString());
});
```
### Promise的API

- Promsise的状态：一个Promise对象中，有一个属性[[PromiseState]]，代表着状态，总共有3种值：pending（未决定、等待）、resolved/fulfilled（成功）、rejected（失败）
- Promise中固有的另外一个属性[[PromiseResult]]，保存着Promise异步任务成功/失败的结果
- Promise的两种流程：
   - newPromise(pending状态) -> 异步操作 -> 成功，执行resolve() -> then()回调onResloved() -> 新的Promise对象
   - newPromise(pending状态) -> 异步操作 -> 失败，执行reject() -> then()回调onRejected() -> 新的Promise对象
1. Promise构造函数：Promise(excutor){}
   - executor函数是执行器(resolve, reject) => {}，该参数内的代码是立即执行的，即同步调用
   - resolve函数是内部定义成功时调用的函数 (value) => {}
   - reject函数时内部定义失败时调用的函数 (reason) => {}
```javascript
let p = new Promise((resolve, reject) => {
	// 同步调用
	console.log('a');
});
console.log('b');
```

2. Promise.prototype.then方法：(onResolved, onRejected) => {}
   - 指定成功/失败的回调，并返回一个新的promise对象
   -  onResolved 函数: 成功的回调函数 (value) => {}  
   -  onRejected 函数: 失败的回调函数 (reason) => {} 
3. Promise.prototype.catch方法：(onRejected) => {}
   -  只接收失败函数的回调
   - 相当于 then(undefined, onRejected)
```javascript
let p = new Promise((resolve, reject) => {
  // resolve(1);
  reject(1);
});
p.catch((reason) => { // 若为resolve(1)则不执行
  console.log(reason); // 1
});
```

4. Promise.resolve方法：(value) => {}
   - value：成功的数据或promise对象
```javascript
// 如果传入的参数为 非Promise 类型的对象, 则返回的结果为成功promise对象
let p1 = Promise.resolve(1); // p1状态为fulfilled，结果为1
// 如果传入的参数为 Promise 对象, 则参数的结果决定了 resolve 的结果
let p2 = Promise.resolve(new Promise((resolve, reject) => {
	// resolve('OK');
	// reject('Error');
}));
console.log(p2); // p2的结果为参数中resolve或reject结果
```

5. Promise.reject方法：(reason) => {}
   - reason：失败的原因，即无论传什么都是失败状态
```javascript
let p1 = Promise.reject(1); // p1状态为rejected，结果为1
let p2 = Promise.reject(new Promise((resolve, reject) => {
  resolve('OK');
  // reject('Error');
}));
console.log(p2); // p2状态为rejected，结果为传入的参数（完整的promise对象）
```

6. Promise.all方法：(promises) => {}
   - promises：由n个promise对象组成的数组
   - 只有数组中所有的promise都成功了才返回成功的promise，若有失败，则按顺序返回第一个失败的promise和reason
   - 返回值为一个新的pomise，成功时结果为数组，失败则为首个失败的结果
```javascript
let p1 = new Promise((resolve, reject) => {resolve(1);})
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);
const res = Promise.all([p1, p2, p3]); // res状态为fulfilled，结果为[1,2,3]

let p4 = Promise.resolve(4);
let p5 = Promise.reject(5);
let p6 = Promise.reject(6);
const res2 = Promise.all([p4, p5, p6]); // res状态为rejected，结果为5
```

7. Promise.race方法：(promises) => {}
   - promises：由n个promise对象组成的数组
   - 返回promise对象，其结果为首个完成promise状态的结果
   - 常见场景：异步请求后台数据时，超时未完成则返回下一个promise
```javascript
let p1 = new Promise((resolve, reject) => {setTimeout(() => {resolve(1);}, 1000);})
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);
const result = Promise.race([p1, p2, p3]); // res状态为fulfilled，结果为2
```
### Promise一些关键问题

- 想让一个一个promise对象的状态改变，只有以下三种方法：
   - resolve()函数
   - reject()函数
   - throw抛出错误
```javascript
let p = new Promise((resolve, reject) => {
	//1. resolve 函数
	resolve('ok'); // pending   => fulfilled (resolved)
	//2. reject 函数
	reject("error"); // pending  =>  rejected
	//3. 抛出错误
	throw '出问题了'; // pending  =>  rejected
});
```

- 当promise指定多个失败/成功的回调时，只要对应的状态改变，则所有相应回调都会触发，按代码顺序执行
```javascript
let p = new Promise((resolve, reject) => {
	resolve('OK');
});
p.then(value => {console.log('1');});
p.then(value => {console.log('2');});
// 按顺序输出 1 2
```

- 改变promise状态和**指定**promise的回调函数（注意不是执行），这两个操作的顺序会根据具体的代码而变化，但不论顺序如何，所有的回调函数都需要对应的状态改变后才能被调用
- 如果改变promise状态的代码在异步函数内，则会先绑定回调函数，再改变promise状态，然后再执行对应的回调代码
- 在执行器中直接调用resolve()/reject()或延迟更长时间调用then()可以先改变状态再指定回调函数，然后执行回调代码
- 无论顺序如何变化，都只有在状态发生改变时对应的回调函数才能拿到结果（数据）
   - 异步函数：异步调用->指定回调->状态改变->执行回调->取得结果
   - 非异步函数：执行代码->状态改变->指定回调->执行回调->取得结果
```javascript
// 非异步
let p = new Promise((resolve, reject) => {
	console.log(1) // 执行顺序1
	resolve(2); // 执行顺序2（改变状态）
});
p.then(value => { // 执行顺序3（指定回调函数）
	console.log('4'); // 执行顺序4（执行回调函数）
});
// 异步
let p1 = new Promise((resolve, reject) => {
	console.log(1) // 执行顺序1
	setTimeout(() => {
		resolve(3); // 执行顺序3（改变状态）
	}, 1000);
});
p1.then(value => { // 执行顺序2（指定回调函数）
	console.log(4); // 执行顺序4（执行回调函数）
})
```

- 调用promise.then()方法时返回的新promise状态由回调函数的执行结果决定
   - 抛出异常时，新promise变为rejected，reason为抛出的异常
   - 返回值为非promise对象时，新promise变为resolved，value为返回的值
   - 返回值为新的promise对象时，新promise的结果变成返回值的promise结果
```javascript
let p = new Promise((resolve, reject) => {resolve('ok');});
let res = p.then(value => {
	//1. 抛出错误
	throw 'error'; // 状态rejected，结果error
	//2. 返回结果是非 Promise 类型的对象
	return 1; // 状态fulfilled，结果1
	//3. 返回结果是 Promise 对象
	return new Promise((resolve, reject) => { // res的状态和结果由该promise的状态和结果决定
		// resolve('success');
		// reject('error');
	});
});
```

- 由于promise.then()方法的返回值依旧为一个promise对象，故promise可以通过then().then().then()的链式调用方式，实现多个操作任务的串联，每一个then()内都可以是同步或异步任务。一个**细节**：promise.then()里若没有进行return的返回值操作，则默认返回状态为fulfilled，值为undefined的promise对象
```javascript
let p = new Promise((resolve, reject) => {
	setTimeout(() => {resolve('OK');}, 1000);
});
p.then(value => {
	return new Promise((resolve, reject) => {
		resolve("success");
	});
}).then(value => {
	console.log(value);
}).then(value => {
	console.log(value);
})
// 输出结果按顺序依次为：success、undefined
```

- 异常穿透：promise.then()的链式调用中，只需要在最后 .catch() 即可捕捉链式调用环节中的任一环节错误
```javascript
let p = new Promise((resolve, reject) => {
	setTimeout(() => {resolve('OK');}, 1000);
});
p.then(value => {
	console.log(1);
}).then(value => {
	throw 'error';
}).then(value => {
	console.log(3);
}).catch(reason => {
	console.log(reason);
});
// 输出结果按顺序依次为：1，error
```

- 中断promise链条：在promise链式调用中，如果只是想单纯的进行某一环节之后的调用，有且只有一个方式——修改当前then()方法返回值为一个pending状态的promise（不应该用异常穿透去实现这个想法）
```javascript
let p = new Promise((resolve, reject) => {resolve('OK');});
p.then(value => {
	console.log(1);
	return new Promise(() => {}); // 链式调用从此处终止
}).then(value => {
	console.log(2);
});
```
### Promise自定义封装

- 水平有限，后续深入理解后补充
### [async](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function) 与 [await](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)

- async函数用来声明一个async function name(){}，其返回值为promise对象，该promise对象的结果由async函数的返回值决定（该特点与then()相似）
```javascript
async function main(){
	//1. 如果返回值是一个非Promise类型的数据
	return 1; // res状态为fulfilled，结果为1
	//2. 如果返回的是一个Promise对象
	return new Promise((resolve, reject) => { // res改回结果为该promise对象返回结果
		// resolve('OK');
		// reject('Error');
	// });
	//3. 抛出异常
	throw "error"; // res状态为rejected，结果为error
}
let res = main();
console.log(res);
```

- await通常情况下右侧表达式为promise对象，会返回promise的成功值，如果是promise的reject状态，则会报错，需要把代码通过try{}catch{}进行捕获错误，可以获取reject的结果值。如果是其他值则直接返回该值
```javascript
async function main(){
	let p = new Promise((resolve, reject) => {
		resolve('success');
		// reject('error');
	})
	//1. 右侧为promise的情况
	let res = await p; // res输出为 success
	//2. 右侧为其他类型的数据
	let res2 = await 10; // res2输出为 10
	//3. 如果promise是失败的状态
	try{
		let res3 = await p;
	}catch(e){
		console.log(e); // 输出 error
	}
}
main();
```

- async与await结合使用，解决回调地狱问题
```javascript
const fs = require('fs');
const util = require('util');
// 将fs.readFile原本为异步回调函数转换为promise实例方法
const mineReadFile = util.promisify(fs.readFile); 
// 回调函数的方式
fs.readFile('./resource/1.html', (err, data1) => {
    if(err) throw err;
    fs.readFile('./resource/2.html', (err, data2) => {
        if(err) throw err;
        fs.readFile('./resource/3.html', (err, data3) => {
            if(err) throw err;
            console.log(data1 + data2 + data3);
        });
    });
});
// async 与 await 处理
async function main(){
    try{
        // 读取第一个文件的内容
        let data1 = await mineReadFile('./resource/1.html');
        let data2 = await mineReadFile('./resource/2.html');
        let data3 = await mineReadFile('./resource/3.html');
        console.log(data1 + data2 + data3);
    }catch(e){ // 统一捕获错误
        console.log(e);
    }
}
main();
```

- 例子，axios的基本原理
```javascript
<button id="btn">点击获取段子</button>
<script>
	// 此处的封装则是axios的简单实现
	function sendAJAX(url){
		return new Promise((resolve, reject) => {
			const xhr = new XMLHttpRequest();
			xhr.responseType = 'json';
			xhr.open("GET", url);
			xhr.send();
			xhr.onreadystatechange = function(){
				if(xhr.readyState === 4){
					if(xhr.status >= 200 && xhr.status < 300){
						resolve(xhr.response);
					}else{
						reject(xhr.status);
					}}}});
}
let btn = document.querySelector('#btn');
btn.addEventListener('click',async function(){ // 此处使用了async
	let duanzi = await sendAJAX('https://api.apiopen.top/getJoke');
	console.log(duanzi);
});
</script>
```












