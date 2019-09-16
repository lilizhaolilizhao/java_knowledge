#mybatis缓存机制详解
mybatis提供了缓存机制减轻数据库压力，提高数据库性能

mybatis的缓存分为两级：一级缓存、二级缓存

一级缓存是SqlSession级别的缓存，缓存的数据只在SqlSession内有效

二级缓存是mapper级别的缓存，同一个namespace公用这一个缓存，所以对SqlSession是共享的

##一级缓存：
mybatis的一级缓存是SqlSession级别的缓存，在操作数据库的时候需要先创建SqlSession会话对象，在对象中有一个HashMap用于存储缓存数据，此HashMap是当前会话对象私有的，别的SqlSession会话对象无法访问。

具体流程：

* 第一次执行select完毕会将查到的数据写入SqlSession内的HashMap中缓存起来

* 第二次执行select会从缓存中查数据，如果select相同切传参数一样，那么就能从缓存中返回数据，不用去数据库了，从而提高了效率

注意事项：

* 如果SqlSession执行了DML操作（insert、update、delete），并commit了，那么mybatis就会清空当前SqlSession缓存中的所有缓存数据，这样可以保证缓存中的存的数据永远和数据库中一致，避免出现脏读

* 当一个SqlSession结束后那么他里面的一级缓存也就不存在了，mybatis默认是开启一级缓存，不需要配置

* mybatis的缓存是基于[namespace:sql语句:参数]来进行缓存的，意思就是，SqlSession的HashMap存储缓存数据时，是使用[namespace:sql:参数]作为key，查询返回的语句作为value保存的。例如：-1242243203:1146242777:winclpt.bean.userMapper.getUser:0:2147483647:select * from user where id=?:19

##二级缓存：
二级缓存是mapper级别的缓存，也就是同一个namespace的mappe.xml，当多个SqlSession使用同一个Mapper操作数据库的时候，得到的数据会缓存在同一个二级缓存区域

二级缓存默认是没有开启的。需要在setting全局参数中配置开启二级缓存

conf.xml：

	<settings>
   		<setting name="cacheEnabled" value="true"/>默认是false:
   	<settings>

在userMapper.xml中配置：

	<cache eviction="LRU" flushInterval="60000" size="512" readOnly="true"/>当前mapper下所有语句开启二级缓存
	
这里配置了一个LRU缓存，并每隔60秒刷新，最大存储512个对象，而却返回的对象是只读的.若想禁用当前select语句的二级缓存，添加useCache="false"修改如下：

	<select id="getCountByName" parameterType="java.util.Map" resultType="INTEGER" statementType="CALLABLE" useCache="false">
	
具体流程：

* 当一个sqlseesion执行了一次select后，在关闭此session的时候，会将查询结果缓存到二级缓存

* 当另一个sqlsession执行select时，首先会在他自己的一级缓存中找，如果没找到，就回去二级缓存中找，找到了就返回，就不用去数据库了，从而减少了数据库压力提高了性能　

注意事项：

* 如果SqlSession执行了DML操作（insert、update、delete），并commit了，那么mybatis就会清空当前mapper缓存中的所有缓存数据，这样可以保证缓存中的存的数据永远和数据库中一致，避免出现脏读

* mybatis的缓存是基于[namespace:sql语句:参数]来进行缓存的，意思就是，SqlSession的HashMap存储缓存数据时，是使用[namespace:sql:参数]作为key，查询返回的语句作为value保存的。例如：-1242243203:1146242777:winclpt.bean.userMapper.getUser:0:2147483647:select * from user where id=?:19







