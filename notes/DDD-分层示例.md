[[DDD]]
## 前置知识

![image-20250629094304545](d:\卡片笔记\想法\DDD-汽车目标管理.assets\image-20250629094304545.png)

**1.应用服务层** **api** **：**对应文件夹为**demo-api**，

demo-api打成jar包供微服务的客户端调用，该jar内定义了dubbo接口协议provider及相应的传输对象dto，provider的请求参数对象`xxxReq`以Req结尾，provider的返回对象为`Result`，Result的内部属性对象`xxxDTO`均以DTO结尾。

**2.应用服务层：**对应的文件夹为**demo-app，**

demo-app依赖demo-api并实现provider接口，dubbo接口provider的实现`providerimpl`、mq的消费者`consumer`、mione的`mischedule`定时任务的task接口及实现`taskimpl`均放在这个文件夹【task接口也是dubbo协议，但命名上区分一下】，demo-app主要负责获取输入，组装上下文，参数校验，调用领域层做业务处理，demo-app将dto转换为领域层的对象后再调用领域层service或gateway[当业务足够简单时可直接调用gateway]；应用服务层的provider、consumer、task之间禁止相互调用。  

**3.领域服务层：**对应的文件夹为**demo-domain**，

demo-domain主要是封装了核心业务逻辑，并通过领域服务`xxxService`向应用服务层输出能力，demo-domain是应用的核心，不依赖任何其他层次；demo-domain依赖的外部资源通过定义的`gateway`接口来获得，`gateway`接口的输入和输出均为领域层定义的对象或基本数据类型，通过`gateway`访问的资源包括(db,redis,rpc,http,search)，这样领域服务层即可以独立于底层数据源、独立于第三方依赖；领域服务service之间禁止相互调用；领域服务层作为核心，其他层次都会直接或间接的依赖领域服务层，异常错误码的定义可以放在该层次。

**4.基础设施层：**对应的文件夹为**demo-infrastructure**，

demo-infrastructure依赖demo-domain，通过`gatewayimpl`来实现demo-domain的`gateway`接口定义，`gateway`接口的输入输出参数均为领域层对象，

`gatewayimpl`的内部实现需要将领域对象转换为数据对象do【数据库的对象xxxDO均以DO结尾】后和数据库通信，数据对象do作为返回值时要先转换为领域对象后返回；当访问数据库的mapper的输入参数超过4个时可封装为`xxxMapperParam`对象作为输入；`gatewayimpl`和第三方通信的话就要将领域对象和第三方接口的输入输出参数进行转换；这样基础设施层就起到了防腐层的作用，底层数据源和外部依赖都需要通过gateway的转义处理，才能被上面的App层和Domain层使用；`gatewayimpl`之间也禁止调用。

再次强调 Repository 的接口是 在 Domain 层，但是实现类是在 Infrastructure 层。

**5.应用启动层：**对应的文件夹为**demo-server**，

为`main`函数的入口，负责dubbo服务的启动，依赖demo-app，demo-infrastructure。