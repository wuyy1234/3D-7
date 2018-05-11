# 3D-7
## 智能巡逻兵

### 提交要求：
> * 游戏设计要求：  
>   * 创建一个地图和若干巡逻兵(使用动画)；  
>   * 每个巡逻兵走一个3~5个边的凸多边型，位置数据是相对地址。即每次确定下一个目标位置，用自己当前位置为原点计算；  
>   * 巡逻兵碰撞到障碍物，则会自动选下一个点为目标；  
>   * 巡逻兵在设定范围内感知到玩家，会自动追击玩家；  
>   * 失去玩家目标后，继续巡逻；  
> * 计分：玩家每次甩掉一个巡逻兵计一分，与巡逻兵碰撞游戏结束；   
>   * 程序设计要求：  
>   * 必须使用订阅与发布模式传消息  
> * subject：OnLostGoal   
> * Publisher: ?   
> * Subscriber: ?   
> * 工厂模式生产巡逻兵  

#### 原始预览效果
 <img src="http://imglf6.nosdn.127.net/img/Z281REhERnhNZlc2R1NNZEpNYmJLR2pFZzFBang5dWlJajZXQURGYUJiOVZBRllTUHpnS05RPT0.gif"  />  
 如果打不开链接：http://imglf6.nosdn.127.net/img/Z281REhERnhNZlc2R1NNZEpNYmJLR2pFZzFBang5dWlJajZXQURGYUJiOVZBRllTUHpnS05RPT0.gif
 
#### 最终加了动画的效果
 <img src="http://imglf6.nosdn.127.net/img/Z281REhERnhNZlc2R1NNZEpNYmJLT2lpMkM4aER2djdMSlNnOU53QUczU1dSVCtDQUpzYUxRPT0.gif"  />  
 如果打不开链接：http://imglf6.nosdn.127.net/img/Z281REhERnhNZlc2R1NNZEpNYmJLT2lpMkM4aER2djdMSlNnOU53QUczU1dSVCtDQUpzYUxRPT0.gif
 
#### 架构图
 <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlc2R1NNZEpNYmJLQ2hiNytzNW1uTlJpRFI1SDd5c3JrYjBMcHRYRjRYc01RPT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  />  
 
#### 主要代码分析
* 订阅与发布模式  
  * 网上有个例子简要讲明了这个用法：
>  有一家三口，妈妈负责做饭，爸爸和孩子负责吃。。。将这三个人，想象成三个类。  
> 妈妈有一个方法，叫做“做饭”。有一个事件，叫做“开饭”。做完饭后，调用开发事件，发布开饭消息。
> 爸爸和孩子分别有一个方法，叫做“吃饭”。  
> 将爸爸和孩子的“吃饭”方法，注册到妈妈的“开饭”事件。也就是，订阅妈妈的开饭消息。让妈妈做完饭开饭时，发布吃饭消息时，告诉爸爸和孩子一声。  
```
class Program
    {
        public static void Main(string[] args)
        {
            //实例化对象
            Mom mom = new Mom();
            Dad dad = new Dad();
            Child child = new Child();
            
            //将爸爸和孩子的Eat方法注册到妈妈的Eat事件
            //订阅妈妈开饭的消息
            mom.Eat += dad.Eat;
            mom.Eat += child.Eat;
            
            //调用妈妈的Cook事件
            mom.Cook();
            
            Console.Write("Press any key to continue . . . ");
            Console.ReadKey(true);
        }
    }
    
    public class Mom
    {
        //定义Eat事件，用于发布吃饭消息
        public event Action Eat;
        
        public void Cook()
        {
            Console.WriteLine("妈妈 : 饭好了");
            //饭好了，发布吃饭消息
            Eat?.Invoke();
        }
    }
    
    public class Dad{
        public void Eat()
        {
            //爸爸去吃饭
            Console.WriteLine("爸爸 : 吃饭了。");
        }
    }
    
    public class Child
    {
        public void Eat(){
            //熊孩子LOL呢，打完再吃
            Console.WriteLine("孩子 : 打完这局再吃。");
        }
    }
```
 * 如果没有使用这个模式，下面是以前打飞碟的代码，可以看到各个代码块的耦合性很高：    
```
public void throwUFO(int num){
			//先从工厂里面拿飞盘
			usingUFOs=UFOFactory.getFactory().creatUFOs(num);
			Scene.getScene ().throwUFO (usingUFOs);
}
```
* 本次使用的实践： 
  * 发布消息：  
```
public delegate void changeScore ();
		public static event changeScore addScoreEvent;

		public delegate void changeGameStatus ();
		public static event changeGameStatus gameOverEvent;
		//发布消息
		public void playerEscapeSuccess(){
			if (addScoreEvent != null) {
				addScoreEvent();
			}
		}
		public void playerCaught(){
			if (gameOverEvent != null) {
				gameOverEvent ();
			}
		}
```
  * 订阅消息：  
```
private int score;
	private int gamestatus;//1为正常，0为结束

	void OnEnable(){
		Controller.addScoreEvent += addScore;
		Controller.gameOverEvent += gameOver;
	}

	void addScore(){
		score++;
	}
	void gameOver(){
		gamestatus = 0;
	}
```
  * 触发发布消息:  
```
public void OnTriggerEnter(Collider other){//玩家触碰，说明逃脱追捕,触发controller发布消息
			Controller.getController ().playerEscapeSuccess ();
}

if (collisionInfo.gameObject.CompareTag ("Player")) {//抓到玩家，触发Controller的事件
			Controller.getController ().playerCaught ();
			collisionInfo.gameObject.GetComponent<Animator> ().SetBool ("isGameOver", true);
}
```





