链接地址: http://www.yiichina.com/doc/guide/2.0

## Yii 2.0 中文权威指南

### 应用结构

#### 应用主体(Application)
- 使用入口脚本创建应用主体(加载配置文件)
- 应用主体是服务定位器，它部署一组提供各种不同功能的应用组件来处理请求。
- 应用主体属性(在应用主体配置文件中):
  - yii\base\Application::id 属性用来区分其他应用的唯一标识ID。
  - yii\base\Application::basePath 属性经常用于派生一些其他重要路径, 系统预定义 @app 代表这个路径。 派生路径可以通过这个别名组成（如@app/runtime代表runtime的路径）
  - yii\base\Application::bootstrap 指定启动阶段bootstrap()需要运行的组件
  ```
  [
    'bootstrap' => [
        // 应用组件ID或模块ID
        'demo',
        // 类名
        'app\components\Profiler',
        // 配置数组
        [
            'class' => 'app\components\Profiler',
            'level' => 3,
        ],
        // 无名称函数
        function () {
            return new app\components\Profiler();
        }
    ],
  ]
  ```
  - yii\base\Application::components 最重要的属性,可以通过表达式 \Yii::$app->ComponentID 全局访问。
  ```
  [
    'components' => [
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'user' => [
            'identityClass' => 'app\models\User',
            'enableAutoLogin' => true,
        ],
    ],
  ]
  ```
  - 应用主体生命周期 http://www.yiichina.com/doc/guide/2.0/structure-applications 最后

#### 应用组件(Application components)
- 首次访问时被实例化,如果想要无论是否被访问都实例化,加入到yii\base\Application::bootstrap中,即引导启动组件
- 核心应用组件: http://www.yiichina.com/doc/guide/2.0/structure-application-components 最后

#### 控制器(Controller)
- 是继承yii\base\Controller类的对象，负责处理请求和生成响应。具体来说,控制器从应用主体接管控制后会分析请求
数据并传送到模型，传送模型结果到视图，最后生成输出响应信息。
- 在yii\web\Application网页应用中，控制器应继承yii\web\Controller 或它的子类。同理在yii\console\Application控制台应用中，控制器继承yii\console\Controller 或它的子类
- 命名规范：由操作组成,命名为actionId。如actionGetName，访问方式为/get-name，与Controller命名访问规则相同
- 路由：
  - 用户通过路由寻找操作 ControllerID/ActionID,如果属于模块，则需要添加ModuleID


#### 从源码入手了解Yii2的处理过程
- 首先，从入口脚本入手。入口脚本是应用启动流程的第一环，用户请求通过入口脚本实例化应用，并将请求转发给应用。
- 入口脚本定义全局常量，注册composer，包含yii类文件，最重要的是加载应用配置并将它传给一个新创建的Application运行。
```
$config = require(__DIR__ . '/../config/web.php');
(new yii\web\Application($config))->run();
```

- $config即配置文件，而Application继承自yii\web\Application.php,这里拥有两个重要方法bootstrap()和handleRequest().
- bootstrap()方法 首先使用$this->getRequest()获取request，$this定义在yii\web\Application.php的父类yii\base\Application.php的构造函数中Yii::$app = $this; Yii::$app代表应用实例，是一个全局可访问的单例，同时也是一个服务定位器，能提供request，response, db等特定功能的组件。然后就是使用Yii::setAlias注册别名.
- run() 位于yii\base\Application.php中，它使用handleRequest()获取用户请求并进行处理
```
$response = $this->handleRequest($this->getRequest());
```
  -用于获取request组件，它代表用户需求并承载用户所有的输入信息。包括getParams(),setParams($Params)和resolve()方法
- handleRequest($request)方法 首先初始化调用resolve()，resolve()使用getParams()获取全部参数并将第一个参数作为路由，正则匹配所有参数并放入[$route, $params]中，将
 
  
  
