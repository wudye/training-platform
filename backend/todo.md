spring:
cloud:
gateway:
routes:
- id: example_route
uri: http://localhost:8080
predicates:
- Path=/example/**
filters:
- name: CircuitBreaker
args:
name: myCircuitBreaker
fallbackUri: forward:/fallback
resilience4j:
circuitbreaker:
instances:
myCircuitBreaker:
registerHealthIndicator: true
slidingWindowSize: 10
minimumNumberOfCalls: 5
permittedNumberOfCallsInHalfOpenState: 3
waitDurationInOpenState: 10s
failureRateThreshold: 50

@RestController
public class FallbackController {
@RequestMapping("/fallback")
public ResponseEntity<String> fallback() {
return ResponseEntity.ok("服务暂时不可用，请稍后再试。");
}
}


not in yml
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("example_route", r -> r.path("/example/**")
                .filters(f -> f.circuitBreaker(config -> 
                    config.setName("myCircuitBreaker")
                          .setFallbackUri("forward:/fallback")))
                .uri("http://localhost:8080"))
            .build();
    }
}

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FallbackController {
@RequestMapping("/fallback")
public ResponseEntity<String> fallback() {
return ResponseEntity.ok("服务暂时不可用，请稍后再试。");
}
}




import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.circuitbreaker.CircuitBreakerRegistry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Resilience4jConfig {

    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(java.time.Duration.ofSeconds(10))
            .slidingWindowSize(10)
            .minimumNumberOfCalls(5)
            .permittedNumberOfCallsInHalfOpenState(3)
            .build();
        return CircuitBreakerRegistry.of(config);
    }
}


