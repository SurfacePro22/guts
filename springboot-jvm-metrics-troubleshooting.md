# Spring Boot 2.7.5 JVM 메트릭 누락 트러블슈팅

## 환경
- Spring Boot 2.7.5, JDK 1.8
- Micrometer 1.9.5 (Boot 관리 버전), spring-boot-starter-actuator, micrometer-registry-prometheus
- 동일한 pom.xml / application.properties인데 한 프로젝트만 JVM 메트릭 누락

## 증상
- `/actuator/prometheus`에 `jvm_*`, `process_*`, `system_*` 메트릭이 없음
- `tomcat_*`, `executor_*` 메트릭은 정상 출력
- 참고: `/actuator/metrics` 404는 `exposure.include`에 `metrics`가 없어서일 뿐, 정상 동작

## 원인
**MeterRegistry가 너무 일찍 생성됨 (BeanPostProcessor 등록 단계에서 끌려옴)**

| 메트릭 | 등록 방식 | 조기 생성 시 |
|---|---|---|
| `jvm_*`, `process_*`, `system_*`, `logback_*` | `MeterBinder` Bean → `MeterRegistryPostProcessor`가 레지스트리 생성 시점에 바인딩 | **누락** |
| `tomcat_*` | `TomcatMetricsBinder`가 기동 완료 후 직접 바인딩 | 정상 |
| `executor_*` | 싱글톤 생성 완료 후 직접 바인딩 | 정상 |

레지스트리가 조기 생성되면 `MeterRegistryPostProcessor`가 적용되지 않아 MeterBinder, 공통 태그(`management.metrics.tags.*`), MeterFilter, MeterRegistryCustomizer가 모두 빠진다.

### 실제 원인 코드 (Spring Batch 설정)
```java
private final JobRegistry jobRegistry;   // 생성자 주입

@Bean
public JobRegistryBeanPostProcessor jrbpp() {   // non-static BPP
    JobRegistryBeanPostProcessor pp = new JobRegistryBeanPostProcessor();
    pp.setJobRegistry(jobRegistry);
    return pp;
}
```
non-static `@Bean` BeanPostProcessor → 설정 클래스 인스턴스를 먼저 생성 → 생성자 의존성 체인이 조기 생성되고, 이 체인 어딘가에서 MeterRegistry까지 조기 생성됨.

## 해결
`static` 메서드로 바꾸고 의존성은 메서드 파라미터로 받는다.
```java
@Bean
public static JobRegistryBeanPostProcessor jobRegistryBeanPostProcessor(JobRegistry jobRegistry) {
    JobRegistryBeanPostProcessor pp = new JobRegistryBeanPostProcessor();
    pp.setJobRegistry(jobRegistry);
    return pp;
}
```
- 그래도 경고가 남으면 파라미터에 `@Lazy` 추가
- 다른 곳에서 안 쓰면 `jobRegistry` 필드는 제거 가능

## 진단 방법
1. `/actuator/prometheus`에서 어떤 메트릭 그룹이 빠졌는지 확인
   - jvm/process/system 전부 없음 → 레지스트리 조기 생성
   - jvm만 없음 → `JvmMetricsAutoConfiguration` exclude, `MeterFilter` deny, `management.metrics.enable.jvm=false`
2. 기동 로그에서 확인
   `Bean '...MeterRegistry' ... is not eligible for getting processed by all BeanPostProcessors`
3. 원인을 정확히 찾으려면 `PrometheusMeterRegistry` 생성자에 브레이크포인트를 걸고 스택 트레이스에서 생성 중인 BeanPostProcessor 확인
4. 필요 시 `/actuator/conditions`, `/actuator/beans` 노출해서 자동 설정과 Bean 존재 여부 확인

## 조기 생성의 흔한 원인 패턴
| 패턴 | 해결 |
|---|---|
| non-static `@Bean` BeanPostProcessor / BeanFactoryPostProcessor (`JobRegistryBeanPostProcessor`, `MapperScannerConfigurer`, `PropertySourcesPlaceholderConfigurer` 등) | `static` 메서드로 변경 |
| 커스텀 BPP가 `@Autowired`로 서비스, DataSource 등을 주입받음 | `ObjectProvider<T>` 또는 `@Lazy` 사용 |
| `GlobalMethodSecurityConfiguration`, `CachingConfigurer`, `AsyncConfigurer` 구현체가 서비스 Bean을 주입받음 | `@Lazy` 또는 `ObjectProvider` 사용 |

## 확인
재기동 후 `/actuator/prometheus`에 `jvm_memory_used_bytes`, `process_cpu_usage`, `system_cpu_usage`가 출력되면 해결.
