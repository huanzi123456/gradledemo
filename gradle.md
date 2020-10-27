#运行初始化任务 https://docs.gradle.org/current/samples/sample_building_java_applications_multi_project.html
gradle init  ---> 
    1.选择使用那种类型作为实现语言!
    2.选择添加那个库
    ....

项目结构:
 ```setting.gradle
        rootProject.name = 'demo'
        include("app", "list", "utilities") : 定义构建由相应文件夹中的三个子项目组成

   build.gradle
        plugins {
            //java插件提供构建java项目所有功能
            id 'java'  
        }
        
        group 'org.example'
        version '1.0-SNAPSHOT'
        
        repositories {
            mavenCentral()
            //存储库作为外部依赖得源
            //jcenter()
        }
        
        //定义junit4.12作为测试框架
        dependencies {
            testCompile group: 'junit', name: 'junit', version: '4.12'
        }
 ```

###任务相关  https://blog.csdn.net/ShelleyLittlehero/article/details/109143731
gradle tasks --all 查看所有任务

> 通过TaskContainer创建任务
tasks.create(){
    doLast{
        println ''
    }
}
>任务之间得依赖
task taskA(dependsOn:taskB) {
    doLast{
        println "executing taskA"
    }
}
>或者在之后声明依赖
hello6.dependsOn hello5


>查看所有得properties
gradle properties

>gradle是一种声明式得构建工具,gradle并不是一开始就顺序执行build.gradle文件中得内容!
而是分为两个阶段:
    配置阶段:读取所有得build.gradle文件得所有内容来配置project和task 
    执行阶段: 按照groovy dsl语法!
    
>概念: bean 和 delegate
```groovy
class GroovyBeanExample {
    private String name
}

def bean = new GroovyBeanExample()
bean.name = 'this is name'
println bean.name
```
groovy会为每个字段自动生成set与get,可以通过对象属性来访问!

###Project类常用api
> 打印project信息
```groovy
this.getProjects()
def getProjects(){
    println '---------------'
    println 'Root Project'
    println '---------------'
    this.getAllprojects().eachWithIndex { Project entry, int index ->
        if(index == 0){
            println "Root project ${entry.name}"
        }else {
            println "+--- project ${entry.name}"
        }
    }
}
this.getSubproject()
def getSubproject(){
    println '---------------'
    println 'Sub Project'
    println '---------------'
    this.getSubprojects().eachWithIndex { Project entry, int index ->
        println "+--- project ${entry.name}"
    }
}

this.getParentPro()
def getParentPro(){
    this.getParent().each {println it.name}
}
this.getRootPro()
def getRootPro(){
    println "The root is ${this.getRootProject().name}"
}
```
###属性相关api
####默认属性: 
```groovy
public interface Project extends Comparable<Project>, ExtensionAware, PluginAware {
    //gradle默认文件名
    String DEFAULT_BUILD_FILE = "build.gradle";
    //默认路径分隔符
    String PATH_SEPARATOR = ":";
  	//输出的默认路径
    String DEFAULT_BUILD_DIR_NAME = "build";
    //配置文件名称
    String GRADLE_PROPERTIES = "gradle.properties";
}
```

####扩展属性   自定义属性
```groovy
//定义扩展属性
ext{
}

subprojects{
	//每个project都定义了扩展属性
	ext{
	}
}

//我们可以在优化一下，将ext定义在根gradle中。根属性，可以被子完全继承，可以直接使用。
```
####gradle.properties定义属性
```groovy
//file: gradle.properties  只能以key-value得形式做
isNeedLoadTest = false

//file: setting.gradle
include ':app',"lib_base"
//需要注意的是gradle.properties的属性 key或者value都是Object类型，需要进行一下转换
if(hasProperty('isNeedLoadTest')?isNeedLoadTest.toBoolean():false){
	include ':test'
}
```

