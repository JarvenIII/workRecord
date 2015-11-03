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

#### 模型(Model)

属性(attributes):yii\base\Model::attributes() 指定模型所拥有的属性。在表里你所有需要使用的字段，都要声明在attributes()里。感谢yii\base\Model支持 ArrayAccess 数组访问 和 ArrayIterator 数组迭代器，共有3种方法访问属性:
```
$model->name = 'example'; //1
$model['name'] = 'example'; //2
//迭代器遍历模型
foreach ($model as $name => $value) {
    echo "$name: $value\n";
}
```

场景(scenarios): 模型可在多个场景下使用。在不同的场景下，模型可能会使用不同的业务规则和逻辑，例如 email 属性在注册时强制要求有，但在登陆时不需要。
```
public function scenarios()
{
    return [
        'login' => ['username', 'password'],
        'register' => ['username', 'email', 'password'],
    ];
}
```
scenarios() 方法默认实现会返回所有yii\base\Model::rules()方法申明的验证规则中的场景，主要在验证 和 属性块赋值 中使用。

验证规则(rules): 模型接收的用户数据应满足的规则，如数据类型、是否为空等。可调用 yii\base\Model::validate() 来验证接收到的数据， 该方法使用yii\base\Model::rules()申明的验证规则来验证每个相关属性。
可以使用”on“指定规则使用的场景：
```
public function rules()
{
  return [
      // 在"register" 场景下 username, email 和 password 必须有值
      [['username', 'email', 'password'], 'required', 'on' => 'register'],

      // 在 "login" 场景下 username 和 password 必须有值
      [['username', 'password'], 'required', 'on' => 'login'],
  ];
} 
```
由于默认yii\base\Model::scenarios()的实现会返回yii\base\Model::rules()所有属性和数据， 如果不覆盖这个方法，表示所有只要出现在活动验证规则中的属性都是安全的。

安全属性(safeAttributes)：不会因用户修改造成安全隐患的属性。例如：一个用户可能具有3个属性\[username, password, role\], 你不会希望用户通过非法发送请求就能够更改role属性，所以username和password是安全属性，但是role不是。

更灵活和强大的将模型转换为数组的方式是使用 yii\base\Model::toArray() 方法

#### 视图(view)

在视图中，可访问 $this 指向 yii\web\View 来管理和渲染这个视图文件。在controller使用render方法传入到view中的参数可以用<?= $name?>来使用。

组织视图：
- 控制器渲染的视图文件默认放在 @app/views/ControllerID 目录下， 其中 ControllerID 对应 控制器 ID, 例如控制器类为 PostController，视图文件目录应为 @app/views/post， 控制器类 PostCommentController对应的目录为 @app/views/post-comment， 如果是模块中的控制器，目录应为 yii\base\Module::basePath 模块目录下的 views/ControllerID 目录。

视图中渲染：
- 视图中的如下代码会渲染该视图所在目录下的 _overview.php 视图文件， 记住视图中 $this 对应 yii\base\View 组件:
```
<?= $this->render('_overview') ?> //在视图中渲染
echo \Yii::$app->view->renderFile('@app/views/site/license.php'); //在任何地方渲染
```

