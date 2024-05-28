Spring3引入了Spring表达式语言(Spring Expression Language，**SpEL**)。前面介绍的注入都是编写Spring配置文件的时候就确定了的。而SpEL它通过运行期间执行的表达式将值装配到Bean的属性或者构造器参数中。

SpEL有许多特性：

1.使用Bean的ID来引用Bean

2.调用方法和访问对象的属性

3.对值进行算数，关系和逻辑运算

4.正则表达式匹配

5.集合操作

## [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#SpEL%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8 "SpEL的基本使用")SpEL的基本使用

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E5%AD%97%E9%9D%A2%E5%80%BC "字面值")字面值

最简单的SpEL表达式仅包含一个字面值。假设现在某个类的count属性的值为5，我们用SpEL表达式表示：

|   |   |
|---|---|
|1|<property name="count" value="#{5}"/>|

`#{ }`符号会提醒Spring，里面的内容是一个SpEL表达式。它们还可以与非SpEL表达式混用：

|   |   |
|---|---|
|1|<property name="message" value="the value is #{5}"/>|

Java基本数据类型都可以出现在SpEL表达式中。表达式中的数字也可以使用科学记数法：

|   |   |
|---|---|
|1|<property name="salary" value="#{1e4}"/>|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E5%BC%95%E7%94%A8Bean-Properties%E5%92%8C%E6%96%B9%E6%B3%95 "引用Bean , Properties和方法")引用Bean , Properties和方法

SpEL表达式能够通过其他Bean的ID进行引用。如：

|   |   |
|---|---|
|1|<constructor-arg value="#{deceivedByLife}"/>|

其等价于：

|   |   |
|---|---|
|1|<constructor-arg ref="deceivedByLife"/>|

carl参赛者是一位模仿高手，kenny唱什么歌，弹奏什么乐器，他就唱什么歌，弹奏什么乐器：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7|<bean id="kenny" class="com.spring.entity.Instrumentalist"  <br>    p:song="May Rain"  <br>    p:instrument-ref="piano"/>  <br><bean id="carl" class="com.spring.entity.Instrumentalist">  <br>    <property name="instrument" value="#{kenny.instrument}"/>  <br>    <property name="song" value="#{kenny.song}"/>  <br></bean>|

As you can see，属性的值由key和value组成。key指定kenny`<bean>` 的id，value指定kenny`<bean>`的song属性。其等价于执行下面的代码：

|   |   |
|---|---|
|1  <br>2|Instrumentalist carl = new Instrumentalist();  <br>carl.setSong(kenny.getSong());|

除了访问类的属性，SpEL表达式还可以访问类的方法。假设现在有个SongSelector类，该类有个`selectSong()`方法，这样的话carl就可以不用模仿别人，开始唱songSelector所选的歌了：

|   |   |
|---|---|
|1|<property name="song" value="#{SongSelector.selectSong()}"/>|

carl有个癖好，歌曲名不是大写的他就浑身难受，我们现在要做的就是仅仅对返回的歌曲调用`toUpperCase()`方法：

|   |   |
|---|---|
|1|<property name="song" value="#{SongSelector.selectSong().toUpperCase()}"/>|

注意：这里我们不能确保不抛出`NullPointerException`，为了避免这个讨厌的问题，我们可以使用SpEL的`null-safe`存取器

|   |   |
|---|---|
|1|<property name="song" value="#{SongSelector.selectSong()?.toUpperCase()}"/>|

yes，就是这么简单。`?.`符号会确保左边的表达式不会为`null`，如果为`null`的话就不会调用`toUpperCase()`方法了。

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E6%93%8D%E4%BD%9C%E7%B1%BB "操作类")操作类

在SpEL表达式中，使用`T()`运算符会调用类的作用域和方法。例如，我们要调用`java.lang.Math`的PI值：

|   |   |
|---|---|
|1|<property name="pi" value="#{T(java.lang.Math).PI}"/>|

获取0~1随机数：

|   |   |
|---|---|
|1|<property name="random" value="#{T(java.lang.Math).random()}"/>|

## [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E5%9C%A8SpEL%E5%80%BC%E4%B8%8A%E6%89%A7%E8%A1%8C%E6%93%8D%E4%BD%9C "在SpEL值上执行操作")在SpEL值上执行操作

SpEL提供了几种运算符：

|运算符类型|运算符|
|---|---|
|算数运算|+, -, *, /, %, ^|
|关系运算|<, >, ==, <=, >=, lt, gt, eq, le, ge|
|逻辑运算|and, or, not, !|
|条件运算|?:(ternary), ?:(Elvis)|
|正则表达式|matches|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E7%AE%97%E6%95%B0%E8%BF%90%E7%AE%97 "算数运算")算数运算

加法运算：

|   |   |
|---|---|
|1|<property name="add" value="#{counter.total+42}"/>|

加号还可以用于字符串拼接：

|   |   |
|---|---|
|1|<property name="blogName" value="#{my blog name is+' '+mrBird }"/>|

