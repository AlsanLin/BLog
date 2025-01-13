## Flutter状态管理-GetX

### 目录
#### 1. [Flutter状态管理篇-GetX](https://pub.dev/packages/get)
#### 2. [Flutter状态管理篇-MobX](https://pub.dev/packages/mobx)
#### 3. [Flutter状态管理篇-BloC](https://pub.dev/packages/flutter_bloc)
#### 4. [Flutter状态管理篇-Redux](https://pub.dev/packages/flutter_redux)
#### 5. [Flutter状态管理篇-Provider](https://pub.dev/packages/provider)

### 安装

在pubspec.yaml添加依赖如下：

```
dependencies:
    get: 4.6.6
```

然后在dart文件导入相关的包

```
import 'package:get/get.dart';

```

### 架构

GetX使用的是MVC架构，使用示例如下：

```
//controller 类继承自 GetxController
class HomeController extends GetxController {
    var title = '';
}

//view 视图类，主要负责界面布局
class HomePage extends GetView<HomeController> {
 @override
  Widget build(BuildContext context) {
    return MainLayout(
      appBar: AppBar(
        title: Text(controller.title),
        centerTitle: true,
      ),
    )
  }
}

```

### 状态管理

GetX 的使用很简单，例如，你可直接定义一个变量，然后在其默认值后而添加 .obs，此时这就变成了一个 GetX 变量, 如：

```
var isEnabled = false.obs;

...

//改变其值
isEnabled.value = true;

```

然后在界面布局中使用Obx(() => )以监测其改动

```
return Scaffold(
      appBar: AppBar(
        title: const Text('Test'),
        centerTitle: true,
      ),
      body: Stack(
        children: [
          Obx(
            () => controller.isEnabled.value 
                ? ListView(
                    children: <Widget>[
                     ...
                    ],
                  )
                : const SizedBox.shrink(),
          ),
        ],
      ), 
    );
  }
```

### 路由管理

```
//导航到下一页
Get.to(NextPage());

//根据路由名称进行导航
Get.tonamed('/next');

//返回到前一页，或者关闭如 snackbar, dialog 之类
Get.back();

//进入到下一页，但不提供返回上一页的按钮，一般可用在启动页或者登入页
Get.off(NextPage());

//进入下一页同时取消所有之前的路由，通常可用在购物车、投票或者测试等页面
Get.offAll(NextPage());

//导航接收返回参数
Get.toNamed('/next', arguments: {"xxx_key":'xxx_value'})?.then((value) {
	//返回参数
	print('select code is : $value');

});

//导航接收传递参数
Get.arguments["xxx_key"]

//如果是栈内简单返回
Get.back()

//如果是栈内多级返回
Get.until((route) => Get.currentRoute == Routes.MAIN);

//如果是栈与栈的返回
Get.offNamed("/xxxPage"); or Get.offAllNamed("/xxxPage");
```

### Bindings使用
	
Bindings 主要配合 GetX 路由和依赖一起使用，作用是在路由跳转页面加载时注入当前页面所需的依赖关系。Bindings的好处是能统一管理页面的依赖关系，当业务复杂时可能一个页面需要注入大量的依赖，此时使用 Bindings 能更方便的维护页面的依赖关系。
	
