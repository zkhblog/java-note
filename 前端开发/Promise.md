① Promise是es6中引入的进行异步编程的新解决方案  
② 当promise改变为对应状态时，指定的多个成功/失败的回调函数，都会被调用  
③ 中断Promise链条的唯一方法：```return new Promise(() => {});```  
④ 应用在读取文件、发送Ajax请求等方面  

优势：  
① 支持链式调用  
② 不用在启动异步任务前指定回调函数，而是在启动异步任务时，返回promise对象，给promise对象绑定回调函数，甚至还可以在异步任务结束后指定多个回调函数来处理成功或失败的结果  

Promise实例对象的两个属性  
① PromiseState：pending（未决定的）/resolved（或fulfilled，意思是成功）/rejected（失败）  
② PromiseResult：保存着异步任务成功或失败的结果  

API  
① Promise构造函数：Promise(executor){}  
executor函数(即(resolve,reject) => {})会在Promise内部立即调用，异步操作在执行器中执行  
② Promise.prototype.then方法：(onResolved,onRejected) => {}  
③ Promise.prototype.catch方法：失败的回调函数  
④ ```Promise.resolve()```  
如果传入的参数为非Promise类型的对象，则返回的结果为成功的Promise对象。如果传入参数为Promise对象，则参数的结果决定了resolve的结果  
⑤ Promise.reject：快速返回一个失败的Promise对象  
⑥ Promise.all：包含n个Promise的数组  
返回一个新的Promise对象，只有所有的Promise都成功才成功，只要有一个失败了就直接返回  
⑦ Promise.trace：第一个完成的Promise的结果状态就是最终的结果状态，不一定是顺序上的第一个，因为第一个是定时器时，最先改变结果状态的可能是第二个Promise  

改变Promise对象的状态的三种方式  
① resolve('ok');  
② reject('error');  
③ 抛出错误：throw 'Error';  



