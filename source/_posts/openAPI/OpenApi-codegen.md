---
title: OpenApi codegen
date: 2019-01-30 17:29:55
tags:
    - openapi generator
    - openapi codegen gradle plugin
    - openapi 상속
---

# openapi generator
<https://github.com/OpenAPITools/openapi-generator>  

Open Api Spec을 통해 code를 generate 해주는 라이브러리  
server stub, client stub 생성 가능  

# gradle plugin  
<https://github.com/OpenAPITools/openapi-generator/tree/master/modules/openapi-generator-gradle-plugin>  

openapi generator를 gradle에서 사용하기 위한 플러그인  

```groovy
dependencies {
    classpath "org.openapitools:openapi-generator-gradle-plugin:3.3.0"
}
```

```groovy
apply plugin: 'org.openapi.generator'

// 위 repository README의 Configuration 참조  
openApiGenerate {
    generatorName = "kotlin"
    inputSpec = "$rootDir/specs/petstore-v3.0.yaml".toString()
    outputDir = "$buildDir/generated".toString()
    apiPackage = "org.openapi.example.api"
    invokerPackage = "org.openapi.example.invoker"
    modelPackage = "org.openapi.example.model"
    modelFilesConstrainedTo = [
            "Error"
    ]
    configOptions = [
        dateLibrary: "java8"
    ]
}
```

# Tip  
## generate 된 코드에 상속 적용하기  
allOf를 사용하면 generate된 코드에 상속을 사용할 수 있다  

```yml
ChildDTO:
    title: ChildDTO
    description: description
    allOf:
        - $ref: '#/components/schemas/ParentDTO'
        - type: object
            required:
                - name
                - age
            properties:
                name:
                    type: string
                    description: 이름
                    example: ㅋㅋ
                age:
                    type: integer
```

아래와 같이 generate 됨  

```java
class ChildDTO extends ParentDTO{
    String name;
    Integer age;
}
```

<!-- more -->