# 1. Logging frameworks in Java

## java.util.logging (JUL)

- JDK framework, comes with Java
- suitable for simpler logging

## SLF4J (Simple Loggig Facade For Java)

- SLF4J is an abstraction for other logging frameworks (logging facade)
- it allows you choosing the framework of your choice - you provide the config (log format, log level, log output...) and the dependency
- apps can change the logging framework without impacting the code implementation - apps are independent of the logging framework (due to the portability of this facade)
- additional benefits:
  - storing the logs in the database
  - rotating the logs - an automated process of managing log file size, preventing the logs from filling the storage space and slowing down the system. It involves renaming an existing log file and creating a new log with the same name to store the new information. This is usually done once a day or week.

## Log4j

Log4j is the open-source logging framework that provides fast logging capabilities for Java applications. It was the most popular Java logging framework until its EOL announced in 2015. Log4j was highly configurable, letting you define the logging behavior at run time and directly log the outputs to various destinations like console, file and databases. It also provided great flexibility to logging by allowing control of the log format and log level of each output.

## Logback

Logback, the successor of Log4j, holds many improvements over Log4j in terms of its performance and native support for SLF4j. Some of the major improvements over Log4j are auto-reloading config files (if the config file is changes, it scans it periodically), filtering capabilities (filtering logs based on some criteria) and automatic removal of old log archival (can define the history slot).

Logback comes with 3 core modules:

- logback-core: This is the base module that lays the foundation required by the following two modules.
- logback-classic: This module contains an improved version of Log4j and also natively implements the SLF4j API that lets you switch between Logback and other frameworks.
- logback-access: This module integrates with servlet containers such as Tomcat and Jetty allowing HTTP-access log function.

## Log4j2

- successor of Logback
- performance features - lazy evaluation of logs based on lambda expressions, async loggers and an option to disable garbage collector operations
