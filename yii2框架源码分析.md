
##从源码入手了解Yii2的处理过程

### yii2如何处理request
入口脚本是应用启动流程的第一环，用户请求通过入口脚本实例化应用，并将请求转发给应用。入口脚本定义全局常量，注册composer，包含yii类文件，最重要的是加载应用配置并将它传给一个新创建的Application运行。
```
$config = require(__DIR__ . '/../config/web.php');
(new yii\web\Application($config))->run();
```
$config即配置文件，而Application是yii\web\Application的对象,这里拥有两个重要方法bootstrap()和handleRequest()。那么是如何加载配置文件的呢？在yii\web\Application里没有相关方法，根据继承关系找到它的父类yii\base\Application里的构造函数__construct里面定义了相关内容：
```
Yii::$app = $this;
$this->setInstance($this);
$this->state = self::STATE_BEGIN;
$this->preInit($config); //加载配置文件，加载默认组件
$this->registerErrorHandler($config); //加载异常处理
Component::__construct($config); //将配置文件的信息赋值给Object，
```
Yii::$app代表应用实例，是一个全局可访问的单例，同时也是一个服务定位器，能提供request，response, db等特定功能的组件，这里将yii\base\Application以及它的父类\yii\base\Module \yii\di\ServiceLocator \yii\base\Component \yii\base\Object的所有public方法交给了Yii::$app。
```
Component::__construct($config);
```
使用了它的父类Object的构造函数__construct：
```
if (!empty($config)) {
    Yii::configure($this, $config);
}
$this->init();
```
Yii::configure($this, $config)将配置文件里包含的所有配置信息都赋值给Object类，而Object类是很多类的基类，这样大多数的类都可以通过Yii::$app来直接访问配置文件了。然后执行初始化，object类的init()是空的，所以按顺序找到它的子类Component->ServiceLocator->Module,在Module里找到了init()方法：
```
if ($this->controllerNamespace === null) {
    $class = get_class($this);
    if (($pos = strrpos($class, '\\')) !== false) {
        $this->controllerNamespace = substr($class, 0, $pos) . '\\controllers';
    }
}
```
这里我的理解是控制器class的位置找到控制器的命名空间。到这里一个应用主体加载完配置文件的配置信息，使用run()方法开始执行。

run() 位于yii\base\Application.php中，它使用handleRequest()获取用户请求并进行处理，主要是加载控制器。
```
$response = $this->handleRequest($this->getRequest());
```
在上面提到yii\base\Application.php有两个重要方法bootstrap()和handleRequest()。bootstrap()方法首先使用$this->getRequest()获取request然后就是使用Yii::setAlias注册别名.并调用父类yii\base\Application的bootstrap()方法，引入extensions.php里包含的扩展。
handleRequest($request)方法首先初始化调用yii\web\Request中resolve():
```
$result = Yii::$app->getUrlManager()->parseRequest($this);
if ($result !== false) {
    list ($route, $params) = $result;
    $_GET = array_merge($_GET, $params);

    return [$route, $_GET];
} else {
    throw new NotFoundHttpException(Yii::t('yii', 'Page not found.'));
}
```
resolve()使用gettUrlManager()->parseRequest($this)获取全部参数,将第一个参数作为路由，正则匹配所有参数并放入[$route, $params](路由，参数数组)数组中。
接下来handleRequest($request)调用runAction()来执行这个数组并检测返回值是不是Response对象，如果是就将结果返回给用户，否则就使用getResponse()获取response，将它的data作为结果返回给用户。
runAction()方法位于yii\base\Application的父类yii\base\Module中，它需要接受2个参数，第一个是路由，第二个是有关此route的参数数组。
```
$parts = $this->createController($route); //通过路由创建控制器
if (is_array($parts)) {
    /* @var $controller Controller */
    list($controller, $actionID) = $parts; //获得controller以及其actionID
    $oldController = Yii::$app->controller;
    Yii::$app->controller = $controller;
    $result = $controller->runAction($actionID, $params); //运行控制器加载操作
    Yii::$app->controller = $oldController; //释放
    return $result;
} else {
    $id = $this->getUniqueId();
    throw new InvalidRouteException('Unable to resolve the request "' . ($id === '' ? $route : $id . '/' . $route) . '".');
}
```
在创建控制器之后调用了yii\base\Controller的runAction()方法来加载action，它同样接受两个参数：actionID和参数数组。关键性代码有两句：
```
$action = $this->createAction($id); //创建action
$result = $action->runWithParams($params); //执行控制器里的actio
```
runWithParams($params)位于yii\base\InlineAction。
```
$args = $this->controller->bindActionParams($this, $params);
return call_user_func_array([$this->controller, $this->actionMethod], $args);
```
将action的参数数组绑定并赋值给控制器，最后用控制器去执行带参数的action。

getResponse() 位于yii\web\Application中，调用yii\web\Component的get()方法以调用response组件。
```
return $this->get('response');
```
get() 位于yii\web\Component.php中
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

### yii2如何管理Url

Yii 2 将Web应用中对于Url的常用要求抽象到了urlManager中，作为web应用的核心组件。

  
