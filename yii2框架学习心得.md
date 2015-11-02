链接地址: http://www.yiichina.com/doc/guide/2.0

## Yii 2.0 中文权威指南

### 应用结构

#### 应用主体(Application)
使用入口脚本创建应用主体(加载配置文件)
应用主体是服务定位器，它部署一组提供各种不同功能的应用组件来处理请求。
应用主体属性(在应用主体配置文件中):
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
首次访问时被实例化,如果想要无论是否被访问都实例化,加入到yii\base\Application::bootstrap中,即引导启动组件
核心应用组件: http://www.yiichina.com/doc/guide/2.0/structure-application-components 最后

#### 控制器(Controller)
控制器是继承yii\base\Controller类的对象，负责处理请求和生成响应。具体来说,控制器从应用主体接管控制后会分析请求
数据并传送到模型，传送模型结果到视图，最后生成输出响应信息。
在yii\web\Application网页应用中，控制器应继承yii\web\Controller 或它的子类。同理在yii\console\Application控制台应用中，控制器继承yii\console\Controller 或它的子类
命名规范：
- 内联操作：由操作组成,命名为actionId。如actionGetName，访问方式为/get-name，与Controller命名访问规则相同
- 独立操作：继承yii\base\Action或它的子类来定义。例如Yii发布的yii\web\ViewAction和yii\web\ErrorAction都是独立操作。要使用独立操作，需要通过控制器中覆盖yii\base\Controller::actions()方法在action map中申明：
```
public function actions()
{
  return [
      // 用类来申明"error" 操作
      'error' => 'yii\web\ErrorAction',

      // 用配置数组申明 "view" 操作
      'view' => [
          'class' => 'yii\web\ViewAction',
          'viewPrefix' => '',
      ],
  ];
}
```
路由：
- 用户通过路由寻找操作 ControllerID/ActionID,如果属于模块，则需要添加ModuleID
通常控制器的名称应该与用途相对应，如控制文章相关资源的控制器应该使用article，但是也可以通过配置yii\base\Application::controllerMap来强制对应(在相关项目中没有找到。。。)
```
[
  'controllerMap' => [
      // 用类名申明 "account" 控制器
      'account' => 'app\controllers\UserController',

      // 用配置数组申明 "article" 控制器
      'article' => [
          'class' => 'app\controllers\PostController',
          'enableCsrfValidation' => false,
      ],
  ],
]
```
