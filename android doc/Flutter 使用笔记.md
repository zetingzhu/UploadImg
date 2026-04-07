Flutter 使用笔记

1. 同步项目环境
```
flutter pub get
```

2. 清除缓存
```
flutter clean
```

3. 诊断开发环境
```
flutter doctor
```
4. 诊断和修复包管理系统的缓存问题
```
   flutter pub cache repair
```

5. 彻底清除包缓存
```
   flutter pub cache clea
```



#  Json 序列化

## 运行代码生成命令
在你的项目根目录下，打开终端，运行以下命令：
```
flutter pub run build_runner build
```

## 持续监听文件修改（可选但推荐）
```
flutter pub run build_runner watch
```

## 执行命令
```
dart run build_runner build --delete-conflicting-outputs
```



# 国际化使用

## 添加国际化依赖
```
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: any
```
## 下一步，先运行 pub get packages，然后引入 flutter_localizations 库，然后为 MaterialApp 指定 localizationsDelegates 和 supportedLocales：

```
import 'package:flutter_localizations/flutter_localizations.dart';
````

```
return const MaterialApp(
  title: 'Localizations Sample App',
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: [
    Locale('en'), // English
    Locale('es'), // Spanish
  ],
  home: MyHomePage(),
);
```

## 在 pubspec.yaml 文件中，启用 generate 标志。该设置项添加在 pubspec 中 Flutter 部分，通常处在 pubspec 文件中后面的部分。
```
# The following section is specific to Flutter.
flutter:
  generate: true # Add this line
```

## 在 Flutter 项目的根目录中添加一个新的 yaml 文件，命名为 l10n.yaml，其内容如下

![](https://raw.githubusercontent.com/zetingzhu/UploadImg/main/img/20250901111734.png)

```arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
```

## 该文件用于配置本地化工具。它为你的项目配置了如下内容：

 - 将 应用资源包 (.arb) 的输入路径指定为 ${FLUTTER_PROJECT}/lib/l10n。
.arb 文件提供了应用的本地化资源。

- 将英文的语言模板设定为 app_en.arb。

- 指定 Flutter 生成本地化内容到 app_localizations.dart 文件。

## 在 ${FLUTTER_PROJECT}/lib/l10n 中，添加 app_en.arb 模板文件。如下：

```
{
  "helloWorld": "Hello World!",
  "@helloWorld": {
    "description": "The conventional newborn programmer greeting"
  }
}
```

## 接下来，在同一目录中添加一个 app_es.arb 文件，对同一条信息做西班牙语的翻译：
```
{
    "helloWorld": "¡Hola Mundo!"
}
```

## 现在，运行 flutter run 命令，你能在 arb-dir 或 output-dir 选项指定的路径下看到生成的文件。同样的，你可以在应用没有运行的时候运行 flutter gen-l10n 来生成本地化文件。

```
flutter gen-l10n
```
