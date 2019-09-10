---
title: 'liquibase'
date: 2018-12-16 21:06:36
tags:
    - liquibase 사용법
---

# 개념  
database schema 변경을 tracking 하여 관리할 수 있게 해주는 open source이다.  
liquibase 문법에 맞춰 xml(yml) 파일을 작성한 뒤 liquibase 를 실행하면 해당 파일의 내용이 데이터베이스에 반영된다.  
반영 방법은 command line, maven, gradle 등 다양한 방법으로 사용가능하다.  
(gradle은 `liquibase-gradle-plugin`을 사용하면 된다)   

> 이 라이브러리를 사용한 이후 부터는 데이터베이스 스키마를 직접 수정하는 일은 지양(금지)해야 한다.  

그리고 어느 라이브러리에서 추가해준 task인지는 모르곘으나..(jhipster애서 generate 된걸 그대로 사용하다보니...)  
gradle에 있는 liquibase 관련 task들 중 `liquibaseDiffChangeLog`를 실행하면 프로젝트에 작성한 entity와 데이터베이스 내 스키마를 참고하여  
다른 부분을 liquibase 문법의 xml 파일로 생성해준다.  
(diff가 완벽하게 생성되지는 않으니, xml 파일을 다시 보면서 잘못된 부분이 없나 최종적으로 확인해야한다. 직접 작성해도 됨)

# 문법
아래는 liquibase의 간단한 예제이다.  

```xml
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd
        http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd">

    <preConditions>
        <runningAs username="liquibase"/>
    </preConditions>

    <changeSet id="1" author="nvoxland">
        <createTable tableName="person">
            <column name="id" type="int" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="firstname" type="varchar(50)"/>
            <column name="lastname" type="varchar(50)">
                <constraints nullable="false"/>
            </column>
            <column name="state" type="char(2)"/>
        </createTable>
    </changeSet>

    <changeSet id="2" author="nvoxland">
        <addColumn tableName="person">
            <column name="username" type="varchar(8)"/>
        </addColumn>
    </changeSet>    
</databaseChangeLog>
```

`changeSet`에 어떤 행위들을 할 지 나열되어 있다.(createTable, addColumn 등)  

이 파일을 liquibase로 실행하면 실제 데이터베이스에 반영될 것이다.  
그리고 변경사항이 반영되면 `DATABASECHANGELOG`라는 테이블에(liquibase에서 사용하는 테이블) 위의 `changeLog id값`으로 로우가 쌓이게 된다.  
이 말은 해당 변경사항은 적용되었다는 뜻이다.  
liquibase는 이 테이블 로우를 참고하여 changeSet이 이미 반영되었으면 skip, 반영되지 않았다면 반영한다.  

## sql 쿼리를 그대로 적용하는법  
```xml
<changeSet author="junyoung.park (generated)" id="20190307124000-7">
    <sql>
        CREATE TABLE BATCH_STEP_EXECUTION_SEQ (
            ID BIGINT NOT NULL,
            UNIQUE_KEY CHAR(1) NOT NULL,
            constraint UNIQUE_KEY_UN unique (UNIQUE_KEY)
        ) ENGINE=InnoDB;
    </sql>
</changeSet>
```

# 여러 파일 적용
한번에 여러 changelog 파일을 실행할 수 있다.  

```xml
<?xml version="1.0" encoding="utf-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.5.xsd">

    <include file="config/liquibase/changelog/00000000_initial_schema.xml" relativeToChangelogFile="false"/>
    <include file="config/liquibase/changelog/2018103001_added_spring_acl_schema.xml" relativeToChangelogFile="false"/>
    <include file="config/liquibase/changelog/2018122001_refactor-column-length.xml" relativeToChangelogFile="false"/>
</databaseChangeLog>
```

이 파일을 liquibase로 실행하면 위에서부터 순서대로 반영한다.  

# 주의사항
liquibase는 변경사항을 반영하기 전에 `DATABASECHANGELOGLOCK` 테이블에 LOCKED=1 인 상태로 레코드를 넣고 락을 건다  
일반적인 상황에서는 문제되지 않지만, 한번에 서버를 2개 이상 띄워야 할 경우 데드락이 발생할 수 있으니 작성전에 이 레코드가 들어있는지 체크해보면 좋다  
혹은 잘 뜨지 않는다면 이 테이블에 데이터가 있는지 체크해보자..  

<!-- more -->