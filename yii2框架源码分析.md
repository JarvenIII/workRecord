
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
resolve()使用getUrlManager()->parseRequest($this)获取全部参数,将第一个参数作为路由，正则匹配所有参数并放入[$route, $params] (路由，参数数组)数组中。
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

Yii 2.0将Web应用中对于Url的常用要求抽象到了urlManager中，作为web应用的核心组件。urlManager类直接继承自yii\base\Component：
```
parent::init();
if (!$this->enablePrettyUrl || empty($this->rules)) {
    return;
}
if (is_string($this->cache)) {
    $this->cache = Yii::$app->get($this->cache, false);
}
if ($this->cache instanceof Cache) {
    $cacheKey = __CLASS__;
    $hash = md5(json_encode($this->rules));
    if (($data = $this->cache->get($cacheKey)) !== false && isset($data[1]) && $data[1] === $hash) {
        $this->rules = $data[0];
    } else {
        $this->rules = $this->buildRules($this->rules);
        $this->cache->set($cacheKey, [$this->rules, $hash]);
    }
} else {
    $this->rules = $this->buildRules($this->rules);
}
```
首先是父类的实例化，很遗憾，我没有在Component和Object类里找到init()方法(只有一个抽象的)。$enablePrettyUrl表明是否使用Url美化，如果不开启使用原有格式，那么路由规则是无效的。初始化前， $this->cache 是缓存组件的ID，是个字符串，需要获取其实例。如果获取不到实例，说明应用不提供缓存功能，那么就将它设为false。如果引用到了缓存，那么就将路由规则缓存起来。以当前的类名作为缓存的key，将路由规则转换为hash值(令其不能被修改)。$data[0]用于储存路由规则，$data[1]存储hash值。如果已经有缓存，就使用$data中的值；如果还没有缓存，那么就使用buildRules()创建路由规则并缓存。如果不使用缓存，那么就直接使用buildRules()创建。
urlManager类里的许多方法都使用到buildRules()方法
```
$compiledRules = [];
$verbs = 'GET|HEAD|POST|PUT|PATCH|DELETE|OPTIONS';
foreach ($rules as $key => $rule) {
    if (is_string($rule)) {
        $rule = ['route' => $rule];
        if (preg_match("/^((?:($verbs),)*($verbs))\\s+(.*)$/", $key, $matches)) {
            $rule['verb'] = explode(',', $matches[1]);
            if (!in_array('GET', $rule['verb'])) {
                $rule['mode'] = UrlRule::PARSING_ONLY;
            }
            $key = $matches[4];
        }
        $rule['pattern'] = $key;
    }
    if (is_array($rule)) {
        $rule = Yii::createObject(array_merge($this->ruleConfig, $rule));
    }
    if (!$rule instanceof UrlRuleInterface) {
        throw new InvalidConfigException('URL rule class must implement UrlRuleInterface.');
    }
    $compiledRules[] = $rule;
}
return $compiledRules;
```
createUrl():
```
$params = (array) $params;
$anchor = isset($params['#']) ? '#' . $params['#'] : '';
unset($params['#'], $params[$this->routeParam]);

$route = trim($params[0], '/');
unset($params[0]);

$baseUrl = $this->showScriptName || !$this->enablePrettyUrl ? $this->getScriptUrl() : $this->getBaseUrl();

if ($this->enablePrettyUrl) {
    /* @var $rule UrlRule */
    foreach ($this->rules as $rule) {
        if (($url = $rule->createUrl($this, $route, $params)) !== false) {
            if (strpos($url, '://') !== false) {
                if ($baseUrl !== '' && ($pos = strpos($url, '/', 8)) !== false) {
                    return substr($url, 0, $pos) . $baseUrl . substr($url, $pos);
                } else {
                    return $url . $baseUrl . $anchor;
                }
            } else {
                return "$baseUrl/{$url}{$anchor}";
            }
        }
    }
    if ($this->suffix !== null) {
        $route .= $this->suffix;
    }
    if (!empty($params) && ($query = http_build_query($params)) !== '') {
        $route .= '?' . $query;
    }
    return "$baseUrl/{$route}{$anchor}";
} else {
    $url = "$baseUrl?{$this->routeParam}=" . urlencode($route);
    if (!empty($params) && ($query = http_build_query($params)) !== '') {
        $url .= '&' . $query;
    }
    return $url . $anchor;
}
```
parseRequest($request):
```
if ($this->enablePrettyUrl) {
    $pathInfo = $request->getPathInfo();
    /* @var $rule UrlRule */
    foreach ($this->rules as $rule) {
        if (($result = $rule->parseRequest($this, $request)) !== false) {
            return $result;
        }
    }

    if ($this->enableStrictParsing) {
        return false;
    }

    Yii::trace('No matching URL rules. Using default URL parsing logic.', __METHOD__);

    $suffix = (string) $this->suffix;
    if ($suffix !== '' && $pathInfo !== '') {
        $n = strlen($this->suffix);
        if (substr_compare($pathInfo, $this->suffix, -$n, $n) === 0) {
            $pathInfo = substr($pathInfo, 0, -$n);
            if ($pathInfo === '') {
                // suffix alone is not allowed
                return false;
            }
        } else {
            // suffix doesn't match
            return false;
        }
    }

    return [$pathInfo, []];
} else {
    Yii::trace('Pretty URL not enabled. Using default URL parsing logic.', __METHOD__);
    $route = $request->getQueryParam($this->routeParam, '');
    if (is_array($route)) {
        $route = '';
    }
    return [(string) $route, []];
}
```
  
