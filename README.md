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
* MyPatrol  
```
public class MyPatrol{
	public int PatrolArea;//记录自己的当前区域，从左到右，上到下，从1到9
	public int PlayerArea;//记录玩家所在区域
	public bool isChasePlayer;
	public GameObject PatrolObj;
	public MyPatrol (){
		isChasePlayer = false;
	}
}
```
* 工厂模式：  
```
public class PatrolFactory :System.Object {

		public GameObject Patrolprefab;
		private static PatrolFactory _PatrolFactory;
		public  List<MyPatrol> usingPatrols{ set; get;}
		private List<MyPatrol> uselessPatrols;
		private MyPatrol _patrol;
		private Vector3[] patrolStartPos;

		public List<MyPatrol> getUsingPatrols(){
			return usingPatrols;
		}
		public List<MyPatrol>  getuselessPatrols(){
			return uselessPatrols;
		}

		public bool getIsChasePlayer(int tagNum){
			foreach (MyPatrol _mypatrol in usingPatrols) {
				if (_mypatrol.PatrolObj.tag.Equals (tagNum.ToString ())) {
					return _mypatrol.isChasePlayer;
				}
			}
			Debug.Log ("cannot find the goal MyPatrol");
			return false;
		}
		public bool getIsChasePlayer(string tagStr){
			foreach (MyPatrol _mypatrol in usingPatrols) {
				if (_mypatrol.PatrolObj.tag.Equals (tagStr)) {
					return _mypatrol.isChasePlayer;
				}
			}
			Debug.Log ("cannot find the goal MyPatrol");
			return false;
		}

		public static PatrolFactory getFactory()
		{
			if (_PatrolFactory == null)
			{
				_PatrolFactory = new PatrolFactory();//相当于创造一个空的工厂，里面有usingPatrols这些指针，但还没有分配空间
				_PatrolFactory.uselessPatrols = new List<MyPatrol>();
				_PatrolFactory.usingPatrols = new List<MyPatrol>();
			}
			return _PatrolFactory;
		}

		public void resetStartPos(){
			patrolStartPos = new Vector3[]{ 
				new Vector3(-10f,0f,13f),new Vector3(-3f,0f,13f),new Vector3(10f,0f,13f),
				new Vector3(-10f,0f,0f),new Vector3(-3f,0f,-3f),new Vector3(10f,0f,0f),
				new Vector3(-10f,0f,-10f),new Vector3(0f,0f,-10f),new Vector3(10f,0f,-10f)
				};
		}
		public List<MyPatrol> creatPatrols()//默认构造9个,初始化位置
		{
			resetStartPos ();
			for (int i = 0; i < 9; i++)
			{	

				if (uselessPatrols.Count == 0)
				{
					_patrol=new MyPatrol();
					_patrol.PatrolObj = GameObject.Instantiate<GameObject>(Patrolprefab);
					_patrol.PatrolObj.transform.position = patrolStartPos [i];
					_patrol.PatrolObj.tag=(i+1).ToString();
					_patrol.PatrolArea = i + 1;
					Controller.getController ().doPatrol (_patrol);

					usingPatrols.Add(_patrol);

				}
				else//从uselessPatrol拿出来，不额外创建
				{
					MyPatrol Patrol = uselessPatrols[0];
					uselessPatrols.RemoveAt(0);
					Patrol.PatrolObj.transform.position = patrolStartPos [i];
					_patrol.PatrolObj.tag=(i+1).ToString();
					Controller.getController ().doPatrol (_patrol);
					usingPatrols.Add(Patrol);
				}
			}
			return this.usingPatrols;
		}
		public void deletePatrol(MyPatrol Patrol)
		{
			int index = usingPatrols.FindIndex(x => x == Patrol);//找到usingPatrols里面要回收的Patrols放回uselessPatrols
			uselessPatrols.Add(Patrol);
			usingPatrols.RemoveAt(index)
		}

	}
```

