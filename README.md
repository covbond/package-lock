# package-lock.json 是什么？

大部分时候我们都不会去关注`package-lock.json`这个文件，因为大部分时候我们都不需要去关注自己使用的库的版本，因为这个原因，读者一般也不需要阅读本文。当我们对需要使用的库有明确的版本要求，或者自己写了一个库，在测试的时候需要知道自己测的是哪一版，这个时候就需要注意`package-lock.json`了。

以下我们通过一些实际试验来了解`package-lock.json`，相关代码：[https://github.com/covbond/package-lock](https://github.com/covbond/package-lock).

## package-lock.json是如何生成的？

命令行切换到[create-lock](https://github.com/covbond/package-lock/tree/master/create-lock)中，执行：

```shell
npm i lodash
```

执行的结果是`package.json`中`dependencies`增加了`lodash`字段，通常是最新版本，在我试验的时候，它是`4.17.15`：

```diff
{
  "name": "create-lock",
  "version": "1.0.0",
  "description": "create package-lock.json",
  "main": "index.js",
  "scripts": {},
  "author": "covbond",
  "license": "MIT",
  "dependencies": {
+    "lodash": "^4.17.15"
  }
}
```

此外，还生成了`package-lock.json`文件：

```javascript
{
  "name": "create-lock",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
    "lodash": {
      "version": "4.17.15",
      "resolved": "https://registry.npmjs.org/lodash/-/lodash-4.17.15.tgz",
      "integrity": "sha512-8xOcRHvCjnocdS5cpwXQXVzmmh5e5+saE2QGoeQmbKmRS6J3VQppPOIt0MnmE+4xlZoumy0GPG0D0MVIQbNA1A=="
    }
  }
}
```

我们注意到`package.json`中`lodash`版本前面多了一个`^`，它表示：以后如果有人把这个项目拉下来安装的话，给他安装的版本要和`4.17.15`兼容，具体来说就是新安装的版本范围是：`>= 4.17.15 且 < 5.0.0`。需要了解更多的话，可以搜索“npm 语义化版本”。

现在的问题是，如果别人使用你的项目的时候，`lodash`已经出到`4.18.0`了，那么当他把项目代码拉下来，执行`npm install`的时候，最终安装的`lodash`会是哪个版本呢？是`4.18.0`还是`4.17.15`？

这和你上传代码文件的时候有没有上传`package-lock.json`是有关的。

由于在写这篇文章的时候，`lodash`版本还没有出到`4.18.0`，为了得到问题的答案，我们可以将`lodash`的版本设置为`^4.0.0`，这是因为这两种情况是类似的，我们可以从`^4.0.0`的情况推测`^4.17.15`的情况。

在[without-lock](https://github.com/covbond/package-lock/tree/master/without-lock)文件夹下我们准备了一份`package.json`文件，其中`lodash`的版本为`^4.0.0`。将命令行切换到这个文件夹下，执行`npm install`，我们会发现，最终安装的版本不是`4.0.0`，而是当前最新的`lodash`版本，这是符合预期的。

但是，如果加上`package-lock.json`文件，最终安装的`lodash`版本会是怎样的呢？

## 使用lock文件

在[with-lock](https://github.com/covbond/package-lock/tree/master/with-lock)文件夹中，我们额外准备了一个`lock`文件（`package-lock.json`名字太长，我们用`lock`指代它），其中`lodash`的版本是`4.0.0`。

我们在这个文件夹下执行`npm install`后，会发现`node_modules`中`lodash`的版本是`4.0.0`。

经过这两个例子的对比，我们就会发现，看起来好像是：如果有`lock`文件，那么最终版本由`lock`文件中的版本决定，否则是由`package.json`中的语义化版本决定的。

那么是不是可以说有`lock`就只用看`lock`，没`lock`就看`package.json`呢？

其实不是这样的，我们可以再做一个试验。

## 手动修改`package.json`会怎样？

有时候我们想更新一个库，就直接编辑`package.json`文件改版本，然后跑`npm install`。这样的话，最后安装的版本是怎样确定的呢？

切换到[modify-package-1](https://github.com/covbond/package-lock/tree/master/modify-package-1)，准备的`package.json`文件中已经将`lodash`的版本手动从`^4.17.15`改成了`^4.0.0`, 这就相当于我们手动修改了`lodash`的版本，执行`npm install`，然后我们会发现`lock`文件一点变化也没有。最终安装的版本还是`4.17.15`。

切换到[modify-package-2](https://github.com/covbond/package-lock/tree/master/modify-package-2)，这次准备的`package.json`文件中版本我们手动改成了`^3.0.0`。如果执行`npm install`，最终会安装什么样的版本呢？

从上一部分得到的结论来推测，我们有`lock`文件，应该全部都听`lock`的，即使`package.json`中版本是`^3.0.0`（`>= 3.0.0 且 < 4.0.0`），也不用管。

可是，当执行`npm install`后，我们发现，最终安装的版本是`3.10.1`（它是满足范围要求的最后一个版本，在[lodash](https://www.npmjs.com/package/lodash)中可以查看这个库的历史版本）。而且`lock`文件被修改了：

```diff
{
  "name": "test-lock",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
    "lodash": {
-      "version": "4.17.15",
-      "resolved": "https://registry.npmjs.org/lodash/-/lodash-4.17.15.tgz",
-      "integrity": "sha512-8xOcRHvCjnocdS5cpwXQXVzmmh5e5+saE2QGoeQmbKmRS6J3VQppPOIt0MnmE+4xlZoumy0GPG0D0MVIQbNA1A=="
+      "version": "3.10.1",
+      "resolved": "https://registry.npmjs.org/lodash/-/lodash-3.10.1.tgz",
+      "integrity": "sha1-W/Rejkm6QYnhfUgnid/RW9FAt7Y="
    }
  }
}
```

从这个例子中我们可以看出，最终结论应该是：对一个库来说，`package.json`中说要这个版本，`lock`中说要那个版本，如果没冲突的话，就全听`lock`的，否则听`package.json`的，而且还会更新`lock`文件。

## 参考资料

这篇文章受[But what the hell is package-lock.json?](https://dev.to/saurabhdaware/but-what-the-hell-is-package-lock-json-b04)启发而写成。

## 运行环境

* node: v12.16.1
* npm: 6.13.4