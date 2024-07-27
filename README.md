# SOLID-principles-with-real-world-examples-and-best-practices

https://www.boxpiper.com/static/433c2c22a5faafa867405b40da29b2ab/be464/solid-design-principle-1.jpg

SOLID is an acronym for five fundamental design principles in object-oriented programming and design. These principles help in creating software that is easy to manage, extend, and understand. The SOLID principles were introduced by Robert C. Martin, also known as Uncle Bob, and they are widely adopted in modern software engineering.
Here's an introduction to each principle:

**SOLID** stands for the following:

**S** = Single Responsibility Principle 
**O** = Open Closed Principle
**L** = Liskov Substitution Principle
**I** = Interface Segregation Principle
**D** = Dependency Inversion Principle

### Single Responsibility Principle (SRP)

**Single Responsibility Principle (SRP)**:
> A class should have one, and only one, reason to change. This means that a class should have only one job or responsibility.

#### Benefits of SRP

1. **Improved Maintainability**: When a class has a single responsibility, it's easier to modify and understand its code. Changes to one part of the system will likely have minimal impact on others.

2. **Better Testability**: Smaller, focused classes are easier to test in isolation.

3. **Enhanced Readability**: Classes with a single responsibility are often more readable because they have a clear, focused purpose.

4. **Easier Debugging**: When a class only handles one aspect of the functionality, finding and fixing bugs becomes simpler.

#### Applying SRP to the `NotificationService` Example

```csharp
public class NotificationService 
{
    public void SendNotification(Notification notification)
    {
        // Send notification
    }

    public void LogNotification(Notification notification)
    {
        // Log notification
    }

    public void ValidateNotification(Notification notification)
    {
        // Validate notification
    }
}
```

In your `NotificationService` class, the responsibilities are mixed: sending, logging, and validating notifications. According to SRP, you should separate these responsibilities into different classes. Here’s how you can refactor your code to adhere to SRP:

1. **NotificationSender**: Responsible for sending notifications.
2. **NotificationLogger**: Responsible for logging notifications.
3. **NotificationValidator**: Responsible for validating notifications.

### Refactored Example

Here's a breakdown of how to apply SRP to your `NotificationService` class:

#### 1. Define Separate Classes for Each Responsibility

**NotificationSender**:
```csharp
public class NotificationSender 
{
    public void Send(Notification notification)
    {
        // Code to send the notification
    }
}
```

**NotificationLogger**:
```csharp
public class NotificationLogger 
{
    public void Log(Notification notification)
    {
        // Code to log the notification
    }
}
```

**NotificationValidator**:
```csharp
public class NotificationValidator 
{
    public bool Validate(Notification notification)
    {
        // Code to validate the notification
        return true; // Return validation result
    }
}
```

#### 2. Create a Service Class to Coordinate the Other Classes

**NotificationService**:
```csharp
public class NotificationService
{
    private readonly NotificationSender _sender;
    private readonly NotificationLogger _logger;
    private readonly NotificationValidator _validator;

    public NotificationService(NotificationSender sender, NotificationLogger logger, NotificationValidator validator)
    {
        _sender = sender;
        _logger = logger;
        _validator = validator;
    }

    public void ProcessNotification(Notification notification)
    {
        if (_validator.Validate(notification))
        {
            _sender.Send(notification);
            _logger.Log(notification);
        }
    }
}
```

### Summary

By separating the responsibilities into distinct classes:

- **NotificationSender**: Manages the process of sending notifications.
- **NotificationLogger**: Handles logging of notifications.
- **NotificationValidator**: Takes care of validation checks.

The `NotificationService` class then acts as a coordinator, using these components to process notifications. This structure adheres to the Single Responsibility Principle, making each class easier to manage and understand.

By incorporating these practices, you'll make your `NotificationService` class more robust, flexible, and maintainable.

### 2. Open/Closed Principle (OCP)
> The Open/Closed Principle (OCP) states that software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. This means that you should be able to extend the behavior of a system without modifying its existing code.

### Example
Let's explore the Open/Closed Principle (OCP) with a real-time example. We'll use an example from the domain of e-commerce, specifically focusing on a discount calculation system. 

