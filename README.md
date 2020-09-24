# Unity-C-delegate-
Unity&amp;C#delegate的委托事件总结



基础知识：
发布器（publisher） 是一个包含事件和委托定义的对象。事件和委托之间的联系也定义在这个对象中。发布器（publisher）类的对象调用这个事件，并通知其他的对象。即下文中的Mom（发布方）

订阅器（subscriber） 是一个接受事件并提供事件处理程序的对象。 在发布器（publisher）类中的委托调用订阅器（subscriber）类中的方法（事件处理程序）。即下文中的Mom,Dad（监听方）

发出者sender：用于记录调用事件的是哪个类

注意：我们通常将 委托类和 委托类的对象都成为委托，但是两者是有区别的。一旦定义了委托类，基本上就可以实例化它的实例，在这一点上和普通类似一致的。即我们也可以有 委托数组。

UnityAction本质上是delegate，且有数个泛型版本（参数最多是4个）,一个UnityAction可以添加多个函数（多播委托）

UnityEvent本质上是继承自UnityEventBase的类，它的AddListener()方法能够注册UnityAction，是临时的
RemoveListener能够取消注册UnityAction，
还有Invoke()方法能够一次性调用所有注册了的UnityAction。
UnityEvent也有数个泛型版本（参数最多也是4个），但要注意的一点是，UnityAction的所有带参数的泛型版本都是抽象类（abstract），所以如果要使用的话，需要自己声明一个类继承之，然后再实例化该类才可以使用。
Unity中通过面板中添加的Listener和通过脚本添加的Listener实际上是两种不同类型的Listener：

在脚本中通过AddListener()添加的是一个0个参数的delegate(UnityAction)回调。是不可序列化的，在Inspector中是无法看到的。这种Listener是常规Listener。
在Inspector中添加的则是永久性的Listener（persistent listener)。他们需要指定GameObject、方法以及方法需要的参数。他们是序列化的，用脚本是无法访问到的。

技巧：可以给方法写不会用到的参数，从而把更多方法注册给委托
比如如果想将一个方法注册给一个委托，但是这个方法又没有相应的参数，那么就可以加无用参数


Action<T>和Func<T>委托
       我们之前都是根据返回值类型和参数列表来定义委托类，然后在根据委托类来生成委托的实例。现在我们还可以使用泛型委托类Action<T>和Func<T>。
       泛型Action<T>委托类表示引用一个 void返回类型的方法，可以传递至多16个不同的参数类型。 Action<in T1,in T2, …,in Tn> (n最大为16,例如Action<in T1,in T2>就表示调用2个参数的方法)。Func<T>委托的使用方式和Action<T>委托类 似. Func<T>允许调用带有返回值的方法。Func<in T1, in T2, ...,in Tn, out TResult> (n的最大值还是16,Func<in T1, in T2, out TResult>表示调用两个参数的方法且返回值类型为TResult)。
我们用Func<T>委托来实现上述委托：
Func<int, int, int> TestDele = add;

      这一条语句就等价于委托类的声明和委托对象的创建。同理，Action<T>也是一样的用法。而且功能上没有任何的不同。唯一不足之处就是参数的个数是有限制的，不过大多数的情况下16个的参数已经足够使用了。

1、匿名委托

       在这之前我们使用委托那么都必须先有一个方法。那么现在我们可以通过另一种方式使委托工作： 匿名方法。用匿名方法来实现委托和之前的定义并没有太大的区别，唯一不同之处就在于实例化。我们就以之前Calc委托类为例： 
