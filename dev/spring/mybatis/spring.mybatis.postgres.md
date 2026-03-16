# Mybatis: postgres 연동

- [Mybatis: postgres 연동](#mybatis-postgres-연동)
  - [build.gradle](#buildgradle)
    - [dependency](#dependency)
    - [version 호환성](#version-호환성)
  - [application.yml](#applicationyml)
  - [Model :: ActorDto.java](#model--actordtojava)
  - [Controller :: DvdRentalController.java](#controller--dvdrentalcontrollerjava)
  - [Service :: ActorService.java](#service--actorservicejava)
  - [DAO(Mapper:Java) :: DvdRentalDao.java](#daomapperjava--dvdrentaldaojava)
  - [Mapper(Xml) :: resource/mapper/DvdRentalDao.xml](#mapperxml--resourcemapperdvdrentaldaoxml)
  - [참고](#참고)

---

## build.gradle

```gradle
plugins {
 id 'org.springframework.boot' version '3.2.2'
}

dependencies {
 implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.3'
}
```

### dependency

springboot 프로젝트에서 mybatis 사용을 위해서는 기본적으로 mybatis-spring-boot-starter 라이브러리만 추가하면 된다.
mybatis starter 라이브러리는 mybatis 사용에 필요한 라이브러리 dependency를 자동으로 추가하도록하는 auto-configuration이 포함되어 있다.

### version 호환성

springframework '3.2.2' 버전에는 mybatis starter '3.0.3' 버전이 호환된다.
호환 버전이 맞지 않으면 다음의 오류가 발생할 수 있다.
> java.lang.IllegalArgumentException: Invalid value type for attribute 'factoryBeanObjectType'

## application.yml

```yml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
    use-generated-keys: true
  mapper-locations: classpath:mapper/**/*.xml

spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/dvdrental
    username: postgres
    password: postgres
```

## Model :: ActorDto.java

```java
package com.example.springwebex.model;

import java.time.LocalDateTime;
import lombok.Data;

@Data
public class ActorDto {
    Integer actorId;
    String firstName;
    String lastName;
    LocalDateTime lastUpdate;
}
```

## Controller :: DvdRentalController.java

```java
package com.example.springwebex.controller;

import java.util.List;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import com.example.springwebex.model.ActorDto;
import com.example.springwebex.service.ActorService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;

@RestController
@RequiredArgsConstructor
@RequestMapping("/dvdrental")
@Slf4j
public class DvdRentalController {
    private final ActorService actorService;

    @GetMapping("/actors")
    public List<ActorDto> getActors() {
        var result = actorService.getActors();
        log.info("{}", result);
        return result;
    }

    @GetMapping("/actor/{id}")
    public ActorDto getActorById(@PathVariable("id") Long id) {
        var result = actorService.getActorById(id);
        log.info("getActors{}", result);
        return result;
    }

    @GetMapping("/actor/insert/{firstName}/{lastName}")
    public void setActor(@PathVariable("firstName") String firstName,
            @PathVariable("lastName") String lastName) {
        actorService.setActor(firstName, lastName);
        log.info("insertActor()");
    }
}
```

## Service :: ActorService.java

```java
package com.example.springwebex.service;

import java.util.List;
import org.springframework.stereotype.Service;
import com.example.springwebex.dao.DvdRentalDao;
import com.example.springwebex.model.ActorDto;
import lombok.RequiredArgsConstructor;

@Service
@RequiredArgsConstructor
public class ActorService {
    private final DvdRentalDao dao;

    public List<ActorDto> getActors() {
        return dao.selectActors();
    }

    public ActorDto getActorById(Long id) {
        return dao.selectActorById(id);
    }

    public void setActor(String firstName, String lastName) {
        var actor = new ActorDto();
        actor.setFirstName(firstName);
        actor.setLastName(lastName);
        dao.insertActor(actor);
    }
}
```

## DAO(Mapper:Java) :: DvdRentalDao.java

```java
package com.example.springwebex.dao;

import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import com.example.springwebex.model.ActorDto;

@Mapper
public interface DvdRentalDao {

    List<ActorDto> selectActors();

    @Select("SELECT * FROM actor WHERE actor_id = #{id}")
    ActorDto selectActorById(@Param("id") Long id);

    void insertActor(ActorDto actorDto);
}
```

## Mapper(Xml) :: resource/mapper/DvdRentalDao.xml

```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.springwebex.dao.DvdRentalDao">

 <select id="selectActors">
  SELECT * FROM actor
 </select>

 <insert id="insertActor" parameterType="com.example.springwebex.model.ActorDto">
  INSERT INTO
      actor (first_name , last_name)
  VALUES (#{firstName}, #{lastName})
 </insert>

</mapper>
```

## 참고

- mybatis starter github
  <https://github.com/mybatis/spring-boot-starter>
- mybatis docs
  <https://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure>
- mybatis starter maven repos page
  <https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/3.0.3>
