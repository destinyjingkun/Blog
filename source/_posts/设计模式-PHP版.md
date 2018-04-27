---
id: 4
title: 设计模式-PHP版
date: 2018-04-26 18:01:59
tags: [设计模式, PHP]
category: PHP
---
###设计模式（php版本）
目录结构
|-database.php
|-factory.php
|-...
|-index.php	//调用的地方
```php
//database.php
class Database{
}
```
`正常为了实例化database对象都是直接new Database()`
####工厂模式
```php
//factory.php
class Factory{
	static function createDatabase(){
		retrun new Database();
	}
}
//index.php
//当一个对象在多个地方需要被调用的话，使用工厂模式可以减少以后修改的成本
$b = new Factory::createDatabase();
```
####单例模式
```php
//database.php
class Database{
	protected $db;
	//确保外部无法直接实例化
	private function __construct(){}
	static function getInstance(){
		if(self::$db){
			return self::$db;
		}else{
			self::$db = new self();
			return self::$db;
		}
	}
}
//factory.php
class Factory{
	static function createDatabase(){
		//改用单例创建,无论调用多少次都只会创建一次
		retrun Database::getInstance();
	}
}
```

####注册器模式（全局模式）

```php
//register.php
//将对象注册成全局变量，省得每次都调用工厂方法实例化
class Register{
	protected static $objects;
	//set obj
	function set($index, $obj){
		self::$objects[$index] = $obj;
	}
	//get obj
	function get($index){
		return self::$objects[$index];
	}
	//remove obj
	function _unset($index){
		unset(self::$objects[$index]);
	}
}
//工厂方法就可以将对象映射到注册树上
//factory.php
class Factory{
	static function createDatabase(){
		//改用单例创建,无论调用多少次都只会创建一次
		$db = Database::getInstance();
		Register::set('db', $db);
		return $db;
	}
}
//index.php 就可以直接从注册树上拿
$db = Register::get('db');
```

####适配器模式
将截然不同的函数接口封装成统一的API
`案例：mysql, mysqli, pdo统一成一致的，类似的还有cache适配器，将memcache, redis, file, apc等统一成一致`
```php
//database.php
interface IDatabase{
	function connect($host, $user, $password, $dbname);
	function query($sql);
	function close();
}
//mysql.php
class Mysql implements IDatabase{
	//实现忽略
	function connect($host, $user, $password, $dbname);
	function query($sql);
	function close();
}
//pdo.php
class Pdo implements IDatabase{
	//实现忽略
	function connect($host, $user, $password, $dbname);
	function query($sql);
	function close();
}
//index.php
//需要用到哪种sql服务就new哪种
$mysqll = new Mysql();
$db->connect('127.0.0.1', 'root', 'root', 'test');
$db->query("show database");
$db->close();
```
####策略模式
`案例：某电商系统需要根据男性用户跟女性用户分别展示不同的类目并根据性别展示不同的广告，如果一直在代码当中使用if判断是男的还是女的,容易造成代码的不可维护性`
```php
//user_strategy.php
interface UserStrategy{
	function showAd();
	function showCategory();
}
//female_user_strategy.php
class FemaleUserStrategy implement UserStrategy{
	function showAd(){ echo "女装"; }
	function showCategory(){ echo "裙子"; }
}
//male_user_strategy.php
class MaleUserStrategy implement UserStrategy{
	//实现忽略
	function showAd(){ echo "男装"; }
	function showCategory(){ echo "衬衫"; }
}
//到这还是跟适配器模式还是比较想象的，下面看下controller当中
//index.php
class Page{
	//这边Page跟外部的UserStrategy或者Female,Male等都进行了解耦，因为都是用了内部的$strategy
	//策略模式的依赖倒置跟控制反转
	protected $strategy;
	function index(){ //进行该有的showAd,showCategory操作而不同去区分是男性还是女性}
	//暴露给外面调用设置策略用的
	function setStrategy(UserStrategy $strategy){
		$this->strategy = $strategy;
	}
}
//外部调用，这样可以确保Page类的index方法里面没有if else判断性别类型
$page = new Page;
if (isset($_GET['female'])){
	$strategy = new FemaleUserStrategy();
}else{
	$strategy = new MaleUserStrategy();
}
$page->setStrategy($strategy);
```
####数据对象映射模式
数据对象映射模式：是将对象和数据存储映射起来，这样对一个对象的操作会映射成对数据存储的操作，典型的有ORM
```php
//user.php
class User{
	//外部允许访问
	public $id;
	public $name;
	public $mobile;

	protected $db;
	//方便演示：这边使用的是构造方法跟析构方法，真正的orm是独立成各种create,update,delete等方法
	function __construct($id){
		$this->db = new Mysql();
		$this->db->connect('127.0.0.1', 'root', 'root', 'test');
		$res = $this->db->query("select * from user limit 1");
		$data = $res->fetch_assoc();
		$this->id = $data['id'];
		$this->mobile = $data['mobile'];
		$this->name = $data['name'];
	}
	//析构方法
	function __destruct(){
		$this->db->query("update user set name ='{$this->name}', mobiel='{$this->mobile}' where id = $this->id");
	}
}
//index.php
$user = new User(1);//拿到对应id为1的用户
$user->mobile = '18059219999';
$user->name = 'test';
```