`^`运算符执行幂运算，其余算数运算符和Java一毛一样，这里不再赘述。

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E5%85%B3%E7%B3%BB%E8%BF%90%E7%AE%97 "关系运算")关系运算

判断一个Bean的某个属性是否等于100：

|   |   |
|---|---|
|1|<property name="eq" value="#{counter.total==100}"/>|

返回值是boolean类型。关系运算符唯一需要注意的是：在Spring XML配置文件中直接写>=和<=会报错。因为这”<”和”>”两个符号在XML中有特殊的含义。所以实际使用时，最号使用文本类型代替符号：

|运算符|符号|文本类型|
|---|---|---|
|等于|==|eq|
|小于|<|lt|
|小于等于|<=|le|
|大于|>|gt|
|大于等于|>=|ge|

如：

|   |   |
|---|---|
|1|<property name="eq" value="#{counter.total le 100}"/>|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E9%80%BB%E8%BE%91%E8%BF%90%E7%AE%97 "逻辑运算")逻辑运算

SpEL表达式提供了多种逻辑运算符，其含义和Java也是一毛一样，只不过符号不一样罢了。

使用and运算符：

|   |   |
|---|---|
|1|<property name="largeCircle" value="#{shape.kind == 'circle' and shape.perimeter gt 10000}"/>|

两边为true时才返回true。

其余操作一样，只不过非运算有`not`和`!`两种符号可供选择。非运算：

|   |   |
|---|---|
|1|<property name="outOfStack" value="#{!product.available}"/>|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E6%9D%A1%E4%BB%B6%E8%BF%90%E7%AE%97 "条件运算")条件运算

条件运算符类似于Java的三目运算符：

|   |   |
|---|---|
|1  <br>2|<property name="instrument"  <br>        value="#{songSelector.selectSong() == 'May Rain' ? piano:saxphone}"/>|

当选择的歌曲为”May Rain”的时候，一个id为piano的Bean将装配到instrument属性中，否则一个id为saxophone的Bean将装配到instrument属性中。注意区别piano和字符串“piano”！

一个常见的三目运算符的使用场合是判断是否为null值：

|   |   |
|---|---|
|1|<property name="song" value="#{kenny.song !=null ? kenny.song:'Jingle Bells'}"/>|

这里，kenny.song引用重复了两次，SpEL提供了三目运算符的变体来简化表达式：

|   |   |
|---|---|
|1|<property name="song" value="#{kenny.song !=null ?:'Jingle Bells'}"/>|

在以上示例中，如果kenny.song不为null，那么表达式的求值结果是kenny.song否则就是“Jingle Bells”。

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F "正则表达式")正则表达式

验证邮箱：

|   |   |
|---|---|
|1  <br>2|<property name="email"  <br>        value="#{admin.email matches '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.com'}"/>|

虽然这个邮箱正则不够健壮，但对于演示matches来说足够啦。

## [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E5%9C%A8SpEL%E4%B8%AD%E7%AD%9B%E9%80%89%E9%9B%86%E5%90%88 "在SpEL中筛选集合")在SpEL中筛选集合

为了展示SpEL操作集合的方法，我们创建一个City类：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23|public class City {  <br>    private String name;  <br>    private String state;  <br>    private int population;  <br>    public String getName() {  <br>        return name;  <br>    }  <br>    public void setName(String name) {  <br>        this.name = name;  <br>    }  <br>    public String getState() {  <br>        return state;  <br>    }  <br>    public void setState(String state) {  <br>        this.state = state;  <br>    }  <br>    public int getPopulation() {  <br>        return population;  <br>    }  <br>    public void setPopulation(int population) {  <br>        this.population = population;  <br>    }  <br>}|

同样，我们在Spring容器中使用`<util:list>`元素配置一个包含City对象的List集合：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18|<util:list id="cities">  <br>    <bean class="com.spring.entity.City" p:name="Chicago"  <br>        p:state="IL" p:population="2853114"/>  <br>    <bean class="com.spring.entity.City" p:name="Atlanta"  <br>        p:state="GA" p:population="537958"/>  <br>    <bean class="com.spring.entity.City" p:name="Dallas"  <br>        p:state="TX" p:population="1279910"/>  <br>    <bean class="com.spring.entity.City" p:name="Houston"  <br>        p:state="TX" p:population="2242193"/>  <br>    <bean class="com.spring.entity.City" p:name="Odessa"  <br>        p:state="TX" p:population="90943"/>  <br>    <bean class="com.spring.entity.City" p:name="El Paso"  <br>        p:state="TX" p:population="613190"/>  <br>    <bean class="com.spring.entity.City" p:name="Jal"  <br>        p:state="NM" p:population="1996"/>  <br>    <bean class="com.spring.entity.City" p:name="Las Cruces"  <br>        p:state="NM" p:population="91865"/>  <br></util:list>|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E8%AE%BF%E9%97%AE%E9%9B%86%E5%90%88%E6%88%90%E5%91%98 "访问集合成员")访问集合成员

