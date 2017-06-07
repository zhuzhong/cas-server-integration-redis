#cas-server-integration-redis

##功能说明

cas应用redis的集群方式,默认情况cas已经提供了很多的集群方式，比如mongo,memcached,ehcache等，但是redis普遍性很高，也是为了公司的应用，所以扩展一下此功能.



##使用配置
	ticketRegistry.xml中
	<bean id="ticketRegistry" class="org.jasig.cas.ticket.registry.DefaultTicketRegistry"/>
	
	替换为如下内容：
	<bean id="ticketRegistry" class="com.aldb.cas.integration.redis.RedisTicketRegistry">
	    <constructor-arg index="0" ref="redisTemplate" />
	    <constructor-arg index="1" value="${tgt.timeToKillInSeconds:1800}" />
	    <constructor-arg index="2" value="${st.timeToKillInSeconds:10}" />
	</bean>

	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
	    <property name="maxIdle" value="200" />
	    <property name="testOnBorrow" value="true" />
	</bean>
	<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
	    <property name="hostName" value="${redis.host}"/>
	    <property name="port" value="6379"/>
	    <property name="poolConfig" ref="jedisPoolConfig"/>
	</bean>
	<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
	    <property name="connectionFactory" ref="jedisConnectionFactory"/>
	</bean>

上面的参数根据自己的实际内容进行更改。

在ticketRegistry.xml中注掉如下bean

	 <bean id="ticketRegistryCleaner" class="org.jasig.cas.ticket.registry.support.DefaultTicketRegistryCleaner"
          c:centralAuthenticationService-ref="centralAuthenticationService"
          c:ticketRegistry-ref="ticketRegistry"/>

    <bean id="jobDetailTicketRegistryCleaner"
          class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean"
          p:targetObject-ref="ticketRegistryCleaner"
          p:targetMethod="clean"/>

    <bean id="triggerJobDetailTicketRegistryCleaner"
          class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean"
          p:jobDetail-ref="jobDetailTicketRegistryCleaner"
          p:startDelay="20000"
          p:repeatInterval="5000000"/>

这三个bean的功能是定期清理过期ticket,由于我们使用了redis的键过期机制所以就不需要这一部分功能。