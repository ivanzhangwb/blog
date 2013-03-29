---
layout: post
date: 2012-11-25 
layout: post
title:"一种感知数据变更解决方案"
categories:
- Java
tags:
- Java
---


问题来源
========
 
在日常的业务开发当中，有很多的业务场景是需要更新信息的，比如说，更新一个公司的名字，
或者更新一个公司的名字和地址，这类业务的一个特点就是如果“公司”这个实体，字段很多的话，就会比较麻烦，
因为我无法知道用户到底更新了几个字段？（这里是指在数据层面上看，从分层结构来看的话，DAO层往往为了通用，接受一个完整的实体DO对象.）  

那么这里就带来一个问题，有什么方式能感知到用户的数据变更，比如说，他只修改了公司名字，那么我最终执行的SQL里面，就只更新公司名字这一个字段呢？

按照以前的方式， 我们更新一条数据的话，首先会将其load出来，然后修改其某几个字段，最终入库的时候，我们会将所有的字段都更新一遍， 这样带来两个问题  

- 如果没有修改过的，也会对这个字段进行更新,只不过把值更新为与原来的值一样。
- 更新数据之前，需要将对应的数据从DB中load出来，比如，我只需要更新id为101的数据，我需要先从DB中查出来，修改数据，再执行SQL更新。


简单的解决方案
-------------

通常我们的应用架构都是分层次的， 一般来说都会有以下几个层:  

- Web ： 负责展示 
- Service ： 负责业务处理 
- DAO ： 负责数据存储

为了解决这个问题，我们可以在Service层与DAO层交互的时候，做一点额外的工作。   
一般来说，同一条数据在每一层的表达形式都不一致， 比如说， 公司这条数据， 从DAO层load出来以后，我一般称其为DO， 而该数据到达Service层以后，可能会将某些数据做特殊处理， 比如说： 把地址这个字符串解析一下，封装为一个表达地址的模型， 这个时候，我们称其为 Model ， 而最终业务处理结束后，我们将数据发送给前台展示的时候，我们则称呼其为VO ，表示其为一种展示模型。  

同样的，数据修改的时候，它的流程则反过来流转:  VO --> MODEL --> DO . 

如果说，我们在service层的时候，就感知到数据的变化，再把这种变化告知给DAO层知道，说， 你就帮我更新这、这、这几个字段，其他的字段你不要去管，是不是就会大大的节省掉SQL的执行时间呢？ 这样的优化对于从大数据量表中更新某一条数据来说，无疑是极其珍贵的。

OK， 下面就详细介绍一下，类似问题的解决方案：  

- 首先在MODEL中添加数据感应装置
- 其次，一旦MODEL有修改了，就通知装置，说，我xx字段更改了
- 在将model交给DAO去做具体更新的同时，把模型的感应器也交给DAO
- DAO根据模型和感应器去进行有针对性的更新操作.

那么具体到代码是怎样的呢?  

 举例来说： 假设我有一个公司的模型,公司内部的changeSet 就相当于一个数据感应器.
 

	public class CompanyModel {
	    private String      name; 
	    private Set<String> changeSet = new HashSet<String>(); //数据变更感应器
	    public String getName() {
	        return name;
	    }
	    public void setName(String name) {
	        this.changeSet.add("name"); //感应到变化
	        this.name = name;
	    }
	    public Set<String> getChangeSet() {
	        return changeSet;
	    }
	    public void setChangeSet(Set<String> changeSet) {
	        this.changeSet = changeSet;
	    }
	}

 
那么， 获取到这些感应之后，我们该如何告知给DAO层呢？ 一般可以这样做：
我们将感应器与需要修改的对象一起告知给DAO，说： `你就按照我给你的感应器，来修改我给你的对象值吧`


	public class IBatisCompanyDAOImpl {
	    public void update(CompanyModel company, Set<String> changeSet) {
	        Map<String, Object> params = new HashMap<String, Object>();
	        for (String change : changeSet) {
	            params.put(change, "");
	        }
	        params.put("this", company); //将model（或者是DO）作为参数传入.
	        getSqlMapClientTemplate().update(UPDATE_INSTBELT_MEMBER,params);
	    }
	}


到此为止，DAO就已经比较明确的知道了，我给他的这次的修改中，他到底需要修改了哪些字段，而不要像以前一样，我给它一个对象，它就傻不溜秋的一股脑给更新掉所有字段了。   
那么，回到DAO这层处理来， 到目前位置，我们只是告诉了DAO层它该怎么干， 那么它最终真正怎么干活，我们还没有告诉它， 所以，接下来我们就要告诉它，它干活的步骤是怎么样的， 所以就有了如下的SQL： 

	<update id="UPDATE_COMPANY" parameterClass="java.util.Map">
				UPDATE COMPANY SET 
				GMT_MODIFIED = now()
				<dynamic>
				  <isNotNull prepend="," property="name">            
		              	NAME = #this.name#
		          </isNotNull>
				</dynamic>
				WHERE MEMBER_ID=#this.memberId#
		</update>

 可以看到，这个SQL里面，它的参数是一个MAP， 参数里面包含了感应器需要修改的字段，当然，还有修改数据的源对象 ， 它首先会去查一下，感应器里面是否有`name` 这个字段，如果有，就把对应字段的值，从数据的源头(我们传入的MODEL) 里面取出来，再进行最终的更新.

至此， 一个简单的数据感应解决方案就完成了。  
当然，这个方案除了解决掉 盲目更新所有字段的问题外，还可以解决一个问题。  
那就是， 如果你要更新ID为101的某几个字段的数据，你不需要首先将101先从DB中load出来，你可以直接new 一个数据对象，将id设置为101， 再set你需要更改的数据值。 （当然，这样的做法是不建议尝试的，因为在业务上有可能会把数据改坏。）