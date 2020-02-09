# Java를 사용한 AWS Lambda

## 소개
AWS Lambda 는 서버, OS, 확장 성 등의 구성을 줄이기 위해 Amazon에서 제공하는 서버리스 컴퓨팅 서비스입니다. AWS Lambda는 AWS 클라우드에서 코드를 실행할 수 있습니다.
AWS Lambda 기능을 트리거하는 다른 AWS 리소스의 이벤트에 대한 응답으로 실행됩니다.

## Maven Dependency
AWS Lambda를 활성화하려면 프로젝트에 다음과 같은 Maven dependency가 필요합니다.
이 의존성은 Maven repository 에서 찾을 수 있습니다.
```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-core</artifactId>
    <version>1.2.0</version>
</dependency>
```

Lambda Application을 빌드하려면 Maven Shade Plugin 도 필요합니다.
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.4.3</version>
    <configuration>
        <createDependencyReducedPom>false</createDependencyReducedPom>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 핸들러 생성
Lambda function를 호출하려면 handler를 지정해야합니다. handler를 작성하는 3가지 방법이 있습니다.
- Custom MethodHandler 생성
- RequestHandler interface 구현
- RequestStreamHandler interface 구현


## Custom MethodHandler
Request의 진입 점이 되는 handler method를 작성합니다. JSON 형식 또는 기본 데이터 유형을 입력 값으로 사용할 수 있습니다.
또한, Context 객체를 사용하면 Lambda 실행 환경에서 사용 가능한 정보에 액세스 할 수 있습니다.
```java
public class LambdaMethodHandler {
    public String handleRequest(String input, Context context) {
        context.getLogger().log("Input: " + input);
        return "Hello World - " + input;
    }
}
```

## RequestHandler Interface
클래스에 RequestHandler를 구현하고 Lambda의 시작점이 되는 handleRequest method를 대체 할 수 있습니다.
```java
public class LambdaRequestHandler
  implements RequestHandler<String, String> {
    public String handleRequest(String input, Context context) {
        context.getLogger().log("Input: " + input);
        return "Hello World - " + input;
    }
}
```

## RequestStreamHandler 인터페이스
클래스에서 RequestStreamHandler를 구현하고 handleRequest method를 대체 할 수도 있습니다.
차이점은 InputStream, ObjectStream 및 Context 객체가 매개 변수로 전달된다는 것입니다.
```java
public class LambdaRequestStreamHandler
  implements RequestStreamHandler {
    public void handleRequest(InputStream inputStream, OutputStream outputStream, Context context) {
        String input = IOUtils.toString(inputStream, "UTF-8");
        outputStream.write(("Hello World - " + input).getBytes());
    }
}
```


## 배포 파일의 작성
모든 것이 구현된 상태에서 Maven 명령어를 실행하여 배포 파일을 만들 수 있습니다.
Target 폴더 아래에 jar 파일이 생성됩니다.
```
mvn clean package shade:shade
```


## 관리 콘솔을 통해 Lambda 함수 생성
AWS Console에 로그인 하고 서비스에서 Lambda를 클릭하십시오. 
새로운 Lambda를 만듭니다.
1. 새로 작성을 선택
2. 기본 정보
    - 함수 이름 : LambdaTest
    - Runtime : java8 선택
    - 권한 : 다른 AWS 리소스가 Lambda 기능에 사용되는 경우 기존 역할을 생성 / 사용하여 액세스를 제공하고 정책 템플릿을 정의하십시오.
      - 기존 Lambda 권한을 가진 새 역할 생성
      - 기존 역할 사용
      - AWS 정책 템플릿에서 새 역할 생성
3. 함수 생성
4. 함수 코드
    - Code entry type : ".zip 또는 Jar 파일 업로드"를 선택 
    - 업로드 버튼을 클릭 
    - Lambda 코드가 포함 된 파일을 선택
    - 핸들러 : com.lj.lambda.batch.LambdaMain::handleRequest
    - 기본 설정
      - 메모리 : Lambda 함수에서 사용할 메모리를 제공하십시오.
      - 제한 시간 : 각 요청에 대해 Lambda 기능을 실행할 시간을 선택하십시오.
    - 설명 : Lambda 함수를 설명
5. Save를 클릭하여 저장합니다.

## 함수 호출
AWS Lambda 함수가 생성되면 몇 가지 데이터를 생성하여 테스트합니다.
1. 목록에서 Lambda 함수를 클릭 한 다음 테스트 버튼을 클릭하십시오
데이터를 보내기위한 더미 값이 포함된 팝업 창이 나타납니다.
2. "Lambda" 입력
3. 저장 및 테스트 버튼을 클릭
4. 화면에서 다음과 같이 출력이 성공적으로 리턴된 실행 결과를 볼 수 있습니다.
```
Hello World - Lambda
```

## 참고 자료 
- [AWS Lambda](https://aws.amazon.com/ko/lambda/)
- [AWS Lambda With Java 
by baeldung](https://www.baeldung.com/java-aws-lambda)
