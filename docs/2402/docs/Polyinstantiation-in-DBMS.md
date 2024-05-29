<!--yml
category: 未分类
date: 2024-05-27 14:46:40
-->

# Polyinstantiation in DBMS

> 来源：[https://coding-land.blogspot.com/2023/06/polyinstantiation-in-dbms.html](https://coding-land.blogspot.com/2023/06/polyinstantiation-in-dbms.html)

Polyinstantiation in DBMS

Polyinstantiation is a security mechanism used in database management systems (DBMS) to prevent information inference attacks. It involves the creation of multiple instances of an object with different levels of sensitivity or security, allowing users with different access levels to access the same object while seeing only the information they are authorized to view.

There are two types of polyinstantiation commonly used in DBMS:

Weak Polyinstantiation: In weak polyinstantiation, multiple instances of an object are created with different sensitivity levels. Each instance contains a subset of the information available in the higher-level instance. This approach ensures that users with lower security clearances cannot infer information from higher-level instances.

Strong Polyinstantiation: Strong polyinstantiation takes the concept further by allowing different instances of an object to have different values for the same attribute. This means that an attribute's value in one instance may not match its value in another instance. Strong polyinstantiation aims to protect against covert channels, where unauthorized users could potentially infer information by analyzing the values of shared attributes.

Here's an example to illustrate polyinstantiation:

Consider a database that stores employee information, including salary details. We have two users with different security clearances: User A has a higher security clearance and can access all salary information, while User B has a lower security clearance and can only access salary information up to a certain threshold.

In a polyinstantiated DBMS, the salary information for each employee can be duplicated into two instances, one for each user:

Instance 1 (User A):

Employee 1: John Doe - Salary: $100,000

Employee 2: Jane Smith - Salary: $80,000

Employee 3: Mark Johnson - Salary: $120,000

Instance 2 (User B):

Employee 1: John Doe - Salary: $80,000

Employee 2: Jane Smith - Salary: $80,000

Employee 3: Mark Johnson - Salary: $80,000

In this example, User A can access the complete salary information, while User B can only see salaries up to $80,000\. By employing polyinstantiation, the DBMS ensures that User B cannot infer the actual salary of any employee beyond the threshold they are authorized to view.

Overall, polyinstantiation is a security feature that helps protect sensitive information and prevent unauthorized users from deducing confidential data by limiting the information available to them based on their security clearances.

User

If salary is 40000

ChatGPT

If the salary is $40,000, the polyinstantiated instances for User A and User B could be as follows:

Instance 1 (User A):

Employee 1: John Doe - Salary: $40,000

Instance 2 (User B):

Employee 1: John Doe - Salary: $40,000

Both User A and User B can see that the salary for Employee 1 (John Doe) is $40,000\. Since the salary is below any specified threshold, User B is allowed to view the complete salary information in this case.