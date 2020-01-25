# samples

## Hello World

每个应用有一个 `main` 函数，使用 `print` （top-level）函数向控制台输出文本。

```dart
void main() {
  print('Hello, World!');
}
```

## Variables

由于**类型推断**，大多数变量不需要*显示类型*。

```dart
var year = 9102;
var money = 0.5;
var name = 'Jack';
var languages = ['C', 'C++', 'Java', 'Python'];
var image = {
  'url': '//path/to/saturn.jpg',
  'tags': ['saturn']
};
```

## Control flow

```dart
if (year >= 2001) {
  print('21st century');
} else if (year >= 1901) {
  print('20th century');
}

for (var object in flybyObjects) {
  print(object);
}

for (int month = 1; month <= 12; month++) {
  print(month);
}

while (year < 2016) {
  year += 1;
}
```

## Functions

指定函数的*参数*和*返回值***类型**

```dart
int fibonacci(int n) {
  if (n < 2) return n;
  
  return fibonacci(n - 1) + fibonacci(n - 2);
}

int addOne(int n) => n + 1;
```

`=>`是只包含一条语句的函数的便利写法，当传递匿名函数作为参数时尤其有用。

```dart
flybyObjects.where((name) => name.contains('turn')).forEach(print);
```

## Comments

```dart
// 单行注释

/// 文档注释

/* Comments like these are also supported. */
```

## Imports

```dart
// 导入核心库
import 'dart:math';

// Importing libraries from external packages
import 'package:test/test.dart';

// Importing files
import 'path/to/my_other_file.dart';
```

## Classes

```dart
class Spacecraft {
  String name;
  DateTime launchDate;

  // 构造器，使用语法糖为成员属性赋值
  Spacecraft(this.name, this.launchDate) {
    // 初始化代码
  }
    
  // 命名构造器，并调用默认构造器
  Spacecraft.unlaunched(String name) : this(name, null);

  // read-only non-final 成员属性
  int get launchDate => launchDate?.year;

  // 成员方法
  void describe() {
    print('Spacecraft: $name');
    if (launchDate != null) {
      int years =
          DateTime.now().difference(launchDate).inDays ~/
              365;
      print('Launched: $launchYear ($years years ago)');
    } else {
      print('Unlaunched');
    }
  }
}

var voyager = Spacecraft('Voyager I', DateTime(1977, 9, 5));
voyager.describe();

var voyager3 = Spacecraft.unlaunched('Voyager III');
voyager3.describe();
```

## Inheritance

遵循单一继承

```dart
class Orbiter extends Spacecraft {
  num altitude;
  Orbiter(String name, DateTime launchDate, this.altitude)
      : super(name, launchDate);
}
```

## Interfaces

没有 `interface` 关键字，所有类*隐式地定义了一个接口*，因此可以*实现任何类*

```dart
class MockSpaceship implements Spacecraft {
  // ···
}
```

### abstract classes

```dart
abstract class Describable {
  void describe();

  void describeWithEmphasis() {
    print('=========');
    describe();
    print('=========');
  }
}
```

## Mixins

 a way of reusing code in multiple class hierarchies. [What are mixins?](https://medium.com/flutter-community/dart-what-are-mixins-3a72344011f3)
 
 ```dart
 class Piloted {
  int astronauts = 1;
  void describeCrew() {
    print('Number of astronauts: $astronauts');
  }
}

class PilotedCraft extends Spacecraft with Piloted {
  // ···
}
 ```

## Async

使用`async`、`await`，避免“回调地狱”、增强代码可读性

```dart
const oneSecond = Duration(seconds: 1);
// ···
Future<void> printWithDelay(String message) async {
  await Future.delayed(oneSecond);
  print(message);
}
```

## Exceptions

使用 `throw` 抛出异常

```dart
if (astronauts == 0) {
  throw StateError('No astronauts.');
}
```

使用`try`、`catch` 捕获异常

```dart
try {
  for (var object in flybyObjects) {
    var description = await File('$object.txt').readAsString();
    print(description);
  }
} on IOException catch (e) {
  print('Could not describe object: $e');
} finally {
  flybyObjects.clear();
}
```

[官方原文](https://dart.dev/samples)