1. **Bindings创建**：创建一个自定义的Bindings继承自Bindings，如下：
	
	```
	class CounterBinding extends Bindings {
	  @override
	  void dependencies() {
		Get.lazyPut(() => CounterController());
	  }
	
	
2. **如何使用**
	
使用分**普通路由**和**别名路由**方式
	
普通路由使用只需要在路由跳转时，将binding参数传入需要的自定义Bindings对象即可：
	
	```
	Get.to(CounterPage(), binding: CounterBinding());

	Get.off(CounterPage(), binding: CounterBinding());

	Get.offAll(CounterPage(), binding: CounterBinding());
	```
	
	别名路由需要先创建GetPage，确定别名与页面关系，并配置到GetMaterialApp的getPages中，通过使用Get.toNamed进行路由跳转，bindings在GetPage创建时传入，如下：
	
	```
	GetPage(name: "/counter", page: () => CounterPage(), binding: CounterBinding());
	```
	
	跳转时只需要使用路由别名即可，如下：
	
	```
	Get.toNamed("/counter");
	```
	
	别名路由相对于普通路由还有一个区别，可以传入bindings数组，这样就可以解决ViewPager等嵌套组件无法自动调用Bindings的问题，使用如下：
	
	```
	GetPage(
	  name: "/counter",
	  page: () => CounterPage(),
	  binding: CounterBinding(),
	  bindings: [PageABinding(), PageBBinding(), PageCBinding()]
	);
	```
	
	最后，在组件中就可以通过Get.find()找到对应的实例，如下：
	
	```
	CounterController controller = Get.find();
	```

### GetxService服务使用


GetX 的服务可以很方便地让你创建全局的通用功能函数，你可以在控制器里去处理 onInit(), onReady(), onClose() 这几个事件，同时你也可以在任何地方使用 Get.find() 获取到所要的服务。如下例子，创建一个存储器的服务以处理 SharedPreferences 相关逻辑
	
	```
	//创建一个服务并继承自 GetxService
	class LocalStorageService extends GetxService {
	Future<T?> getValue<T>(
		LocalStorageKeys key, [
		T Function(Map<String, dynamic>)? fromJson,
	  ]) async {
		SharedPreferences prefs = await SharedPreferences.getInstance();
		switch (T) {
		  case int:
			return prefs.getInt(key.name) as T?;
		  case double:
			return prefs.getDouble(key.name) as T?;
		  case String:
			return prefs.getString(key.name) as T?;
		  case bool:
			return prefs.getBool(key.name) as T?;
		  default:
			assert(fromJson != null, 'fromJson must be provided for Object values');
			if (fromJson != null) {
			  final stringObject = prefs.getString(key.name);
			  if (stringObject == null) return null;
			  final jsonObject = jsonDecode(stringObject) as Map<String, dynamic>;
			  return fromJson(jsonObject);
			}
		}
		return null;
	  }

	  void setValue<T>(LocalStorageKeys key, T value) async {
		SharedPreferences prefs = await SharedPreferences.getInstance();
		switch (T) {
		  case int:
			prefs.setInt(key.name, value as int);
			break;
		  case double:
			prefs.setDouble(key.name, value as double);
			break;
		  case String:
			prefs.setString(key.name, value as String);
			break;
		  case bool:
			prefs.setBool(key.name, value as bool);
			break;
		  default:
			assert(
			  value is Map<String, dynamic>,
			  'value must be int, double, String, bool or Map<String, dynamic>',
			);
			final stringObject = jsonEncode(value);
			prefs.setString(key.name, stringObject);
		}
	  }
	}

	```
	
	
然后，在main.dart中初始化服务，如下：
	
	
	```
	Get.put(LocalStorageService());

	```
	
	
之后，就可以在任何地方获取和使用了
	
	
	```
	//获取服务 
	var localStorage = Get.find<LocalStorageService>();

	...

	//保存值
	localStorage.setValue<String>('currentLanguage', 'en');

	//获取值
	var currentLanguage =
		  await localStorage.getValue<String>('currentLanguage');

	```



### 高级API使用

```
// 获取当前页面中的参数，通常是前一页传递过来的路由里的参数
Get.arguments

// 前一页路由的名字
Get.previousRoute

// 获取原始路由， 如 rawRoute.isFirst()
Get.rawRoute

// 访问路由的 API
Get.routing

// 判断 snackbar 是否打开
Get.isSnackbarOpen

// 判断对话框是否打开
Get.isDialogOpen

// 判断 bottomsheet 是否打开
Get.isBottomSheetOpen

// 判断当前使用的是哪个平台
GetPlatform.isAndroid
GetPlatform.isIOS
GetPlatform.isMacOS
GetPlatform.isWindows
GetPlatform.isLinux
GetPlatform.isFuchsia

// 判断设备型号
GetPlatform.isMobile
GetPlatform.isDesktop
GetPlatform.isWeb


// 相当于: MediaQuery.of(context).size.height,
// 不可修改的
Get.height
Get.width

// 当前导航的上下文
Get.context

// 对于 snackbar/dialog/bottomsheet 的上下文，全局可用
Get.contextOverlay

...

```