### Scenario
Imagine you are building an e-commerce platform that applies different types of discounts to customer orders. Initially, you have a few discount types, but you want to ensure that your system can easily accommodate new discount types without modifying existing code.

### Initial Implementation (Not Following OCP)

Here’s a simplified version of a discount calculation system that violates the Open/Closed Principle. 

```csharp
public class DiscountCalculator
{
    public double CalculateDiscount(string discountType, double orderAmount)
    {
        if (discountType == "Percentage")
        {
            return orderAmount * 0.10; // 10% discount
        }
        else if (discountType == "Fixed")
        {
            return 15; // $15 discount
        }
        else
        {
            throw new ArgumentException("Unknown discount type");
        }
    }
}
```

In this code, if you want to add a new discount type (e.g., a Buy-One-Get-One-Free discount), you must modify the `DiscountCalculator` class. This violates the OCP because the class is not closed for modification.

### Refactored Example (Following OCP)

To adhere to the Open/Closed Principle, we can refactor the code to use polymorphism. Here’s how you can do it:

#### Define an Interface for Discounts

```csharp
public interface IDiscount
{
    double Apply(double orderAmount);
}
```

#### Implement Concrete Discount Types

**PercentageDiscount**:
```csharp
public class PercentageDiscount : IDiscount
{
    private readonly double _percentage;

    public PercentageDiscount(double percentage)
    {
        _percentage = percentage;
    }

    public double Apply(double orderAmount)
    {
        return orderAmount * (_percentage / 100);
    }
}
```

**FixedAmountDiscount**:
```csharp
public class FixedAmountDiscount : IDiscount
{
    private readonly double _amount;

    public FixedAmountDiscount(double amount)
    {
        _amount = amount;
    }

    public double Apply(double orderAmount)
    {
        return _amount;
    }
}
```

#### Create a Discount Calculator that Uses Dependency Injection

```csharp
public class DiscountCalculator
{
    private readonly IDiscount _discount;

    public DiscountCalculator(IDiscount discount)
    {
        _discount = discount;
    }

    public double CalculateDiscount(double orderAmount)
    {
        return _discount.Apply(orderAmount);
    }
}
```

### Adding New Discount Types

To add a new discount type, such as a `BuyOneGetOneFreeDiscount`, you simply create a new class that implements the `IDiscount` interface:

**BuyOneGetOneFreeDiscount**:
```csharp
public class BuyOneGetOneFreeDiscount : IDiscount
{
    public double Apply(double orderAmount)
    {
        // Assumes a fixed price of $20 per item
        return orderAmount >= 20 ? 20 : 0; // 20 is the discount for a BOGO offer
    }
}
```

### Usage Example

Here’s how you use the `DiscountCalculator` with different types of discounts:

```csharp
class Program
{
    static void Main()
    {
        double orderAmount = 100;

        IDiscount percentageDiscount = new PercentageDiscount(10); // 10% discount
        IDiscount fixedAmountDiscount = new FixedAmountDiscount(15); // $15 discount
        IDiscount bogoDiscount = new BuyOneGetOneFreeDiscount(); // BOGO discount

        DiscountCalculator calculator;

        calculator = new DiscountCalculator(percentageDiscount);
        Console.WriteLine($"Percentage Discount: {calculator.CalculateDiscount(orderAmount)}");

        calculator = new DiscountCalculator(fixedAmountDiscount);
        Console.WriteLine($"Fixed Amount Discount: {calculator.CalculateDiscount(orderAmount)}");

        calculator = new DiscountCalculator(bogoDiscount);
        Console.WriteLine($"BOGO Discount: {calculator.CalculateDiscount(orderAmount)}");
    }
}
```

### Summary

In this improved design:

- **Discount Types** (`PercentageDiscount`, `FixedAmountDiscount`, `BuyOneGetOneFreeDiscount`) are **open for extension**: You can add new discount types by creating new classes that implement the `IDiscount` interface.
- **DiscountCalculator** is **closed for modification**: It doesn’t need to be changed when new discount types are added. It works with any implementation of the `IDiscount` interface.

This design adheres to the Open/Closed Principle, making it more flexible and easier to extend as new requirements arise.
### 3. Liskov Substitution Principle (LSP)
> The Liskov Substitution Principle (LSP) states that objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program. In other words, a subclass should be able to substitute its superclass without causing errors or unexpected behaviors.

