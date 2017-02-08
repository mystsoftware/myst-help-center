## {{page.title }} 

MyST uses JDBC to execute SQL against databases. Many databases are supported through JDBC drivers. The most common of these are:
 * Oracle Database
 * MySQL
 * PostgreSQL
 * DB2
 
When seeding data to NoSQL databases, you may wish to [create a custom deployment type](application-deployment/other-databases.md). However, some NoSQL databases do have JDBC Drivers written for them. For example, there is a [JDBC Driver for Apache Cassandra](https://github.com/adejanovski/cassandra-jdbc-wrapper) which wraps the CQL protocol to allow common SQL operations such as INSERT, UPDATE, CREATE TABLE etc.