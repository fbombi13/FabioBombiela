---
title: 'Performance Testing Best Practices: Gatling, JMeter, and Locust'
date: 2024-09-10
permalink: /posts/2024/09/performance-testing-best-practices/
tags:
  - performance-testing
  - gatling
  - jmeter
  - locust
  - load-testing
excerpt: 'A comprehensive guide to choosing and using the right performance testing tool for your needs.'
---

## Introduction

Performance testing is critical for ensuring your application can handle real-world load. In this guide, I'll share best practices from my experience using Gatling, JMeter, and Locust for enterprise performance testing.

## Choosing the Right Tool

Each tool has its strengths. Here's when to use each:

### Gatling
**Best for:** High-performance scenarios, complex user journeys

**Pros:**
- Minimal resource overhead
- Scala DSL for powerful scripting
- Beautiful HTML reports
- Excellent for CI/CD integration

**Cons:**
- Steeper learning curve (Scala)
- Limited protocol support vs JMeter

### JMeter
**Best for:** Protocol-level testing, extensive plugin ecosystem

**Pros:**
- Supports many protocols (HTTP, JDBC, LDAP, SOAP, etc.)
- Huge plugin library
- GUI for test development
- Large community

**Cons:**
- Higher resource consumption
- Can be complex for simple scenarios

### Locust
**Best for:** Python developers, distributed testing

**Pros:**
- Python-based (easy to learn)
- Great for distributed load generation
- Real-time web UI
- Simple to extend

**Cons:**
- Less mature than JMeter/Gatling
- Fewer built-in features

## Getting Started with Each Tool

### Gatling Example

```scala
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import scala.concurrent.duration._

class BasicSimulation extends Simulation {
  
  val httpProtocol = http
    .baseUrl("https://api.example.com")
    .acceptHeader("application/json")
    .userAgentHeader("Gatling Performance Test")
  
  val scn = scenario("API Load Test")
    .exec(
      http("Get Users")
        .get("/users")
        .check(status.is(200))
    )
    .pause(1.second)
    .exec(
      http("Create User")
        .post("/users")
        .body(StringBody("""{"name": "Test User"}"""))
        .check(status.is(201))
    )
  
  setUp(
    scn.inject(
      rampUsers(100) during (60.seconds),
      constantUsersPerSec(50) during (5.minutes)
    )
  ).protocols(httpProtocol)
}
```

### JMeter Example

While JMeter is typically GUI-based, you can also script it:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2">
  <hashTree>
    <TestPlan>
      <stringProp name="TestPlan.comments">API Load Test</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <ThreadGroup>
        <stringProp name="ThreadGroup.num_threads">100</stringProp>
        <stringProp name="ThreadGroup.ramp_time">60</stringProp>
        <HTTPSamplerProxy>
          <stringProp name="HTTPSampler.domain">api.example.com</stringProp>
          <stringProp name="HTTPSampler.path">/users</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
        </HTTPSamplerProxy>
      </ThreadGroup>
    </TestPlan>
  </hashTree>
</jmeterTestPlan>
```

### Locust Example

```python
from locust import HttpUser, task, between

class APIUser(HttpUser):
    wait_time = between(1, 3)
    
    @task(3)
    def get_users(self):
        self.client.get("/users")
    
    @task(1)
    def create_user(self):
        self.client.post("/users", json={
            "name": "Test User",
            "email": "test@example.com"
        })
    
    def on_start(self):
        # Login or setup
        self.client.post("/login", json={
            "username": "testuser",
            "password": "password"
        })
```

## Performance Testing Best Practices

### 1. Define Clear Objectives

Before testing, answer:
- What is the expected load? (users, requests/sec)
- What are acceptable response times?
- What is the breaking point?
- Which scenarios are most critical?

### 2. Model Realistic User Behavior

```python
# Bad: Constant hammering
for i in range(1000):
    make_request()

# Good: Realistic think time
class RealisticUser(HttpUser):
    wait_time = between(2, 5)  # Think time
    
    @task
    def browse_products(self):
        self.client.get("/products")
        time.sleep(random.uniform(3, 7))  # Reading time
        self.client.get(f"/products/{random.randint(1, 100)}")
```

### 3. Ramp Up Gradually

```scala
// Gatling ramp-up strategy
setUp(
  scn.inject(
    nothingFor(4.seconds),
    atOnceUsers(10),
    rampUsers(50) during (30.seconds),
    constantUsersPerSec(20) during (5.minutes),
    rampUsersPerSec(20) to 50 during (2.minutes),
    constantUsersPerSec(50) during (10.minutes)
  )
)
```

### 4. Monitor System Resources

```python
import psutil

def monitor_resources():
    cpu_percent = psutil.cpu_percent(interval=1)
    memory = psutil.virtual_memory()
    disk = psutil.disk_io_counters()
    
    return {
        'cpu': cpu_percent,
        'memory_used': memory.percent,
        'disk_read': disk.read_bytes,
        'disk_write': disk.write_bytes
    }
