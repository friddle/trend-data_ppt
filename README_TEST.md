Log4j配置规范
---------------
---------------

###Log4j分级含义
级别      |含义       |示范
-------- |---------  |
ERROR    |错误日志    |Appapi的错误拦截代码
WARN     |指系统可以继续运行，但是存在潜在风险|appapi的访问请求超时错误
INFO     |信息点，比如访问请求。定时运行任务|appapi访问请求日志
DEBUG    |系统级别的日志输出|appapi的redis和jdbc的操作


###Log4j的规范目标
项目       |详细描述                 |怎样做              |负责人
----------|------------------------|--------------------|---------------
appapi|redis的操作不应该是debug级别而应该是INFO级别|RT|胡青云
appapi|输出请求日志到INFO|自定义INFO格式,可以固化为Class|胡青云，李泽文
appapi|记录超时JDBC,Redis，访问请求到DEBUG|RT|胡青云
pcm|录超时JDBC,Redis，访问请求到DEBUG|RT|易磊

###Log4j的格式设定

####Log4jRequest请求格式为
  请求Class定义为:(请求)USER_REQUEST
  参数假如为空，则用“-”取代
  请求格式为 REQUEST：` “REQUEST名称” 请求来源 请求耗时 请求大小 返回大小 “请求端” `

  message： ` REQUEST: “ybb.thrift.SendSMSVerifyCodeReq” - 0.151 9,797 9,797 "Dalvik/1.6.0 (Linux; U; Android 4.4.2; H60-L01 Build/HDH60-L01)"`
>  记住REQUEST名称，请求端需要加引号  

####Log4jError请求格式
   请求Class定义为：REQUEST_ERROR
   请求格式为 REQUEST_ERROR: “REQUEST名称” 请求来源 请求参数（json） 错误Message
    message： ` REQUEST: “ybb.thrift.SendSMSVerifyCodeReq”  "Dalvik/1.6.0 (Linux; U; Android 4.4.2; H60-L01 Build/HDH60-L01)"` {user:1} "the type"

####Log4j接口请求WARNING格式
   请求Class定义为:(JDBC)JDBC_WARN,(REDIS)REDIS_WARN,(接口)INTERFACE_WARN
   请求格式为 WARN： “REQUEST名称” 请求耗时 请求参数（json）
   message：  “ybb.thrift.SendSMSVerifyCodeReq” 0.14 “{username:1111}” 


###Log4j：示范代码   

```   

    public class RequestLog{
        public static class Params
        {
            int request_size;
            int response_size;
            String request_name;
            String request_client;
            String request_source;
            int timestamp;
    
            public String formatLog()
            {
                String.format(RequestFormat, this.request_name,this.request_source,this.timestamp,this.request_size,this.response_size,this.request_client);
            }
        }
    
        public static Logger logger= LoggerFactory.getLogger("USER_REQUEST");
        public static final String RequestFormat="REQUEST:\"%s\" %s %s %s %s \"%s\"";
    
        public static void OutputRequest(Request request, Response response, int timestamp)
        {
            Params param=new Params();
            Request request1=request.deepCopy();
            param.request_size=request1.getBody().length;
            param.request_name=request1.getHeader().requestName;
            param.timestamp=timestamp;
            logger.info(param.formatLog());
        }
    }

``

