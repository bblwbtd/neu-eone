# 东北大学一网通办爬虫
## 背景
&emsp;&emsp;2019年初，[东北大学](https://www.neu.edu.cn/)为了解决各个学校系统相互独立、账号分散容易忘记的情况，推出了[东北大学一网通办系统](https://eone.neu.edu.cn/).只要有这个系统的密码，就可以直接登陆教务处、图书馆、校园卡服务中心、学生邮箱、BB系统。此爬虫就是利用了这个特点实现的，在此之前，如东大方圆这样的微信小程序必须同时绑定多个账号才能使用全部功能，现在理论上可以只绑定一网通办就能实现大部分功能（不能实现的如：数学预约、考试系统），并且一些技术先进团队如weNEU已经使用这套**方案**（指使用一网通办进行各平台登录）进行信息的获取.
## 系统要求
* 安装Python3.0+
* 安装了Python的[requests](https://2.python-requests.org/en/master/)第三方库
* 能够访问互联网（无需教育网）
## 进展
已成功实现以下功能
* [东北大学图书馆](http://202.118.8.7:8991/F/)
  * 查询当前借阅书籍
  * 查询历史借阅书籍
  * 续借
  * 图书馆馆藏查询
* [东北大学教务处（新）](http://219.216.96.4/eams/homeExt.action)
  * 查课表
  * 查询个人信息
  * 查询课程信息
  * 查询成绩（暂不支持查询GPA）
* [东北大学一网通办中心](https://portal.neu.edu.cn/)
  * 校园网使用情况
  * 校园卡余额、补助余额
  * 邮箱邮件数量
  * 报销情况
* [东北大学校园卡服务中心](http://ecard.neu.edu.cn)
  * 卡状态（是否丢失）
  * 门禁记录
  * 消费记录
  * 挂失卡
## 未来要做的
* 支持卡挂失功能
* 完善文档
* 根据可靠消息，数学机考预约系统将支持一网通办，支持后将加入相关功能
* 由于目前教务处正进行数据迁移，在迁移完成后，将提供GPA查询，空教室查询（如果提供）
## 使用
在您的项目使用`import nespider`引入模块后即可使用，在这个模块中，提供了一个NeuStu类，通过这个类您可以实现获取各个系统的信息并进行简单操作，另外，对于与用户无关的操作如图书馆馆藏查询，本模块提供了LibrarySearch类。  
### 实例化
实例化NeuStu类需要传入两个参数：
* 一网通办账号（学生证号）
* 一网通办密码（默认为身份证/护照后六位）
例如用户账号为`20190000`，密码为`a123456` 。如果你想实例一个名为user的对象你需要使用以下语句：  
`user = NeuStu('20190000','a123456')`  
这时你便可以得到一个NeuStu的实例.  

实例化LibrarySearch类只需要传入一个参数，即书籍名称,然后使用get_books(page)方法进行查询,page代表查询的页数，每页十个结果。如果你想知道一共有多少个结果，只需要获取sum这个成员的值即可。下面是一段代码实例
```
# 初始化搜索引擎，搜索与‘java’有关的书籍  
books = LibrarySearch('java')  
# 打印搜索到的书籍数  
print(books.sum)  
# 获取打印第二页的内容  
print(books.get_books(2))  
```
***
### NeuStu类实例的成员方法与成员变量
对于每一个NeuStu实例，本模块提供了以下实例方法与变量：
* **card_info**  
学生卡信息 这是一个字典对象，*card_balance*代表余额，*sub_card_balance*代表补助余额
* **net_info**  
校园网使用信息 这是一个字典对象，*sum_bytes*代表本月使用的总流量（单位：B），*sum_seconds*代表使用总时长（单位：秒），*sum_times*代表连接次数（存疑），*user_balance*校园网账号余额（单位：元），*user_charge*猜测为本月消费
* **email_info**  
这是一个字典对象 *mbox_msgcnt*代表邮箱中的邮件总数。*mbox_msgsize*代表邮箱容量（存疑），*mbox_newmsgcnt*、*mbox_newmsgsize*代表的数值暂不清楚
* **finance_info**  
理论应返回字典对象 目前可能由于学校原因不能使用，不过这项似乎代表了财政报销信息
* **mobile_info**  
这是一个字典对象 *EMAIL*、*ID_NUMBER*、*MOBILE*分别代表了该一网通办账号所绑定的邮箱、学号、手机号。
* **is_card_lost**  
这是一个布尔值 为True代表校园卡已经挂失，为False代表校园卡出于正常状态
* **user_info**  
这是一个字典对象 内容包含了从新教务处所获得的大部分信息，每个键的意义容易理解，在此先不做说明
* **library_info**  
这是一个字典对象 其下正常应包括两个键**extend_url**、**book_data**，前一个键所对应的值代表了续借链接（字符串），通过该链接与书籍代码相连可以实现续借，**book_data**所对应的值是一个数组，数组的每个元素都是字典并代表了当前借阅的所有书籍，对于每本书对应字典每个键的意义，容易字面理解，在此先不做说明
* **get_course(semester)**  
此方法获取一个semester值，此值代表了所查询课表的学期，如2019年春季学期的学期代码为30.方法将返回一个列表，代表了本学期所有课程以及其相关信息如：上课教室，上课周，上课时间，任课教师
* **get_card_trade(start_date, end_date)**  
此方法获取两个参数，分别代表了查询起始日期与结束日期，如你要查询2019年5月1日至2019年5月10日则你需要使用`user.get_card_trade('2019-05-01','2019-05-10')`，此方法将返回一个列表记录交易记录。 
**注意**：使用此方法查询时间过长或交易量过多将导致响应时间过长，请谨慎使用，或自行使用多线程进行封装（为了学校服务器考虑，强烈不建议使用多线程）
* **get_door_info(start_date, end_date)**  
使用方法与 **get_card_trade(start_date, end_date)**  
基本相同，也要注意同样的问题
* **librar_continue_borrow(book_list)**  
此方法接受一个列表，列表中是所借书籍的代码，即**library_info**->**book_data**->**book_code**所对应的值（字符串），如果允许将会对列表中的书籍进行借阅，但由于没有足够的样本，还不能判断是否成功续借，但如果图书馆允许，即可成功借阅
* **lost_card**  
此功能待开发，用于挂失校园卡
***
## 备注
* 你可能注意到，各个域名下的cookies并没有使用双下划线开头命名，这意味这这些cookies是公有的，这方便您传递给其他服务端或客户端，进行更高效率的访问.
* 引入此模块时要注意，这个模块名并非neuspider而是nespider，虽然我是作者，但是我也不知道为什么我要这么做.当然你可以自行修改，以适应你的项目.
## 注意事项
* 如果你想使用此爬虫来实现类似东大方圆、东大葫芦、weNeu这样的信息聚合平台，请尽量使用代理访问服务器，因为一网通办的登陆记录（含IP），都将无一遗漏地记录在用户登录日志中，用户读取这些日志便可得知登录IP，这对于服务器拥有者可能是危险的.即使日志是可删除的，但只能全部清空，我强烈不建议你每次登录成功都清除登录记录，这可能会给你的用户带来困扰.
* 接上条，即使你不想暴露自己的IP清除了一网通办的登录记录，但教务处仍然会有不可清除的登录记录.
* 东北大学的大部分子域名并没有提供`robots.txt`来规定网站访问，所有我们无从得知学校的爬虫政策.对于爬虫的合法性无法做出准确判定.

## 开源协议
**MIT**  
这意味着您可以尽情享受开源的乐趣，但作者对他人使用本爬虫所造成的一切后果不负责任
