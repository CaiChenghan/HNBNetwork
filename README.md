## iOS网络层设计

> ### 总体思想：提供稳定的数据服务：稳定的网络连接+稳定的数据接口。
- 稳定的网络连接：网络连接尽量做到简洁、独立。
- 稳定的数据接口：脱离业务，能提供接口相匹配的数据。

> ### 总层次：连接层+扩展层+业务层
- 连接层：AFNetworking、Socket、NSURLRequest或者第三方SDK封装的网络请求。
- 扩展层：基于连接层向上的封装，主要用于功能的扩展。例如缓存、弹窗、数据转换、网络监测等。
- 业务层：业务接口层+业务数据层
- 业务接口层：可根据业务的对象、功能模块划分不同的业务接口，防止接口爆炸及方便接口管理。为链接层的向上封装，可采用类的继承或者类的扩展来实现，需封装基础类。
- 业务数据层：可与接口层对应，也可独立作用于具体业务逻辑，提供业务相匹配的逻辑数据。

> ### 缓存
- 对api接口进行数据缓存。主要为缓存周期、缓存策略、缓存清理以及缓存读取。缓存周期可设定为7天，超过7天的缓存不予读取。缓存策略可为执行缓存和不执行缓存。可自行检查清理超过期限的缓存。对于缓存数据是否可用，例如接口返回数据结构的改变，可用缓存版本号来进行判断。

> ### 弹窗
- 可接入MBProgressHUD或者自定义弹窗，弹窗策略可分为自行弹窗、弹窗和不弹窗。自行弹窗可根据网络环境自行处理，例如保存接口返回时间，根据接口返回时间自行决定是否执行弹窗。

> ### 数据转换
- 数据转换，包括字典转模型，字典转JSON，用于返回数据格式控制。例如期待返回对象模型，则只需要传入模型class即可，否则传nil。

> ### 网络监测
- 加入网络自行监测机制，用于判断网络请求的执行与否，弱化网络请求超时、无网络判定。

> ### 关于iOS网络层优化问题
- 目前对于iOS网络层的优化，基本是围绕接口请求并发数进行处理的，主要为减少同时进行的网络请求数量，同时释放无用的网络请求。
- iOS网络并发数问题：对于同一host，**最大并发数默认为4**，可查看HTTPMaximumConnectionsPerHost字段。经本人测试下载，发现同一个域名下，正在下载的网络连接数，不会超过4个。对于新加入的域名文件，也会执行网络连接，但相同域名下连接同样不会超过4个。即使请求方法执行了，也不会发出任何请求。
- 请求释放问题：对于iOS的请求，在请求发起后，即使请求使用者销毁了，请求也不会销毁，除非使用者主动取消了请求。

> ### 请求的自动释放：
- 主要思路为：请求对象销毁，释放该对象的所有请求。因此，我们只需要设计一个请求自动释放类即可。故请求在设计的时候，需要带入请求自动释放类。而这个请求自动释放类类，可以通过runtime绑定到objc上，也就是请求对象上。这样，在创建请求的时候，就将请求对象的请求，“绑定”到请求自动释放类上，进而释放请求。相当于一个桥，一边连接着请求对象，一边连接着对象所持有的请求，调用者走了，桥就处理调用者使用过的请求。

---
## MyNetwork

> ### 主要文件为如下
- 连接层：MyAFNetwork
- 扩展层：MyRequestManager
- 业务接口层：MyDataFetcher
- 业务数据层：MyDataHandler

> ### 使用：
- 数据请求业务，需通过MyDataFetcher扩展具体的业务api。如果有特殊的数据处理，可在MyDataHandler中处理，默认不做处理。具体可参考MyDataFetcher头文件方法。


