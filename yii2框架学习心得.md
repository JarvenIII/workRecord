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

视图名：
- 视图名以双斜杠 // 开头，对应的视图文件路径为 @app/views/ViewName，在 yii\base\Application::viewPath 路径下找，例如 //site/about 对应到 @app/views/site/about.php。
- 视图名以单斜杠/开始，视图文件路径以当前使用模块 的yii\base\Module::viewPath开始， 如果不存在模块，使用@app/views/ViewName开始，例如，如果当前模块为user， /user/create 对应成 @app/modules/user/views/user/create.php, 如果不在模块中，/user/create对应@app/views/user/create.php。
- 在视图中渲染时，_overview.php应该和渲染它的视图文件位于同一个文件夹内

布局(layout)：
- 特殊视图，代表多个视图的公共部分；像普通视图一样创建，默认存入views/layouts中。 模块中使用的布局应存储在yii\base\Module::basePath模块目录下的views/layouts路径下。
```
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $content string 字符串 */
?>
<?php $this->beginPage() ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <?= Html::csrfMetaTags() ?>
    <title><?= Html::encode($this->title) ?></title>
    <?php $this->head() ?>
</head>
<body>
<?php $this->beginBody() ?>
    <header>My Company</header>
    <?= $content ?>
    <footer>&copy; 2014 by My Company</footer>
<?php $this->endBody() ?>
</body>
</html>
<?php $this->endPage() ?>
```
 输入代码 public $layout = 'post' 来使用布局。
 
在view中设置页面标题：
- 在试图中声明： 
```
<?php
$this->title = 'My page title';
?>
```
在<head>标签中要保证：
```
<title><?= Html::encode($this->title) ?></title>
```
注册Meta元标签和链接标签：
```
<?php
$this->registerMetaTag(['name' => 'keywords', 'content' => 'yii, framework, php']);
$this->registerLinkTag([
    'title' => 'Live News for Yii',
    'rel' => 'alternate',
    'type' => 'application/rss+xml',
    'href' => 'http://www.yiiframework.com/rss.xml/',
]);
?>
```

#### 模块(module)

模块被组织成一个称为yii\base\Module::basePath的目录，此目录下都有一个继承yii\base\Module的模块类并能自动加载。
要在应用中使用模块，只需要将模块加入到应用主体配置的yii\base\Application::modules属性的列表中。

#### 过滤器(filter)

过滤器可包含 预过滤（过滤逻辑在动作之前） 或 后过滤（过滤逻辑在动作之后），也可同时包含两者。

使用过滤器：
本质上是一种特殊的行为(behaviors)，可以在controller中覆盖behaviors()来申明过滤器，默认应用到所有actions。
```
public function behaviors()
{
    return [
        [
            'class' => 'yii\filters\HttpCache',
            'only' => ['index', 'view'],
            'lastModified' => function ($action, $params) {
                $q = new \yii\db\Query();
                return $q->from('user')->max('updated_at');
            },
        ],
    ];
} //使用except声明不应用到哪些actions
```

可在 模块或应用主体 中申明过滤器，但是应该用路由代替actionID。

预过滤从底层到顶层，后过滤反之。

核心过滤器：
yii\filters\AccessControl 访问控制 => 授权
```
use yii\filters\AccessControl;

public function behaviors()
{
    return [
        'access' => [
            'class' => AccessControl::className(),
            'only' => ['create', 'update'],
            'rules' => [
                // 允许认证用户
                [
                    'allow' => true,
                    'roles' => ['@'],
                ],
                // 默认禁止其他用户
            ],
        ],
    ];
}
```
使用以上代码可以在controller中使用ACF，@代表认证用户，？代表访问用户，无论是否认证。
可以加入denyCallback属性来制定无授权用户访问操作的PHP回调函数。
