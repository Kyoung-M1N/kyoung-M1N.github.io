---
title: S3에 파일 업로드 하기
author: kymin
date: 2024-07-27 17:41
categories: [Aws]
tags: [aws, spring]


---

## Amazon S3?

AWS S3는 아마존이 제동하는 객체 스토리지 서비스로 데이터가 저장되는 공간을 버킷(bucket)이라고 하며, 쉽게 말해 외부 저장공간이라고 할 수 있다.

스토리지 서비스이기 때문에 `ls`, `mv`, `cp`, `rm` 등 파일과 관련된 명령어만 사용이 가능하다.

## 스프링에서 파일 업로드

### 설정

스프링에서 S3의 버킷에 파일을 업로드 하기 위해서는 스프링이 버킷에 접근할 수 있어야 한다.

따라서 버킷을 생성하거나 이미 생성된 버킷에 대해 ACL을 활성화하고 퍼블릭 엑세스를 허용하여, S3만 객체 소유권을 가질 수 있도록 되어있는 설정을 해제해야 한다.

또한 IAM에서 AccessKey와 SecretKey를 발급받고 해당 키에 권한을 부여하여 키값을 가진 사용자만 접근할 수 있도록 해야한다.

> 자세한 방법에 참고자료의 게시물에 나와있다.

### 구현

- build.gradle에 의존성 설정

  build.gradle에 AWS의 기능을 사용하기 위해 SDK의 라이브러리에 대한 의존성을 추가한다.

  ```groovy
  dependencies {
      ...
      implementation 'software.amazon.awssdk:s3:<version>'
    	...
  }
  ```

- application.yml에 리소스 입력

  application.yml에 AWS의 기능을 사용하는 데에 필요한 값들을 넣어놓는다.

  ```yaml
  ...
  aws:
    region: [AWS S3의 Region]
    access-key: [AWS IAM에서 발급받은 Access Key]
    secret-key: [AWS IAM에서 발급받은 Secret Key]
    s3:
      bucket-name: [S3 버킷 이름]
  ...
  ```
  
  
  
- 코드 구현

  먼저 AWS S3의 기능을 사용하기 위해 credential을 설정해주는 config파일을 만들고 `S3Client`를 빈으로 등록해준다.

  ```java
  @Configuration
  public class S3Config {
  	@Value("${aws.access-key}")
  	private String accessKey;
  
  	@Value("${aws.secret-key}")
  	private String secretKey;
  
  	@Value("${aws.region}")
  	private String region;
  
  	@Bean
  	public S3Client s3Client(){
  		AwsBasicCredentials awsBasicCredentials = AwsBasicCredentials.create(accessKey, secretKey);
  		return S3Client.builder()
  				.credentialsProvider(StaticCredentialsProvider.create(awsBasicCredentials))
  				.region(Region.of(region))
  				.build();
  	}
  }
  ```
  
  다음으로 빈으로 등록된 config를 이용하여 S3에 대한 기능을 코드로 작성한다.
  
  ```java
  @Component
  @RequiredArgsConstructor
  public class ImageUploader {
  	private static final List<String> ALLOWED_EXTENSIONS = Arrays.asList("jpg", "jpeg", "png");
  	private final S3Client s3Client;
  
  	@Value("${aws.s3.bucket-name}")
  	private String bucketName;
  
  	// https://tao-tech.tistory.com/27
  
  	public List<String> upload(final long memberId, final String contractId, final List<MultipartFile> images) {
  		validateFiles(images);
  
  		return IntStream.range(0, images.size())
  				.mapToObj(idx -> {
                      // S3에 저장하기 위한 적절한 파일명 생성
  					String filenameToUpload = memberId + "-" + contractId + "-" + idx;
  					return uploadToS3(filenameToUpload, images.get(idx));
  				})
  				.toList();
  	}
  
  	private void validateFiles(final List<MultipartFile> images) {
  		images.forEach(image -> {
  			String fileName = image.getOriginalFilename();
  
  			// 파일 존재 유무 검증
  			if (fileName == null || fileName.isEmpty()) {
  				throw new ImageFileNotFoundException();
  			}
  
  			// 파일 확장자 존재 유무 검증
  			if (fileName.lastIndexOf(".") == -1) {
  				throw new FileExtensionNotFoundException();
  			}
  
  			if (!ALLOWED_EXTENSIONS.contains(fileName.substring(fileName.lastIndexOf(".") + 1))) {
  				throw new InvalidFileExtensionException();
  			}
  		});
  	}
  
  	private String uploadToS3(final String filenameToUpload, MultipartFile image) {
  
  		String extension = Objects.requireNonNull(image.getOriginalFilename())
  				.substring(image.getOriginalFilename().lastIndexOf(".") + 1);
  
  		try (InputStream inputStream = image.getInputStream()) {
  			PutObjectRequest putObjectRequest = PutObjectRequest.builder()
  					.bucket(bucketName)
  					.key(filenameToUpload)
  					.acl(ObjectCannedACL.PUBLIC_READ)
  					.contentType("image/" + extension)
  					.contentLength(image.getSize())
  					.build();
  			s3Client.putObject(putObjectRequest, RequestBody.fromInputStream(inputStream, image.getSize()));
  		} catch (Exception e) {
  			throw new RuntimeException(e);
  		}
  
  		return s3Client.utilities()
  				.getUrl(url -> url.bucket(bucketName).key(filenameToUpload)).toString();
  	}
  }
  ```
  
  이 때, S3에 저장하기 위해 적절한 파일명을 생성하여 지정해주어야 한다. 보통은 UUID를 통해 생성하고 데이터베이스에 별도로 저장하는 방식을 사용한다.
  
  또한 S3에 객체를 전달할 때에는 클라이언트로부터 전달받은 `MultipartFile`타입의 객체를 전달하는 것이 아니라 파일의 `InputStream`을 전달한다.
  
  > `MultipartFile`타입의 객체를 `File`로 변환한 뒤에 `File`타입의 객체를 전달하는 방법도 있지만 파일을 전달하기 위한 임시파일을 생성해야 하므로 효율성이 떨어지게 된다.
  >
  > `InputStream`을 통해 버킷에 파일을 업로드 하기 위해서는 반드시 `metadata`를 생성해야 했지만 업데이트 이후 `PutObjectRequest`가  `contentLength`와 같은  `metadata`를 포함하도록 변경되었다.
  
  버킷에 이미 같은 이름의 파일이 존재하면 해당 파일을 덮어쓰기 처리되기 때문에 다르게 저장되어야 하는 파일의 이름이 중복되지 않도록 유의해야 한다.
  
  또한 삭제 시 해당 파일이 버킷에 존재하지 않더라도 에러 메시지 대신 성공 메시지가 발생한다.



---

##### 참고자료 :

[https://velog.io/@yes_jihyeon](https://velog.io/@yes_jihyeon/AWS-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%A0%80%EC%9E%A5%EC%9D%84-%EC%9C%84%ED%95%9C-S3-%EB%B2%84%ED%82%B7-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0-SpringBootGradle)

[https://www.catsriding.com](https://www.catsriding.com/posts/aws-s3-file-upload-with-spring)

[https://antstudy.tistory.com](https://antstudy.tistory.com/308)

[https://ittrue.tistory.com](https://ittrue.tistory.com/167)