```

### 5. Analyze Results Properly

Key metrics to track:
- **Response Time Percentiles** (P50, P95, P99)
- **Throughput** (requests/second)
- **Error Rate** (%)
- **Resource Utilization** (CPU, Memory, Network)

```python
def analyze_results(results):
    response_times = [r.response_time for r in results]
    
    metrics = {
        'p50': np.percentile(response_times, 50),
        'p95': np.percentile(response_times, 95),
        'p99': np.percentile(response_times, 99),
        'mean': np.mean(response_times),
        'max': np.max(response_times),
        'throughput': len(results) / total_time,
        'error_rate': sum(r.failed for r in results) / len(results)
    }
    
    return metrics
```

## Advanced Techniques

### Distributed Load Generation

**Locust distributed setup:**

```bash
# Master
locust -f locustfile.py --master

# Workers (on multiple machines)
locust -f locustfile.py --worker --master-host=<master-ip>
```

**Gatling distributed with Docker:**

```yaml
# docker-compose.yml
version: '3'
services:
  gatling-master:
    image: denvazh/gatling
    command: -s MySimulation
    
  gatling-worker-1:
    image: denvazh/gatling
    command: -s MySimulation
    
  gatling-worker-2:
    image: denvazh/gatling
    command: -s MySimulation
```

### Performance Profiling Integration

```python
# Locust with custom metrics
from locust import events

@events.request.add_listener
def on_request(request_type, name, response_time, response_length, **kwargs):
    # Send to monitoring system
    send_to_grafana({
        'metric': 'response_time',
        'value': response_time,
        'endpoint': name
    })
```

### Correlation and Dynamic Data

```scala
// Gatling: Extract and reuse values
val scn = scenario("Dynamic Data")
  .exec(
    http("Login")
      .post("/login")
      .check(jsonPath("$.token").saveAs("authToken"))
  )
  .exec(
    http("Get Profile")
      .get("/profile")
      .header("Authorization", "Bearer ${authToken}")
  )
```

## CI/CD Integration

### GitHub Actions with Gatling

```yaml
name: Performance Tests

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run Gatling Tests
        run: |
          mvn gatling:test
      
      - name: Upload Results
        uses: actions/upload-artifact@v2
        with:
          name: gatling-results
          path: target/gatling/
      
      - name: Check Performance Thresholds
        run: |
          python check_thresholds.py \
            --p95-threshold 2000 \
            --error-rate-threshold 1
```

### Performance Thresholds

```python
def check_thresholds(results, thresholds):
    violations = []
    
    if results['p95'] > thresholds['p95']:
        violations.append(
            f"P95 response time {results['p95']}ms exceeds threshold {thresholds['p95']}ms"
        )
    
    if results['error_rate'] > thresholds['error_rate']:
        violations.append(
            f"Error rate {results['error_rate']}% exceeds threshold {thresholds['error_rate']}%"
        )
    
    if violations:
        raise Exception("\n".join(violations))
```

## Common Pitfalls to Avoid

### 1. Testing from Single Location

**Problem:** Doesn't represent real user distribution  
**Solution:** Use distributed load from multiple regions

### 2. Ignoring Warm-up Period

```python
# Include warm-up in your test
class LoadTest(HttpUser):
    def on_start(self):
        # Warm-up: prime caches, establish connections
        for _ in range(10):
            self.client.get("/health")
```

### 3. Not Cleaning Up Test Data

```python
def cleanup_test_data():
    # Delete test users created during load test
    test_users = User.objects.filter(email__contains='loadtest')
    test_users.delete()
```

### 4. Testing Production Directly

**Never test production!** Use:
- Staging environment
- Performance test environment
- Production-like infrastructure

## Real-World Example: E-commerce Load Test

```python
from locust import HttpUser, task, between, SequentialTaskSet

class UserBehavior(SequentialTaskSet):
    @task
    def browse_homepage(self):
        self.client.get("/")
    
    @task
    def search_products(self):
        self.client.get("/search?q=laptop")
    
    @task
    def view_product(self):
        self.client.get("/products/123")
    
    @task
    def add_to_cart(self):
        self.client.post("/cart", json={"product_id": 123, "quantity": 1})
    
    @task
    def checkout(self):
        self.client.post("/checkout", json={
            "payment_method": "credit_card",
            "shipping_address": "123 Test St"
        })

class EcommerceUser(HttpUser):
    tasks = [UserBehavior]
    wait_time = between(2, 5)
    weight = 3  # 75% of users

class BrowsingUser(HttpUser):
    @task
    def browse_only(self):
        self.client.get("/")
        self.client.get("/products")
    
    wait_time = between(1, 3)
    weight = 1  # 25% of users
```

## Conclusion

Key takeaways for successful performance testing:

1. **Choose the right tool** for your specific needs
2. **Model realistic user behavior** with proper think times
3. **Ramp up gradually** to identify breaking points
4. **Monitor system resources** alongside application metrics
5. **Integrate with CI/CD** for continuous performance validation
6. **Analyze results properly** using percentiles, not just averages

Performance testing is not a one-time activity—it should be continuous throughout development.

## Resources

- [My Performance Testing Project](/portfolio/02-performance-testing/)
- [Performance Testing Workshop](/talks/2023-performance-testing)
- [Gatling Documentation](https://gatling.io/docs/)
- [JMeter Documentation](https://jmeter.apache.org/)
- [Locust Documentation](https://docs.locust.io/)

Questions about performance testing? Let's connect on [LinkedIn](https://linkedin.com/in/fabio-bombiela-ramirez)!
