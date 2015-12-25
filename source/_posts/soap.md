title: "php使用soap通信(一)"
date: 2015-04-07 09:56:30
categories: PHP
tags: soap

-----
做外部可调用的接口，要求根据一个几个参数返回所有的字段信息，需要使用soap通信。
<!--more-->
**什么是soap通信**
假设现在有三方系统互相需要通信互相调用，这三个系统一个使用C#语言，一个使用Java语言，一个使用PHP语言，这三方系统的数据结构彼此不相同，如何实现互通？
这时候就需要一个独立于语言和平台之外的东西来结构化三方的数据来实现彼此互通的目的，于是我们需要使用soap。简单的说，soap是一种基于xml的http通信协议。

**维基百科搬运：**
>SOAP（原为Simple Object Access Protocol的首字母缩写，即简单对象访问协议）是交换数据的一种协议规范，使用在计算机网络Web服务（web service）中，交换带结构信息。SOAP为了简化网页服务器（Web Server）从XML数据库中提取数据时，节省去格式化页面时间，以及不同应用程序之间按照HTTP通信协议，遵从XML格式执行资料互换，使其抽象于语言实现、平台和硬件。此标准由IBM、Microsoft、UserLand和DevelopMentor在1998年共同提出，并得到IBM，莲花（Lotus），康柏（Compaq）等公司的支持，于2000年提交给万维网联盟（World Wide Web Consortium；W3C），目前SOAP 1.1版是业界共同的标准，属于第二代的XML协定（第一代具主要代表性的技术为XML-RPC以及WDDX）。

用一个简单的例子来说明SOAP使用过程，一个SOAP消息可以发送到一个具有Web Service功能的Web站点，例如，一个含有房价信息的数据库，消息的参数中标明这是一个查询消息，此站点将返回一个XML格式的信息，其中包含了查询结果（价格，位置，特点，或者其他信息）。由于数据是用一种标准化的可分析的结构来传递的，所以可以直接被第三方站点所利用。

#PHP使用soap通信的方法
soap有两种操作模式，**NO-WSDL**和**WSDL**，WSDL是基于XML的网络服务描述语言，用于描述网络服务，也可以定义网络服务

##1. NO-WSDL方式使用soap
首先我们来定义客户端**client**和服务端**server**，在客户端调用服务端提供的接口，得到返回的数据后，就完成了一次soap通信

**client.php客户端代码：**
```php
ini_set("soap.wsdl_cache_enabled", "0");  //设置soap缓存为0
try{
    $client = new SoapClient(null, array('location' => 'http://localhost/server.php','uri'=>'http://localhost')); //location写服务端的位置，uri必须和服务端的uri一致
    $client->__getFunctions(); //可以得到服务端所有可以调用的方法
    $result = $client->sayHello();
    echo "<pre>".print_r($result)."</pre>";
}catch(SoapFault $e){
	echo "<pre>".print_r($e)."</pre>";
    echo $e->getMessage();
}
```
**server.php服务端代码：**
```php
// 服务端接口类
class server{
	public $message;
    public function sayHello(){
    	return "hello";
    }
    public function add($a, $b){
    	return $a + $b;
    }
}

// soap服务端定义
try{
	ini_set('soap.wsdl_catch_enable','0'); //关闭wdsl缓存
	$server = new SoapServer(null, array('uri' => 'http://localhost'));
	$server->setClass('server');// 设置处理请求的class
	$server->handle();
}catch(SoapFault $e){
	echo $e->faultString; // 打印出错信息
}
```

##2. WSDL方式使用soap
**关于WSDL，维基百科搬运：**
>WSDL（Web服务描述语言，Web Services Description Language）是为描述Web服务发布的XML格式。W3C组织（World Wide Web Consortium）没有批准1.1版的WSDL，当前的WSDL版本是2.0，是W3C的推荐标准（recommendation）（一种官方标准），并将被W3C组织批准为正式标准。

>在诸多技术文献中通常将Web服务描述语言简写为WSDL，读音通常发为："wiz-dəl"。

>WSDL描述Web服务的公共接口。这是一个基于XML的关于如何与Web服务通讯和使用的服务描述；也就是描述与目录中列出的Web服务进行交互时需要绑定的协议和信息格式。通常采用抽象语言描述该服务支持的操作和信息，使用的时候再将实际的网络协议和信息格式绑定给该服务。

so，首先我们需要一个wsdl文件，其他流程和不用wsdl一样，只是发送的请求和返回的响应经过wsdl描述之后起到了跨平台跨语言的作用。

我们使用以下代码来简单的生成一个wsdl文件

