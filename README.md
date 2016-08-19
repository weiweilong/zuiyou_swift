# zuiyou_swift
swift3.0 仿最右

开发环境XCode8 beta2
iOS10 beta5

由于swift3.0 需要XCode8支持，现在还未发布正式版本，所以用的都是deta版

从新建工程到现在，先说说swift3.0中的坑。

首先 因为现在项目大多用到了第三方类库，有些是还不支持swift的，所以必须用到oc和swift混编。
新建swift工程

1、手动拖入第三方类库，在拖入工程中时(第一次创建oc文件)，XCode会提示是否创建桥接文件，点击确定，在工程中会生成一个以工程名开头的Bridging-Header.h文件，
  在Build Settings 查看Objective-C Bridging Header是否配置正确，工程名+文件名，在创建好桥接文件后，需要在swift用OC的类，只需要在桥接文件中导入即可
  OC中的方法在swift中同样能使用，按照swift的语法调用就可以了

2、使用pod导入安装第三方类库。配置好podfile，install就好了，使用时导入框架的头文件就行，比如“import AFNetworking”，但是注意每次重新编辑podfile执行pod install之后，需要配置每个类库对应targets的签名文件，因为XCode8中有Automatically manage siging自动配置签名文件这个功能了，只需要选择Team的账户就可以了，这确实解决了以前证书的麻烦。

我是用的是pod，下面的问题是在使用pod导入类库遇到的，使用手动拖入可能会有不同问题。以pod为例。
  
  OC和swift混编环境就配置好了，可以开始使用swift了
  
  swift2中的有一些方法在swift3不能用或者做了修改
  
  在swift中导入AFNetworking，没有这个AFHTTPSessionManager.manager这个方法，需要自己创建单例。
  首先子类化一个RequestClient，继承AFHTTPSessionManager。然后实现单例，但是问题来了。
  1、以前使用dispatch_once实现单例，现在这个方法没有了，不能用了
  
  之前：class var sharedInstance :RequestClient {
          struct Static {
              static var onceToken: dispatch_once_t = 0
              static var instance:RequestClient? = nil
              
              static let onceToken = RequestClient()
          }
          
          dispatch_once(&Static.onceToken, { () -> Void in
              //string填写相应的baseUrl即可
              var url:NSURL = NSURL(string: "")!
              Static.instance = RequestClient(baseURL: url)
          })
          //返回本类的一个实例
          return Static.instance!

        }
    
    现在：static let sharedInstance: RequestClient = {
            let instance = RequestClient()
            // setup code
            return instance
          }()

定义为static就可以了

2、在进行完数据请求后，将json转换成model对象，开始打算使用ObjectMapper的，但是在导入ObjectMapper (1.4.0)后发现还不支持3.0，很多错误，就使用了之前封装好的OC类，然后进行混编。

3、字符串截取操作
        let index = text.index(text.startIndex, offsetBy: 8)
        let begStr = text.substring(to: index)

4、对未知类型的判断，之前直接 object != nil 就行，需要对类使用? 修饰


暂时碰到这些坑，欢迎补充！