### Example

You are developing a payment processing system that handles different types of payments. Initially, you might have a `PaymentMethod` class, and you want to extend it with specific types of payment methods like `CreditCard` and `PayPal`. To adhere to the LSP, the subclasses should properly fit the expectations set by the base class.

### Initial Implementation (Violating LSP)

**Base PaymentMethod Class**:
```csharp
public abstract class PaymentMethod
{
    public abstract void ProcessPayment(double amount);

    // Some common properties or methods for payment methods
}
```

**CreditCard Class**:
```csharp
public class CreditCard : PaymentMethod
{
    public override void ProcessPayment(double amount)
    {
        // Process credit card payment
        Console.WriteLine($"Processing credit card payment of ${amount}");
    }
}
```

**PayPal Class** (Problematic):
```csharp
public class PayPal : PaymentMethod
{
    public override void ProcessPayment(double amount)
    {
        // PayPal might require additional steps
        if (amount <= 0)
        {
            throw new ArgumentException("Amount must be positive for PayPal payments.");
        }
        Console.WriteLine($"Processing PayPal payment of ${amount}");
    }
}
```

**PaymentProcessor Class**:
```csharp
public class PaymentProcessor
{
    public void MakePayment(PaymentMethod paymentMethod, double amount)
    {
        paymentMethod.ProcessPayment(amount);
    }
}
```

**Usage**:
```csharp
class Program
{
    static void Main()
    {
        var creditCard = new CreditCard();
        var payPal = new PayPal();
        
        var processor = new PaymentProcessor();
        
        processor.MakePayment(creditCard, 100);  // Works fine
        processor.MakePayment(payPal, -50);      // Throws exception
    }
}
```

**Problems**:
- The `PayPal` class introduces an additional validation that isn’t present in the base class `PaymentMethod`, violating the expectation that `ProcessPayment` should work uniformly across all subclasses. This can lead to exceptions or inconsistent behavior when using polymorphism.

### Improved Implementation (Following LSP)

To follow the Liskov Substitution Principle, we need to ensure that subclasses can be substituted without breaking the behavior expected from the base class.

**Base PaymentMethod Class**:
```csharp
public abstract class PaymentMethod
{
    public abstract void ProcessPayment(double amount);

    protected void ValidateAmount(double amount)
    {
        if (amount <= 0)
        {
            throw new ArgumentException("Amount must be positive.");
        }
    }
}
```

**CreditCard Class**:
```csharp
public class CreditCard : PaymentMethod
{
    public override void ProcessPayment(double amount)
    {
        ValidateAmount(amount); // Ensure validation is consistent
        // Process credit card payment
        Console.WriteLine($"Processing credit card payment of ${amount}");
    }
}
```

**PayPal Class**:
```csharp
public class PayPal : PaymentMethod
{
    public override void ProcessPayment(double amount)
    {
        ValidateAmount(amount); // Ensure validation is consistent
        // Process PayPal payment
        Console.WriteLine($"Processing PayPal payment of ${amount}");
    }
}
```

**PaymentProcessor Class**:
```csharp
public class PaymentProcessor
{
    public void MakePayment(PaymentMethod paymentMethod, double amount)
    {
        paymentMethod.ProcessPayment(amount);
    }
}
```

**Usage**:
```csharp
class Program
{
    static void Main()
    {
        var creditCard = new CreditCard();
        var payPal = new PayPal();
        
        var processor = new PaymentProcessor();
        
        processor.MakePayment(creditCard, 100);  // Works fine
        processor.MakePayment(payPal, 50);       // Works fine
    }
}
```

**Summary**

- **Base Class (`PaymentMethod`)**: Defines a common interface for processing payments and includes a method for amount validation that can be reused by subclasses.
- **Subclasses (`CreditCard`, `PayPal`)**: Both subclasses implement the `ProcessPayment` method and use the base class’s validation method to ensure consistent behavior.
- **PaymentProcessor**: Interacts with any `PaymentMethod` without needing to know the specifics of each payment method, ensuring that all `PaymentMethod` implementations are interchangeable.

By following the Liskov Substitution Principle, the system maintains consistent behavior across different payment methods, preventing issues related to method expectations and ensuring that derived classes can substitute their base classes without causing unexpected errors or exceptions.

