#Resilience4j is a lightweight, easy-to-use fault tolerance library inspired by  Netflix Hystrix,
but designed for Java 8 and functional programming. 

##Core modules of Resilience4j
resilience4j-circuitbreaker: Circuit breaking
resilience4j-ratelimiter: Rate limiting
resilience4j-bulkhead: Bulkheading
resilience4j-retry: Automatic retrying (sync and async)
resilience4j-cache: Result caching
resilience4j-timelimiter: Timeout handling

#What is Rate Limiting?
Rate Limiter limits the number of requests for a given period.

resilience4j.ratelimiter.instances.getMessageRateLimit.limit-for-period=2
resilience4j.ratelimiter.instances.getMessageRateLimit.limit-refresh-period=5s
resilience4j.ratelimiter.instances.getMessageRateLimit.timeout-duration=0

The above properties represent that only 2 requests are allowed in 5 seconds duration. Also, there is no timeout duration which means after completion of 5 seconds, the user can send request again.


#What is Retry?
Suppose Microservice ‘A’  depends on another Microservice ‘B’. 
Let’s assume Microservice ‘B’ is a faulty service and its success rate is only upto 50-60%.
If Microservice ‘A’ retries to send request 2 to 3 times, the chances of getting response 
increases. Obviously, we can achieve this functionality with the help of annotation @Retry provided by Resilience4j without writing a code explicitly.

Update application.proerpties
resilience4j.retry.instances.getInvoiceRetry.max-attempts=5
resilience4j.retry.instances.getInvoiceRetry.wait-duration=2s
resilience4j.retry.instances.getInvoiceRetry.retry-exceptions=org.springframework.web.client.ResourceAccessException

By default the retry mechanism makes 3 attempts if the service fails for the first time. But here we have configured for 5 attempts, each after 2 seconds interval.


#What is Circuit Breaker ?
Circuit Breaker is a pattern in developing the Microservices based applications in order to 
tolerate any fault. As the name suggests, ‘Breaking the Circuit’. Suppose a Microservice ‘A’ is internally calling another Microservice ‘B’ and ‘B’ has some fault.
In order to escape the multiple microservices from becoming erroneous as a result of cascading 
effect, we stop calling the faulty Microservice ‘B’.

application.properties
resilience4j.circuitbreaker.instances.getInvoiceCB.failure-rate-threshold=80
resilience4j.circuitbreaker.instances.getInvoiceCB.sliding-window-size=10
resilience4j.circuitbreaker.instances.getInvoiceCB.sliding-window-type=COUNT_BASED
resilience4j.circuitbreaker.instances.getInvoiceCB.minimum-number-of-calls=5
resilience4j.circuitbreaker.instances.getInvoiceCB.automatic-transition-from-open-to-half-open-enabled=true
resilience4j.circuitbreaker.instances.getInvoiceCB.permitted-number-of-calls-in-half-open-state=4
resilience4j.circuitbreaker.instances.getInvoiceCB.wait-duration-in-open-state=1s

1) ‘failure-rate-threshold=80‘ indicates that if 80% of requests are getting failed, open the circuit ie. Make the Circuit Breaker state as Open.
2) ‘sliding-window-size=10‘ indicates that if 80% of requests out of 10 (it means 8) are failing, open the circuit.
3) ‘sliding-window-type=COUNT_BASED‘ indicates that we are using COUNT_BASED sliding window. Another type is TIME_BASED.
4) ‘minimum-number-of-calls=5‘ indicates that we need at least 5 calls to calculate the failure rate threshold.
5) ‘automatic-transition-from-open-to-half-open-enabled=true‘ indicates that don’t switch directly from the open state to the closed state, consider the half-open state also.
6) ‘permitted-number-of-calls-in-half-open-state=4‘ indicates that when on half-open state, consider sending 4 requests. If 80% of them are failing, switch circuit breaker to open state.
7) ‘wait-duration-in-open-state=1s’ indicates the waiting time interval while switching from the open state to the closed state.


#What is Bulkhead?
sing Bulkhead we can limit the number of concurrent requests. 
We can achieve this functionality easily with the help of annotation @Bulkhead without writing a specific code.
Update application.properties
resilience4j.bulkhead.instances.getMessageBH.max-concurrent-calls=5
resilience4j.bulkhead.instances.getMessageBH.max-wait-duration=0
‘max-concurrent-calls=5’ indicates that if the number of concurrent calls exceed 5, activate the fallback method.

‘max-wait-duration=0’ indicates that don’t wait for anything, show response immediately based on the configuration.

#What is Time Limiting or Timeout Handling?
Time Limiting is the process of setting a time limit for a Microservice to respond. 
Suppose Microservice ‘A’ sends a request to Microservice ‘B’, it sets a time limit for the 
Microservice ‘B’ to respond. If  Microservice ‘B’ doesn’t respond within that time limit,
then it will be considered that it has some fault. 
We can achieve this functionality easily with the help of annotation @Timelimiter without 
writing a specific code.
Update application.properties
resilience4j.timelimiter.instances.getMessageTL.timeout-duration=1ms
resilience4j.timelimiter.instances.getMessageTL.cancel-running-future=false

‘timeout-duration=1ms’ indicates that the maximum amount of time a request can take to respond 
is 1 millisecond

‘cancel-running-future=false’ indicates that do not cancel the Running Completable Futures 
After TimeOut.

In order to test the functionality, Run the application as it is. You will get TimeOutException 
on the Browser. When you change the value of timeout-duration=1s, you will receive “Executing Within the time Limit…” message in the browser.


Reference:
https://javatechonline.com/how-to-implement-fault-tolerance-in-microservices-using-resilience4j/
