##简介
`Revisionable` 是一个Laravel包，允许您保留您的模型的修订历史，而不用考虑。
对于一些背景和信息，[请参阅这篇文章](http://www.chrisduell.com/blog/development/keeping-revisions-of-your-laravel-model-data/)

链接：[https://github.com/VentureCraft/revisionable](https://github.com/VentureCraft/revisionable)。

##使用第三方 Auth / Eloquent 拓展
Revisionable 支持 Auth powered by
[**Sentry by Cartalyst**](https://cartalyst.com/manual/sentry).
[**Sentinel by Cartalyst**](https://cartalyst.com/manual/sentinel).

##安装

`composer venturecraft/revisionable`

更新包

`composer update`

最后，您还需要在程序包（Laravel 5.x）上运行迁移

`php artisan migrate --path=vendor/venturecraft/revisionable/src/migrations`

如果你要copy migration 到app/database/migrations文件夹下，你要修改类名: CreateRevisionsTable 改成 CreateRevisionTable（去掉‘s’，否则会报你有一个重复的类的错）

`cp vendor/venturecraft/revisionable/src/migrations/2013_04_09_062329_create_revisions_table.php app/database/migrations/`

如果您想要保留修订历史的任何模型，包括可修正的命名空间并在模型中使用RevisionableTrait，例如。，如果您正在使用另一个trait，那么一定要在模型中重写引导方法;

    namespace MyApp\Models;
    
    class Article extends Eloquent {
    use \Venturecraft\Revisionable\RevisionableTrait;
    
	    public static function boot()
	    {
	    parent::boot()
	    }
    }

如果需要，您可以通过$revisionEnabled在模型中设置为false 来禁用修订。如果您要临时禁用修订，或者如果要创建自己的扩展版本的基础模型（所有模型都可以扩展），但是您希望关闭某些模型，则可以使用该功能。

    namespace MyApp\Models;
    
    class Article extends Eloquent {
	    use Venturecraft\Revisionable\RevisionableTrait;
	
	    protected $revisionEnabled = false;
    }

您还可以通过$historyLimit在停止修订之前设置为要保留的修订版本数来进行许多修订后，禁用修订版本。

	namespace MyApp\Models;

	class Article extends Eloquent {
	    use Venturecraft\Revisionable\RevisionableTrait;
	
	    protected $revisionEnabled = true;
	    protected $historyLimit = 500; //Stop tracking revisions after 500 changes have been made.
	}

为了保持对历史的限制，但是如果要删除旧的修订版本，而不是停止跟踪修订，则可以通过设置来适应该功能$revisionCleanup。

	namespace MyApp\Models;

	class Article extends Eloquent {
	    use Venturecraft\Revisionable\RevisionableTrait;
	
	    protected $revisionEnabled = true;
	    protected $revisionCleanup = true; //Remove old revisions (works only when used with $historyLimit)
	    protected $historyLimit = 500; //Maintain a maximum of 500 changes at any point of time, while cleaning up old revisions.
	}

###存储软删除

默认情况下，如果您的模型支持软删除，则可修订版将存储此模式，并将任何还原作为模型上的更新。
您可以通过添加 `deleted_at` 到 `$dontKeepRevisionOf` 数组里来选择忽略删除和恢复。
要更好地格式化 `deleted_at` 条目的输出，您可以使用`isEmpty`格式化程序（请参阅
[格式输出](https://github.com/VentureCraft/revisionable#format-output)以获得一个示例）。

###存储新建

默认情况下，新模型的创建不会作为修订版本存储。仅存储对模型的后续更改。如果要将创建存储为修订版本，可以通过将以下内容添加到模型中来设置`revisionCreationsEnabled`为覆盖此行为`true`：

    protected  $revisionCreationsEnabled  =  true ;

###更多的控制

毫无疑问，在某些情况下，您不想仅在模型的某些字段中存储修订历史记录，这有两种不同的方式。在您的模型中，您可以指定明确要跟踪哪些字段，并忽略所有其他字段：

    protected  $keepRevisionOf  =  array（' title '）;    

或者，您可以指定明确不想跟踪哪些字段。所有其他字段将被跟踪。

    protected  $dontKeepRevisionOf  =  array（' category_id '）;    

###Events
每次创建模型修订时，触发事件。你可以监听`revisionable.created`，
`revisionable.saved`或`revisionable.deleted`。

    public function boot()
    {
            parent::boot();

            Event::listen('revisionable.*', function($model, $revisions) {
                // Do something with the revisions or the changed model. 
                dd($model, $revisions);
            });
    }

###格式化输出


注意点：[Event 监听model变更](https://github.com/VentureCraft/revisionable/issues/233)laravel5.3跟5.4的区别

5.3版本：

	// app/Providers/EventServiceProviders.php
	public function boot(DispatcherContract $events)
	{
	    parent::boot($events);
	
	    $events->listen('revisionable.*', function($eventName, $data) {
	        // $eventName = revisionable.created, revisionable.saved or revisionable.deleted
	        // $data['model']
	        // $data['revisions']
	    });
	}

5.4版本：

	// app/Providers/EventServiceProviders.php
	public function boot()
	{
        parent::boot();

        Event::listen('revisionable.*', function($model, $revisions) {
            // Do something with the revisions or the changed model. 
            dd($model, $revisions);
        });
	}