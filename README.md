Yii2 Queue Chain
================

Цепочки заданий на базе `yii\queue`.

**Внимание:** 'это экспериментальное расширение, и оно в процессе стабилизации АПИ. По итогам код
перейдет в `yiisoft/yii2-queue`.

Настройка
---------

 ```php
 <?php
 return [
    'components' => [
        'queue' => [
            'class' => \yii\queue\redis\Queue::class,
            'as chain' => [
                'class' => \yii\queue\chain\ChainBehavior::class,
                'storage' => \yii\queue\chain\RedisStorage::class,
            ],
        ],
    ],
 ];
 ```

Использование
-------------

Пример группового задания:

```php
class MyGroupJob extends BaseObject implements JobInterface, ChainJobInterface
{
    /**
     * @return string уникальный идентификатор группы заданий.
     * @inheritdoc ChainJobInterface
     */
    public function getGroupId()
    {
        return 'group-123';
    }
    
    /**
     * @inheritdoc JobInterface
     */
    public function execute($queue)
    {
        //...
        return 12345;
    }
    
    /**
     * Финализация запустится один раз после того, как выполнятся все задания группы.
     * @inheritdoc ChainJobInterface
     */
    public function finalizeGroup($queue, array $results)
    {
        $queue->push(new MyFinalizeJob(['results' => $results]));
    }
}
```

Узнать прогресс выполнения группы заданий:

```php
list($done, $pushed) = Yii::$app->queue->getGroupProgress('group-123');
$percent = $done * 100 / $pushed;
```
