---
title: presigned url
date: 2019-02-08 18:58:38
tags:
    - aws presigned url
---

서버에서 직접 multipart를 받아 S3 버킷에 업로드하면 서버쪽에서 multipart 파일을 쥐고 있어야 하는 둥 리소스 낭비가 크므로 S3에 접근할 수 있는 Presigned url을 생성하여 클라이언트가 직접 업로드하게 한다.  

<https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/PresignedUrlUploadObjectJavaSDK.html>  
presigned url 이란 AWS S3 버킷에 바로 파일을 업로드 할 수 있는 URL을 말한다.  

# 생성 및 사용
java sdk는 maven repository에서 간단하게 검색 가능하다  
<https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk>  

위 라이브러리를 임포트 하고 아래와 같이 작성하면 된다  

```java
@Autowired // spring 사용 시 DI 가능하다!
private final AmazonS3 s3Client;

@Value("${aws.s3.bucket}")
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

이렇게 생성된 presigned url에 PUT 요청으로 multipart data를 보내면 파일을 S3에 업로드할 수 있게 된다.  
업로드는 expiration time 동안 유효하다.  

> 참고로 위의 소스에는 접근하는 S3에 대한 인증정보가 없는데, 이는 `AmazonS3`에서 사용하는 `ProfileCredentialProvider` 때문이다.  
> 이 클래스는 저장된 credential을 읽어서 인증하는데, 실행되는 환경에 따라 이 credential 을 찾는 우선순위가 있다.  
> (ec2에서 실행되면 ec2에 지정된 credential, pc에 저장된 credential 등등)  

# 업로드한 파일 접근
업로드 이후 presigned-url에서 query param을 제거하고 GET 요청하면 해당 파일을 보거나(이미지) 다운로드 받을 수 있는데, 그냥 접근하면 access denied가 발생한다.  
그러므로 presigned url 생성할 때 `public read` 권한을 줘야하고, 아래와 같이 하면 된다.  
<http://zstd.github.io/amazon-presigned-url/>  

```java
GeneratePresignedUrlRequest generatePresignedUrlRequest = new GeneratePresignedUrlRequest(bucketName, objectKey)
        .withMethod(HttpMethod.PUT)
        .withExpiration(expiration);

// 이부분
generatePresignedUrlRequest.addRequestParameter(
            Headers.S3_CANNED_ACL,
            CannedAccessControlList.PublicRead.toString());
```

<!-- more -->