定义一个ChoseCity类：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9|public class ChoseCity {  <br>    private City city;  <br>    public void setCity(City city) {  <br>        this.city = city;  <br>    }  <br>    public City getCity() {  <br>        return city;  <br>    }  <br>}|

选取集合中的某一个成员，并赋值给city属性中：

|   |   |
|---|---|
|1  <br>2  <br>3|<bean id="choseCity" class="com.spring.entity.ChoseCity">  <br>    <property name="city" value="#{cities[0]}"/>  <br></bean>|

实例化这个Bean：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8|public class ChoseaCity {  <br>    public static void main(String[] args) {  <br>        String conf="applicationContext.xml";  <br>        ApplicationContext ac=new ClassPathXmlApplicationContext(conf);  <br>        ChoseCity c=(ChoseCity)ac.getBean("choseCity");  <br>        System.out.println(c.getCity().getName());  <br>    }  <br>}|

输出：

|   |   |
|---|---|
|1|Chicago|

随机的选择一个city：

|   |   |
|---|---|
|1  <br>2  <br>3|<bean id="choseCity" class="com.spring.entity.ChoseCity">  <br>    <property name="city" value="#{cities[T(java.lang.Math).random()*cities.size()]}"/>  <br></bean>|

中括号`[]`运算符始终通过索引访问集合中的成员。

`[]`运算符同样可以用来获取java.util.Map集合中的成员。例如，假设City对象以其名字作为键放入Map集合中，在这种情况下，我们可以像下面那样获取键为Dallas的entry：

|   |   |
|---|---|
|1|<property name="chosenCity" value="#{cities['Dallas']}"/>|

`[]`运算符的另一种用法是从java.util.Properties集合中取值。例如，假设我们需要通过`<util:properties>`元素在Spring中加载一个properties配置文件：

|   |   |
|---|---|
|1|<util:properties id="settings" loaction="classpath:settings.properties"/>|

现在要在这个配置文件Bean中访问一个名为twitter.accessToken的属性：

|   |   |
|---|---|
|1|<property name="accessToken" value="#{settings['twitter.accessToken']}"/>|

`[]`运算符同样可以通过索引来得到某个字符串的某个字符，例如下面的表达式将返回s：

|   |   |
|---|---|
|1|'This is a test'[3]|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E6%9F%A5%E8%AF%A2%E9%9B%86%E5%90%88%E6%88%90%E5%91%98 "查询集合成员")查询集合成员

如果我们想从城市cities集合中查询出人口大于10000的城市，在SpEL中只需要用一个查询运算符 `.?[]`就可以简单做到。修改choseCity类：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10|public class ChoseCity {  <br>    private List<City> city;  <br>      <br>    public List<City> getCity() {  <br>        return city;  <br>    }  <br>    public void setCity(List<City> city) {  <br>        this.city = city;  <br>    }  <br>}|

修改Bean：

|   |   |
|---|---|
|1  <br>2  <br>3|<bean id="choseCity" class="com.spring.entity.ChoseCity">  <br>    <property name="city" value="#{cities.?[population gt 100000]}"/>  <br></bean>|

实例化该Bean：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10|public class ChoseCitys {  <br>    public static void main(String[] args) {  <br>        String conf="applicationContext.xml";  <br>        ApplicationContext ac=new ClassPathXmlApplicationContext(conf);  <br>        ChoseCity c=(ChoseCity)ac.getBean("choseCity");  <br>        for(City city:c.getCity()){  <br>            System.out.println(city.getName());  <br>        }  <br>    }  <br>}|

输出：

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5|Chicago  <br>Atlanta  <br>Dallas  <br>Houston  <br>El Paso|

SpEL还提供了其他两种查询运算符：`.^[]`和`.$[]`，从集合查询中查出第一个匹配项和最后一个匹配项。例如查询第一个符合查询条件的城市：

|   |   |
|---|---|
|1|<property name="firstBigCity" value="#{cities.^[population gt 100000]}"/>|

### [](https://mrbird.cc/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E6%8A%95%E5%BD%B1%E9%9B%86%E5%90%88 "投影集合")投影集合

集合投影就是从集合的每一个成员中选择特定的属性放入到一个新的集合中。SpEL的投影运算符`.![]`完全可以做到这一点。

例如，我们仅需要包含城市名称的一个String类型的集合：

|   |   |
|---|---|
|1|<property name="cityNames" value="#{cities.![name]}"/>|

再比如，得到城市名字加州名的集合：

|   |   |
|---|---|
|1|<property name="cityNames" value="#{cities.![name+','+state]}"/>|

把符合条件的城市的名字和州名作为一个新的集合：

|   |   |
|---|---|
|1|<property name="cityNames" value="#{cities.?[population gt 100000].![name+','+state]}"/>|

总之SpEL非常强大，但SpEL最终也只是一个字符串，不易于测试，也没IDE语法检查的支持。所以：建议不要把过多的逻辑放入SpEL表达式中