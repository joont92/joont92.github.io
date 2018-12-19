---
title: liquibase
date: 2018-12-16 21:06:36
tags:
    - liquibase 사용법
---

초기 셋팅법은 정확하게 모르고, 기본적인 사용법만 아는 상태임.  
(jhipster에서 자동으로 liquibase 관련 설정까지 generate 해줬었다)  
<https://www.liquibase.org/>  

# 개념
gradle에 있는 liquibase 관련 task들 중  
`liquibaseDiffChangeLog`를 실행하면 liquibase 설정에 작성한 jdbc정보를 참고하여 해당 데이터베이스와 프로젝트의 entity들을 서로 비교하고, 서로 다른 부분을 xml 파일로 떨궈준다.  
파일 내용 형태는 아래와 같다.  

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

보다시피 `changeSet` 안에 어떤 행위들을 할 지 나열되어 있다.(createTable, addColumn 등)  

diff가 완벽하게 생성되지는 않으니, xml 파일을 다시 보면서 잘못된 부분이 없나 최종적으로 확인해야한다.  

# 적용(jhipster 프로젝트)
확인이 완료되면 `master.xml`에 해당 파일의 경로를 아래와 같이 추가한다.  
(파일명은 적절히 수정한다)  

```xml
<include file="config/liquibase/changelog/20181212-modify-somthing.xml" relativeToChangelogFile="false"/>
```

이후 spring boot 프로젝트드를 다시 실행하면 위의 파일이 같이 실행되면서 변경내용이 실제 db에 반영된다.  
즉, 데이터베이스 스키마에 변경이 필요할 경우 직접 스키마를 수정해선 안되고 entity를 수정해서 liquibase를 통해 반영하는 형태로 진행해야 한다.  

변경사항이 반영되면 `DATABASECHANGELOG`라는 테이블에(liquibase에서 사용하는 테이블) 위의 `changeLog id값`으로 로우가 쌓이게 된다.  
이 말은 해당 변경사항은 적용되었다는 뜻이다.  

liquibase는 이 테이블을 참고하여 테이블에 추가적으로 alter를 해야하는지 하지말아야 하는지를 결정한다.  



<!-- more -->