**创建一个soapDiscovery.class.php**
```php
class SoapDiscovery {
    private $class_name = '';
    private $service_name = '';

    /**
     * SoapDiscovery::__construct() SoapDiscovery class Constructor.
     *
     * @param string $class_name
     * @param string $service_name
     **/
    public function __construct($class_name = '', $service_name = '') {
        $this->class_name = $class_name;
        $this->service_name = $service_name;
    }

    /**
     * SoapDiscovery::getWSDL() Returns the WSDL of a class if the class is instantiable.
     *
     * @return string
     **/
    public function getWSDL() {
        if (empty($this->service_name)) {
            throw new Exception('No service name.');
        }
        $headerWSDL = "<?xml version=\"1.0\" ?>\n";
        $headerWSDL.= "<definitions name=\"$this->service_name\" targetNamespace=\"urn:$this->service_name\"
          xmlns:wsdl=\"http://schemas.xmlsoap.org/wsdl/\"
          xmlns:soap=\"http://schemas.xmlsoap.org/wsdl/soap/\" 
          xmlns:tns=\"urn:$this->service_name\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" 
          xmlns:SOAP-ENC=\"http://schemas.xmlsoap.org/soap/encoding/\" 
          xmlns=\"http://schemas.xmlsoap.org/wsdl/\">\n";
        $headerWSDL.= "<types xmlns=\"http://schemas.xmlsoap.org/wsdl/\" />\n";

        if (empty($this->class_name)) {
            throw new Exception('No class name.');
        }

        $class = new ReflectionClass($this->class_name);

        if (!$class->isInstantiable()) {
            throw new Exception('Class is not instantiable.');
        }

        $methods = $class->getMethods();

        $portTypeWSDL = '<portType name="'.$this->service_name.'Port">';
        $bindingWSDL = '<binding name="'.$this->service_name.'Binding" type="tns:'.$this->service_name."Port\">\n
        <soap:binding style=\"rpc\" transport=\"http://schemas.xmlsoap.org/soap/http\" />\n";
        $serviceWSDL = '<service name="'.$this->service_name."\">
        \n<documentation />
        \n<port name=\"".$this->service_name.'Port" binding="tns:'.$this->service_name."Binding\">
        <soap:address location=\"http://".$_SERVER['SERVER_NAME'].':'.$_SERVER['SERVER_PORT'].$_SERVER['PHP_SELF']."\" />
        \n</port>\n</service>\n";
        $messageWSDL = '';
        foreach ($methods as $method) {
            if ($method->isPublic() && !$method->isConstructor()) {
                $portTypeWSDL.= '<operation name="'.$method->getName()."\">\n".
                '<input message="tns:'.$method->getName()."Request\" />\n<output message=\"tns:"
                .$method->getName()."Response\" />\n</operation>\n";
                $bindingWSDL.= '<operation name="'.$method->getName()."\">\n".'<soap:operation soapAction="urn:'.
                $this->service_name.'#'.$this->class_name.'#'.$method->getName()."\" />\n
                <input><soap:body use=\"encoded\" namespace=\"urn:$this->service_name\" encodingStyle=\"
                http://schemas.xmlsoap.org/soap/encoding/\" />\n</input>\n<output>\n<soap:body use=\"encoded\"
                namespace=\"urn:$this->service_name\" encodingStyle=\"http://schemas.xmlsoap.org/soap/encoding/\" />
                \n</output>\n</operation>\n";
                $messageWSDL.= '<message name="'.$method->getName()."Request\">\n";
                $parameters = $method->getParameters();
                foreach ($parameters as $parameter) {
                    $messageWSDL.= '<part name="'.$parameter->getName()."\" type=\"xsd:string\" />\n";
                }
                $messageWSDL.= "</message>\n";
                $messageWSDL.= '<message name="'.$method->getName()."Response\">\n";
                $messageWSDL.= '<part name="'.$method->getName()."\" type=\"xsd:string\" />\n";
                $messageWSDL.= "</message>\n";
            }
        }
        $portTypeWSDL.= "</portType>\n";
        $bindingWSDL.= "</binding>\n";
        return sprintf('%s%s%s%s%s%s', $headerWSDL, $portTypeWSDL, $bindingWSDL, $serviceWSDL, $messageWSDL, 
            '</definitions>');
    }

    /**
     * SoapDiscovery::getDiscovery() Returns discovery of WSDL.
     *
     * @return string
     **/
    public function getDiscovery() {
        return "<?xml version=\"1.0\" ?>\n<disco:discovery xmlns:disco=\"http://schemas.xmlsoap.org/disco/\" 
        xmlns:scl=\"http://schemas.xmlsoap.org/disco/scl/\">\n<scl:contractRef ref=\"http://".$_SERVER['SERVER_NAME'].
        ':'.$_SERVER['SERVER_PORT'].$_SERVER['PHP_SELF']."?wsdl\" />\n</disco:discovery>";
    }
}
```

