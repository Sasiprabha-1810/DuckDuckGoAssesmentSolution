**Additional Questions:**
**You have a requirement which states
        "The system should scale to 2000 users".**
1.) What other questions would you need to ask in order to get enough information to create a workload model.

Answer : 
1.User Distribution and Behaviour:
	1.what are the peak and average concurrent user counts ?
	2.what type of user access the system(eg: anonymous,authenticated)?
	3.What are the most common user modules or business transactions?
2.Transaction Details:
	1.What is the expected think time between actions ?
	2.How long does the typical session lasts ?
	3.What are the critical transactions we must include in workload?
3.Load Pattern:
	1.Are there any specific hours or days when the peak loads occur?
	2.Should the user load ramp up period gradually or start at full capacity?
4.Performance SLAs:
	1.What are the acceptable response time for each transactions?
	2.What is the expectable error rate and system throughput?
5.System Architecture:
	1.What are the components in scope? frontend, api,database ?
6.Test Environment:
	1.Is the test environment production clone ?
 
 **Can you re-write the requirement so that it is more descriptive and testable. (You can use
made up values?**

Answer: 
	The application shall support 2000 concurrent users during peak usage period,executing following transactions
with 40% search operation,30% homepage loads, 20% suggestions dropdowns and 10% settings navigation.
	Concurrent user - 2000
	The duration - 5 mins
Each user performs one action every 7 seconds(approx 3 minute session with 5 seconds think time between actions)
The result in a total of approximately 286 transactions per second(TPS)during the test and 
total transactions is approximately 85800

**Question 2: Example of Involvement in the Resolution of a Performance / Scalability Defect**

a. How was the defect found?
The defect was discovered during a performance test of an ERP system's "Generate Invoice Summary" transaction. 
This feature is used by finance teams to generate consolidated invoice data across multiple business units. Under a load of 200 virtual users, the average response time exceeded 25 seconds, while the SLA was set at 5 seconds. Additionally, some transactions were failing due to backend timeouts.

b. How was it diagnosed?
Using LoadRunner, I simulated the transaction and captured the response patterns. The server-side analysis (via logs and APM tools like appdynamics) revealed that:

A stored procedure triggered by the report generation was consuming significant CPU and running long queries with full table scans.
The ERP middleware logs showed that memory utilization peaked during this transaction, triggering long GC cycles.

c. How was it resolved?
I collaborated with the development and database teams to address the issue:
The stored procedure was optimized by rewriting inefficient joins and applying appropriate indexing on invoice_header and transaction_detail tables.
The business logic in the application layer was restructured to push more computation into the optimized stored procedure, reducing post-processing time.
We implemented result set pagination, ensuring only relevant data (e.g., monthly batch) was fetched based on input parameters.
I also advised the infrastructure team to tune JVM memory settings and increase the thread pool size to improve concurrency handling.

d. How was the resolution proven?
After implementing the fixes, I executed the same LoadRunner scenario. The response time for the "Generate Invoice Summary" transaction reduced from 25 seconds to under 4 seconds, well within SLA. No timeouts or application errors were observed.

We followed up with endurance (soak) testing for 4 hours, and the application showed consistent performance without memory leaks 
or GC overhead. The results were shared with stakeholders, and the changes were approved for deployment.

**As well as creating a script in a tool to do the performance testing, what other factors would need to be considered for a performance testing engagement?**

1. Understanding Business Requirements and SLAs
Identify business-critical workflows and their expected response times, concurrency levels, and throughput.

Clarify performance objectives, such as maximum acceptable response times under peak load, system availability, or failover behavior.

2. Workload Modeling
Define realistic user behavior patterns (e.g., how users navigate an ERP system during a workday).

Establish user concurrency, pacing, and think times to simulate real-world usage.

Include background jobs or scheduled tasks that might impact performance (e.g., batch processing in ERP systems).

3. Test Environment Readiness
Ensure the performance test environment mirrors production in terms of architecture, hardware, and configurations.

Coordinate with environment teams to reset or refresh data between test runs to maintain consistency.

Validate that monitoring tools are set up and functional (e.g., AppDynamics, Dynatrace, or server logs).

4. Test Data Strategy
Prepare realistic and sufficient test data, especially for complex applications like ERP where data volumes directly impact performance.

Ensure test data covers different scenarios (e.g., various invoice types or user roles).

5. Monitoring and Diagnostics Setup
Monitor infrastructure metrics (CPU, memory, disk I/O, network).

Track application-specific metrics (DB query times, thread usage, garbage collection).

Enable logging and tracing to support post-test analysis and bottleneck identification.

6. Risk Management
Identify system limitations, such as third-party integrations, that could affect test outcomes.

Plan for rollback and cleanup procedures in case the test causes disruptions (e.g., excessive DB writes or memory usage).

7. Defect Management and Retesting
Collaborate with developers to log and prioritize performance-related defects.

Ensure iterative tuning and retesting after each code or infrastructure change.

8. Communication and Reporting
Regularly update stakeholders on test status, risks, and findings.

Provide clear, actionable reports that explain test results in both technical and business terms.

Include recommendations for improvements (e.g., database tuning, hardware scaling, or code optimizations).

9. Compliance and Security
Ensure test plans comply with data privacy and security standards, especially when using production-like data.

Secure access to test environments and prevent unintended alerts (e.g., from security monitoring tools).

By addressing these factors, performance testing becomes a strategic activity that not only verifies technical readiness 
but also builds confidence in the system’s ability to handle real-world usage
