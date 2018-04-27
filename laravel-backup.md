# Laravel-backup # 
参考链接1：[https://docs.spatie.be/laravel-backup/v3](https://docs.spatie.be/laravel-backup/v3)

参考链接2：[https://github.com/spatie/laravel-backup](https://github.com/spatie/laravel-backup)

## 介绍 ##
这个Laravel 5软件包创建了应用程序的备份。备份是一个ZIP文件，其中包含您指定的目录中的所有文件以及数据库的转储。备份可以存储在 您在Laravel 5中配置的任何[文件系统](https://laravel.com/docs/5.4/filesystem)上。

对备份感到偏执？不要！您可以将应用程序一次备份到多个文件系统。

一旦安装，备份您的文件和数据库是非常容易的。只是发出这个命令：

    php artisan backup:run
## 安装和设置 ##
###基本安装###
    composer require spatie/laravel-backup

这里面要注意如果不是PHP 7 和 Laravel 5.3，那么

    composer require "spatie/laravel-backup:^3.0.0"
You'll need to register the serviceprovider:

    // config/app.php
    
    'providers' => [
    // ...
    Spatie\Backup\BackupServiceProvider::class,
    ];
To publish the config file to config/laravel-backup.php run:


    php artisan vendor:publish --provider="Spatie\Backup\BackupServiceProvider"
如果要备份到特定磁盘而不是所有磁盘，请运行：

    php artisan backup:run --only-to-disk=name-of-your-disk
如果要备份到自定义目录可以进行一下配置

//app/config/filesystems.php


    'disks'   => [

        // 添加laravel-backup备份文件目录
        'backup' => [
            'driver'     => 'local',
            'root'       => env('BACKUP_PATH'),
            'visibility' => 'public',
        ], 
    ],
//app/config/laravel-backup.php
   
     'destination' => [

       /*
        * The disk names on which the backups will be stored. 
        */
        'disks' => [
            'local',
			//'backup',
			//'admin',
        ],
    ],

如果只需要备份db运行：

    php artisan backup:run --only-db
如果只需要备份文件，并且要跳过转储数据库，请运行：

    php artisan backup:run --only-files
您可以运行以下操作来清理备份：

    php artisan backup:clean

默认情况下，包不会删除最新的备份，无论其大小或年龄。
确定应删除哪些备份

该部分配置决定了哪些备份应该被删除。

//config/laravel-backup.php

    'cleanup' => [
        /*
         * The strategy that will be used to cleanup old backups. The default strategy
         * will keep all backups for a certain amount of days. After that period only
         * a daily backup will be kept. After that period only weekly backups will
         * be kept and so on.
         *
         * No matter how you configure it the default strategy will never
         * deleted the newest backup.
         */
        'strategy' => \Spatie\Backup\Tasks\Cleanup\Strategies\DefaultStrategy::class,

        'defaultStrategy' => [

            /*
             * The number of days that all backups must be kept.
             */
            'keepAllBackupsForDays' => 7,

            /*
             * The number of days that all daily backups must be kept.
             */
            'keepDailyBackupsForDays' => 16,

            /*
             * The number of weeks of which one weekly backup must be kept.
             */
            'keepWeeklyBackupsForWeeks' => 8,

            /*
             * The number of months of which one monthly backup must be kept.
             */
            'keepMonthlyBackupsForMonths' => 4,

            /*
             * The number of years of which one yearly backup must be kept.
             */
            'keepYearlyBackupsForYears' => 2,

            /*
             * After cleaning up the backups remove the oldest backup until
             * this amount of megabytes has been reached.
             */
            'deleteOldestBackupsWhenUsingMoreMegabytesThan' => 5000,
        ],
    ],
此包提供了一种有意义的方法来确定应删除哪些旧备份。我们称此为DefaultStrategy。这是它的工作原理：

- 规则＃1：它不会删除最新的备份，无论其大小或年龄
- 规则＃2：它将保留指定的天数的所有备份 keepAllBackupsForDays
- 规则＃3：它将只保留对于keepDailyBackupsForDays规则＃2所覆盖之前的所有备份指定的天数的日常备份
- 规则＃4：它将只保留每个备份的规定keepMonthlyBackupsForMonths为所有备份的规定＃3规定＃
- 规则5：它只会保留对于keepYearlyBackupsForYears规则＃4所覆盖的所有备份所指定的年数的年度备份
- 规则6：它将开始删除旧的备份，直到使用的存储空间低于指定的数量deleteOldestBackupsWhenUsingMoreMegabytesThan。
- 当然，默认配置中使用的数字可以根据自己的需要进行调整。
## 制定自己的策略 ##

例如：

    use Spatie\Backup\BackupDestination\BackupCollection;
    use Spatie\Backup\Tasks\Cleanup\CleanupStrategy;
    class BackupClean extends CleanupStrategy{
    
    	public function deleteOldBackups(BackupCollection $backups){
    		// Retrieve an instance of `Spatie\Backup\BackupDestination\Backup`
    		$backup = $backups->oldest();
    
    		// Bye bye backup
    		$backup->delete();
    	}
    }

如果您的要求没有涵盖`DefaultStrategy`，您可以创建自己的自定义策略。

扩展抽象类`Spatie\Backup\Tasks\Cleanup\CleanupStrategy`。你只需要实现这个方法：

    use Spatie\Backup\BackupDestination\BackupCollection;
    
    public function deleteOldBackups(BackupCollection $backupCollection)
本`BackupCollection`类是从扩展`Illuminate\Support\Collection`，并包含`Spatie\Backup\BackupDestination\Backup`按年龄排序的对象。最新的备份是集合中的第一个。

使用该集合，您可以轻松地手动删除最旧的备份：

    // Retrieve an instance of `Spatie\Backup\BackupDestination\Backup`
    $backup = $backups->oldestBackup();
    
    // Bye bye backup
    $backup->delete();
不要忘记cleanup.strategy在laravel-backup配置文件的密钥中指定自定义策略的完整类名。

    'cleanup' => [
	    /*
	     * The strategy that will be used to cleanup old backups.
	     * The youngest backup will never be deleted.
	     */
	    // 'strategy' => \Spatie\Backup\Tasks\Cleanup\Strategies\DefaultStrategy::class,
	    'strategy' => \App\BackupClean::class,
    namespace App;

## 监控所有备份的运行状况 ##
    //app/Console/Kernel.php
    
    protected function schedule(Schedule $schedule)
    {
       $schedule->command('backup:monitor')->daily()->at('03:00');
    }
## 指定应监视哪些备份 ##

    //config/laravel-backup.php

    /*
     *  In this array you can specify which backups should be monitored.
     *  If a backup does not meet the specified requirements the
     *  UnHealthyBackupWasFound-event will be fired.
     */
    'monitorBackups' => [
        [
            'name' => env('APP_URL'),
            'disks' => ['local'],
            'newestBackupsShouldNotBeOlderThanDays' => 1,
            'storageUsedMayNotBeHigherThanMegabytes' => 5000,
        ],

        /*
        [
            'name' => 'name of the second app',
            'disks' => ['local', 's3'],
            'newestBackupsShouldNotBeOlderThanDays' => 1,
            'storageUsedMayNotBeHigherThanMegabytes' => 5000,
        ],
        */
    ],

您可以执行此命令查看所有受监视的目标文件系统的状态。

    php artisan backup:list
## 发送通知 ##

该软件包可以让您知道您的备份（不）OK。当某个事件发生时，它可以通过一个或多个渠道通知您。
//config/laravel-backup.php

     'notifications' => [

        'notifications' => [
            \Spatie\Backup\Notifications\Notifications\BackupHasFailed::class         => ['mail'],
            \Spatie\Backup\Notifications\Notifications\UnhealthyBackupWasFound::class => ['mail'],
            \Spatie\Backup\Notifications\Notifications\CleanupHasFailed::class        => ['mail'],
            \Spatie\Backup\Notifications\Notifications\BackupWasSuccessful::class     => ['mail'],
            \Spatie\Backup\Notifications\Notifications\HealthyBackupWasFound::class   => ['mail'],
            \Spatie\Backup\Notifications\Notifications\CleanupWasSuccessful::class    => ['mail'],
        ],

        /*
         * Here you can specify the notifiable to which the notifications should be sent. The default
         * notifiable will use the variables specified in this config file.
         */
        'notifiable' => \Spatie\Backup\Notifications\Notifiable::class,

        'mail' => [
            'to' => '13967741060@163.com',
        ],

        'slack' => [
            'webhook_url' => '',

            /*
             * If this is set to null the default channel of the webhook will be used.
             */
            'channel' => null,
        ],
    ],

邮箱通知

.env 配置邮箱信息

    MAIL_DRIVER=smtp
    MAIL_HOST=smtp.163.com
    MAIL_PORT=465
    MAIL_USERNAME=13967741060@163.com
    MAIL_PASSWORD=Wcw846758255
    MAIL_ENCRYPTION=ssl
    MAIL_FROM_ADDRESS=13967741060@163.com
    MAIL_FROM_NAME=

## 创建自定义发件人 ##
默认情况下备份包可以通过以下方式通知您：

- 在日志中写东西
- 通过发送邮件
- 如果maknz/slack已安装，请在[Slack](https://slack.com/)上发布消息

如果您想通过其他频道收到通知，您可以创建自己的发件人。有效的发件人是实现接口的任何对象Spatie\Backup\Notifications\SendsNotifications。

    namespace Spatie\Backup\Notifications;
    
    interface SendsNotifications
    {
    /**
     * @param string $type
     *
     * @return \Spatie\Backup\Notifications\SendsNotifications
     */
    public function setType($type);

    /**
     * @param string $subject
     *
     * @return \Spatie\Backup\Notifications\SendsNotifications
     */
    public function setSubject($subject);

    /**
     * @param string $message
     *
     * @return \Spatie\Backup\Notifications\SendsNotifications
     */
    public function setMessage($message);

    public function send();

    }
如果你选择扩展Spatie\Backup\Notifications\BaseSender你只需要实现send- 功能。

您可以通过monitor.events在laravel-backup配置文件的一个键中指定完整的类名来使用您的自定义发件人。

    // ...
    'whenBackupHasFailed' => ['log', 'mail', App\Backup\MyCustomSender::class],
    // ...
## 添加额外的文件到zip ##
该包附带BackupManifestWasCreated事件，使您能够将其他文件添加到备份zip文件。

当备份进程启动时，软件包将创建所有选择备份的文件的清单。一旦清单创建完毕，就会创建一个包含清单中所有文件的zip文件。zip文件将被复制到您配置的备份目标。

但是，如果您有需要向特定备份添加附加文件的情​​况，则可以在创建清单和压缩文件的结束之间进行此操作。

在清单创建之后，在创建zip文件之前，Spatie\Backup\Events\BackupManifestWasCreated触发事件。这是什么样子：

    namespace Spatie\Backup\Events;
    
    use Spatie\Backup\Tasks\Backup\Manifest;
    
    class BackupManifestWasCreated
    {
	    /** @var \Spatie\Backup\Tasks\Backup\Manifest */
	    public $manifest;
	    
	    public function __construct(Manifest $manifest)
	    {
	    	$this->manifest = $manifest;
	    }
    }

您可以使用该事件将额外的文件添加到清单中，如下面的示例中，额外的文件作为数组传递给addFiles（）方法。

    use Spatie\Backup\Events\BackupManifestWasCreated;
    
    Event::listen(BackupManifestWasCreated::class, function (BackupManifestWasCreated $event) {
       $event->manifest->addFiles([$path1, $path2, ...]);
    });