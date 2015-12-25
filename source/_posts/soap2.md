title: "php使用soap通信(二)"
date: 2015-04-08 11:48:56
categories: PHP
tags: soap

---

第一次返回的数据达不到要求，客户要求需要50几个字段，但是始终没办法在wsdl文件中定义50多个标签，第一篇中的wsdl模板太过简单，没有自定义类型。
<!--more-->

经过一番疯狂的查资料，wsdl文件中可以自定义参数和返回值为复杂类型。
网上有很多生成wsdl的库，我使用的是这一个：[wsdl-creator](https://github.com/piotrooo/wsdl-creator)

它是基于php类使用注释和反射机制，支持php 5.3

作者的github里有使用方法的详细介绍。这里只简单的描述一下。

##获取wsdl-creator
原作者使用composer管理这个东西，你可以使用composer建立一个项目。
```bash
	composer create-project piotrooo/wsdl-creator projectName
```
**注：如果半天拉不下来，说明需要VPN**
为了防止关键时候没有VPN的尴尬，我拉下来后放在了自己的github
```bash
	git clone https://github.com/lybc/wsdl-creator.git
```


##如何使用wsdl-creator生成wsdl
1. 创建一个类里面写好供客户调用的接口，因为使用了wsdl生成库，所以接口类的注释是有讲究的
```php
class ServiceOrder{
    /**
     * @type int
     */
    public $id;
    /**
     * @type string
     */
    public $name;
    /**
     * @type string
     */
    public $message;
    /**
     * @param string $a
     * @param string $b
     *
     * @return object $ServiceOrder @int=$id @string=$name @string=$message
     */
    public function returnObj($a = null, $b = null){
        $this->id = $a;
        $this->name = $b;
        $this->message = $a . $b;
        return $this
    }
}
```
2. 创建一个文件根据ServiceOrder类调用wsdl-creator中的方法
```php
	require_once 'vendor/autoload.php';
	use WSDL\WSDLCreator;
    
    // 第一个参数是接口类的类名，第二个参数是服务端的URL
    $wsdl = new WSDL\WSDLCreator('ServiceOrder', 'http://localhost/wsdl-creator/ServiceOrderServer.php');
    $wsdl->renderWSDL();
```
**注：参数必须写正确，如果参数错误调用时会报错**

写好后执行这个文件，如果成功会在浏览器中得到一段XML，`ctrl + S`保存为你需要的文件名`*.wsdl`

得到wsdl文件之后即可像第一篇中一样编写服务端和客户端，通过WSDL互相取值。下面贴一段我测试时使用python调用接口的代码。
```python
	from suds.client import Client
    url = "http://localhost/mgs/bin/soap/ServiceOrder.wsdl"; # wsdl文件路径
    c = Client(url)
    result = c.service
    obj = result.returnObj("a","b");
    print(obj)
```
