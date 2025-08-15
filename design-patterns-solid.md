# Design Patterns & SOLID Principles Interview Questions

## Table of Contents
1. [SOLID Principles](#solid-principles)
2. [Creational Patterns](#creational-patterns)
3. [Structural Patterns](#structural-patterns)
4. [Behavioral Patterns](#behavioral-patterns)
5. [Enterprise Patterns](#enterprise-patterns)
6. [Anti-Patterns](#anti-patterns)
7. [Pattern Selection & Trade-offs](#pattern-selection--trade-offs)

## SOLID Principles

### Q1: Explain SOLID principles with examples.
**Answer:**

### S - Single Responsibility Principle
A class should have only one reason to change.

```java
// Bad: Multiple responsibilities
public class Employee {
    private String name;
    private BigDecimal salary;
    
    public void calculatePay() { }
    public void saveToDatabase() { }  // Database responsibility
    public void sendEmail() { }       // Email responsibility
}

// Good: Single responsibility
public class Employee {
    private String name;
    private BigDecimal salary;
    
    public BigDecimal calculatePay() { 
        return salary;
    }
}

public class EmployeeRepository {
    public void save(Employee employee) { }
}

public class EmailService {
    public void sendEmail(Employee employee) { }
}
```

### O - Open/Closed Principle
Software entities should be open for extension but closed for modification.

```java
// Bad: Modifying existing code for new shapes
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.width * r.height;
        } else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return Math.PI * c.radius * c.radius;
        }
        // Need to modify this method for new shapes
        return 0;
    }
}

// Good: Extension through abstraction
public interface Shape {
    double calculateArea();
}

public class Rectangle implements Shape {
    private double width, height;
    
    public double calculateArea() {
        return width * height;
    }
}

public class Circle implements Shape {
    private double radius;
    
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea();
    }
}
```

### L - Liskov Substitution Principle
Objects of a superclass should be replaceable with objects of subclasses without breaking the application.

```java
// Bad: Square violates LSP when substituting Rectangle
public class Rectangle {
    protected int width, height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // Breaks Rectangle's behavior
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}

// Good: Proper abstraction
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width, height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    public int getArea() {
        return side * side;
    }
}
```

### I - Interface Segregation Principle
Clients should not be forced to depend on interfaces they don't use.

```java
// Bad: Fat interface
public interface Worker {
    void work();
    void eat();
    void sleep();
}

public class Robot implements Worker {
    public void work() { }
    public void eat() { 
        // Robots don't eat - forced to implement
        throw new UnsupportedOperationException();
    }
    public void sleep() { 
        // Robots don't sleep
        throw new UnsupportedOperationException();
    }
}

// Good: Segregated interfaces
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public class Human implements Workable, Eatable, Sleepable {
    public void work() { }
    public void eat() { }
    public void sleep() { }
}

public class Robot implements Workable {
    public void work() { }
}
```

### D - Dependency Inversion Principle
High-level modules should not depend on low-level modules. Both should depend on abstractions.

```java
// Bad: High-level class depends on low-level class
public class EmailService {
    public void sendEmail(String message) {
        // Send email logic
    }
}

public class NotificationService {
    private EmailService emailService = new EmailService();
    
    public void notify(String message) {
        emailService.sendEmail(message);
    }
}

// Good: Both depend on abstraction
public interface MessageSender {
    void send(String message);
}

public class EmailService implements MessageSender {
    public void send(String message) {
        // Send email
    }
}

public class SMSService implements MessageSender {
    public void send(String message) {
        // Send SMS
    }
}

public class NotificationService {
    private MessageSender messageSender;
    
    public NotificationService(MessageSender messageSender) {
        this.messageSender = messageSender;
    }
    
    public void notify(String message) {
        messageSender.send(message);
    }
}
```

## Creational Patterns

### Q2: Explain Singleton pattern and its implementations.
**Answer:**
Singleton ensures only one instance of a class exists.

```java
// 1. Eager Initialization
public class EagerSingleton {
    private static final EagerSingleton INSTANCE = new EagerSingleton();
    
    private EagerSingleton() {}
    
    public static EagerSingleton getInstance() {
        return INSTANCE;
    }
}

// 2. Lazy Initialization with Double-Checked Locking
public class LazySingleton {
    private static volatile LazySingleton instance;
    
    private LazySingleton() {}
    
    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}

// 3. Bill Pugh Solution (Nested Class)
public class BillPughSingleton {
    private BillPughSingleton() {}
    
    private static class SingletonHelper {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }
    
    public static BillPughSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}

// 4. Enum Singleton (Recommended)
public enum EnumSingleton {
    INSTANCE;
    
    public void doSomething() {
        // Business logic
    }
}
```

### Q3: Explain Factory Method and Abstract Factory patterns.
**Answer:**

**Factory Method:** Defines interface for creating objects, but lets subclasses decide which class to instantiate.

```java
// Factory Method
public interface Vehicle {
    void drive();
}

public class Car implements Vehicle {
    public void drive() {
        System.out.println("Driving a car");
    }
}

public class Bike implements Vehicle {
    public void drive() {
        System.out.println("Riding a bike");
    }
}

public abstract class VehicleFactory {
    public abstract Vehicle createVehicle();
    
    public void deliverVehicle() {
        Vehicle vehicle = createVehicle();
        vehicle.drive();
    }
}

public class CarFactory extends VehicleFactory {
    public Vehicle createVehicle() {
        return new Car();
    }
}

public class BikeFactory extends VehicleFactory {
    public Vehicle createVehicle() {
        return new Bike();
    }
}
```

**Abstract Factory:** Provides interface for creating families of related objects.

```java
// Abstract Factory
public interface GUIFactory {
    Button createButton();
    TextField createTextField();
}

public class WindowsFactory implements GUIFactory {
    public Button createButton() {
        return new WindowsButton();
    }
    
    public TextField createTextField() {
        return new WindowsTextField();
    }
}

public class MacFactory implements GUIFactory {
    public Button createButton() {
        return new MacButton();
    }
    
    public TextField createTextField() {
        return new MacTextField();
    }
}

public class Application {
    private Button button;
    private TextField textField;
    
    public Application(GUIFactory factory) {
        button = factory.createButton();
        textField = factory.createTextField();
    }
}
```

### Q4: Explain Builder pattern.
**Answer:**
Builder constructs complex objects step by step.

```java
public class User {
    private final String firstName;  // Required
    private final String lastName;   // Required
    private final int age;          // Optional
    private final String phone;     // Optional
    private final String address;   // Optional
    
    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }
    
    public static class UserBuilder {
        private final String firstName;
        private final String lastName;
        private int age;
        private String phone;
        private String address;
        
        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.UserBuilder("John", "Doe")
    .age(30)
    .phone("123-456-7890")
    .address("123 Main St")
    .build();
```

### Q5: Explain Prototype pattern.
**Answer:**
Prototype creates objects by cloning existing instances.

```java
public abstract class Shape implements Cloneable {
    private String id;
    protected String type;
    
    public String getId() {
        return id;
    }
    
    public void setId(String id) {
        this.id = id;
    }
    
    @Override
    public Shape clone() {
        try {
            return (Shape) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

public class Rectangle extends Shape {
    public Rectangle() {
        type = "Rectangle";
    }
}

public class Circle extends Shape {
    public Circle() {
        type = "Circle";
    }
}

public class ShapeCache {
    private static Map<String, Shape> shapeMap = new HashMap<>();
    
    public static void loadCache() {
        Circle circle = new Circle();
        circle.setId("1");
        shapeMap.put(circle.getId(), circle);
        
        Rectangle rectangle = new Rectangle();
        rectangle.setId("2");
        shapeMap.put(rectangle.getId(), rectangle);
    }
    
    public static Shape getShape(String shapeId) {
        Shape cachedShape = shapeMap.get(shapeId);
        return cachedShape.clone();
    }
}
```

## Structural Patterns

### Q6: Explain Adapter pattern.
**Answer:**
Adapter allows incompatible interfaces to work together.

```java
// Target interface
public interface MediaPlayer {
    void play(String filename);
}

// Adaptee
public class AdvancedMediaPlayer {
    public void playVlc(String filename) {
        System.out.println("Playing vlc file: " + filename);
    }
    
    public void playMp4(String filename) {
        System.out.println("Playing mp4 file: " + filename);
    }
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        advancedPlayer = new AdvancedMediaPlayer();
    }
    
    @Override
    public void play(String filename) {
        if (filename.endsWith(".vlc")) {
            advancedPlayer.playVlc(filename);
        } else if (filename.endsWith(".mp4")) {
            advancedPlayer.playMp4(filename);
        }
    }
}

// Client
public class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String filename) {
        if (filename.endsWith(".mp3")) {
            System.out.println("Playing mp3 file: " + filename);
        } else {
            mediaAdapter = new MediaAdapter(filename);
            mediaAdapter.play(filename);
        }
    }
}
```

### Q7: Explain Decorator pattern.
**Answer:**
Decorator adds new functionality to objects without altering their structure.

```java
// Component
public interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete Component
public class SimpleCoffee implements Coffee {
    public String getDescription() {
        return "Simple Coffee";
    }
    
    public double getCost() {
        return 2.0;
    }
}

// Decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
    
    public double getCost() {
        return decoratedCoffee.getCost();
    }
}

// Concrete Decorators
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.5;
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.2;
    }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
// Output: Simple Coffee, Milk, Sugar $2.7
```

### Q8: Explain Proxy pattern.
**Answer:**
Proxy provides a placeholder or surrogate for another object to control access to it.

```java
// Subject
public interface Image {
    void display();
}

// Real Subject
public class RealImage implements Image {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + fileName);
    }
    
    @Override
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}

// Proxy
public class ProxyImage implements Image {
    private RealImage realImage;
    private String fileName;
    
    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}

// Different types of proxies
// 1. Virtual Proxy - Lazy initialization
// 2. Protection Proxy - Access control
public class ProtectionProxy implements BankAccount {
    private RealBankAccount realAccount;
    private String userRole;
    
    public ProtectionProxy(String userRole) {
        this.userRole = userRole;
        this.realAccount = new RealBankAccount();
    }
    
    @Override
    public void withdraw(double amount) {
        if ("ADMIN".equals(userRole)) {
            realAccount.withdraw(amount);
        } else {
            throw new SecurityException("Insufficient privileges");
        }
    }
}

// 3. Remote Proxy - Hide remote communication
// 4. Caching Proxy - Cache results
public class CachingProxy implements DataService {
    private RealDataService realService;
    private Map<String, String> cache = new HashMap<>();
    
    @Override
    public String getData(String key) {
        if (cache.containsKey(key)) {
            return cache.get(key);
        }
        String data = realService.getData(key);
        cache.put(key, data);
        return data;
    }
}
```

### Q9: Explain Facade pattern.
**Answer:**
Facade provides a simplified interface to a complex subsystem.

```java
// Complex subsystem
public class CPU {
    public void freeze() { }
    public void jump(long position) { }
    public void execute() { }
}

public class Memory {
    public void load(long position, byte[] data) { }
}

public class HardDrive {
    public byte[] read(long lba, int size) {
        return new byte[size];
    }
}

// Facade
public class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}

// Client
ComputerFacade computer = new ComputerFacade();
computer.start();  // Simple interface to complex operations
```

## Behavioral Patterns

### Q10: Explain Observer pattern.
**Answer:**
Observer defines one-to-many dependency between objects so that when one object changes state, all dependents are notified.

```java
// Subject
public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// Observer
public interface Observer {
    void update(String message);
}

// Concrete Subject
public class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Concrete Observer
public class NewsChannel implements Observer {
    private String news;
    
    @Override
    public void update(String news) {
        this.news = news;
        System.out.println("NewsChannel received: " + news);
    }
}

// Java Built-in Observer (PropertyChangeListener)
public class Stock {
    private PropertyChangeSupport support = new PropertyChangeSupport(this);
    private double price;
    
    public void addPropertyChangeListener(PropertyChangeListener listener) {
        support.addPropertyChangeListener(listener);
    }
    
    public void setPrice(double newPrice) {
        double oldPrice = this.price;
        this.price = newPrice;
        support.firePropertyChange("price", oldPrice, newPrice);
    }
}
```

### Q11: Explain Strategy pattern.
**Answer:**
Strategy defines a family of algorithms, encapsulates each one, and makes them interchangeable.

```java
// Strategy Interface
public interface PaymentStrategy {
    void pay(double amount);
}

// Concrete Strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using Credit Card");
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using PayPal");
    }
}

// Context
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout() {
        double total = calculateTotal();
        paymentStrategy.pay(total);
    }
    
    private double calculateTotal() {
        return items.stream()
            .mapToDouble(Item::getPrice)
            .sum();
    }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment("1234-5678"));
cart.checkout();

cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
cart.checkout();
```

### Q12: Explain Template Method pattern.
**Answer:**
Template Method defines skeleton of algorithm in base class, letting subclasses override specific steps.

```java
public abstract class DataProcessor {
    // Template method
    public final void process() {
        readData();
        processData();
        saveData();
    }
    
    protected abstract void readData();
    protected abstract void processData();
    
    // Hook method with default implementation
    protected void saveData() {
        System.out.println("Saving to default storage");
    }
}

public class CSVDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }
}

public class XMLDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading XML file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing XML data");
    }
    
    @Override
    protected void saveData() {
        System.out.println("Saving to XML database");
    }
}
```

### Q13: Explain Chain of Responsibility pattern.
**Answer:**
Chain of Responsibility passes requests along a chain of handlers until one handles it.

```java
public abstract class SupportHandler {
    protected SupportHandler nextHandler;
    protected int level;
    
    public void setNextHandler(SupportHandler nextHandler) {
        this.nextHandler = nextHandler;
    }
    
    public void handleRequest(int level, String message) {
        if (this.level >= level) {
            handle(message);
        } else if (nextHandler != null) {
            nextHandler.handleRequest(level, message);
        } else {
            System.out.println("Request cannot be handled");
        }
    }
    
    protected abstract void handle(String message);
}

public class Level1Support extends SupportHandler {
    public Level1Support() {
        this.level = 1;
    }
    
    @Override
    protected void handle(String message) {
        System.out.println("Level 1 Support handling: " + message);
    }
}

public class Level2Support extends SupportHandler {
    public Level2Support() {
        this.level = 2;
    }
    
    @Override
    protected void handle(String message) {
        System.out.println("Level 2 Support handling: " + message);
    }
}

public class Level3Support extends SupportHandler {
    public Level3Support() {
        this.level = 3;
    }
    
    @Override
    protected void handle(String message) {
        System.out.println("Level 3 Support handling: " + message);
    }
}

// Setup chain
SupportHandler level1 = new Level1Support();
SupportHandler level2 = new Level2Support();
SupportHandler level3 = new Level3Support();

level1.setNextHandler(level2);
level2.setNextHandler(level3);

// Use chain
level1.handleRequest(1, "Simple issue");
level1.handleRequest(2, "Complex issue");
level1.handleRequest(3, "Critical issue");
```

### Q14: Explain Command pattern.
**Answer:**
Command encapsulates a request as an object, allowing you to parameterize clients with different requests.

```java
// Command Interface
public interface Command {
    void execute();
    void undo();
}

// Receiver
public class Light {
    private boolean isOn = false;
    
    public void turnOn() {
        isOn = true;
        System.out.println("Light is ON");
    }
    
    public void turnOff() {
        isOn = false;
        System.out.println("Light is OFF");
    }
}

// Concrete Commands
public class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOn();
    }
    
    @Override
    public void undo() {
        light.turnOff();
    }
}

public class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOff();
    }
    
    @Override
    public void undo() {
        light.turnOn();
    }
}

// Invoker
public class RemoteControl {
    private Command command;
    private Stack<Command> history = new Stack<>();
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
        history.push(command);
    }
    
    public void pressUndo() {
        if (!history.isEmpty()) {
            Command lastCommand = history.pop();
            lastCommand.undo();
        }
    }
}

// Macro Command
public class MacroCommand implements Command {
    private List<Command> commands;
    
    public MacroCommand(List<Command> commands) {
        this.commands = commands;
    }
    
    @Override
    public void execute() {
        commands.forEach(Command::execute);
    }
    
    @Override
    public void undo() {
        // Undo in reverse order
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
}
```

## Enterprise Patterns

### Q15: Explain Repository pattern.
**Answer:**
Repository mediates between domain and data mapping layers.

```java
// Domain Model
public class User {
    private Long id;
    private String username;
    private String email;
    // getters, setters
}

// Repository Interface
public interface UserRepository {
    User findById(Long id);
    List<User> findAll();
    List<User> findByEmail(String email);
    void save(User user);
    void delete(User user);
}

// Repository Implementation
@Repository
public class JpaUserRepository implements UserRepository {
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public User findById(Long id) {
        return entityManager.find(User.class, id);
    }
    
    @Override
    public List<User> findAll() {
        return entityManager
            .createQuery("SELECT u FROM User u", User.class)
            .getResultList();
    }
    
    @Override
    public List<User> findByEmail(String email) {
        return entityManager
            .createQuery("SELECT u FROM User u WHERE u.email = :email", User.class)
            .setParameter("email", email)
            .getResultList();
    }
    
    @Override
    @Transactional
    public void save(User user) {
        if (user.getId() == null) {
            entityManager.persist(user);
        } else {
            entityManager.merge(user);
        }
    }
    
    @Override
    @Transactional
    public void delete(User user) {
        entityManager.remove(entityManager.contains(user) ? 
            user : entityManager.merge(user));
    }
}

// Service using Repository
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User getUserById(Long id) {
        return userRepository.findById(id);
    }
}
```

### Q16: Explain Unit of Work pattern.
**Answer:**
Unit of Work maintains a list of objects affected by a business transaction and coordinates writing changes.

```java
public interface UnitOfWork {
    void registerNew(Entity entity);
    void registerModified(Entity entity);
    void registerDeleted(Entity entity);
    void commit();
    void rollback();
}

public class UnitOfWorkImpl implements UnitOfWork {
    private Set<Entity> newEntities = new HashSet<>();
    private Set<Entity> modifiedEntities = new HashSet<>();
    private Set<Entity> deletedEntities = new HashSet<>();
    
    @Override
    public void registerNew(Entity entity) {
        newEntities.add(entity);
    }
    
    @Override
    public void registerModified(Entity entity) {
        if (!newEntities.contains(entity)) {
            modifiedEntities.add(entity);
        }
    }
    
    @Override
    public void registerDeleted(Entity entity) {
        if (newEntities.remove(entity)) {
            return;
        }
        modifiedEntities.remove(entity);
        deletedEntities.add(entity);
    }
    
    @Override
    @Transactional
    public void commit() {
        try {
            insertNew();
            updateModified();
            deleteRemoved();
            clear();
        } catch (Exception e) {
            rollback();
            throw new RuntimeException("Commit failed", e);
        }
    }
    
    private void insertNew() {
        newEntities.forEach(entity -> 
            entityManager.persist(entity));
    }
    
    private void updateModified() {
        modifiedEntities.forEach(entity -> 
            entityManager.merge(entity));
    }
    
    private void deleteRemoved() {
        deletedEntities.forEach(entity -> 
            entityManager.remove(entity));
    }
    
    private void clear() {
        newEntities.clear();
        modifiedEntities.clear();
        deletedEntities.clear();
    }
}
```

### Q17: Explain DTO (Data Transfer Object) pattern.
**Answer:**
DTO carries data between processes, reducing network calls.

```java
// Entity
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String password;  // Sensitive
    private String email;
    private LocalDateTime createdAt;
    // Many other fields
}

// DTO
public class UserDTO {
    private Long id;
    private String username;
    private String email;
    // Only necessary fields, no sensitive data
    
    // Constructor for mapping
    public UserDTO(User user) {
        this.id = user.getId();
        this.username = user.getUsername();
        this.email = user.getEmail();
    }
}

// Mapper
@Component
public class UserMapper {
    public UserDTO toDTO(User user) {
        return new UserDTO(user);
    }
    
    public List<UserDTO> toDTOs(List<User> users) {
        return users.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }
    
    public User toEntity(UserDTO dto) {
        User user = new User();
        user.setUsername(dto.getUsername());
        user.setEmail(dto.getEmail());
        return user;
    }
}

// Controller using DTO
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public UserDTO getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return userMapper.toDTO(user);
    }
}
```

## Anti-Patterns

### Q18: What are common anti-patterns to avoid?
**Answer:**

**1. God Object/Class:**
- Class that knows too much or does too much
- Solution: Split into smaller, focused classes

**2. Spaghetti Code:**
- Code with complex and tangled control structure
- Solution: Refactor, use proper design patterns

**3. Copy-Paste Programming:**
- Duplicate code instead of creating reusable components
- Solution: DRY principle, extract common functionality

**4. Magic Numbers/Strings:**
```java
// Bad
if (status == 1) { }
if (type.equals("PREMIUM")) { }

// Good
if (status == Status.ACTIVE.getValue()) { }
if (type.equals(UserType.PREMIUM.name())) { }
```

**5. Premature Optimization:**
- Optimizing before knowing where bottlenecks are
- Solution: Profile first, optimize based on data

**6. Anemic Domain Model:**
```java
// Bad: Only getters/setters, no behavior
public class Order {
    private List<Item> items;
    private BigDecimal total;
    // Only getters and setters
}

// Good: Rich domain model
public class Order {
    private List<Item> items;
    private BigDecimal total;
    
    public void addItem(Item item) {
        items.add(item);
        recalculateTotal();
    }
    
    public void applyDiscount(Discount discount) {
        // Business logic
    }
}
```

## Pattern Selection & Trade-offs

### Q19: How to choose the right design pattern?
**Answer:**
**Consider:**
1. **Problem context**: What specific problem are you solving?
2. **Complexity**: Don't over-engineer simple problems
3. **Team familiarity**: Use patterns the team understands
4. **Performance requirements**: Some patterns add overhead
5. **Maintainability**: Will it make code easier to maintain?

**Decision Matrix:**
- **Creation problems** → Creational patterns
- **Structure problems** → Structural patterns
- **Behavior problems** → Behavioral patterns
- **Distributed systems** → Enterprise patterns

### Q20: What are trade-offs of common patterns?
**Answer:**

**Singleton:**
- ✅ Controlled access to single instance
- ❌ Global state, testing difficulties, concurrency issues

**Factory:**
- ✅ Decouples creation from usage
- ❌ Can add complexity for simple cases

**Observer:**
- ✅ Loose coupling, dynamic relationships
- ❌ Memory leaks if not properly unsubscribed

**Decorator:**
- ✅ Flexible alternative to subclassing
- ❌ Many small objects, debugging complexity

**Strategy:**
- ✅ Algorithm families, runtime switching
- ❌ Clients must know strategies

**Always consider:**
- YAGNI (You Aren't Gonna Need It)
- KISS (Keep It Simple, Stupid)
- Start simple, refactor to patterns when needed