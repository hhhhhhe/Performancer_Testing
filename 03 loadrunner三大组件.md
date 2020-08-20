## LoadRunner三大组件

### 脚本用户脚本（Virtual user generator）:录制、调试脚本

> - 测试人员被LoadRunner的Vuser（虚拟用户）代替，测试人员执行的操作以Vuser Script（虚拟用户脚本）的方式固定下来。
> - 一条计算机可以运行多个Vuser，因此LoadRunner又减少了性能测试对硬件的要求。
> - Vuser在方案中执行的操作是用Vuser脚本描述的。运行场景时，每个Vuser去执行Vuser脚本。
> - Vuser脚本记录了用户的动作，并且包含一系列度量并记录服务器性能的函数，从而方便计算性能指标。
> - 这就像一个真实的用户一边做操作，一边拿着秒表记录时间一样。

### 控制台（controller）:设置场景参数，管理虚拟用户
> - 是运行性能测试的司令部，Controller负责生成性能测试场景，管理和协调多个虚拟用户，在实际运行时，Controller运行任务分派给各个Load generator，同时还联机监测软件系统各个节点的性能，并收集结果数据，提供给LoadRunner的Analysis.
1. ①Load generator:就是pc，要利用各pc机的资源，比如一台虚拟机可以模拟出的用户数为1000人，若要对5000人进行在线测试，则可以将其他的pc联入，输入其ip地址即可。
在Controller中，”Scenario Scripts”的Load Generators中点击”Add”，输入ip地址；然后点击工具栏的load generator，点击connect进行连接，即可利用该物理机的资源。（新物理机要有load generator软件）
Load generator，通俗来讲，是controller的“手下”，Controller发号命令，Load generator负责实施执行。通常在一台机器上安装了LoadRunner后，就自动安装了Load generator，而一个Controller可以控制多态机器上的Load generator，让他们同意听从指挥，共同完成任务。
2. ②代理程序（Agent）:部署在各个客户端，协同得到步调一致的虚拟用户
在load generator中，我们知道Controller可以向它发布命令，各物理机要能听到，就用的是代理程序，所以要启动该程序。
agent负责实时侦听来自控制器的指令，以达到协调各压力生成器中虚拟用户的作用
3. ③在做联机测试时，联机的机器要满足两个条件：
> - 1）安装load generator
> - 2）启动agent：所有程序—>HP LoadRunner—>Advanced Settings—>LoadRunner Agent Process
### 结果分析器（analysis）：生成测试报告
> - ④监控器：在性能测试过程中，要监控所有的服务器的重要资源。
> - ⑤ 以管理员身份打开Controller后，有Select Scenario Type
> - ①Manual Scenario Type手动设置场景（create Vuser groups 、specify the scripts、load generators、number of Vusers）
–Use the Percentage mode…:定义虚拟用户总数，为每个脚本分配一定比例的虚拟用户。比如：虚拟用户总数为20，有两个脚本001_login 40% 和002_lookFlight 60%（若修改其中一个比例，另一个比例会自动1-这个比例）
因为Controller启动缓慢， 若要进行模式转化，则Senario–>Convert Scenario to the Vuser Group Mode 再将脚本引入即可。
> - –若不选择上面的，则会按个数，比如：虚拟用户总数为20，有两个脚本001_login 8 和002_lookFlight 16（若修改其中一个个数，另一个个数会自动更改）
> - –企业中，一般的并发测试达到几百用户居多，所以百分比用的较少
> - ②Global-Oriented Scenario:定义一个在测试需要实现的目标，lr会自动建立场景。这种方式会隐式自动设置一些内容，所以运行过程容易出错，出错时还得自己查找错误，不如手动设置方便。
> - 1）VuGen对AUT进行捕捉和录制（选择正确的协议，模拟Java客户端或ie客户端），形成脚本。对于脚本可以在run-time Settings中进行设置（比如action循环执行多次），进而形成场景
> - 2）Controller，对VU进行部署（schedules)，连同场景，形成各种测试场景（性能测试策略，如基准测试）。场景可以启动或者停止，包括对load generator的控制，还可以在测试过程中，对AUT的服务器进行监控。
> - 3）测试过程中，形成的海量数据，在测试结束后，同一提交给Analysis中，形成各式图表。

## 二、LoadRunner的工作原理

录制时，LoadRunner记录下客户端和服务器两者之间的对话
回放时，LoadRunner模拟真正的客户端，向服务器发出请求，并根据脚本来验证服务器的应答。

## 三、测试基本流程

lr自带的tours系统：jojo/bean
一般情况下，在录制脚本之前，登录录制到init中，关心的操作录制在action中，退出录制到end。
vuser_init:虚拟用户的初始化函数，一般将用户初始化的操作放在这里，如登录操作、分配内存等。
Action：是虚拟用户要做的业务。用户的业务操作，也就是测试内容的主体。在VU里设置迭代循环选项时，只有Action有效，被重复运行，而init和end部分则在脚本的运行过程中只会执行一次。
vuser_end:与vuser_init相对应，vuser_end做收尾工作。在vuser_init中如果是登录，vuser_end里就是退出登录；在vuser_init中如果是申请内存，比如使用malloc函数，在vuser_end中就应该释放内存，使用free函数。
问题：录制登录，应该将录制放在action中
> - 1、点击“new”，在“new Virtual User”中选择Web[HTTP/HTML]，然后点击”Create”
> - 2、点击“start recording”按钮，在弹出的窗口中将URL Address:写上
http://localhost:1080/WebTours/ ，也可以修改录制到哪个action中（暂时用不到），点击“OK”
> - 3、开始录制。若要开始事务点击“insert start transaction”，然后输入新事务名称；若要结束事务，则要点击“insert end transaction”，输入要结束的事务名称。
（在此过程中，可以调整录入代码的页面Action、vsuer_init、vuser_end）
在此实例中，将登陆过程录在action中，将退出系统录在vsuer_end中
> - 4、查看报告，Tools–>General Optional–>Replay–>After Replay（选择visual test result ）
在结果报告中会有前面带有√、X、！的，若全要查看这些快照，则View–>Expand All
，当再次点击这些标识时，就会有快照显示
## 四、LoadRunner的脚本中，可以调用三种函数：

VU通用函数，一般以lr开头，比如lr_start_transaction
协议有关函数，不同类型的Vuser的函数一般以本协议类型开头。如web（HTTP/HTML）类型的，web_url就是一个协议函数，web前缀说明它属于Web HTTP协议的。
语言相关函数。Vu脚本是用C语言写的，那么C语言的标准函数或dll都可以在这里被加载和使用。
四、注意lr的脚本运行得出的result中全部为pass时，不一定证明脚本正确，因为lr只是在网络层面上验证了服务器收到了客户端发送的数据包并返回，但是，返回的应答中数据是否正确（应答页面是否正确）没有验证，这些检查，要在以后的检查点中进行学习。