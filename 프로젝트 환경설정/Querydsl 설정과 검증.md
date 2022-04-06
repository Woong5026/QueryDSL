build.gradle에 주석을 참고해서 querydsl 설정 추가

* build.gradle

```java

plugins {
	id 'org.springframework.boot' version '2.5.0'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	//querydsl 추가
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
	id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	//querydsl 추가
	implementation 'com.querydsl:querydsl-jpa'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
configurations {
	querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝

```

스프링 부트 2.6 이상, Querydsl 5.0 지원 방법 메뉴얼


```java

buildscript {
   ext {
      queryDslVersion = "5.0.0"
   }
}

plugins {
   id 'org.springframework.boot' version '2.6.0'
   id 'io.spring.dependency-management' version '1.0.11.RELEASE'
   //querydsl 추가
   id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
   id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
   compileOnly {
      extendsFrom annotationProcessor
   }
}

repositories {
   mavenCentral()
}

dependencies {
   implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
   implementation 'org.springframework.boot:spring-boot-starter-web'
   compileOnly 'org.projectlombok:lombok'
   runtimeOnly 'com.h2database:h2'
   //querydsl 추가
   implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
   implementation "com.querydsl:querydsl-apt:${queryDslVersion}"


   annotationProcessor 'org.projectlombok:lombok'
   testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
   useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"

querydsl {
   jpa = true
   querydslSourcesDir = querydslDir
}
sourceSets {
   main.java.srcDir querydslDir
}
compileQuerydsl{
   options.annotationProcessorPath = configurations.querydsl
}
configurations {
   compileOnly {
      extendsFrom annotationProcessor
   }
   querydsl.extendsFrom compileClasspath
}
//querydsl 추가 끝

```

#### Querydsl 환경설정 검증

* Hello

```java

@Entity
@Getter
@Setter
public class Hello {

    @Id @GeneratedValue
    private Long id;
}

```

검증용 Q 타입 생성

Gradle IntelliJ 사용법

Gradle -> Tasks -> other -> compileQuerydsl (더블 클릭)

![image](https://user-images.githubusercontent.com/78454649/155884912-18b3b63c-7c71-4496-a56c-16fbafbfb0bb.png)

Gradle 콘솔 사용법

프로젝트 디렉토리 이동 -> ./gradlew compileQuerydsl

 <br/><br/>

Q 타입 생성 확인

build -> generated -> querydsl -> study -> entity -> QHello.java 파일이 생성되어 있어야 함

 <br/><br/>

+) 참고

Q타입은 컴파일 시점에 자동 생성되므로 버전관리(Git)에 포함하지 않는 것이 좋다.

앞서 설정에서 생성 위치를 gradle build 폴더 아래 생성되도록 했기 때문에 git에 포함하지 않도록 했다.


<br/>

---

+) unknown Entity

테스트 시 persist 에 unknown Entity Hello 하고 에러가 뜨면 

```java

@SpringBootApplication
@EntityScan(basePackages = {"entity"}) // 이 부분 추가
public class PracQueryDslApplication {

```

스프링 부트에서 패키지 스캔을 하는데, @EntityScan의 기본 패키지 설정 위치가 @SpringBootApplication이 있는 위치라고 생각