###文件属性相关操作
![avatar](https://img-blog.csdnimg.cn/2020101609372973.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpcmtzbWFsbGVy,size_16,color_FFFFFF,t_70#pic_center)
####文件路径
```groovy
println getRootDir().absoulutePath() //根工程
println getBuildDir().absolutePath()  //当前位置下的build文件夹路径
println getProjectDir().absolutePath() //当前project模块
```
####文件定位
```groovy
def getContent(String path){
	try{
		//file 从当前位置定位文件
		//project 还有 files api ，通过批量定位文件
		def f = file(path)
		return f.text
	}catch(GradleException e){
		e.printStackTrack()
	}
	return null
}
````
####文件copy
```groovy
//将当前文件的path复制都 根目录下的build的文件
copy{
	from fiile('path1')
	into getRootProject().getBuildDir()
	//还可以排除
	exclude{
	}
	//重命名
	rename{
	}
}
```
####文件树遍历
```groovy
fileTree('path'){FileTree fileTree ->
	fileTree.visit{ FileTreeElement element->
		copy{
			from element.file
			into getRootProject().getBuildDir().path+"/test"
		}
	}
}
```
###其他api
![avator](https://img-blog.csdnimg.cn/2020101609525842.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpcmtzbWFsbGVy,size_16,color_FFFFFF,t_70#pic_center)
####依赖相关api
```groovy
buildscript{ ScriptHandler scriptHandler->
	//配置工程的仓库地址
	scriptHandler.repositories{ RepositoryHandler handler->
		handler.jcenter()
		handler.mavenCentral()
		handler.mavenLocal()
		handler.ivy{
		}
		handler.maven{
			name 'personal'
			url 'http://local.....'
			credenticals{
				username = 'admin'
				password = 'admin123'
			}
		}
	}
	//配置工程插件地址
	scriptHandler.dependencies{
		classpath 'com.android.tools.buid:gradle:2.2.2'
		classpath 'com.tencent'
	}
}


dependencies{
	//文件夹 -> fileTree ,否则file
	compile fileTree(include:['*.jar'],dir:'libs') 
	compile(){
		//排除依赖
		exclude module:'xxx'
		exclude group:'yyyy'
		//禁止传递依赖
		transive false
	}
}
```

####外部命令
```groovy
task(name: 'apkcopy'){
	//doLast确保 该方法在gradle运行阶段执行
	doLast{
		def sourcePath = this.buildDir.path + '/outputs/apk'
		def destPath = '绝对目的路径'
		def cmd = 'mv -f ${sourcePath} ${destPath}'
		exec{
			try{
				executable 'bash'
				args '-c' , cmd
				println 'the command is execute success.'
			}catch(GradleException e){
				e.printStackTrace()
				println 'the command is execute failed.'
			}
		}
	}
}

// ./gradlew apkcopy
```

###task作用 查看所有得task   gradle task --all
####task的创建方式,通过task函数去创建,通过task容器去创建
```groovy
//直接通过task函数去创建
task helloTask {
    println 'I am HelloTask'
}

//使用TaskContainer创建
this.tasks.create(name: 'helloTaks2'){
    println 'I am helloTaks2'
}

//运行Task：
//  ./gradlew helloTask
//  ./gradlew helloTask2
```

####task执行详解
```groovy
//配置task
//1. 在定义的时候配置
task helloTaks(group:'zfc',description:'task study'){
    println 'I am HelloTask'
    doFirst {
        println 'the task group is :' + group
    }
    doFirst {}
}
//需要注意的是 外部的doFirst要优先于内部的doFirst执行
helloTaks.doFirst {
    println 'the task group is :' + description
}

//使用TaskContainer创建
this.tasks.create(name: 'helloTaks2'){
    //分组
    setGroup('zzz')
    //注释
    setDescription("tasks study")
    println 'I am helloTaks2'
}
```
#### task依赖设置
```groovy
task taskX{
    doLast {
        print 'taskX'
    }
}
task taskY{
    doLast {
        print 'taskY'
    }
}
//在创建Task的时候，设置其依赖的task
task taskZ(dependsOn:[taskX]){
    doLast {
        print 'taskZ'
    }
}
//通过调用对象方法创建依赖关系
taskZ.dependsOn('taskY')
```

```groovy
task noLib  {
    doLast {
        println 'noLib'
    }
}
task lib2 {
    doLast {
        println 'lib2'
    }
}
task taskZ(dependsOn:[taskX]){
    //该task依赖所有的lib开头的task，但是
    //需要注意的是 这里只有lib1生效了，因此lib2声明在后面
    dependsOn this.tasks.findAll { task->
        return task.name.startsWith("lib")
    }
    doLast {
        print 'taskZ'
    }
}
task lib2 {
    doLast {
        println 'lib2'
    }
}
```

```groovy
//dependOn 在闭包里面设置dependOn来添加依赖!
task noLib  {
    doLast {
        println 'noLib'
    }
}
task lib2 {
    doLast {
        println 'lib2'
    }
}
task taskZ(dependsOn:[taskX]){
    //该task依赖所有的lib开头的task，但是
    //需要注意的是 这里只有lib1生效了，因此lib2声明在后面
    dependsOn this.tasks.findAll { task->
        return task.name.startsWith("lib")
    }
    doLast {
        print 'taskZ'
    }
}
task lib2 {
    doLast {
        println 'lib2'
    }
}
```









附录: 命令
####运行守护进程状态
gradle --status

####禁用守护进程
org.gradle.daemon=false --> 到文件«USER_HOME»/.gradle/gradle.properties

####停止现有守护进程
gradle --stop

