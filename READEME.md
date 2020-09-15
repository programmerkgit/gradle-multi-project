# Getting started multi project
Gradleを利用して、複数のプロジェクトを作成する手順を紹介します。

Gradleでは複数のJVM言語を利用してプロジェクトを作成できるため、今回はKotlinでプロジェクトを設定しながらGroovyとJavaを利用したパッケージを作成してみます。

## 初期化
Gradleのプロジェクトを初期化します。

```shell script
gradle init
# project type -> 1
# build script DSL -> 2
```

[Gradle init](https://firebasestorage.googleapis.com/v0/b/storage-ui.appspot.com/o/1600170031133?alt=media&token=23094721-0cfe-49a3-ac5a-f5b78f4cb61a)

### build.gradle.ktsの記述
マルチプロジェクトでは、可能な限りトップレベルのビルドスクリプトに共通設定を設定し、個別の設定のみサブプロジェクトに記述するようにします。

全てのプロジェクトがjcentorリポジトリを利用するように設定しましょう。

また、IntellijIDEを利用している場合、記述する前に`build.gradle.kts`ファイル上で右クリックをし、importしておくとコードの補完が効くようになります。

**`buiild.gradle.kts`**ファイル
```kotlin
allproject{
    repositories {
        jcentor()
    }
}
```

## サブプロジェクトの作成
サブプロジェクトを作成していきます。サブディレクトリ配下にも`build.gradle.kts`ファイルを配置します。

```
$ mkdir subproject
$ cd subproject/
$ touch build.gradle.kts
```

### サブプロジェクトの読み込み
プロジェクトルートの`setting.gradle.kts`ファイルに、サブプロジェクトを読み込む設定を記述します

**`build.gradle.kts`**
```kotlin
include("subproject")
```

### サブプロジェクトの記述
今回はGroovy libraryを記述してみます。
コード補完が効くようにルート配下の`build.gradle.kts`を再importしておきましょう。
IntellijIDAの右側のGradleタブのリロードマークからファイルのリロードが叶です。

[Gradle reload](https://firebasestorage.googleapis.com/v0/b/storage-ui.appspot.com/o/1600170869819?alt=media&token=0500e880-4669-4c13-a258-25088b9caf06)

**`subproject/build.gradle.kts`**の記述

```
dependencies {
    compile("org.codehaus.groovy:groovy:2.4.10")
    testCompile("org.spockframework:spock-core:1.0-groovy-2.4") {
        exclude(module = "groovy-all")
    }
}
```
## groovyパッケージを作成

`subproject/src/main/groovy`ディレクトリを作成ます。

```
cd subproject
mkdir -p src/main/groovy/greeter
mkdir -p src/test/groovy/greeter
```

## GreetingFormatter クラスを作成します。

**`src/main/groovy/greeter/GreetingFormatter.groovy`**
```
package greeter

import groovy.transform.CompileStatic

@CompileStatic
class GreetingFormatter {
    static String greeting(final String name) {
        "Hello, ${name.capitalize()}"
    }
}
```

## Spock Frameworkを利用してtest(GreetingFormatterSpec)を作成します

**`greeting-library/src/test/groovy/greeter/GreetingFormatterSpec.groovy
`**

```kotlin

```

## ビルド
ルート配下でビルドを実行します。
```shell script
./gradlew build

```

subprojectのbuild.gradle.ktsファイルが読み込まれビルドされます。

```
$ ./gradlew build

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.3/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 12s
4 actionable tasks: 4 execute
```

`subproject/build`ファイルが作成されている事がわかります。

## Javaパッケージを追加する
Groovyプロジェクトに加えて、Javaのプロジェクトも追加してみましょう。
Groovyの時と同様にサブプロジェクトを作り、`gradle.build.kts`ファイルを作成しルートにimportします。
IDEによるルートの`gradle.build.kts`ファイルのリロードまで実行しましょう。

```
mkdir subproject2
cd subproject2
touch build.gradle.kts

```

ルートにサブプロジェクトを読み込み、build.gradle.ktsをリロード。

**`setting.gradle.kts`**
```kotlin
include("subproject")
include("subproject2")
```

javaとApplicationをインポート。

**`subproject2/build.gradle.kts`**
```kotlin
plugins {
    java        
    application 
}

```

Application pluginを入れることで、UNIX系オペレイティングシステムとWindowsの両方で実行できる一つの実行ファイルが生成されるようになり、起動が簡単になります。

### ディレクトリの作成

```
mkdir -p subproject2/src/main/java/greeter
```

### ファイルの作成
**`greeter/src/main/java/greeter/Greeter.java`** 
```java
package greeter;

public class Greeter {
    public static void main(String[] args) {
        final String output = GreetingFormatter.greeting(args[0]);
        System.out.println(output);
    }
}
```

ここで、groovyプロジェクトで定義したGreetingFormatterを利用しています。
GreetingFormatterを読み込むために、build.gradle.ktsファイルを変更します。
**`subproject2/build.gradle.kts`**
```kotlin
plugins {
    java
    application
}

application {
    mainClassName = "greeter.Greeter"
}

dependencies {
    compile(project(":subproject"))
}
```

`:ディレクトリ名`で読み込むプロジェクトを指定します。
ここで、またルートは以下の`build.gradle.kts`を読み込んでおきましょう。

## ビルド
Groovyプロジェクトを読み込んだJavaプロジェクトをビルドしてみましょう。

```
$ ./gradlew build

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.3/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 2s
9 actionable tasks: 5 executed, 4 up-to-date
```
ここまででビルドの完了です