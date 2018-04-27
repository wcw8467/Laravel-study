##Laravel---模型标签关系编辑

####方案一

+ form表单

		$关联模型变量 = 关联模型::pluck('name', 'id');
		$form->listbox('关联模型方法名', '中文名')->options();
		或
		$form->tags('关键字');
+ store

		// 手动添加-中间表
        if (is_array($request->get('关联模型方法名'))) {
            $该模型变量->关联模型方法名()->sync(array_filter($request->get('关联模型方法名')));
        } 
		
+ Update

		// 手动添加-中间表
        if (is_array($request->get('关联模型方法名'))) {
            $该模型变量->关联模型方法名()->sync(array_filter($request->get('关联模型方法名')));
        } 

####方案二
### laravel-tagging
安装步骤

1. composer 加载包 `composer require rtconner/laravel-tagging "~2.2"`
2. 安装并进行迁移
```
 'providers' => array(
	\Conner\Tagging\Providers\TaggingServiceProvider::class,
);
```
3. 加载包
``
 php artisan vendor:publish --provider="Conner\Tagging\Providers\TaggingServiceProvider"
``
4. 合并`php artisan migrate`

5. 设置你的模型(这里我使用article模型举例)

		
		class Article extends \Illuminate\Database\Eloquent\Model {
			// 在模型里使用taggable
			use \Conner\Tagging\Taggable;
		}

- 获取模型已有的标签

		Article::where('id', '=', $id)->with('tagged')->first();
  		foreach ($article->tags as $tag) {
            echo $tag->name . ' with url slug of ' . $tag->slug;
        }

- 给模型添加标签

		标签可以是数组,也可以单个
		$tag   = ['水果', '学习', '运动', '休息'];
		$article             = new Article();
		// 添加标签
		$article->tag($tag);

- 重新保存标签

		$article = Article::where('id', '=', $id)->with('tagged')->first();
    	$arr     = ['学习', '运动'];
		//重新保存标签
		$article->retag($arr)


- 删除模型的一个或多个标签
		
		// 可以是数组,也可以是字符串
		$tag     = '水果';
		$article = Article::where('id', '=', $id)->first();
		// 删除指定标签
		$article->untag($tag)
		
- 删除模型的所有的标签

		$article = Article::where('id', '=', $id)->first();
		// 删除所有的标签
		$article->untag()

- 展示当前模型的所有标签1
	
		$article = Article::where('id', '=', $id)->first();
		// 获取当前文章的所有标签
		$tag     = $article->tagNames();

- 展示模型的所有标签2

		$article = Article::existingTags();
		
- 查找指定标签的模型,有其中一个标签即可
		
		$articles = Article::withAnyTag(['水果','学习'])->get();

- 查找指定标签的模型,需全部满足条件
		
		$articles = Article::withAllTags(['水果', '学习'])->get();

- 查找没有指定标签的模型	

		$articles = Article::withoutTags(['水果'])->get();

- 查找使用超过n次的模型

		$articles = \Conner\Tagging\Model\Tag::where('count','>',$num)->get();


- delete current tags and save new tags `$directors->retag(array('Gardening', 'Cookie'));`,`$directors->retag('Fruit', 'Fish');`


- 创建标签组`php artisan tagging:create-group MyTagGroup`
- 给某个标签设置标签组`$tag = Tag::where(['id'=>16])->first(); $tag->setGroup('MyTagGroup');`
- 获取某个组中的所有的标记` $tag = Tag::inGroup('MyTagGroup')->get();`
- 检查某个标签是否存在组中`$tag->isInGroup('MyTagGroup')`
- 注意
- 不支持不同组相同标签


### laravel-tags:
标签将被存储在tags-table中。使用这些功能时，我们会确保标签是唯一的，而且模型只能附加一个标签。

添加一个标签
`$yourModel->attachTag('tag 1');`

添加多个标签
`$yourModel->attachTags(['tag 2', 'tag 3']);`

使用一个实例using an instance of \Spatie\Tags\Tag
`$yourModel->attach(\Spatie\Tags\Tag::createOrFind('tag4'));`

分离标签
标签分离如下

- 使用一个字符串`$yourModel->detachTag('tag 1');`
- 使用一个数组`$yourModel->detachTags(['tag 2', 'tag 3']);`
- //using an instance of \Spatie\Tags\Tag`$yourModel->detach(\Spatie\Tags\Tag::Find('tag4'));`

- 同步标签`$yourModel->syncTags(['tag 2', 'tag 3']);`


管理标签
- 创建一个标签`$tag = Tag::create(['name' => 'my tag']);`
- 更新一个标签`$tag->name = 'another tag'; $tag->save();`
- 创建一个标签如果它还不存在`$tag = Tag::findFromString('another tag');`
- 删除一个标签` $tag->delete();`
- 返回具有给定标记的一个或多个模型的模型。`YourModel::withAnyTags(['tag 1', 'tag 2'])->get();`
- 返回的模型都给有着这些标签`YourModel::withAllTags(['tag 1', 'tag 2'])->get();`
- 返回给类型中具备所有标签的模型`YourModel::withAllTags(['tag 1', 'tag 2'], 'myType')->get();`
- 在你的应用程序的当前区域设置存储`$tag = Tag::findOrCreate('my tag');`
-  添加一些其他语言的翻译` $tag->setTranslation('name', 'fr', 'mon tag');$tag->save();`,翻译功能需要支持json列
-  创建一个标签具有一定的类型`$tagWithType = Tag::findOrCreate('headline', 'newsTag');`
-  除了字符串，基本使用部分中提到的所有方法也可以实例化Tag。`$newsItem->attachTag($tagWithType);`
-    要获取具有特定类型的所有标签,使用getWithType方法
 ``` 
    $tagA = Tag::findOrCreate('tagA', 'firstType');
    $tagB = Tag::findOrCreate('tagB', 'firstType');
    $tagC = Tag::findOrCreate('tagC', 'secondType');
    $tagD = Tag::findOrCreate('tagD', 'secondType');
    // 使用以下方法将会返回$tagA和$tagB
    Tag::getWithType('firstType');
``` 

-  从model对象中,您还可以通过以下tagWithType方法获取特定类型的所有标签 ,返回一个集合`$nwesItem->tagsWithType('firstType');`
-  在使用了spatie/eloquent-sortable你可以使用该包提供的任何模型`$orderedTage = Tags::ordered()->get();`
-  设置一个完全新的排序`Tags::setNewOrder($arrayWithTagIds);$myModel->moveOrderUp(); $myModel->moveOrderDown();`

- 将标签移到第一个或者是最后一个位置`$tag->moveToStart(); $tag->moveToEnd();`
- 交换排序`$tag->swapOrder($anotherTag);`
 