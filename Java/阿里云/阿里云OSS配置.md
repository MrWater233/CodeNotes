# 依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>2.8.3</version>
</dependency>
```

# 后端上传方式

1. `application.yaml`

   ```yaml
   aliyun:
     endpoint: xxx
     accessKeyId: xxx
     accessKeySecret: xxx
     oss:
       bucketName: xxx
   ```

2. `com.weixin.room.util.OssUtil`

   ```java
   import com.aliyun.oss.OSSClient;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Component;
   import org.springframework.web.multipart.MultipartFile;
   
   import java.io.IOException;
   import java.net.URL;
   import java.text.SimpleDateFormat;
   import java.util.Date;
   import java.util.UUID;
   
   /**
    * @author WanJingmiao
    * @Description
    * @date 2020/5/18 11:11
    * @Version
    */
   @Component
   public class OssUtil {
       @Value("${aliyun.endpoint}")
       private String endpoint;
       @Value("${aliyun.accessKeyId}")
       private String accessKeyId;
       @Value("${aliyun.accessKeySecret}")
       private String accessKeySecret;
       @Value("${aliyun.oss.bucketName}")
       private String bucketName;
   
       public String upload(MultipartFile file) throws IOException {
           String fileName = file.getOriginalFilename();
           SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
           System.out.println(fileName);
           //获取文件后缀名
           String suffixName = fileName.substring(fileName.lastIndexOf("."));
           //生成上传的文件名
           String objectName = sdf.format(new Date()) + "/" + UUID.randomUUID().toString() + suffixName;
           OSSClient ossClient = new OSSClient(endpoint, accessKeyId, accessKeySecret);
           ossClient.putObject(bucketName, objectName, file.getInputStream());
           //设置有效期为10年
           Date expiration = new Date(System.currentTimeMillis() + 3600 * 1000 * 24 * 3650);
           URL url = ossClient.generatePresignedUrl(bucketName, objectName, expiration);
           ossClient.shutdown();
           return url.toString();
       }
   }
   ```

# 后端签名方式