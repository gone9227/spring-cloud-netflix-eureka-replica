# spring-cloud-netflix-eureka-replica

## Spring boot + Spring Cloud Gateway + Eureka

### Eureka란?

- Eureka는 **AWS와 같은 Cloud 시스템에서 서비스의 로드 밸런싱과 실패처리 등을 유연하게 가져가 위해 각 서비스들의 IP / Port / InstanceId를 가지고 있는 REST 기반의 미들웨어 서버**
- **Eureka는 마이크로 서비스 기반의 아키텍처의 핵심 원칙 중 하나인 Service Discovery의 역할을 수행**합니다. **MSA에서는 Service의 IP와 Port가 일정하지 않고 지속적으로 변화**. 그렇기 때문에 Client에 Service의 정보를 수동으로 입력하는 것은 한계가 있음. Service Discovery란 이런 MSA의 상황에 적합함.

### Eureka Server

- Service Discovery
- Eureka Replica 모드 - Peer Awareness
    - Eureka 서버의 고가용성, Failover에 대비해 Eureka 인스턴스를 두 개 이상 띄울 수 있음
    - 주의할 점 (If unavailable-replicas로 나오면 확인할 점)
        - Hostname이 같으면 Replica 모드가 동작하지 않음 (같은 호스트에서 테스트하고자 하면 호스트에 2개 이상의 Alias를 주어야 함)
        - Application 명이 달라야 함
        - Eureka Replica 모드는 마스터와 서브 구조가 아닌 Peer들 끼리 서로를 바라보고 있는 구조임. 서로 하나의 연결 고리만 있어도 다 Replication이 가능함
    - 구현하기
        1. build.gradle 에 디펜던시 추가
            
            ```groovy
            implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
            ```
            
        2. hosts 파일에 host의 alias 추가해주기
            - 윈도우엔 `C:\Windows\System32\drivers\etc` 에 위치
            
            ```html
            127.0.0.1       eureka1
            127.0.0.1       eureka2
            ```
            
        3. application.yml 설정
            
            ```yaml
            management:
              endpoint:
                health:
                  enabled: true
                  show-details: always
                shutdown:
                  enabled: true
              endpoints:
                web:
                  base-path: /
                  # By default, only 'health' and 'info' are accessible via web
                  exposure:
                    include: '*'
            
            ---
            # Eureka Instance 1
            spring:
              profiles: eureka1
            
            server:
              port: 8787
              enableSelfPreservation: false
            
            # Eureka
            eureka:
              instance:
                appName: EurekaReplicaTestApp
                hostname: eureka1
              client:
                register-with-eureka: true
                fetch-registry: true
                registryFetchIntervalSeconds: 10
                serviceUrl:
           # 서로의 Host:port 를 설정함
                  defaultZone: http://eureka2:8788/eureka/
            
            ---
            # Eureka Instance 2
            spring:
              profiles: eureka2
            
            server:
              port: 8788
              enableSelfPreservation: false
            
            # Eureka
            eureka:
              instance:
                appName: EurekaReplicaTestApp
                hostname: eureka2
              client:
                register-with-eureka: true
                fetch-registry: true
                registryFetchIntervalSeconds: 10
                serviceUrl:
                  defaultZone: http://eureka1:8787/eureka/
            ```
            
        4. [Application.java](http://Application.java) 에 @EnableEurekaServer 추가

### Eureka Client

1. build.gradle 추가
    
    ```groovy
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    ```
    
2. Application.java에 @EnableDiscoveryClient 추가
3. application.yml 설정
    
    ```yaml
    eureka:
      client:
        # eureka 서버에 등록할지 여부
        register-with-eureka: true
        fetch-registry: true
        serviceUrl:
        # Eureka Server 주소 입력
          defaultZone: http://eureka1:8787/eureka/, http://eureka2:8788/eureka/
        healthcheck:
          enabled: true
        refresh:
          enabled: true
    ```
    

### References

[https://thepracticaldeveloper.com/spring-boot-service-discovery-eureka/](https://thepracticaldeveloper.com/spring-boot-service-discovery-eureka/)
