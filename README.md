# spring-ws访问webService
## 使用spring-ws访问webService支持认证，废话不多说，直接上代码
### 引入依赖：
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-ws</artifactId>
            <version>1.3.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.3</version>
        </dependency>

### jdk生成wsdl代码：
    无认证：
      wsimport -s d:\wsdl -p com.example.demo.request -encoding utf-8 http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl
    有认证：
      wsimport -s d:\wsdl -p com.example.demo.request -encoding utf-8 http://www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl -Xauthfile D:\authfile.txt
      比无认证多了Xauthfile，新建Xauthfile，内容如下：
      http://用户名:密码@www.webxml.com.cn/WebServices/IpAddressSearchWebService.asmx?wsdl
      ### 注意：协议要与wsdl的url一致，如wsdl的url用http则才有http，用https就采用https
 
        1、通过-s命令指定您的java项目src路径
        2、通过-p命令指定需生成包结构(指定之后会自动生成)
        3、通过-Xauthfile命令指定访问带有401认证的webservice授权文件(文件路径或文件名可以随意更改 特别简单，请放心)。
### 编写代码：
          @Configuration
          public class WebServiceClientConfig {

              // 用户名
              String userName = "用户名";
              //密码
              String password = "密码";
              
              @Bean
              public Jaxb2Marshaller marshaller() {
                  Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
                  //会扫描此类下面的对应的 jaxb2实体类 因为是使用 Marshaller和 unmarshaller来进行xml和bean直接转换的
                  //具体是判断此路径下是否包含 ObjectFactory.class 文件
                  //设置 JAXBContext 对象
                  //marshaller.setContextPath("com.chinaredstar.progress.utils.webservice");
                  //或者使用 以下方式设置
                  marshaller.setPackagesToScan("com.chinaredstar.progress.utils.webservice");
          //        marshaller.setClassesToBeBound(classesToBeBound);
                  return marshaller;
              }

              @Bean
              public WebServiceMessageSender messageSender() {
                  HttpComponentsMessageSender messageSender = new HttpComponentsMessageSender();
                  //设置默认超时时间
                  messageSender.setReadTimeout(10000);
                  messageSender.setConnectionTimeout(10000);
                  //认证
                  Credentials credentials = new Credentials() {
                      @Override
                      public Principal getUserPrincipal() {
                          return new BasicUserPrincipal(userName);
                      }

                      @Override
                      public String getPassword() {
                          return password;
                      }
                  };
                  messageSender.setCredentials(credentials);
                  return messageSender;
              }

              /*
               * 创建bean
               */
              @Bean
              public SapWebServiceClient wsClient(Jaxb2Marshaller marshaller, WebServiceMessageSender messageSender) {
                  SapWebServiceClient client = new SapWebServiceClient();
                  //默认对应的ws服务地址 client请求中还能动态修改的
                  //client.setDefaultUri("你的webService url");
                  client.setMarshaller(marshaller);//指定转换类
                  client.setUnmarshaller(marshaller);

                  client.setMessageSender(messageSender);

                  return client;
              }
          }
 
##         
        /**
         * 编写客户端 继承WebServiceGatewaySupport 类 方便调用
         *
         * @author zhujia
         */
        @Slf4j
        public class SapWebServiceClient extends WebServiceGatewaySupport {

            //银行入账明细查询URL
            String bankAccountingDetailsUrl = "你的url";

            /**
             * 银行入账明细查询
             *
             * @author zhujia
             */
            public ZXFZBANKSTMTResponse getBankAccountingDetails(ZXFZBANKSTMT req) {
                //使用 marshalSendAndReceive 进行调用
                log.info("调用SAP接口，银行入账明细查询参数：{}", JSON.toJSONString(req));
                ZXFZBANKSTMTResponse res = (ZXFZBANKSTMTResponse) super.getWebServiceTemplate().marshalSendAndReceive(bankAccountingDetailsUrl, req);
                log.info("调用SAP接口，银行入账明细查询结果：{}", JSON.toJSONString(res));
                return res;
            }

        }
  
 ### 测试
 
       public class WebServiceTest extends BaseTest {

          @Autowired
          SapWebServiceClient sapWebServiceClient;

          @Test
          public void getBankAccountingDetails() {
              ZXFZBANKSTMT zxfzbankstmt = new ZXFZBANKSTMT();
              zxfzbankstmt.setZBUKRS("收款公司");
              zxfzbankstmt.setZZFKR("付款方全称");
              zxfzbankstmt.setZZFKZH("付款方银行账号");
              zxfzbankstmt.setWRBTR(new BigDecimal(123.12));
              zxfzbankstmt.setBLDATF("2018-03-04");
              zxfzbankstmt.setBLDATT("2018-03-05");
              ZXFZBANKSTMTResponse response = sapWebServiceClient.getBankAccountingDetails(zxfzbankstmt);
              System.err.println(JSON.toJSONString(response));
          }
      }
        
  