##### 其余代码：  
* SSActionManager:  
```
public class SSActionManager : MonoBehaviour {
	private Dictionary<int, SSAction> actions = new Dictionary<int, SSAction>();  
	private List<SSAction> waitingAdd = new List<SSAction>();  
	private List<int> waitingDelete = new List<int>();  

	// Use this for initialization  
	void Start()  {  
	}  

	// Update is called once per frame  
	protected void Update()  
	{  
		//Debug.Log ("waitingAdd: " + waitingAdd.Count);
		foreach (SSAction ac in waitingAdd) {
			actions[ac.GetInstanceID()] = ac;  

		} 
		waitingAdd.Clear();  

		foreach (KeyValuePair<int, SSAction> kv in actions)  
		{  
			SSAction ac = kv.Value;  
			if (ac.destroy)  
			{  
				waitingDelete.Add(ac.GetInstanceID());  
			}  
			else if (ac.enable)  
			{  
				//Debug.Log("")
				ac.Update();  
				//ac.FixedUpdate();  
			}  
		}  

		foreach (int key in waitingDelete)  
		{  
			SSAction ac = actions[key]; 
			actions.Remove(key); 
			DestroyObject(ac);  
		}  
		waitingDelete.Clear();  
	}  

	public void RunAction(GameObject gameobject, SSAction action, ISSActionCallback manager)  
	{  
		//先将该物体原有动作删除
		for (int i = 0; i < waitingAdd.Count; i++) {
			if (waitingAdd[i].gameobject.Equals(gameobject)) {
				SSAction ac = waitingAdd[i];
				waitingAdd.RemoveAt(i);
				i--;
				DestroyObject(ac);
			}
		}

		//Debug.Log ("SS tag: " + gameobject.tag);
		action.gameobject = gameobject;  
		action.transform = gameobject.transform;  
		action.callback = manager;  
		//action.enable = true;

		///
		waitingAdd.Add(action); 
		//Debug.Log ("action: " + action);
		action.Start();  
	}  
}

```
* SSAction  
```
public enum SSActionEventType : int { Started, Competeted }  

public interface ISSActionCallback  
{  
	void SSActionEvent(SSAction source, SSActionEventType events = SSActionEventType.Competeted,  
		int intParam = 0, string strParam = null, Object objectParam = null);  
}  

public class SSAction : ScriptableObject   {

	public bool enable = true;  
	public bool destroy = false;  

	public GameObject gameobject { get; set; }  
	public Transform transform { get; set; }  
	public ISSActionCallback callback { get; set; }  

	protected SSAction() { }  

	public virtual void Start()  
	{  
		throw new System.NotImplementedException();  
	}  

	public virtual void Update()  
	{  

		throw new System.NotImplementedException();  
	} 


	public void reset(){
		enable = false;
		destroy = true;
		gameobject = null;
		transform = null;
		callback = null;
	}
}

```
* Interact
```
public class Interact : MonoBehaviour {

	private GUIStyle fontStyle1,fontStyle2;
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

	// Use this for initialization
	void Start () {
		fontStyle1 = new GUIStyle ();
		fontStyle2= new GUIStyle ();
		fontStyle1.fontSize = 40;
		fontStyle2.fontSize = 80;
		score = 0;
		gamestatus = 1;
	}
	
	// Update is called once per frame
	void Update () {
		
	}
	void OnGUI(){
		GUI.Label (new Rect (Screen.width / 2-100, Screen.height / 2-250, 100, 50), "score: "+score, fontStyle1);
		if (gamestatus == 0) {
			GUI.Label (new Rect (Screen.width / 2-150, Screen.height / 2+50, 100, 50), "game over", fontStyle2);
		}
	}
}

```
* userGUI:  
```
public class userGUI : MonoBehaviour {

		public userGUI _userGUI;
		private int player_speed;
		private int rotate_speed ;
		public GameObject meObj;
		public static int playerArea{ set; get;}//玩家所在区域，从左到右，上到下，从1到9
		public  static Vector3 PlayerCurrentPos{set; get;} 
		private Rigidbody rb;

		/*public static Vector3 getPlayerCurrentPos(){
		return meObj.transform.position;
	
		}*/
			/*public userGUI getUSERGUI(){
			if (_userGUI == null) {
				_userGUI = new userGUI ();
				return _userGUI;
			}
			return _userGUI;
		}*/

		// Use this for initialization
		void Start () {
			player_speed = 10;
			rotate_speed = 180;
			rb = GetComponent<Rigidbody> ();
		}

		// Update is called once per frame
		void Update () {

			PlayerCurrentPos = meObj.transform.position;

			//获取方向键的偏移量
			float translationX = Input.GetAxis("Horizontal");
			float translationZ = Input.GetAxis("Vertical");

			//移动和旋转
			meObj.gameObject.transform.Translate(0, 0, translationZ * player_speed * Time.deltaTime);
			meObj.gameObject.transform.Rotate(0, translationX * rotate_speed * Time.deltaTime, 0);
			//事实判断玩家所在区域位置
			float posX=meObj.transform.position.x;
			float posZ = meObj.transform.position.z;
			if (posZ > 5) {
				if (posX < -5) {
					playerArea = 1;
				} else if (posX < 5) {
					playerArea = 2;
				} else {
					playerArea = 3;
				}
			} else if (posZ > -5) {
				if (posX < -5) {
					playerArea = 4;
				} else if (posX < 5) {
					playerArea = 5;
				} else {
					playerArea = 6;
				}

			} else {
				if (posX < -5) {
					playerArea = 7;
				} else if (posX < 5) {
					playerArea = 8;
				} else {
					playerArea = 9;
				}
			}
			//Debug.Log ("player area: " + playerArea);
		}
		void OnCollisionStay(Collision collisionInfo){

		}

		public void OnTriggerEnter(Collider other){//玩家触碰，说明逃脱追捕,触发controller发布消息
			Controller.getController ().playerEscapeSuccess ();
		}

	}
```
* ChaseAction  
```
public class ChaseAction : SSAction {
	private Vector3 Pos;
	private static ChaseAction action ;

	public static ChaseAction getSSAction(){//避免创造太多chaseAction
		if (action == null) {
			action= ScriptableObject.CreateInstance<ChaseAction> ();
		}
		return action;
	}

	// Update is called once per frame
	public override void Update () {
		//Debug.Log ("attract patrol No.: " + this.gameobject.tag);
		//根据isChasePlayer来判断
		if (PatrolFactory.getFactory ().getIsChasePlayer (this.gameobject.tag)) {
			Vector3 playerPos = userGUI.PlayerCurrentPos;
			Vector3 patrolPos = this.gameobject.transform.position;
			this.transform.LookAt (playerPos);
		} else {
			
		}
	}

}
```