### 4. Interface Segregation Principle (ISP)
> The Interface Segregation Principle (ISP) states that clients should not be forced to depend on interfaces they do not use. It encourages creating smaller, more focused interfaces that are specific to the needs of the client.

This principle promotes the creation of smaller, more specific interfaces rather than large, general-purpose ones. The idea is to ensure that classes only implement methods that are relevant to them, thus avoiding unnecessary dependencies and complexity.

### Example: Printer and Scanner

Let’s consider a real-world scenario involving a multi-function printer. This printer can print, scan, and fax. Initially, you might have a single interface that includes all these operations. However, not all clients need all functionalities, and such a large interface can lead to unwanted dependencies.

### Initial Implementation (Violating ISP)

Here’s an example of a large interface:

```csharp
public interface IMultiFunctionDevice
{
    void Print(string document);
    void Scan(string document);
    void Fax(string document);
}
```

**Problems**:
- **Unnecessary Implementations**: If you have a simple printer that does not support scanning or faxing, it still needs to implement all methods, potentially leaving methods unimplemented or throwing exceptions.
- **Coupling**: Clients that use the interface have to deal with methods that are irrelevant to them, increasing complexity.

### Improved Implementation (Following ISP)

To adhere to the Interface Segregation Principle, you should break down the large interface into smaller, more specific interfaces.

#### Define Specific Interfaces

**Printer Interface**:
```csharp
public interface IPrinter
{
    void Print(string document);
}
```

**Scanner Interface**:
```csharp
public interface IScanner
{
    void Scan(string document);
}
```

**Fax Interface**:
```csharp
public interface IFax
{
    void Fax(string document);
}
```

#### Implement Classes for Different Devices

**SimplePrinter**:
```csharp
public class SimplePrinter : IPrinter
{
    public void Print(string document)
    {
        // Implementation for printing
        Console.WriteLine($"Printing document: {document}");
    }
}
```

**MultiFunctionPrinter**:
```csharp
public class MultiFunctionPrinter : IPrinter, IScanner, IFax
{
    public void Print(string document)
    {
        // Implementation for printing
        Console.WriteLine($"Printing document: {document}");
    }

    public void Scan(string document)
    {
        // Implementation for scanning
        Console.WriteLine($"Scanning document: {document}");
    }

    public void Fax(string document)
    {
        // Implementation for faxing
        Console.WriteLine($"Faxing document: {document}");
    }
}
```

#### Client Code

Clients can now depend on only the interfaces they need:

**PrintClient**:
```csharp
public class PrintClient
{
    private readonly IPrinter _printer;

    public PrintClient(IPrinter printer)
    {
        _printer = printer;
    }

    public void PrintDocument(string document)
    {
        _printer.Print(document);
    }
}
```

**ScanClient**:
```csharp
public class ScanClient
{
    private readonly IScanner _scanner;

    public ScanClient(IScanner scanner)
    {
        _scanner = scanner;
    }

    public void ScanDocument(string document)
    {
        _scanner.Scan(document);
    }
}
```

**FaxClient**:
```csharp
public class FaxClient
{
    private readonly IFax _fax;

    public FaxClient(IFax fax)
    {
        _fax = fax;
    }

    public void FaxDocument(string document)
    {
        _fax.Fax(document);
    }
}
```

### Summary

In this improved design:

- **Interfaces** (`IPrinter`, `IScanner`, `IFax`) are smaller and more specific, adhering to the Interface Segregation Principle.
- **Implementations** like `SimplePrinter` and `MultiFunctionPrinter` only implement the interfaces that are relevant to their functionality.
- **Clients** only depend on the interfaces they need, which simplifies their dependencies and avoids unnecessary complexity.

By following ISP, the system becomes more modular and easier to maintain, as changes to one part of the system are less likely to affect unrelated parts.

### 5. Dependency Inversion Principle (DIP)
> The Dependency Inversion Principle (DIP) states that high-level modules should not depend on low-level modules, but both should depend on abstractions (interfaces or abstract classes). It also states that abstractions should not depend on details; details should depend on abstractions.