**再创建一个createWSDL文件**
```php
include 'soapDiscovery.class.php';
include 'server.php';
try{
$disco = new SoapDiscovery('server','server'); //第一个参数是说明需要基于哪个类生成WSDL文件，第二个参数是指生成的wsdl的名称可以随意写
$result = $disco->getWSDL(); // 调用soapDiscovery中的getWSDL方法
// echo "<pre>".printf($result)."</pre>";
$crFile = fopen("newServer.wsdl", "w"); // 写入到文件
fwrite($crFile, $result);
}catch(Exception $e){
	echo $e->getMessage();
}
```

执行createWSDL文件中的代码后，会得到一个名为newServer.wsdl的文件，文件代码如下
```xml
<?xml version="1.0" ?>
<definitions name="service" targetNamespace="urn:service"
          xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
          xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" 
          xmlns:tns="urn:service" xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
          xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" 
          xmlns="http://schemas.xmlsoap.org/wsdl/">
<types xmlns="http://schemas.xmlsoap.org/wsdl/" />
<portType name="servicePort"><operation name="sayHello">
<input message="tns:sayHelloRequest" />
<output message="tns:sayHelloResponse" />
</operation>
<operation name="add">
<input message="tns:addRequest" />
<output message="tns:addResponse" />
</operation>
</portType>
<binding name="serviceBinding" type="tns:servicePort">

<soap:binding style="rpc" transport="http://schemas.xmlsoap.org/soap/http" />
<operation name="sayHello">
<soap:operation soapAction="urn:service#server#sayHello" />
<input>
<soap:body use="encoded" namespace="urn:service" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" />
</input>
<output>
<soap:body use="encoded" namespace="urn:service" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" />
</output>
</operation>
<operation name="add">
<soap:operation soapAction="urn:service#server#add" />
<input><soap:body use="encoded" namespace="urn:service" encodingStyle="
http://schemas.xmlsoap.org/soap/encoding/" />
</input>
<output>
<soap:body use="encoded" namespace="urn:service" encodingStyle="http://schemas.xmlsoap.org/soap/encoding/" />
</output>
</operation>
</binding>
<service name="service">
<documentation />
<port name="servicePort" binding="tns:serviceBinding"><soap:address location="http://localhost:80/createWSDL.php" />
</port>
</service>
<message name="sayHelloRequest">
</message>
<message name="sayHelloResponse">
<part name="sayHello" type="xsd:string" />
</message>
<message name="addRequest">
<part name="a" type="xsd:string" />
<part name="b" type="xsd:string" />
</message>
<message name="addResponse">
<part name="add" type="xsd:string" />
</message>
</definitions>
```

得到wsdl文件之后，将客户端和服务端的代码做一定改变使其通过wsdl文件描述
**client.php**
```php
ini_set('soap.wsdl_catch_enable','0'); //关闭wdsl缓存
try{
	$client = new SoapClient('http://localhost/server.php?wsdl');
	var_dump($client->sayHello());
}catch(soapFault $e){
	echo $e->getMessage();
}
```

**server.php**
```php
class service{
	public $name = "lybc";
	public $age = "19";
	public $abc = "lasjdfljaskljfljsdalkfjlasf";

	public function sayHello(){
		return $this;
	}

	public function add($a , $b)
	{
		$arr = array('name' => 'lybc','age' => '19','abc' => 'asfasfsadf');
		// return serialize($arr);
		return arr;
	}

	public function link($a , $b)
	{
		return $a . $b;
	}
}

ini_set('soap.wsdl_catch_enable','0'); //关闭wdsl缓存
$server = new SoapServer("newServer.wsdl",array('soap_version' => SOAP_1_2));
$server->setClass("service");
$server->handle();
```

完成好一切之后在访问localhost/client.php，会得到一个蛋疼的结果
>string(6) "Object"

因为我在service这个class中sayHello方法返回的是一个对象，而经过WSDL文件描述后返回的是一个string类型，因此会得到这样一个蛋疼的返回结果。那么如何解决

**修改wsdl文件**
在wsdl文件中找到下面这段代码
```xml
<message name="sayHelloResponse">
<part name="sayHello" type="xsd:string" />
</message>
```
将其中的`type="xsd:string"`改成`type="xsd:anyType"`即可成功的返回一个对象

为了不同平台的使用建议server端返回一段json类型的字符串让其他平台解析，这样可以保证效率
