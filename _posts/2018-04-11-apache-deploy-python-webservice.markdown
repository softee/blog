---
layout: post
title:  "create python webservice and deployed by apache"
date:   2018-04-11 14:36:30 +0800
categories: python
---
-   environment
    -   python version:2.7.9
    -   os: window 7 32bit
    -   windows runtime: VC9
    -   apache version: 2.4.12
-   server
    -   download and install python module `soaplib` for server
        -   download address: <https://github.com/soaplib/soaplib.git>
        -   cd soaplib
        -   python setup.py install
        
    -   server code. create file server.wsgi as belows:

            # coding:utf-8
            # author:peter 2018-04-11

            import soaplib
            from soaplib.core.service import rpc, DefinitionBase
            from soaplib.core.model.primitive import String, Integer, Boolean
            from soaplib.core.server import wsgi
            from soaplib.core.model.clazz import Array
            from soaplib.core.service import soap
            from soaplib.core.model.clazz import ClassModel
            from wsgiref.simple_server import make_server


            class Data(ClassModel):
                """
                :function: create `Data` dictionary as datatype
                """
                __namespace__ = "Data"
                username = String
                emotion = String


            class ExportService(DefinitionBase):
                """
                :function:create functions `say_hello` and `get_recommend`
                :return: dictionay
                """
                @soap(String, Integer, _returns=Array(String))
                def say_hello(self, name, times):
                    results = []
                    for i in range(0, times):
                        results.append('Hello, %s' % name)
                    return results

                @soap(Data, _returns=Boolean)
                def get_recommend(self, data):
                    print data.username
                    print data.emotion
                    return 1
                    
            try:
                soap_application = soaplib.core.Application([ExportService], 'tns')
                application = wsgi.Application(soap_application)
                server = make_server('localhost', 8888, application)
                server.serve_forever()
            except ImportError:
                print "Error: example server code requires Python >= 2.5"
                
-   client
    -   install python module `suds` for client.
        -   `pip install suds-jurko`
        
    -   client code. create file client.py as belows:
            
            # coding:utf-8
            # author:peter 2018-04-11

            from suds.client import Client

            def get_client():
                """
                :function:call webservice
                :return:
                """
                client = Client('http://localhost:8888/?wsdl')
                client.options.cache.clear()
                return client


            def get_recommend():
                """
                :function:call method `get_recommend` by dictionary `data`
                :return:
                """
                data={
                    "username":"alle",
                    "emotion":"1-2-3"
                }
                print 'call get_recommend'
                client = get_client()
                result = client.service.get_recommend(data)
                print result


            def say_hello():
                """
                :function:call method `say_hello` by passing two params `name` and `times`
                :return:
                """
                name = '123'
                times = 2
                print 'call say_hello'
                client = get_client()
                result = client.service.say_hello(name,times)
                print result


            if __name__ == '__main__':
                print '====start======'
                # get_recommend()
                say_hello()
                print '=====end======'
-   test
    -   open cmd
        -   run server: `python server.wsgi`
    -   open a new cmd
        -   run client: `python client.py`
    
-   deployed by apache2.4
    -   download windows version, select apache-2.4.12
        -   <https://www.apachehaus.com/cgi-bin/download.plx>
    -   download module for vc9
        -   mod_wsgi-windows-4.4.6.tar.gz
        -   unzip file and copy file `Apache24-win32-VC9\mod_wsgi-py27-VC9.so` to `apache\modules\`
        -   or download <https://github.com/GrahamDumpleton/mod_wsgi> and install mod_wsgi
        
    -   modify apache config 
        -   file: apache/conf/httpd.conf
        -   change params as your environment
            -   apache address `Define SRVROOT "D:/Apache24"`
            -   port `Listen 8080`
            -   server name `ServerName 127.0.0.1:8080`
            -   cancel setting
                
                    #<Directory />
                    #    AllowOverride none
                    #    Require all denied
                    #</Directory>

        -   add codes at the end:
            
                LoadModule wsgi_module modules/mod_wsgi-py27-VC9.so
                AddType text/html .py

                WSGIPythonHome D:/Python27
                WSGIScriptAlias / D:/webservice/serve.wsgi
                <Directory D:/Webplat/Runtime/>
                    AllowOverride None
                    Options None
                    Require all granted
                </Directory>
                WSGIApplicationGroup %{GLOBAL}

                AcceptFilter http none 
                AcceptFilter https none 
                EnableSendfile Off  
                EnableMMAP off
    -   run apache
        -   cd /d D:\Webplat\Apache24\bin
        -   httpd -k restart
        -   call apache service
            -   open Browser and enter <http://localhost:8080>
        -   call python service
            -   open Browser and enter <http://localhost:8888?wsdl>,then you will see service infomation.
    -   test client
        -   open cmd
        -   run client `python client.py`