* CCActionManager:  
```
public class CCActionManager:  SSActionManager,ISSActionCallback {
	public PatrolAction _PatrolAction;
	public ChaseAction _ChaseAction;

	public void DoPatrolAction(MyPatrol _MyPatrol){
		_PatrolAction = PatrolAction.getSSAction ();
		//Debug.Log("CCActionManager: MyPatrol obj: "+_MyPatrol.PatrolObj);

		this.RunAction (_MyPatrol.PatrolObj, _PatrolAction, this);
	}
	public void DoChaseAction(MyPatrol _MyPatrol){
		_ChaseAction = ChaseAction.getSSAction ();
		this.RunAction (_MyPatrol.PatrolObj, _ChaseAction, this);
	}
	public void SSActionEvent(SSAction source, SSActionEventType events = SSActionEventType.Competeted,  
		int intParam = 0, string strParam = null, Object objectParam = null)  {
	}
}

```
*  PatrolBehaviour
```
public class PatrolBehaviour : MonoBehaviour {
	public void OnCollisionStay(Collision collisionInfo){//scriptableObject无法使用这个方法
		if (collisionInfo.gameObject.CompareTag ("Player")) {//抓到玩家，触发Controller的事件
			Controller.getController ().playerCaught ();
			collisionInfo.gameObject.GetComponent<Animator> ().SetBool ("isGameOver", true);
		} else if( !collisionInfo.gameObject.CompareTag ("Plane")){//撞到墙,转向90度
			this.gameObject.transform.Rotate(0, 10 ,0);
		}
	}
	public void OnTriggerStay(Collider other){
		//Debug.Log ("OnTriggerStay");
		this.gameObject.transform.Rotate(0, 10 ,0);//避免出界
	}
}
```

