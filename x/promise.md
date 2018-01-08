## Promise大法好
---

现实生活中如果做太多承诺是件麻烦事，但在js里面就不一样啦。可以在心情好的时候就向你的代码来一记`new Promise()`，毕竟最后兑现这个承诺的不是你~所幸我成为前端时前后端分离已经大行其道，感谢Ajax。现在写代码好像已经离不开`Promise`了，刚开始还有点糊里糊涂，不知道为什么有的表达式后面就可以`then`了，也搞不太清`Promise`的几种状态。后来看了几篇很有价值的文档，也经过日积月累的实践，终于是能比较有底气地为自己的代码“做出一个承诺”了。这篇md就来总结一下自己理解的`Promise`，以及几个具有代表性的应用`Promise`的场合。


### Promise迷思
曾经对`Promise`产生过很多疑问，虽然可以用起来，但是始终不能掌握它的要领。看MDN上的文档，全面是很全面，但感觉有点太“干”了，咽不下去啊。后来果然还是实践出真知，现在已经对当时的一些“迷思”有一个清楚的认识了。

#### 1. `new Promise()`生成的对象到底代表了什么？这个对象的不同状态都是什么意思？这些状态对此对象又意味着什么？

当时这三连击拷问了我很久啊。首先，`new Promise()`生成的是一个`Promise`对象(哈哈哈好像冷笑话我忍不住了)。在我理解，这个`Promise`对象代理了一个你在生成此对象时不一定产生了最终结果的值或者过程，此值或者过程通常来自一个异步操作，你对ta有一定预期——你希望在这个过程结束或者这个值有确定结果后去做一些事情。自然，这个对象大体会有两类状态：代理的值/过程已经产生了最终结果，或者没有。在MDN的文档里，一个`Promise`对象有下面三种状态：

* *pending*: 初始，等待
* *fulfilled*: 成功
* *rejected*: 失败

第一种状态表示代理值/过程未产生最终结果，后面两种状态分别表示代理值/过程最终产生了成功/失败的结果。这种成功/失败的界定完全取决于你对这个`Promise`的预期是怎样的，你完全可以自行判定某种结果归为成功还是失败。在`Promise`对象代理的值或者过程有最终结果之前，你是不能对ta做任何事的，因为你还没有拿到你关心的结果。三连击KO，嘿嘿！

#### 2. `.then()`代表了什么？`.then()`又为何能够串联？

接着前面三连击的回答，既然你首先在代码中生成了一个`Promise`对象，那你必然关心某个未来的值/过程的最终结果，那么`.then()`就是你可以在这个最终结果产生后做一些事情的地方。像这样调用一个`Promise`对象`then`方法：`.then(onFulfilled, onRejected)`，那么在此`Promise`对象的状态为*fulfilled*或者*rejected*，即其代理的值/过程有最终结果的时候，你在`.then()`中写好的回调函数`onFulfilled`/`onRejected`就会被调用，这两个回调就是你针对成功/失败的不同结果所做的处理。

`.then()`能够串联，是因为调用`then`方法返回的就是一个`Promise`对象；对于一个`Promise`对象，自然可以按照上面的逻辑继续调用`then`方法啦。


### Promise应用
目前我遇到的`Promise`按照代理内容物的不同，大体可以分为两类——代理值和代理过程，下面就分别举例来说明一下~

#### 1. 代理值

这一类比较典型的应用是在网络请求中，用`Promise`对象来代理请求成功/失败的数据。比如下面是一个用`Promise`封装，基于Ajax的`get`请求：
    
``` javascript  
const get = (url, param) => {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        xhr.onload = () => {
            if (xhr.status == 200) {
                resolve(JSON.parse(xhr.response))
            } else {
                reject(Error(`${xhr.status}[${xhr.statusText}]`))
            }
        };
        
        xhr.onError = () => {
            reject(Error('Network Error!'))
        }
        
        let paramStr = '';
        if (param !== undefined) {
            paramStr = Object.keys(param).map(pk => `${pk}=${param[pk]}`).join('&');
        }
        
        const urlWithParam = param !== undefined ? `${url}?{paramStr}` : url;
        
        xhr.open('GET', urlWithParam);
        xhr.send();
    })
}
```

此`get`方法返回的是一个`Promise`对象，其代理的就是这个`get`请求的结果。这个结果是我们在构造`Promise`对象时，传入的参数(一个函数)中调用`resolve`或者`reject`获得的。调用`resolve`或者`reject`时传入的参数，就是我们在后面调用`Promise`对象的`then`方法时，其成功/失败的回调函数获得的参数。比如：

``` javascript
get('/whatever/url')
.then(data => {
    // do whatever you want with the successful response
    // ...
}, err => {
    // handle errors that can happen
    // ...
})
```

在上面的代码中，调用`then`方法时传入的第一个成功回调能够拿到`resolve`后的结果，第二个失败回调能够拿到`reject`后的结果。这样我们就可以在相应的回调函数中进行后续的操作了。

当然，这只是`Promise`的初级用法。前面的【迷思】部分提到过，`.then()`返回的是一个`Promise`对象，所以可以进行串联——我们可以利用这一点，将后续所有可能的异步操作放到一个个级联的`then`方法里面，避免陷入回调深渊。比如以下场景中，我们计划先获取用户购物车中的商品列表，再根据商品`id`获取其主要信息(名称，图片)等，然后展现在页面上：