####观察者模式（Observer）
观察者模式：当一个对象状态发生改变时，依赖它的对象全部会收到通知，并自动更新
场景：一个事件发生后，要执行一连串更新操作，传统的编程方式，就是在事件的代码之后直接加入处理逻辑，当更新的逻辑增多之后，代码会变得难以维护，这种方法是耦合的，增加新的逻辑需要修改事件的主体代码，而观察者模式实现了低耦合，非侵入式的通知与更新机制
```php
//index.php
class Event{
	function trigger(){
		echo 'event happened!';
		//传统方法,直接加在后面
		echo 'new event1 add';
		echo 'new event2 add';
		echo 'new event3 add';
	}
}
//传统调用
$event = new Event;
$event->trigger();
//...............................
//修改成观察者模式
//首先声明一个抽象类
//event_generator.php
abstract class EventGenerator{
	private $observers = array();
	//添加观察者
	function addObserver(Observer $obs){
		$this->observers[] = $obs;
	}
	//通知其它事件进行更新
	function notify(){
		foreach($this->observers as $obs){
			$obs->update();
		}
	}
}
//observer.php
interface Observer{
	function update($event_info = null);
}
//所以Event就要改成如下
class Event extends EventGenerator{
	function trigger(){
		echo "event";
		//下面的时间就交给基类去通知其它观察者进行更新
		$this->notify();
	}
}
//然后就可以添加新的观察者，而对原代码不会进行修改
class Observer1 implements Observer{
	function update($event_info = null){
		echo "逻辑1";
	}
}
class Observer2 implements Observer{
	function update($event_info = null){
		echo "逻辑2";
	}
}
class Observer3 implements Observer{
	function update($event_info = null){
		echo "逻辑3";
	}
}
//调用就可以更改为
$event = new Event;
$event->addObserver(new Observer1());
$event->addObserver(new Observer2());
$event->addObserver(new Observer3());
$event->trigger();
```
####原型模式
1.原型模式与工厂模式类似，都是用来创建对象
2.与工厂模式的实现不同，原型模式是先创建好一个原型对象，然后通过clone原型对象来创建新的对象。这样就免去了类创建时重复的初始化操作
3.原型模式适用于大对象的创建，创建一个大对象需要很大的开销，如果每次new就会消耗很大，而原型模式仅需要内存拷贝即可
实例：假定有一个Canvas的类
```php
//canvas.php 画布类
class Canvas{
	public $data;
	//打印一个矩形*
	function init($width = 20, $height = 10){
		$data = array();
		for($i =0; $i< $height; $i++){
			for($j = 0; $j< $width; $j++){
				$data[$i][$j] = '*'
			}
		}
		$this->data = $data;
	}
	function draw(){
		foreach($this->data as $line){
			foreach($line as $char){
				echo $char;
			}
			echo "<br/>\n";
		}
	}
	//绘制一个空的矩形
	function rect($a1, $a2, $b1, $b2){
		foreach($this->data as $k1=>$line ){
			if($k1 < $a1 or $k1 > $a2) continue;
			foreach($line as $k2 =>$char){
				if($k2 < $b1 or $k2 > $b2) continue;
				$this->data[$k1][$k2] = '&nbsp;';
			}
		}
	}
}
```
```php
//index.php
$canvas = new Canvas();
$canvas->init();
$canvas->rect(3,6,4,12);
$canvas->draw();
//假如我们需要绘制2次，因为第一次调用rect()的时候已经对data进行了赋值，导致我们第二次再使用的时候只能重新实例化，而每次实例化的时候为了创建一个data我们都得执行200次的循环，消耗非常大
$canvas = new Canvas();
$canvas->init();
$canvas->rect(1,3,2,6);
$canvas->draw();
//而是改成.................
$prototype = new Canvas();
$prototype->init();

$canvas1 = clone $prototype;
$canvas1->rect(3,6,4,12);
$canvas1->draw();

$canvas2 = clone $prototype;
$canvas2->rect(1,3,2,6);
$canvas2->draw();
```
####装饰器模式
1.装饰器模式（Decorator）,可以动态地添加修改类的功能
2.一个类提供了一项功能，如果要在修改并添加额外的功能，传统的编程模式，需要写一个子类继承它，并重新实现类的方法
3.使用装饰器模式，仅需要再运行时添加一个装饰器对象即可实现，可以实现最大的灵活性
```php
//了解Python的装饰器可能就比较容易理解了
//使用上面的Canvas类，假设我们的draw需要改成红色的，传统的做法是新增一个子类，然后重写draw这个方法
class Canvas2 extends Canvas{
	function draw(){
		echo "<div style='color:red'>";
		parent::draw();
		echo "</div>";
	}
}
//然后调用子类的draw()，如果下次还需要变成绿色的？或者改变draw大小等，一直重写就达不到灵活性，最好的方法就是把它做成一个装饰器
//draw_decorator.php
interface DrawDecorator{
	function beforeDraw();
	function afterDraw();
}
//canvas.php
class Canvas{
	protected $decorators = array();//用来保存装饰器对象
	//在draw之前跟之后调用，源代码就不用更改
	function draw(){
		$this->beforeAdd();
		//原来逻辑代码
		$this->afterAdd();
	}
	function addDecorator(DrawDecorator $decorator){
		$this->decorators[] = $decorator;
	}
	//然后遍历逐个调用装饰器的方法
	function beforeAdd(){
		foreach($this->decorators as $dec){
			$dec->beforeDraw();
		}
	}
	function afterAdd(){
		//这边注意的话decorators的话要进行反转，before是先进先出，after是后进先出
		$decorators = array_reverse($this->decorators);
		foreach($decorators as $dec){
			$dec->afterDraw();
		}
	}
}
//接下来只需要去实现装饰器的对象即可
将上面的传统方法改成如下的装饰器对象
//color_decorator.php
class ColorDecorator implements DrawDecorator{
	protected $color;
	function __construct($color = 'red'){
		$this->color = $color;
	}
	function beforeDraw(){
		echo "<div style='color:{$this->color}'>";
	}
	function afterDraw(){
		echo "</div>";
	}
}
//index.php
$canvas = new Canvas();
$canbas = init();
//在绘制之前加入颜色装饰器即可
$red_docorator = new ColorDecorator('green');
$canvas->addDecorator($red_decorator);
//需要修改大小只需要再添加一个SizeDecorator即可，这样就实现了解耦
$canvas->rect(3,6,4,12);
$canvas->draw();
```
####迭代器模式
1.迭代器模式，在不需要了解内部实现的前提下，遍历一个聚合对象的内部元素
2.相比于传统的编程模式，迭代器模式可以隐藏遍历元素的所需的操作
```php
//all_user.php
//迭代器需要继承spr的迭代器接口
class AllUser implements Iterator{
	protected $ids;
	protected $data; //保存从数据库取到的数据
	protected $index; //表示迭代器的当前位置
	function __construct(){
		$db= Factory::getDatabase();
		$result = $db->query("select id from user");
		$this->ids = $result->fetch_all(MYSQL_ASSOC);
	}
	//需要实现迭代器的5个方法
	//调用顺序： 2
	function current(){
		$id = $this->ids[$this->index]['id'];//拿到对应数据库当中的ID;
		return Factory::getUser($id); //返回用户对象
	}
	//调用顺序： 4
	function next(){
		$this->index ++;
	}
	//调用顺序： 1
	function valid(){
		//代表没有下一个元素了
		return $this->index < count($this->ids);
	}
	//调用顺序： 0
	function rewind(){
		$this->index = 0;
	}
	//调用顺序： 5 获取当前的索引
	function key(){
		return $this->index;
	}
}
//index.php
$users = new AllUser();
//而不是$users= new User().find_all();这样的形式
foreach ($users as $user){
	var_dump($user);
}
```
####代理模式
1.在客户端与实体之间建立一个代理对象（proxy）,客户端对实体进行操作全部委派给代理对象，隐藏实体的具体实现细节
2.Proxy还可以与业务分离，部署到另外的服务器，业务代码种通过RPC来委派任务
```php
//index.php
//传统方法,不管是读写等操作都要显示的知道他的操作工程
$db = Factory::getDatabase('slave');
$info = $db->query("select name from user where id = 1 limit 1");

$db1 = Factory::getDatabase('master');
$db1->query("update user name='evan' where id=1 limit 1");
//代理模式
//iuserproxy.php
interface IUserProxy{
	function getUsername($id);
	function setUsername($id, $name);
}
//proxy.php
class Proxy implements IUserProxy{
	function getUsername($id){
		$db = Factory::getDatabase('slave');
$info = $db->query("select name from user where id = 1 limit 1");
	}
	function setUsername($id, $name){
		$db = Factory::getDatabase('master');
		$db->query("update user name='evan' where id=1 limit 1");
	}
}
//index.php
//这样在整个逻辑里面就没有对数据库操作的详细流程。跟java的service层有点相似
$proxy = new Proxy();
$proxy->getUsername($id);
$proxy->setUsername($id, $proxy);
```

