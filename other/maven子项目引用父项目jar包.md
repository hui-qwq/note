# maven子项目引用父项目jar包

如果父项目pom中使用的是：
<dependencies>
   ....
</dependencies>方式，
则子项目pom会自动使用pom中的jar包。

如果父项目pom使用
<dependencyManagement>
   <dependencies>
     ....
   </dependencies>
</dependencyManagement>方式，
则子项目pom不会自动使用父pom中的jar包，如果需要使用，就要给出groupId和artifactId，无需给出version