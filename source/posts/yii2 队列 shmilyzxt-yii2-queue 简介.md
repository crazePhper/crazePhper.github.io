---
layout: post
title: YII2 队列 shmilyzxt-yii2-queue 简介
date: '2016-03-27 19:53'
comments: true
customizeurl: YII2-shmilyzxt-yii2-queue
tags:
  - php
  - YII2
abbrlink: 49049
---

- 在yii2论坛中看到一个关于队列的帖子,感觉不错.http://www.yiichina.com/extension/1084 
(注:SendMail 错写为 SendMial,粘贴时要注意了.)

- 在使用的过程中觉得挺好的,建议看一下关于队列的原理.

- `shmilyzxt/yii2-queue` 简单解释:
1. 我用的yii2高级版,我们从配置开始看代码,这里我用的是mysql队列,首先配置文件,我把`queue`配置项写在根目录`common\config\main-local.php`下的 `components`数组下,更改一下数据库配置.复制`composer`安装后复制
`vendor\shmilyzxt\yii2-queue\jobs\jobs.sql`
`vendor\shmilyzxt\yii2-queue\failed\failed.sql`
2个sql文件到数据库中建立队列数据表和执行任务失败时的数据表.

1. 推送任务开始语法:`\Yii::$app->queue->pushOn(new SendMial(),['email'=>'49783121@qq.com','title'=>'test','content'=>'email test'],'email');` 我们到`vendor\shmilyzxt\queue\queues\DatabaseQueue.php`去看看代码,`pushOn()`方法写在了`DatabaseQueue`类的父类`vendor\shmilyzxt\queue\base\Queue.php`中:

```php
//入队列
public function pushOn($job, $data = '', $queue = null)
    {
		//canPush 检查队列是否已达最大任务量
        if ($this->canPush()) {  
			//beforePush 入队列前的事件
            $this->trigger(self::EVENT_BEFORE_PUSH); 
			//入队列
            $ret = $this->push($job, $data, $queue);
			//afterPush 入队列后的事件
            $this->trigger(self::EVENT_AFTER_PUSH);
            return $ret;
        } else {
            throw new \Exception("max jobs number exceed! the max jobs number is {$this->maxJob}");
        }
    }
```
注释:这里最好去看看yii2 event事件类,[http://www.digpage.com/event.html](http://www.digpage.com/event.html "http://www.digpage.com/event.html")
<!-- more -->
关于入队列: `$this->push($job, $data, $queue);`,这里在配合`queue`类文件查看,相关函数跳转,处理一下数据记录到数据库中.(函数走向:`getQueue()-->createPayload()-->pushToDatabase()`),`pushOn()`最终返回数据插入数据库的结果,成功`$ret`是1.

3.后台运行命令处理队列,例:`php ./yii worker/listen default 10 128 3 0` 其中`default`是队列的名称,上面推送了一个`email`队列 应该改为`email`.
启动命令后,我们来看代码:首先执行:`WorkerController`控制器 `actionListen`方法,我们跟着代码进入到 `vendor\shmilyzxt\queue\Worker.php -- listen`方法中,这里其实就是一直在循环,执行操作队列的任务:
```php
/**
  * 启用一个队列后台监听任务
  * @param Queue $queue
  * @param string $queueName 监听队列的名称(在pushon的时候把任务推送到哪个队列，则需要监听相应的队列才能获取任务)
  * @param int $attempt 队列任务失败尝试次数，0为不限制
  * @param int $memory 允许使用的最大内存
  * @param int $sleep 每次检测的时间间隔
  */
    public static function listen(Queue $queue, $queueName = 'default', $attempt = 10, $memory = 512, $sleep = 3, $delay = 0){
        while (true){
            try{
				//DatabaseQueue从数据库队列取出一个可用任务(实例),并且更新任务
                $job = $queue->pop($queueName);
            }catch (\Exception $e){
                throw $e;
                continue;
            }

            if($job instanceof Job){
				//判断执行错误的次数是否大于传入的执行次数
                if($attempt > 0 && $job->getAttempts() > $attempt){
                    $job->failed();
                }else{
                    try{
                        //throw new \Exception("test failed");
                        $job->execute();

                    }catch (\Exception $e){
						//执行失败,判断是否被删除,重新入队
                        if (! $job->isDeleted()) {
                            $job->release($delay);
                        }
                    }
                }
            }else{
                self::sleep($sleep);
            }


            if (self::memoryExceeded($memory)) {
                self::stop();
            }
        }
    }
```
注释:在`$queue->pop($queueName);`是`vendor\shmilyzxt\queue\queues\DatabaseQueue.php`方法内使用事务执行SQL,并且创建`vendor\shmilyzxt\queue\jobs\DatabaseJob.php`的实例
```php
	//取出一个任务
    public function pop($queue = null)
    {
        $queue = $this->getQueue($queue);

        if (!is_null($this->expire)) {
            //$this->releaseJobsThatHaveBeenReservedTooLong($queue);
        }

        $tran = $this->connector->beginTransaction();
		//判断是否有一个可用的任务需要执行
        if ($job = $this->getNextAvailableJob($queue)) {
            $this->markJobAsReserved($job->id);
            $tran->commit();

            $config = array_merge($this->jobEvent, [
                'class' => 'shmilyzxt\queue\jobs\DatabaseJob',
                'queue' => $queue,
                'job' => $job,
                'queueInstance' => $this,
            ]);

            return \Yii::createObject($config);

        }
        $tran->commit();
        return false;
    }
```
至于:`$job->execute();`是`DatabaseJob`继承父类`Job`执行的,顺着代码找下去是`yii\base\Component trigger`执行的事件,
```php
/**
 * 执行任务
 */
public function execute()
{
   $this->trigger(self::EVENT_BEFORE_EXECUTE, new JobEvent(["job" => $this, 'payload' => $this->getPayload()]));//beforeExecute 执行任务之前的一个事件 在JobEvent中并没有什么可执行的代码
   $this->resolveAndFire();//真正执行的任务的方法
}
```
```php
 /**
   * 真正任务执行方法（调用hander的handle方法）
   * @param  array $payload
   * @return void
 */
    protected function resolveAndFire()
    {
        $payload = $this->getPayload();
        $payload = unserialize($payload); //反序列化数据
        $type = $payload['type'];
        $class = $payload['job'];

        if ($type == 'closure' && ($closure = (new Serializer())->unserialize($class[1])) instanceof \Closure) {
            $this->handler = $this->getHander($class[0]);
            $this->handler->closure = $closure;
            $this->handler->handle($this, $payload['data']);
        } else if ($type == 'classMethod') {
            $payload['job'][0]->$payload['job'][1]($this, $payload['data']);
        } else if ($type == 'staticMethod') {
            $payload['job'][0]::$payload['job'][1]($this, $payload['data']);
        } else {//执行的`SendMail`类的`handle($job,$data)`方法
            $this->handler = $this->getHander($class);
            $this->handler->handle($this, $payload['data']);
        }

        //执行完任务后删除
        if (!$this->isDeletedOrReleased()) {
            $this->delete();
        }
    }
```
最后到了执行的`SendMail`类的`handle($job,$data)`,在这里就是推送到队列的对象和数据,接着就是我们的处理逻辑了.
```php
public function handle($job,$data)
    {
        if($job->getAttempts() > 3){
            $this->failed($job);
        }

        $payload = $job->getPayload();
        echo '<pre>';print_r($payload);
        //$payload即任务的数据，你拿到任务数据后就可以执行发邮件了
        //TODO 发邮件
    }
```
