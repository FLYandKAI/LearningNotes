# java8新特性（补充）



现在很多大型的互联网公司都对外提供了API服务，比如百度的地图，微博的新闻，天气预报等等。很少有网站或网络应用汇以完全隔离的方式工作，而是采用混聚的方式：它会使用来自多个源的内容，将这些内容聚合在一起，方便用户使用。

比如实现一个功能，你需要在微博中搜索某个新闻，然后根据当前坐标获取天气预报。这些调用第三方信息的时候，不想因为等待搜索新闻时，放弃对获取天气预报的处理，于是我们可以使用 分支/合并框架 及并行流 来并行处理，将他们切分为多个子操作，在多个不同的核、CPU甚至是机器上并行的执行这些子操作。

相反，如果你想实现并发，而不是并行，或者你的主要目标是在同一个CPU上执行几个松耦合的任务，充分利用CPU的核，让其足够忙碌，从而最大化程序的吞吐量，那么你其实真正想做的是避免因为等待远程服务的返回，或者对数据库的查询，而阻塞线程的执行，浪费宝贵的计算资源，因为这种等待时间可能会很长。**Future接口，尤其是它的新版实现CompletableFuture是处理这种情况的利器。**

## Stream流

举个例子，返回低热量的菜肴名称的， 并按照卡路里排序，一个是用Java 7写的，另一个是用Java 8的流写的。

比较一下。不用太担心 Java 8代码怎么写，我们在接下来的几节里会详细解释。

 之前（Java 7）： 

```
List<Dish> lowCaloricDishes = new ArrayList<>(); 
for(Dish d: menu){     
   if(d.getCalories() < 400){          
		lowCaloricDishes.add(d);     
		} 
    }
        Collections.sort(lowCaloricDishes, new Comparator<Dish>() {      
        public int compare(Dish d1, Dish d2){         
        return Integer.compare(d1.getCalories(), d2.getCalories());     
        } }); 
        List<String> lowCaloricDishesName = new ArrayList<>(); 
        for(Dish d: lowCaloricDishes){     
        lowCaloricDishesName.add(d.getName()); } 
```

在这段代码中，你用了一个“垃圾变量”lowCaloricDishes。它唯一的作用就是作为一次 性的中间容器。

在Java 8中，实现的细节被放在它本该归属的库里了。 之后（Java 8）： 

```
import static java.util.Comparator.comparing; 
import static java.util.stream.Collectors.toList;
List<String> lowCaloricDishesName = menu.stream()
										.filter(d -> d.getCalories() < 400)                     						                                                 .sorted(comparing(Dish::getCalories))                     
										.map(Dish::getName)                     
										.collect(toList());  
```

为了利用多核架构并行执行这段代码，你只需要把stream()换成parallelStream()： 

```
List<String> lowCaloricDishesName =menu.parallelStream()                    
									   .filter(d -> d.getCalories() < 400)        
									   .sorted(comparing(Dishes::getCalories))                    
									   .map(Dish::getName)                    
									   .collect(toList()); 
```

代码是以声明性方式写的：说明想要完成什么（筛选热量低的菜肴）而不是说明如何实现一个操作（利用循环和if条件等控制流语句）。这种方法加上行为参数化让你可以轻松应对变化的需求：你很容易再创建一个代码版本，利用 Lambda表达式来筛选高卡路里的菜肴，而用不着去复制粘贴代码

**流到底是什么呢？**

简短的定义就是“从支持数据处理操作的源生成的元素序列”。让 我们一步步剖析这个定义。

 元素序列——就像集合一样，流也提供了一个接口，可以访问特定元素类型的一组有序 值。因为集合是数据结构，所以它的主要目的是以特定的时间/空间复杂度存储和访问元 素（如ArrayList 与 LinkedList）。但流的目的在于表达计算，比如你前面见到的 filter、sorted和map。集合讲的是数据，流讲的是计算。我们会在后面几节中详细解 释这个思想。

 源——流会使用一个提供数据的源，如集合、数组或输入/输出资源。 请注意，从有序集 合生成流时会保留原有的顺序。由列表生成的流，其元素顺序与列表一致。

 数据处理操作——流的数据处理功能支持类似于数据库的操作，以及函数式编程语言中 的常用操作，如filter、map、reduce、find、match、sort等。流操作可以顺序执 行，也可并行执行。 此外，流操作有两个重要的特点。 

 流水线——很多流操作本身会返回一个流，这样多个操作就可以链接起来，形成一个大 的流水线。

 内部迭代——与使用迭代器显式迭代的集合不同，流的迭代操作是在背后进行的。

```
import static java.util.stream.Collectors.toList;
List<String> threeHighCaloricDishNames =   menu.stream()        
											   .filter(d -> d.getCalories() > 300)  
											   .map(Dish::getName)        
											   .limit(3)        
											   .collect(toList());   
System.out.println(threeHighCaloricDishNames); 
```

我们先是对menu调用stream方法，由菜单得到一个流。

