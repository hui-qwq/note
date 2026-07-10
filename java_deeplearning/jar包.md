# jar包

## 一、什么是jar包

jar包就是 Java Archive File，顾名思义，它的应用是与 Java 息息相关的，是 Java 的一种文档格式，是一种与平台无关的文件格式，可将多个文件合成一个文件。jar 包与 zip 包非常相似——准确地说，它就是 zip 包，所以叫它文件包。jar 与 zip 唯一的区别就是在 jar 文件的内容中，包含了一个 META-INF/MANIFEST.MF 文件，该文件是在生成 jar 文件的时候自动创建的，作为jar里面的"详情单"，包含了该Jar包的版本、创建人和类搜索路径Class-Path等信息，当然如果是可执行Jar包，会包含Main-Class属性，表明Main方法入口，尤其是较为重要的Class-Path和Main-Class。

因为jar包主要是对class文件进行打包，而java编译生成的class文件是平台无关的，这就意味着jar包是跨平台的，所以不必关心涉及具体平台的问题。

jar还能打包静态资源文件如.html、.css以及.js等项目所需的一切，这也就意味着咱们能将自己的项目打成jar，即不管是web应用还是底层框架，都能打成jar包。