Calc TestDele = delegate(int a,int b){        //代码块(因为Calc委托类是有返回值的，所以函数体内必须有return语句)}；      //这里有一个分号，千万不能漏 
      通过使用匿名方法，由于不必创建单独的方法，因此减少了实例化委托所需的编码系统开销。而且使用匿名方法可以有效减少要编写的代码，有助于降低代码的复杂度。 
      然而我们在使用匿名委托的时候我们要遵守 两个原则：1、匿名方法中不能有跳转语句(break, goto或continue)跳转到匿名方法的外部，反之，外部代码也不能跳转到该匿名方法内部。2、在匿名方法中不能访问不安全代码。 
注意：不能访问在匿名方法外部使用的ref和out参数。 
2、Lambda表达式

      由于Lambda表达式的出现使得我们的代码可以变得更加的简洁明了。我们现在来看看Lambda表达式的使用语法： 
[委托类] [委托对象名] = ( [参数列表] ) => { /*代码块*/ };      //结尾还是有一个分号 
       我们值得注意的是lambda表达式的参数列表，我们只需给出变量名即可，其余的编译器会自动和委托类进行匹配。如果委托类使用返回值的，那么代码块就需要return一个返回值。我们用一个例子来说明上述问题: 
Func<int, int> TestLam = (x) => { return x*x; }; 
       在这里我们是使用一个参数的为例，上面的写法是Lambda表达式的正常写法，但是当参数只有一个时，x两边的括号就可以去除，那么现在代码就变成这样了： 
Func<int, int> TestLam = x => { return x*x; }; 
        当Lambda表达式代码块中只有一条语句，那么我们就可以把花括号丢了。如果这一条语句还是包含return的语句，那么我们在去除花括号的同时，必须将return同时删去。现在上述代码就变成了这样： 
Func<int, int> TestLam = x => x*x; 
注意：Lambda表达式可以用于类型为委托的任意地方。 

3、闭包
       通过lambda表达式可以访问lambda表达式外部的变量，于是我们就引出了一个新的概念-----闭包。我们来看一个例子：
int someVal = 5;Func<int, int> f = x => x+someVal;
      现在我们很容易知道f(3)的返回值是8，我们继续： 
someVal = 7; 
       我们现在将someVal的值改为7，那么这时我们在调用f(3)，现在就会很神奇的发现f(3)的返回值变成了10。这就是闭包的 特点，这个特点在编程上很大程度上能给我们带来一定的好处。但是有利终有弊，如果我们使用不当，那么这就变成了一个非常危险的功能。 
       我现在再来看看在foreach语句中的闭包，我们现在看看下面这段代码：
List<int> values = new List<int>() { 10, 20, 30 };
var funcs = new List<Func<int>>();
foreach (var val in values)
{   funcs.Add(() => val);}
foreach (var f in funcs)
{   Console.WriteLine(f());}
        用我们刚才的知识来判断的话，输出结果应该是3个30。然而在C#4.0确实是这样，然而C#5.0会在foreach创建的while循环的代码块中创建一个不同的局部循环变量，所以这时在C#5.0中我们输出的结果应该是分别输出10,20和30。

    using UnityEngine;
    using System;//EventArgs需要
    using UnityEngine.Events;//Unity事件需要
    using UnityEngine.UI;//展示注册已有的UI事件
    public class Mom : MonoBehaviour {
       //本代码是以总结顺序写的，声明-定义-注册-调用。
       //用法1：一个对象可以一口气执行多个方法，并且可顺序执行其他类的方法，用于监听-执行
       //1.声明一个C#委托协定，指定方法签名
       public delegate int MyDelegate(int i, int j);
       //委托也是一个类型，需要声明对象
       MyDelegate myDelegate;
       //2.C#事件的声明：event+委托+事件名
       public event MyDelegate CSharpEvent;
     ###  //事件就是一个委托对象，而委托可以是一种类型  ###
       //3.C#自身提供了一种比较好的委托类型:EventHandler
       //sender表示事件源，e表示事件相关的信息
       public delegate void EventHandler(object sender, EventArgs e);
       EventHandler CSharpDelegateHandler;
       //4.另一种更好的委托方式是使用泛型参数的委托类型：EventHandler<TEventArgs>,其签名如下：
       //如果产生额外参数，第二个参数是从 EventArgs 派生的类型并提供所有字段或属性需要保存事件数据。使用 EventHandler<TEventArgs> 的优点在于，如果事件生成事件数据，则无需编写自己的自定义委托代码。
       public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);
       //5.我们也可以为这些委托声明事件（EventHandler事件处理器就是事件）
       public event EventHandler CSharpEventHandler;
       //6.使用UnityEvent(可视) 和 UnityAction(不可视)
       //使用类创建的话一条语句就等价于委托类的声明和委托对象的创建
       public UnityAction unityAction;//本质是delegate
       public UnityEvent unityEvent = new UnityEvent();//也可以在定义时new。本质上是继承自UnityEventBase的类
       
     //7.Unity泛型委托
       //因为UnityEvent<T0>是抽象类，所以需要声明一个类来继承它
       public class UnityActionWithParameter: UnityEvent<int>{
              //嵌套类
       }
       public UnityActionWithParameter unityActionWithParameter = new UnityActionWithParameter();
       public UnityAction<int> unityIntAction;
       public UnityEvent<int,int> unityIntEvent;
       //8.泛型委托Action<T>和Func<T>
       public Action<int, int> CSharpIntAction;//使用类创建的话一条语句就等价于委托类的声明和委托对象的创建
       public Func<int, int, int> CSharpIntFunc;//最后一个int是函数返回类型
       //注册的方法要求：返回类型+参数的类型+参数个数     参数名可以不同
       int Add(int a, int b)
       {
              return a + b;
       }
       int Subtarct(int a,int b)
       {
              return a - b;
       }
       void NoParameter()
       {
              //无参方法
       }
       void OneIntParameter(int i)
       {
              //一个int参数,且返回void
       }
       void TwoIntParameter(int a,int b)
       {
              //2个int参数,且返回void
       }
       //针对EventHandler的注册方法
       void OnGameStart(object sender, EventArgs e)
       {
              //你要执行的代码
       }
       //用法2：委托充当函数指针
       static void functionPointer(MyDelegate function,int a,int b)
       {
              function(a, b);
       }
       void Start()
       {
              //委托对象的初始化和赋值 和普通类一致。注意由于声明的委托不包含0参构造函数，所以必须要注册参数
              myDelegate =new MyDelegate(Add);
              //C#事件添加方法
              CSharpEvent += new MyDelegate(Add);
              //其实这几个无论用=，+=都是可以的
              CSharpDelegateHandler = new EventHandler(OnGameStart);
              //另外还有其他的注册方式，这里就拿C#标准委托的事件为例
              CSharpEventHandler += OnGameStart;
              CSharpEventHandler -= OnGameStart;//当监听到后不需要执行这个方法时（取消监听）
              CSharpEvent += Add;//同样可以
              Dad dad = new Dad();
              CSharpEvent += dad.Add;//如果我们想注册外部类的方法也是可以的（注意要public）
              unityAction = new UnityAction(NoParameter);
              unityAction += NoParameter;
              //注意UnityEvent只能用AddListener临时注册方法
              unityEvent.AddListener(NoParameter);
              unityEvent.RemoveListener(NoParameter);
              unityIntAction += OneIntParameter;//可以使用+=
              unityIntAction = new UnityAction<int>(OneIntParameter);//注册要求一个int参数,且返回void
              unityActionWithParameter.AddListener(OneIntParameter);//其实我不确定上面那个能不能用，实例中是只用了这行的事件的
              unityIntEvent.AddListener(TwoIntParameter);//注册要求2个int参数,且返回void
              CSharpIntAction += TwoIntParameter;//可以使用+=
              CSharpIntAction = new Action<int, int>(TwoIntParameter);
              CSharpIntFunc += Add;//要求返回int类型
              CSharpIntFunc = new Func<int, int, int>(Add);
              //9.lambda方式可以添加包含任意参数的函数，非常方便。以下是为onClick事件注册方法
              GetComponent<Button>().onClick.AddListener(() =>
              {//意思是用(参数列表)执行以下函数体的内容
               //你要执行的代码，也可以执行方法，可以使用参数
              });
              //以下是调用。一般放在监听成功的位置
              //C#委托的调用：委托对象及参数
              myDelegate(1, 2);
              myDelegate.Invoke(1,2);//调用与上面等价
              //C#事件的调用，同委托完全一样
              CSharpEvent(1, 2);
              CSharpEvent.Invoke(2, 4);
              //C#标准委托调用，注意不需任何参数时就这样
              if(CSharpDelegateHandler!=null)//可以加个安全检查
                      CSharpDelegateHandler(this, EventArgs.Empty);
              unityAction();//无参调用
              unityAction.Invoke();
              unityEvent.Invoke();//UnityEvent只能使用Invoke()调用
              unityIntAction(1);//1个int参数
              unityActionWithParameter.Invoke(1);//继承UnityAction<int>的事件
              unityIntEvent.Invoke(1,2);//在Invoke中提供实参
              CSharpIntAction(1, 2);//Invoke就省略不写了
              CSharpIntFunc(1, 2);
              //函数指针的使用
              functionPointer(myDelegate, 2, 4);//可以用已有的委托对象做参数
              functionPointer(new MyDelegate(Subtarct), 4, 2);//也可以new一个新对象
              //另外如果对象是public的那么我们也可以在外部类执行以上代码
       }
}
