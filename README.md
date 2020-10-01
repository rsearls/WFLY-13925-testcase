# WFLY-13925-testcase

Project testcase for https://github.com/rsearls/WFLY-13925-testcase

Issue:
	This project defines a simple REST endpoint.
	When the archive is built and deployed *without* archive
	org.jboss.resteasy:resteasy-client-microprofile
	(see pom.xml lines 18-22) the endpoint can be reached
	successfully.  When the archive is built and deployed *with*
	org.jboss.resteasy:resteasy-client-microprofile the endpoint
	can not be reached.

Steps to reproduce.

	Case 1, the successful case.
		build this project
		deploy the app to wildfly-21.0.0.Beta1-SNAPSHOT
		Use cURL to call the endpoint
			curl -v http://localhost:8080/rest-client-wfly/ping

    Case 2, the unsuccessful case.
		Edit pom.xml and uncomment archive org.jboss.resteasy:resteasy-client-microprofile
		build this project
		deploy the app to wildfly-21.0.0.Beta1-SNAPSHOT
		Use cURL to call the endpoint
			curl -v http://localhost:8080/rest-client-wfly/ping

Current debugging informaton

This appears to be an issue in wfly deployment processing.
I can confirm when deploying the app that contains resteasy-client-microprofile
the servlet executed by undertow is io.undertow.servlet.handlers.DefaultServlet.
When the app does not contain resteasy-client-microprofile the servlet executed
by undertow is org.jboss.resteasy.plugins.server.servlet.HttpServlet30Dispatcher.

Setting a breakpoint in io.undertow.servlet.core.ManagedServlet$DefaultInstanceStrategy  
line 300, method start()  you will see ServletInfo{mappings=[], servletClass=class io.undertow.servlet.handlers.DefaultServlet, name='default'}
when the app contains resteasy-client-microprofile and value
ServletInfo{mappings=[/*], servletClass=class org.jboss.resteasy.plugins.server.servlet.HttpServlet30Dispatcher, name='io.xstefank.RestApplication'}
when it does not.

In wfly I looked at the jaxrs deployment code.  I can see that for both
apps, servletClass is assigned org.jboss.resteasy.plugins.server.servlet.HttpServlet30Dispatcher
but when resteasy-client-microprofile is present some wfly process that runs after
jaxrs overwrites this information with servletClass=class io.undertow.servlet.handlers.DefaultServlet.





	

