### Design Patterns Used in Spring Boot

In Spring Boot applications, several design patterns are commonly employed:

1. **Dependency Injection (DI)**: Spring Boot extensively utilizes the DI pattern, achieved through Spring's Inversion of Control (IoC) container. It promotes loose coupling and modularity by injecting dependencies into classes, improving testability and flexibility.

2. **Model-View-Controller (MVC)**: Spring Boot follows the MVC design pattern for structuring web applications. It separates the application logic into three components: the model (data and business logic), the view (user interface), and the controller (handles requests and orchestrates the model and view).

3. **Builder**: Spring Boot uses the builder pattern to create complex objects with a fluent and readable API. Builders, such as `DataSourceBuilder`, simplify the configuration process by providing customizable options.

4. **Factory**: Spring Boot employs the factory pattern to create and configure instances of Spring beans and components. Factories encapsulate creation logic and provide a centralized point for object creation.

5. **Template Method**: Spring Boot utilizes the template method pattern, as seen in Spring's `JdbcTemplate`. Template methods define an algorithm's skeleton, allowing subclasses to override specific steps while maintaining a common structure.

6. **Observer**: Spring Boot utilizes the observer pattern in the event-driven programming model. Components can publish events, and other components can listen for and react to those events, following the observer pattern.

7. **Decorator**: Spring Boot uses the decorator pattern, particularly with aspects of AOP (Aspect-Oriented Programming). AOP allows for the separation of cross-cutting concerns by dynamically adding behavior around method invocations.

These are some of the design patterns commonly used in Spring Boot applications. Spring Boot incorporates various other patterns and best practices to promote modularity, maintainability, and extensibility in software development.

