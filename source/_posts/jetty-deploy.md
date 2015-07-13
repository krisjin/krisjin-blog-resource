title: jetty部署配置
date: 2015-07-13 20:22:39
categories: jetty
tags: [jetty,deploy]
---



执行下命令：

	mkdir -p /opt/data/app/bin   ##启动shell 目录
 
	mkdir -p /opt/data/app/log  ## 应用日志目录

	mkdir -p /opt/data/app/jetty  ##jetty安装目录

	mkdir -p /opt/data/app/iopush  ##创建应用目录，并创建一个iopush应用

	
	cd /opt/data/app/iopush

	vi iopush-content.xml

在iopush-content.xml文件中添加以下配置：

	
	 <?xml version="1.0"?>
	 <!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">
	 <Configure class="org.eclipse.jetty.webapp.WebAppContext">
	   <Set name="contextPath">/</Set>
	   <Set name="war">/opt/data/app/iopush/iopush/iopush.war</Set>
	   <Set name="extractWAR">true</Set>
	   <Set name="copyWebDir">true</Set>
	   <Set name="tempDirectory">/opt/data/app/iopush/iopush/temp</Set>
	   <Array id="plusConfig" type="java.lang.String">
	     <Item>org.eclipse.jetty.webapp.WebInfConfiguration</Item>
	     <Item>org.eclipse.jetty.webapp.WebXmlConfiguration</Item>
	     <Item>org.eclipse.jetty.webapp.MetaInfConfiguration</Item>
	     <Item>org.eclipse.jetty.webapp.FragmentConfiguration</Item>
	     <Item>org.eclipse.jetty.plus.webapp.EnvConfiguration</Item>
	     <Item>org.eclipse.jetty.plus.webapp.PlusConfiguration</Item>
	     <Item>org.eclipse.jetty.annotations.AnnotationConfiguration</Item>
	     <Item>org.eclipse.jetty.webapp.JettyWebXmlConfiguration</Item>
	     <Item>org.eclipse.jetty.webapp.TagLibConfiguration</Item>
	   </Array>
	   <Set name="defaultsDescriptor"><Property name="jetty.home" default="."/>/etc/webdefault.xml</Set>
	   <Set name="ConfigurationClasses"><Ref id="plusConfig"/></Set>
	 </Configure>

	
将iopush.war 放在 /opt/data/app/iopush/  目录下
创建 temp目录： mkdir -p /opt/data/app/iopush/temp


在/opt/data/bin 创建iopush 运行文件，内容如下：

	# chkconfig: 2345 20 80
	# description: jetty server instance  
	# use the JETTY_HOME/bin/jetty.sh to start,stop,status the jetty servers.
	export JAVA_HOME=/opt/jdk7
	export PATH=${JAVA_HOME}/bin:$PATH
	export JETTY_HOME=/opt/data/jetty
	export JETTY_PORT=8099
	export JETTY_MONITOR_PORT=9002
	export JETTY_HOST=192.168.133.130
	export JETTY_LOGS=/opt/data/log
	export JETTY_PID=/opt/data/iopush.pid
	export JETTY_SERVER_NAME=iopush
	
	#JVM_0PTIONS=" -server "
	JVM_0PTIONS="-server -XX:+UseConcMarkSweepGC -XX:ParallelCMSThreads=2 -XX:+CMSClassUnloadingEnabled -XX:+UseCMSCompactAtFullCollecti
	on -XX:CMSInitiatingOccupancyFraction=80"
	export JAVA_OPTIONS="-Xmx1000m -Xss256k ${JVM_0PTIONS} -Dgsns.apps=/opt/data/app/iopush -Djetty.host=${JETTY_HOST} -Dserver_name=${J
	ETTY_SERVER_NAME}"
	/opt/data/jetty/bin/jetty.sh $* 1>> ${JETTY_LOGS}/jetty_stdout.log 2>>${JETTY_LOGS}/jetty_stderr.log



修改/opt/data/jetty/etc/jetty.xml文件，添加如下内容：

	
	
        <!-- Create the deployment manager                                   -->     
    <Call name="addBean">
      <Arg>   
        <New id="DeploymentManager" class="org.eclipse.jetty.deploy.DeploymentManager">
          <Set name="contexts">
            <Ref id="Contexts" />
          </Set>  
          <Call name="setContextAttribute">
            <Arg>org.eclipse.jetty.server.webapp.ContainerIncludeJarPattern</Arg>
            <Arg>.*/servlet-api-[^/]*\.jar$</Arg>
          </Call>
        </New>
      </Arg>
    </Call>

        <!-- Add a WebAppProvider to the deployment manager                  -->
    <Ref id="DeploymentManager">
          <Call id="contextprovider" name="addAppProvider">
            <Arg>
              <New class="org.eclipse.jetty.deploy.providers.ContextProvider">
                <Set name="monitoredDirName"><Property name="gsns.apps" /></Set>
                <Set name="scanInterval">-1</Set>
              </New>
            </Arg>
          </Call>
          <Call id="webappprovider" name="addAppProvider">
            <Arg>
              <New class="org.eclipse.jetty.deploy.providers.WebAppProvider">
                <Set name="monitoredDirName"><Property name="jetty.home" default="." />/webapps</Set>
                <Set name="defaultsDescriptor"><Property name="jetty.home" default="."/>/etc/webdefault.xml</Set>
                <Set name="scanInterval">-1</Set>
                                <Set name="extractWars">true</Set>
              </New>
            </Arg>
          </Call>
    </Ref>

        <!-- Config stdout log -->
        <New id="ServerStdoutLog" class="java.io.PrintStream">
      <Arg>
        <New class="org.eclipse.jetty.util.RolloverFileOutputStream">
          <Arg><Property name="jetty.logs" default="./logs"/>/yyyy_mm_dd.stdout.log</Arg>
          <Arg type="boolean">true</Arg>
          <Arg type="int">7</Arg>
          <Arg><Call class="java.util.TimeZone" name="getTimeZone"><Arg>GMT</Arg></Call></Arg>
          <Get id="ServerStdoutLogName" name="datedFilename"/>
        </New>
      </Arg>
    </New>
        <!-- Config stderr log -->
        <New id="ServerErrLog" class="java.io.PrintStream">
      <Arg>
        <New class="org.eclipse.jetty.util.RolloverFileOutputStream">
          <Arg><Property name="jetty.logs" default="./logs"/>/yyyy_mm_dd.stderr.log</Arg>
          <Arg type="boolean">true</Arg>
          <Arg type="int">7</Arg>
          <Arg><Call class="java.util.TimeZone" name="getTimeZone"><Arg>GMT</Arg></Call></Arg>
          <Get id="ServerErrLogName" name="datedFilename"/>
        </New>
      </Arg>
    </New>
    <Call class="org.eclipse.jetty.util.log.Log" name="info"><Arg>Redirecting stdout to <Ref id="ServerStdoutLogName"/></Arg></Call>
    <Call class="org.eclipse.jetty.util.log.Log" name="info"><Arg>Redirecting stderr to <Ref id="ServerErrLogName"/></Arg></Call>
    <Call class="java.lang.System" name="setErr"><Arg><Ref id="ServerErrLog"/></Arg></Call>
    <Call class="java.lang.System" name="setOut"><Arg><Ref id="ServerStdoutLog"/></Arg></Call>