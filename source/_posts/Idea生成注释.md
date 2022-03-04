---
title: Idea使用
date: 2021-10-27 09:35:56
tags: 
- Idea
index_img: /img/idea.png

---



Idea生成类和方法注释

<!--more-->

### 新建类注释

Settings -> Editor -> File and Code Templates

File选项卡 下Class 选项

```JAVA
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
#parse("File Header.java")
 /**  
    * @Author 作者名字
    * @Date ${DATE}/${TIME} 		// 时间
    * @ProjectName ${PROJECT_NAME}	// 工程名
    * @ClassName: ${NAME}			// 类名
    * @Description: TODO			// 描述
    */
public class ${NAME} {
}
```

### 方法注释

Settings -> Editor -> Live Templates



1.点击 + 号

![](/img/markdown/image-20211027095320407.png)



2.新建一个组 MyGroup

![](/img/markdown/image-20211027095101893.png)

3.选中刚才的组,再点击 + 号

![](/img/markdown/image-20211027095020949.png)

4.修改<abbreviation> 选项卡

![](/img/markdown/image-20211027095512737.png)

Template text 中内容为

```JAVA
**
* @Author tho
* @Date $date$ $times$
$param$
* @Return $return_type$
* @Description: todo
*/
```





java 中注释格式其中一种为 

```JAVA
/**
*
*/
```

/x + tab 可以生成

```JAVA
/ + 下面文本的内容 
```

```java
选择x只是个人习惯 不能选择已经存在的可以使用tab生成的内容,而x按起来比较方便,就选择 x
x 也可以改成任何不会冲突的命令
    必须使用/ + 某个指令 + 快捷键(Tab)生成
    不能将/ 写到下面文本中, 
	直接使用 某个指令 + 快捷键   生成的内容会有问题
```

5.选择对应的函数来生成时间等信息

![](/img/markdown/image-20211027100010970.png)

![](/img/markdown/image-20211027100106443.png)

- date() 和 time() 函数idea自带 可以直接输入
- param 和 return_type 选项要选择自定义脚本来显示,idea自带显示不是很好

- param对应脚本(直接将以下代码贴进Expression 选项内)

  ```JS
  groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\n' : '')}; return result", methodParameters())
  ```

- return_type对应脚本

  ```JS
  groovyScript("def result='';  def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split('<').toList(); for(i = 0; i < params.size(); i++) {if(i!=0){result+='<';};  def p1=params[i].split(',').toList();  for(i2 = 0; i2 < p1.size(); i2++)  { def p2=p1[i2].split('\\\\.').toList();  result+=p2[p2.size()-1]; if(i2!=p1.size()-1){result+=','}  } ; };  return result", methodReturnType())
  ```

6.点击如下按钮将模板设置配置到全局

![](/img/markdown/image-20211027100357703.png)

![](/img/markdown/image-20211027100412702.png)

7.之后新建的类会自动生成注释

方法上的注释 需要使用 /x + tab