数据源是菜肴列表（菜 单），它给流提供一个元素序列。接下来，对流应用一系列数据处理操作：filter、map、limit 和collect。除了collect之外，所有这些操作都会返回另一个流，这样它们就可以接成一条流 水线，于是就可以看作对源的一个查询。最后，collect操作开始处理流水线，并返回结果（它 和别的操作不一样，因为它返回的不是流，在这里是一个List）。在调用collect之前，没有任何结果产生，实际上根本就没有从menu里选择元素。你可以这么理解：链中的方法调用都在排队等待，直到调用collect

 filter——接受Lambda，从流中排除某些元素。在本例中，通过传递lambda d -> d.getCalories() > 300，选择出热量超过300卡路里的菜肴。 

 map——接受一个Lambda，将元素转换成其他形式或提取信息。在本例中，通过传递方 法引用Dish::getName，相当于Lambda d -> d.getName()，提取了每道菜的菜名。

```
一个非常常见的数据处理套路就是从某些对象中选择信息。比如在SQL里，你可以从表中选择一列。Stream API也通过map和flatMap方法提供了类似的工具。 
流支持map方法，它会接受一个函数作为参数。
这个函数会被应用到每个元素上，并将其映射成一个新的元素（使用映射一词，是因为它和转换类似，但其中的细微差别在于它是“创建一 个新版本”而不是去“修改”）。
例如，下面的代码把方法引用Dish::getName传给了map方法， 来提取流中菜肴的名称： 
List<String> dishNames = menu.stream()                              
							 .map(Dish::getName)                              
							 .collect(toList()); 
因为getName方法返回一个String，所以map方法输出的流的类型就是Stream<String>。
 让我们看一个稍微不同的例子来巩固一下对map的理解。
 给定一个单词列表，你想要返回另一个列表，显示每个单词中有几个字母。怎么做呢？
 你需要对列表中的每个元素应用一个函数。 这听起来正好该用map方法去做！应用的函数应该接受一个单词，并返回其长度。你可以像下面 这样，给map传递一个方法引用String::length来解决这个问题： 
List<String> words = Arrays.asList("Java 8", "Lambdas", "In", "Action"); 
List<Integer> wordLengths = words.stream()                                  
								 .map(String::length)                                  
								 .collect(toList()); 
现在让我们回到提取菜名的例子。如果你要找出每道菜的名称有多长，怎么做？你可以像下 面这样，再链接上一个map： 

List<Integer> dishNameLengths = menu.stream()                                     
									.map(Dish::getName)                                     	
									.map(String::length)                                     
									.collect(toList());
```

 limit——截断流，使其元素不超过给定数量。 

 collect——将流转换为其他形式。在本例中，流被转换为一个列表。它看起来有点儿 像变魔术，现在，你可以把collect看作能够接受各种方案作为参数，并将流中的元素累积成为一个汇总结果的操作。这里的 toList()就是将流转换为列表的方案。 

collectors

主要提供了三大功能：  将流元素归约和汇总为一个值 

​									     元素分组

 									    元素分区 

```
比如：
long howManyDishes = menu.stream().collect(Collectors.counting());
这还可以写得更为直接： 
long howManyDishes = menu.stream().count();
最大值和最小值
Comparator<Dish> dishCaloriesComparator =Comparator.comparingInt(Dish::getCalories); 
Optional<Dish> mostCalorieDish =menu.stream()         
									.collect(maxBy(dishCaloriesComparator)); 
例如，通过一次summarizing操作你可以就数出菜单中元素的个数，并得 到菜肴热量总和、平均值、大值和小值： 
IntSummaryStatistics menuStatistics =menu.stream().collect(summarizingInt(Dish::getCalories)); 
这个收集器会把所有这些信息收集到一个叫作IntSummaryStatistics的类里，它提供了 方便的取值（getter）方法来访问结果。打印menuStatisticobject会得到以下输出： 
IntSummaryStatistics{count=9, sum=4300, min=120,average=477.777778, max=800}
分组：
用Collectors.groupingBy工厂方法返回 的收集器就可以轻松地完成这项任务，如下所示： 
Map<Dish.Type, List<Dish>> dishesByType =menu.stream().collect(groupingBy(Dish::getType)); 
其结果是下面的Map： 
{FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza],  MEAT=[pork, beef, chicken]} 
但是，分类函数不一定像方法引用那样可用，因为你想用以分类的条件可能比简单的属性访问器要复杂。
例如，你可能想把热量不到400卡路里的菜划分为“低热量”（diet），热量400到700 卡路里的菜划为“普通”（normal），高于700卡路里的划为“高热量”（fat）。由于Dish类的作者 没有把这个操作写成一个方法，你无法使用方法引用，但你可以把这个逻辑写成Lambda表达式：
public enum CaloricLevel { DIET, NORMAL, FAT } 
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(         groupingBy(dish -> {               
		if (dish.getCalories() <= 400) 
				return CaloricLevel.DIET;                
		else if (dish.getCalories() <= 700) 
				return     CaloricLevel.NORMAL;         
		else return CaloricLevel.FAT;         
        } )); 
```



举例：

![image-20201130154150997](C:\Users\LangFo.zheng\AppData\Roaming\Typora\typora-user-images\image-20201130154150997.png)

