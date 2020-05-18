1. 在AccessKey管理中配置用户组并配置SMS权限，再生成用户并加入用户组并开启代码控制，记下用户的accessKeyId和accessSecret

2. 在SMS控制台中配置相应的模板和签名

3. 代码编写

   ```java
   @Value("${aliyun.accessKeyId}")
   String accessKeyId;
   @Value("${aliyun.accessSecret}")
   String accessSecret;
   @Value("${aliyun.sms.templateCode}")
   String templateCode;
   @Value("${aliyun.sms.signName}")
   String signName;
   public Boolean sendMessage(String telephone,String code) {
           DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId, accessSecret);
           IAcsClient client = new DefaultAcsClient(profile);
   
           CommonRequest request = new CommonRequest();
           request.setMethod(MethodType.POST);
           request.setDomain("dysmsapi.aliyuncs.com");
           request.setVersion("2017-05-25");
           request.setAction("SendSms");
           request.putQueryParameter("PhoneNumbers",telephone);
           //签名
           request.putQueryParameter("SignName",signName);
           //模板code
           request.putQueryParameter("TemplateCode",templateCode);
           //构建短信验证码
           HashMap<String, Object> map = new HashMap<>();
           //对应创建模板的参数
           map.put("code", code);
           request.putQueryParameter("TemplateParam", JSONObject.toJSONString(map));
   
           try {
               CommonResponse response = client.getCommonResponse(request);
               System.out.println(response.getData());
               return response.getHttpResponse().isSuccess();
        } catch (ServerException e) {
               e.printStackTrace();
           } catch (ClientException e) {
               e.printStackTrace();
           }
           return false;
       }
   ```
   
   