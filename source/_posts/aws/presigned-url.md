---
title: presigned url
date: 2019-02-08 18:58:38
tags:
    - aws presigned url
---

<https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/PresignedUrlUploadObjectJavaSDK.html>  
presigned url 이란 AWS S3 버킷에 바로 파일을 업로드 할 수 있는 URL을 말한다.  

java sdk는 maven repository에서 간단하게 검색 가능하다  
<https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk>  

위 라이브러리를 임포트 하고 아래와 같이 작성하면 된다  

```java
@Autowired
private final AmazonS3 s3Client;

@Value("${cloud.aws.s3.bucket}")
private String bucketName;

public String getPresignedUrl(){
    String fileName = UUID.randomUUID().toString();

    Date expiration = new Date();
    long expTimeMillis = expiration.getTime();
    expTimeMillis += 1000 * 60 * 60; // 1시간
    expiration.setTime(expTimeMillis);

    GeneratePresignedUrlRequest generatePresignedUrlRequest = new GeneratePresignedUrlRequest(bucketName, objectKey)
        .withMethod(HttpMethod.PUT)
        .withExpiration(expiration);

    generatePresignedUrlRequest.addRequestParameter(Headers.S3_CANNED_ACL,
        CannedAccessControlList.PublicRead.toString());

    URL url = s3Client.generatePresignedUrl(generatePresignedUrlRequest);

    return url.toExternalForm();
}
```

이렇게 생성된 presigned url에 PUT 요청으로 multipart data를 보내면 바로 파일을 업로드할 수 있게 된다.  



<!-- more -->