companyAdList.stream()是把LIST变成流，Collectors.toMap是把流收集到一个`Map`实例，是把ID作为key，第二个参数就是map得value，第三个参数的作用是当出现一样的key值得时候如何取舍其中V1代表旧值，v2代表新值，示例中取旧值

companyAdList.stream()是把LIST变成流，map做一个映射，获得所有公司的ID，然后调用distinct来过滤重复的字符

![image-20201130162539565](C:\Users\LangFo.zheng\AppData\Roaming\Typora\typora-user-images\image-20201130162539565.png)

这里的filter是选出所有更新人ID都大于0的AdsRecommendVo实体类

## 归约

在我们研究如何使用reduce方法之前，先来看看如何使用for-each循环来对数字列表中的元素求和： 

```
int sum = 0; for (int x : numbers) {     sum += x; }
```

 numbers中的每个元素都用加法运算符反复迭代来得到结果。通过反复使用加法，你把一个数字列表归约成了一个数字。

这段代码中有两个参数： 

 总和变量的初始值，在这里是0； 

 将列表中所有元素结合在一起的操作，在这里是+。

 要是还能把所有的数字相乘，而不必去复制粘贴这段代码，岂不是很好？这正是reduce操 作的用武之地，它对这种重复应用的模式做了抽象。你可以像下面这样对流中所有的元素求和：

```
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
```

 reduce接受两个参数： 

 一个初始值，这里是0；

  一个BinaryOperator<T>来将两个元素结合起来产生一个新值，这里我们用的是 lambda (a, b) -> a + b。 你也很容易把所有的元素相乘，只需要将另一个Lambda：(a, b) -> a * b传递给reduce 操作就可以了： 

```
int product = numbers.stream().reduce(1, (a, b) -> a * b);
```

让我们深入研究一下reduce操作是如何对一个数字流求和的。首先，0作为Lambda（a）的 第一个参数，从流中获得4作为第二个参数（b）。0 + 4得到4，它成了新的累积值。然后再用累 积值和流中下一个元素5调用Lambda，产生新的累积值9。接下来，再用累积值和下一个元素3 调用Lambda，得到12。最后，用12和流中最后一个元素9调用Lambda，得到最终结果21。 

你可以使用方法引用让这段代码更简洁。在Java 8中，Integer类现在有了一个静态的sum 方法来对两个数求和，这恰好是我们想要的，用不着反复用Lambda写同一段代码了： 

```
int sum = numbers.stream().reduce(0, Integer::sum);
```

 无初始值 reduce还有一个重载的变体，它不接受初始值，但是会返回一个Optional对象： 

```
Optional<Integer> sum = numbers.stream().reduce((a, b) -> (a + b));
```

 为什么它返回一个Optional<Integer>呢？考虑流中没有任何元素的情况。reduce操作无 法返回其和，因为它没有初始值。这就是为什么结果被包裹在一个Optional对象里，以表明和 可能不存在。

代码清单5-1 找出2011年的所有交易并按交易额排序（从低到高）

```
 List<Transaction> tr2011 =     transactions.stream()                 
								            .filter(transaction -> transaction.getYear() == 2011)                 
											.sorted(comparing(Transaction::getValue))                 
											.collect(toList());
```

 交易员都在哪些不同的城市工作过 

```
List<String> cities =     transactions.stream()                 
									  .map(transaction -> transaction.getTrader().getCity())                 
									  .distinct()                 
									  .collect(toList()); 
```

有没有交易员是在米兰工作的 

```
boolean milanBased =     transactions.stream()                															  											 .anyMatch(transaction -> transaction.getTrader()                                           
									 .getCity()                                                    
									 .equals("Milan")); 
```

找到交易额小的交易 

```
Optional<Transaction> smallestTransaction =     transactions.stream()                 
															.reduce((t1, t2) -> t1.getValue() < t2.getValue() ? t1 : t2); 
```

## ava8中的字符串连接收集器

在JDK8中，可以采用函数式编程（使用 `Collectors.joining 收集器`）的方式对字符串进行更优雅的连接。`Collectors.joining 收集器` 支持灵活的参数配置，可以指定字符串连接时的 **分隔符**，**前缀** 和 **后缀** 字符串。

```
// 定义人名数组
final String[] names = {"Zebe", "Hebe", "Mary", "July", "David"};
Stream<String> stream1 = Stream.of(names);
Stream<String> stream2 = Stream.of(names);
Stream<String> stream3 = Stream.of(names);
// 拼接成 [x, y, z] 形式
String result1 = stream1.collect(Collectors.joining(", ", "[", "]"));
// 拼接成 x | y | z 形式
String result2 = stream2.collect(Collectors.joining(" | ", "", ""));
// 拼接成 x -> y -> z] 形式
String result3 = stream3.collect(Collectors.joining(" -> ", "", ""));
System.out.println(result1);
System.out.println(result2);
System.out.println(result3);
```

程序输出结果如下：

```text
[Zebe, Hebe, Mary, July, David]
Zebe | Hebe | Mary | July | David
Zebe -> Hebe -> Mary -> July -> David
```