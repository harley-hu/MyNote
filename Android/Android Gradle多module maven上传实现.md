# Android Gradle 多module Maven上传实现

## 一、背景
通常在建立工程时，做模块化划分时，需要将代码模块化，因而会导致一个工程往往依赖多个模块，当将这些模块作为SDK上传Maven库时，每个module只会上传其模块内的代码，而其依赖的module都是即使是工程依赖，都会变成Maven依赖，最后生成的POM文件其依赖会出现groupId、version不对的情况,如下图：

![maven_pom](https://github.com/harley-hu/MyNote/raw/main/assets/maven_pom.webp)

现参考[填坑：有工程依赖时，Gradle uploadArchives to Nexus/Maven]完整实现多module maven上传功能。

## 二、实现
在根目录的`build.gradle`文件`allprojects`或着`subprojects`代码块中添加如下代码：
```grovy
  allprojects{
      apply plugin: 'maven'
      //请不用手动在此处点击上传，请使用命令行针对子module进行上传
      //def repository='file://'+dir.filePath
      def GROUP_ID='xxx'
      def VERSION='1.0.1'
      uploadArchives {
          repositories {
              mavenDeployer {
                  repository(url: uri(repository)) {
                      authentication(userName: "$mavenUser", password: "$mavenPassword")
                  }
                  pom.version = VERSION
                  pom.groupId = GROUP_ID

                  pom.whenConfigured { pom ->
                      pom.dependencies.forEach { dep ->
                          if (dep.getVersion() == "unspecified") {
                              dep.setGroupId(GROUP_ID)
                              dep.setVersion(VERSION)
                          }
                      }
                  }
              }
          }
      }
    }
```
上述代码主要功能是在每个module执行`uploadArchives` Task时，会更改pom依赖设置，将所有工程依赖的版本号设置成`VERSION`，`groupId`设置成`GROUP_ID`,默认的`artifactId`为module的名字。如果想修改`artifactId`，则可以在具体的module的`build.gradle`中设置`archivesBaseName`:
```
archivesBaseName = '$artifactId'
```
一般情况下，使用gradle maven 插件上传时，可以不在上传时设置`groupId`、`version`,直接在`build.gradle`声明：
```
group = 'org.gradle.sample'
version = '1.0'
```
当执行一次`uploadArchives`时，就可以对所有module执行maven上传任务，同样在根目录`build.gradle`中设置task依赖，具体如下：
```
project.afterEvaluate {
    getSubprojects().forEach() { project ->
        if (//指定条件) {
            def uploadTask = project.tasks.getByName('uploadArchives')
            uploadArchives.dependsOn uploadTask
        }
    }
}
```
因此当执行root project的`uploadArchives`时，会依次执行符合条件moudle的上传任务。


参考文档：
1.[填坑：有工程依赖时，Gradle uploadArchives to Nexus/Maven]
2.





[填坑：有工程依赖时，Gradle uploadArchives to Nexus/Maven]:https://www.jianshu.com/p/98d42f0d36da