``` javascript
// get cartList
get('/cartList.json')
.then(data => {
    // get product detail
    // ...
})
.then(data => {
    // display
    // ...
})
```

在第一个`get`请求后，假设我们在`then`方法的成功回调中拿到的`data`是如下格式：

``` json
{
    list: [
        {
            id: "avb5487fh",
            amount: 2
        }, {
            id: "54gret46g",
            amount: 1
        }, {
            id: "vjh39hd48",
            amount: 1
        }, ...
    ]
}
```

`list`属性中每个对象都包含了一个商品的`id`和数量，我们需要根据`id`再获得所有商品的信息，这又包含若干次请求。由于请求都是异步的，所以我们可以将这若干次请求一起发出，等到它们返回结果后再集中展示。这时就可以用到一个方法——`Promise.all()`。`Promise.all()`接收一组`Promise`，返回一个`Promise`对象，代理这组`Promise`的执行情况，比如如果所有`Promise`都成功`resolve`后，`Promise.all()`的值就是一个数组，包含所有`Promise`对象的最终结果，其顺序与`Promise`对象数组的顺序保持一致。所以我们上面的代码可以这样完善：

``` javascript
// get cartList
get('/cartList.json')
.then(data => {
    // get product detail
    const productReq = data.map(p => get('/product.json', { id: p.id }));
    
    return Promise.all(productReq)
})
.then(data => {
    // display all products
    // ...
})
```

这样我们就能在后面的`then`方法中进行商品展示了。但是，由于`Promise.all()`在所有`Promise`成功(或某一个失败)后才会返回有最终结果的`Promise`对象，所以我们是在拿到所有商品的信息后才进行展示，这中间无法呈现内容给用户。如果可以拿到一条信息就展示一条，体验会好很多。

首先，要保证所有商品请求并行发出，其次我们需要为了顺序而创建一个展示序列。这里我学习借鉴了Google Developers上面一篇[关于Promise的文章](https://developers.google.com/web/fundamentals/primers/promises)，里面的用法当时真的惊艳了我！具体是这样操作的：

``` javascript
// get cartList
get('/cartList.json')
.then(data => {
    // get product detail
    const productReq = data.map(p => get('/product.json', { id: p.id }));
    
    productReq.reduce((sequence, productPromise) => {
        return sequence.then(() => {
            return productPromise
        }).then(() => {
            // display each product
        })
    }, Promise.resolve())
})
```

就是这样，来自顶级互联网公司的大佬把`Promise`可以串联的特性和`reduce`产生单一对象的特点完美地融合在了一起！当时这里我看了好半天，现在再来缕一遍：

1. `map`产生一组`Promise`对象，将所有获取商品信息的请求发出；
2. 对这组请求进行`reduce`操作，在一个已经成功的`Promise`基础上(`Promise.resolve()`)进行后续操作，保证我们一直在同一个`Promise`上调用`then`方法，并保证了展示的顺序。这个`Promise`最终串联下来的效果类似下面：

    ``` javascript
    Promise.resolve()
    .then(productPromise_0)
    .then(() => {
        // display product 0
    })
    .then(productPromise_1)
    .then(() => {
        // display product 1
    })
    ...
    .then(productPromise_n-1)
    .then(() => {
        // display product n-1
    })
    ```

    而`` `productPromise_${i}` ``就是我们已经在第一步的`map`操作中发送的若干请求之一，只要执行到``.then(`productPromise_${i}`)``时这个请求已经返回成功的结果，就能马上执行其后的`.then()`了。这样一来，我们就可以按顺序展示已经拿到的商品信息，而不用等所有结果都返回再统一展示了。Google Developers的那篇文章中有实际的例子，比较直观的反映了用户体验上的差别，值得一看~
    
#### 2. 代理过程

代理过程其实比较难概括性地举例，我只能说出一两个应用场景。比如，对于某段过程，你不知道或者不关心ta会持续多长时间，但是你需要在这段过程结束后进行一些其他操作。通常这两部分代码是分离的，描述过程的代码又通常是一个通用的公用方法，你不好深入其中去评估ta的具体执行时间，更不可能在那里植入自己的代码。这时候你就会期盼，如果那段过程代码返回的是一个`Promise`对象就好了，你可以轻松在后面`.then()`一个，做你爱做的事啦。

想象这样一个场景——一个未登录用户打开了一个需要登录的页面，你需要在页面加载后先展示一个吐司提示用户未登录，然后在吐司消失后跳转至登录页面。吐司作为公用的组件，你可能不知道ta在页面上会停留多久，你也不能在吐司的代码中添加自己的逻辑；就算知道吐司2s后就会消失，通过手动`setTimeout()`将后续操作延迟至吐司消失后，也肥肠蠢——万一哪天产品说，不行，咱这吐司消失得太快了，用户连上面的字儿都没看清呢，给我延长到10s再消失~~！你就要哭瞎啦~所以这里，可以将展示吐司的方法返回一个`Promise`对象，并在吐司消失的特定时间`resolve`之，我们就可以在调用ta之后再调用`then`方法，将我们后续的操作写进回调里，干净又整洁，也大大提高了代码的可维护性。


慢慢写到现在，感觉自己对`Promise`的认识又深刻了一些些。以后有时间，还是要多多总结才是。能写一段代码不代表理解了其中的奥义，而能把这件事讲清楚……也不一定意味着正确理解了写下的每一句话，哈哈哈。多多加油，继续学习和实践吧~(这个Promise我就写这儿了，不怕立flag，吼吼~~)












