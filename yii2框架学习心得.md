
#### 从源码入手了解Yii2的处理过程
- 1. 如何处理request。入口脚本是应用启动流程的第一环，用户请求通过入口脚本实例化应用，并将请求转发给应用。
  - 入口脚本定义全局常量，注册composer，包含yii类文件，最重要的是加载应用配置并将它传给一个新创建的Application运行。
```
$config = require(__DIR__ . '/../config/web.php');
(new yii\web\Application($config))->run();
```

  - $config即配置文件，而Application继承自yii\web\Application.php,这里拥有两个重要方法bootstrap()和handleRequest().run() 位于yii\base\Application.php中，它使用handleRequest()获取用户请求并进行处理
```
$response = $this->handleRequest($this->getRequest());
```
  - 用于获取request组件，它代表用户需求并承载用户所有的输入信息。包括getParams(),setParams($Params)和resolve()方法
  - bootstrap()方法 首先使用$this->getRequest()获取request，$this定义在yii\web\Application.php的父类yii\base\Application.php的构造函数中Yii::$app = $this; Yii::$app代表应用实例，是一个全局可访问的单例，同时也是一个服务定位器，能提供request，response, db等特定功能的组件，这里将yii\base\Application以及它的父类\yii\base\Module \yii\di\ServiceLocator \yii\base\Component \yii\base\Object的所有public方法交给了Yii::$app。然后就是使用Yii::setAlias注册别名.并调用父类yii\base\Application的bootstrap()方法，引入extensions.php里包含的扩展。
  - handleRequest($request)方法 首先初始化调用request组件中resolve()，resolve()使用getParams()获取全部参数并将第一个参数作为路由，正则匹配所有参数并放入[$route, $params](路由，参数数组)数组中，调用runAction()来执行这个数组并检测返回值是不是Response对象，如果是就将结果返回给用户，否则就使用getResponse()获取response，将它的data作为结果返回给用户。
  - runAction()方法 位于yii\base\Controller中，它需要接受2个参数，第一个是actionID，第二个是有关此action的参数数组。
  - getResponse() 位于yii\web\Application中，调用yii\web\Component的get()方法以调用response组件
  - get() 位于yii\web\Component.php中
```
$getter = 'get' . $name;
if (method_exists($this, $getter)) {
    // read property, e.g. getName()
    return $this->$getter();
} else {
    // behavior property
    $this->ensureBehaviors();
    foreach ($this->_behaviors as $behavior) {
        if ($behavior->canGetProperty($name)) {
            return $behavior->$name;
        }
    }
}
if (method_exists($this, 'set' . $name)) {
    throw new InvalidCallException('Getting write-only property: ' . get_class($this) . '::' . $name);
} else {
    throw new UnknownPropertyException('Getting unknown property: ' . get_class($this) . '::' . $name);
}
        
``` 
- 2. 如何管理Url
  
  
