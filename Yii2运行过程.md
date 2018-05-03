1. 入口文件 ，`web` 目录下 `index.php`文件,加载已经合并的配置文件执行 `(new yii\web\Application($config))->run()` .

2. `yii\web\Application` 类看一下 构造函数`__construct` :

   ```php
   /**
        * Constructor.
        * @param array $config name-value pairs that will be used to initialize the object properties.
        * Note that the configuration must contain both [[id]] and [[basePath]].
        * @throws InvalidConfigException if either [[id]] or [[basePath]] configuration is missing.
        */
       public function __construct($config = [])
       {
           Yii::$app = $this;
           static::setInstance($this);

           $this->state = self::STATE_BEGIN;

           $this->preInit($config);

           $this->registerErrorHandler($config);

           Component::__construct($config);
       }
   ```

   第一行把当前类的赋值给了 `Yii::$app` `Yii`类文件仅继承了`BaseYii` 文件.第二行是