**Example:**
Dependency Inversion Principle (DIP) in the context of a notification system where `EmailSender` and `SMSSender` both implement an interface `INotificationSender`. We'll demonstrate how to refactor the design to adhere to DIP, ensuring that high-level modules (such as `NotificationService`) depend on abstractions rather than concrete implementations.

### 1. Define the Abstraction

Start by defining an interface `INotificationSender` that abstracts the sending of notifications. This interface will be implemented by concrete classes like `EmailSender` and `SMSSender`.

```csharp
public interface INotificationSender
{
    void Send(Notification notification);
}
```

### 2. Implement Concrete Classes

Create concrete implementations of the `INotificationSender` interface for different notification methods.

**EmailSender Class**:
```csharp
public class EmailSender : INotificationSender
{
    public void Send(Notification notification)
    {
        // Logic to send email notification
        Console.WriteLine($"Sending email to {notification.Recipient} with message: {notification.Message}");
    }
}
```

**SMSSender Class**:
```csharp
public class SMSSender : INotificationSender
{
    public void Send(Notification notification)
    {
        // Logic to send SMS notification
        Console.WriteLine($"Sending SMS to {notification.Recipient} with message: {notification.Message}");
    }
}
```

### 3. Refactor the NotificationService

Refactor `NotificationService` to depend on the `INotificationSender` interface rather than a concrete implementation. This makes `NotificationService` flexible and able to work with any `INotificationSender` implementation.

**NotificationService Class**:
```csharp
public class NotificationService
{
    private readonly INotificationSender _notificationSender;

    // Dependency Injection via Constructor
    public NotificationService(INotificationSender notificationSender)
    {
        _notificationSender = notificationSender;
    }

    public void ProcessNotification(Notification notification)
    {
        _notificationSender.Send(notification);
    }
}
```

### 4. Define the Notification Class

Ensure the `Notification` class contains the properties needed for sending notifications.

**Notification Class**:
```csharp
public class Notification
{
    public string Recipient { get; set; }
    public string Message { get; set; }
}
```

### 5. Usage Example

Instantiate `NotificationService` with different `INotificationSender` implementations and use it to process notifications.

**Main Program**:
```csharp
class Program
{
    static void Main()
    {
        // Create instances of senders
        INotificationSender emailSender = new EmailSender();
        INotificationSender smsSender = new SMSSender();

        // Create instances of NotificationService with different senders
        var emailNotificationService = new NotificationService(emailSender);
        var smsNotificationService = new NotificationService(smsSender);

        // Create a notification
        var notification = new Notification
        {
            Recipient = "user@example.com",
            Message = "Your order has been shipped!"
        };

        // Send notifications using different services
        emailNotificationService.ProcessNotification(notification);
        
        notification.Recipient = "123-456-7890"; // Change recipient for SMS
        smsNotificationService.ProcessNotification(notification);
    }
}
```

### Summary

- **Abstraction (`INotificationSender`)**: Defines a contract for sending notifications, allowing different implementations to be used interchangeably.
- **Concrete Implementations (`EmailSender`, `SMSSender`)**: Implement the `INotificationSender` interface to provide specific methods for sending notifications.
- **Refactored Service (`NotificationService`)**: Depends on the `INotificationSender` abstraction, allowing it to work with any concrete sender implementation. This makes `NotificationService` flexible and adheres to the Dependency Inversion Principle.
- **Dependency Injection**: Used to inject dependencies into `NotificationService`, promoting loose coupling and enhancing testability.

By adhering to DIP, your design becomes more modular and adaptable. Adding new notification methods or changing existing ones only requires implementing the `INotificationSender` interface and updating the dependency injection setup, rather than modifying the core service logic.

### Summary of SOLID Principles

- **Single Responsibility Principle (SRP)**: Each class should have one reason to change.
- **Open/Closed Principle (OCP)**: Classes should be open for extension but closed for modification.
- **Liskov Substitution Principle (LSP)**: Subtypes should be substitutable for their base types without altering correctness.
- **Interface Segregation Principle (ISP)**: Clients should not be forced to depend on interfaces they do not use.
- **Dependency Inversion Principle (DIP)**: Depend on abstractions, not on concrete implementations.

By following these SOLID principles, you can design software that is more modular, flexible, and easier to maintain and extend over time. Each principle contributes to better code organization, reduced coupling between components, and improved code quality.
