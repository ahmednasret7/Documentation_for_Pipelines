# Java Spring Applications - DevOps Essentials
This guide provides a foundational understanding of key practices for managing Java Spring applications as a DevOps engineer.

## Build and Dependency Management

### Maven
- Purpose: Maven is a build tool that helps generate application packages (JAR/WAR files) for deployment.
- How to Use:
  - Ensure the **pom.xml** file lists all dependencies, plugins, and configuration settings.
  - Run **mvn clean install** to build the application and resolve dependencies.
- Best Practices: Use Maven’s dependency management to enforce consistent versions and avoid conflicts.


### Gradle
  - Purpose: Gradle is another build tool that can also package applications as JAR or WAR files, with flexible dependency resolution.
  - How to Use:
Define dependencies and settings in the **build.gradle** file.
Run **gradle build** to compile the application and package it for deployment.
  - Best Practices: Use dependency constraints in **build.gradle** to enforce versions and reduce conflict risk.


### Application Configuration Management
#### Profiles
- Purpose: Spring Boot profiles allow you to manage separate configuration files for different environments, e.g., development, testing, and production.
- Common Files:
  - **application-dev.yml**: Development environment settings.
  - **application-prod.yml**: Production environment settings.
- How to Apply Profiles:
  - Set the **spring.profiles.active** environment variable to switch profiles (**spring.profiles.active=dev** for development).
  - Use **application-{profile}.yml** files to separate configurations like database settings, API endpoints, and logging levels.
- Best Practices: Keep sensitive data out of these files. Use placeholders and external secrets management where possible.


#### Externalized Configuration
- Purpose: Tools like Spring Cloud Config help manage configuration settings externally, allowing you to update configurations without modifying the code or redeploying.
- How to Use:
  - Configure Spring Cloud Config to read properties from an external source, such as a Git repository or a configuration server.
  - Ensure the application points to the config server by setting **spring.cloud.config.uri**.
- Best Practices: Store environment-specific properties in separate repositories or paths, and use secure access methods (like tokens or credentials) to manage access.