# software-design-patterns-
LEGACY CODE ARCHAEOLOGY
## Purpose
- Apply pattern recognition to realistic code
- Practice reading and analyzing unfamiliar codebases
- Build collaborative problem-solving skills
- Prepare for afternoon presentations

**Team 1: Payment Processing System**

```java
// File: PaymentProcessor.java
public interface PaymentStrategy {
    boolean processPayment(double amount, String currency);
    String getProviderName();
    boolean supportsRefund();
}

// File: CreditCardPayment.java
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String expiryDate;
    private String cvv;
    
    public CreditCardPayment(String cardNumber, String expiryDate, String cvv) {
        this.cardNumber = cardNumber;
        this.expiryDate = expiryDate;
        this.cvv = cvv;
    }
    
    @Override
    public boolean processPayment(double amount, String currency) {
        // Connect to card processor API
        System.out.println("Processing $" + amount + " via credit card ending in " 
            + cardNumber.substring(cardNumber.length() - 4));
        // Simulate processing
        return true;
    }
    
    @Override
    public String getProviderName() {
        return "CreditCard";
    }
    
    @Override
    public boolean supportsRefund() {
        return true;
    }
}

// File: PayPalPayment.java
public class PayPalPayment implements PaymentStrategy {
    private String email;
    private String authToken;
    
    public PayPalPayment(String email) {
        this.email = email;
        this.authToken = authenticateWithPayPal(email);
    }
    
    private String authenticateWithPayPal(String email) {
        // OAuth flow simulation
        return "token_" + email.hashCode();
    }
    
    @Override
    public boolean processPayment(double amount, String currency) {
        System.out.println("Processing $" + amount + " via PayPal account: " + email);
        return true;
    }
    
    @Override
    public String getProviderName() {
        return "PayPal";
    }
    
    @Override
    public boolean supportsRefund() {
        return true;
    }
}

// File: CryptoPayment.java
public class CryptoPayment implements PaymentStrategy {
    private String walletAddress;
    
    public CryptoPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public boolean processPayment(double amount, String currency) {
        System.out.println("Initiating crypto transfer of " + amount + " " + currency 
            + " to wallet: " + walletAddress);
        return true;
    }
    
    @Override
    public String getProviderName() {
        return "Cryptocurrency";
    }
    
    @Override
    public boolean supportsRefund() {
        return false; // Crypto payments are non-refundable
    }
}

// File: ShoppingCart.java
public class ShoppingCart {
    private List<CartItem> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;
    
    public void addItem(CartItem item) {
        items.add(item);
    }
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public double getTotal() {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
    
    public boolean checkout() {
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment method not selected");
        }
        
        double total = getTotal();
        System.out.println("Checking out with " + paymentStrategy.getProviderName());
        
        boolean success = paymentStrategy.processPayment(total, "USD");
        
        if (success) {
            items.clear();
            System.out.println("Order complete!");
        }
        
        return success;
    }
}

// Usage example
public class Main {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(new CartItem("Laptop", 999.99, 1));
        cart.addItem(new CartItem("Mouse", 29.99, 2));
        
        // Customer chooses payment method at runtime
        cart.setPaymentStrategy(new CreditCardPayment("4111111111111111", "12/25", "123"));
        cart.checkout();
        
        // Different customer, different method
        ShoppingCart cart2 = new ShoppingCart();
        cart2.addItem(new CartItem("Book", 19.99, 1));
        cart2.setPaymentStrategy(new PayPalPayment("customer@email.com"));
        cart2.checkout();
    }
}
```

---

**Team 2: Notification System**

```java
// File: NotificationListener.java
public interface NotificationListener {
    void onNotification(String eventType, Map<String, Object> data);
}

// File: NotificationService.java
public class NotificationService {
    private Map<String, List<NotificationListener>> listeners = new HashMap<>();
    
    public void subscribe(String eventType, NotificationListener listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }
    
    public void unsubscribe(String eventType, NotificationListener listener) {
        List<NotificationListener> eventLiteners = listeners.get(eventType);
        if (eventListeners != null) {
            eventListeners.remove(listener);
        }
    }
    
    public void notify(String eventType, Map<String, Object> data) {
        List<NotificationListener> eventListeners = listeners.get(eventType);
        if (eventListeners != null) {
            for (NotificationListener listener : eventListeners) {
                try {
                    listener.onNotification(eventType, data);
                } catch (Exception e) {
                    System.err.println("Listener error: " + e.getMessage());
                    // Continue notifying other listeners
                }
            }
        }
    }
}

// File: EmailNotifier.java
public class EmailNotifier implements NotificationListener {
    private String recipientEmail;
    
    public EmailNotifier(String email) {
        this.recipientEmail = email;
    }
    
    @Override
    public void onNotification(String eventType, Map<String, Object> data) {
        String subject = "Notification: " + eventType;
        String body = formatEmailBody(eventType, data);
        sendEmail(recipientEmail, subject, body);
    }
    
    private String formatEmailBody(String eventType, Map<String, Object> data) {
        StringBuilder sb = new StringBuilder();
        sb.append("Event: ").append(eventType).append("\n\n");
        data.forEach((key, value) -> sb.append(key).append(": ").append(value).append("\n"));
        return sb.toString();
    }
    
    private void sendEmail(String to, String subject, String body) {
        System.out.println("Sending email to " + to);
        System.out.println("Subject: " + subject);
        System.out.println("Body: " + body);
    }
}

// File: SlackNotifier.java
public class SlackNotifier implements NotificationListener {
    private String webhookUrl;
    private String channel;
    
    public SlackNotifier(String webhookUrl, String channel) {
        this.webhookUrl = webhookUrl;
        this.channel = channel;
    }
    
    @Override
    public void onNotification(String eventType, Map<String, Object> data) {
        String message = formatSlackMessage(eventType, data);
        postToSlack(message);
    }
    
    private String formatSlackMessage(String eventType, Map<String, Object> data) {
        return String.format("*%s*\n%s", eventType, data.toString());
    }
    
    private void postToSlack(String message) {
        System.out.println("Posting to Slack channel " + channel + ": " + message);
    }
}

// File: DatabaseAuditLogger.java
public class DatabaseAuditLogger implements NotificationListener {
    private Connection connection;
    
    public DatabaseAuditLogger(Connection connection) {
        this.connection = connection;
    }
    
    @Override
    public void onNotification(String eventType, Map<String, Object> data) {
        String sql = "INSERT INTO audit_log (event_type, event_data, created_at) VALUES (?, ?, ?)";
        try (PreparedStatement stmt = connection.prepareStatement(sql)) {
            stmt.setString(1, eventType);
            stmt.setString(2, data.toString());
            stmt.setTimestamp(3, new Timestamp(System.currentTimeMillis()));
            stmt.executeUpdate();
        } catch (SQLException e) {
            System.err.println("Failed to log audit event: " + e.getMessage());
        }
    }
}

// Usage
public class OrderService {
    private NotificationService notifications;
    
    public OrderService(NotificationService notifications) {
        this.notifications = notifications;
    }
    
    public void placeOrder(Order order) {
        // Process order...
        order.setStatus(OrderStatus.CONFIRMED);
        
        // Notify all interested parties
        Map<String, Object> eventData = new HashMap<>();
        eventData.put("orderId", order.getId());
        eventData.put("customerId", order.getCustomerId());
        eventData.put("total", order.getTotal());
        eventData.put("status", order.getStatus());
        
        notifications.notify("ORDER_PLACED", eventData);
    }
}
```

---

**Team 3: Configuration Manager**

```java
// File: ConfigurationManager.java
public class ConfigurationManager {
    private static volatile ConfigurationManager instance;
    private Properties properties;
    private String environment;
    
    // Private constructor prevents direct instantiation
    private ConfigurationManager() {
        this.properties = new Properties();
        this.environment = System.getenv("APP_ENV");
        if (this.environment == null) {
            this.environment = "development";
        }
        loadConfiguration();
    }
    
    // Double-checked locking for thread safety
    public static ConfigurationManager getInstance() {
        if (instance == null) {
            synchronized (ConfigurationManager.class) {
                if (instance == null) {
                    instance = new ConfigurationManager();
                }
            }
        }
        return instance;
    }
    
    private void loadConfiguration() {
        String configFile = "config-" + environment + ".properties";
        try (InputStream input = getClass().getClassLoader().getResourceAsStream(configFile)) {
            if (input != null) {
                properties.load(input);
                System.out.println("Loaded configuration from " + configFile);
            } else {
                System.err.println("Config file not found: " + configFile);
            }
        } catch (IOException e) {
            throw new RuntimeException("Failed to load configuration", e);
        }
    }
    
    public String get(String key) {
        return properties.getProperty(key);
    }
    
    public String get(String key, String defaultValue) {
        return properties.getProperty(key, defaultValue);
    }
    
    public int getInt(String key, int defaultValue) {
        String value = properties.getProperty(key);
        if (value == null) return defaultValue;
        try {
            return Integer.parseInt(value);
        } catch (NumberFormatException e) {
            return defaultValue;
        }
    }
    
    public boolean getBoolean(String key, boolean defaultValue) {
        String value = properties.getProperty(key);
        if (value == null) return defaultValue;
        return Boolean.parseBoolean(value);
    }
    
    public String getEnvironment() {
        return environment;
    }
    
    // For testing: allows resetting the singleton
    // Should only be used in tests!
    static void resetForTesting() {
        instance = null;
    }
}

// Usage throughout the application
public class DatabaseConnection {
    public Connection connect() {
        ConfigurationManager config = ConfigurationManager.getInstance();
        String url = config.get("database.url");
        String user = config.get("database.user");
        String password = config.get("database.password");
        int poolSize = config.getInt("database.pool.size", 10);
        
        // Create connection...
        return DriverManager.getConnection(url, user, password);
    }
}

public class EmailService {
    public void sendEmail(String to, String subject, String body) {
        ConfigurationManager config = ConfigurationManager.getInstance();
        String smtpHost = config.get("email.smtp.host");
        int smtpPort = config.getInt("email.smtp.port", 587);
        boolean useTls = config.getBoolean("email.smtp.tls", true);
        
        // Send email...
    }
}
---

**Code Module 4: Report Generation System**

```java
// File: Report.java
public class Report {
    private String title;
    private String header;
    private List<String> sections = new ArrayList<>();
    private String footer;
    private String format; // "PDF", "HTML", "CSV"
    private boolean includeCharts;
    private boolean includeTableOfContents;
    private String watermark;
    private String author;
    private LocalDateTime generatedAt;
    
    // Private constructor - use Builder
    private Report(Builder builder) {
        this.title = builder.title;
        this.header = builder.header;
        this.sections = builder.sections;
        this.footer = builder.footer;
        this.format = builder.format;
        this.includeCharts = builder.includeCharts;
        this.includeTableOfContents = builder.includeTableOfContents;
        this.watermark = builder.watermark;
        this.author = builder.author;
        this.generatedAt = LocalDateTime.now();
    }
    
    // Getters...
    public String getTitle() { return title; }
    public String getFormat() { return format; }
    // ... etc
    
    public void render() {
        System.out.println("=== " + title + " ===");
        System.out.println("Format: " + format);
        System.out.println("Author: " + author);
        System.out.println("Generated: " + generatedAt);
        if (watermark != null) {
            System.out.println("Watermark: " + watermark);
        }
        if (includeTableOfContents) {
            System.out.println("\nTable of Contents:");
            for (int i = 0; i < sections.size(); i++) {
                System.out.println("  " + (i + 1) + ". Section " + (i + 1));
            }
        }
        System.out.println("\n" + header);
        for (String section : sections) {
            System.out.println("\n" + section);
        }
        if (includeCharts) {
            System.out.println("\n[Charts would be rendered here]");
        }
        System.out.println("\n" + footer);
    }
    
    // Builder class
    public static class Builder {
        // Required parameters
        private final String title;
        
        // Optional parameters with defaults
        private String header = "";
        private List<String> sections = new ArrayList<>();
        private String footer = "";
        private String format = "PDF";
        private boolean includeCharts = false;
        private boolean includeTableOfContents = false;
        private String watermark = null;
        private String author = "Unknown";
        
        // Builder constructor with required params
        public Builder(String title) {
            this.title = title;
        }
        
        public Builder header(String header) {
            this.header = header;
            return this;
        }
        
        public Builder addSection(String section) {
            this.sections.add(section);
            return this;
        }
        
        public Builder footer(String footer) {
            this.footer = footer;
            return this;
        }
        
        public Builder format(String format) {
            this.format = format;
            return this;
        }
        
        public Builder withCharts() {
            this.includeCharts = true;
            return this;
        }
        
        public Builder withTableOfContents() {
            this.includeTableOfContents = true;
            return this;
        }
        
        public Builder watermark(String watermark) {
            this.watermark = watermark;
            return this;
        }
        
        public Builder author(String author) {
            this.author = author;
            return this;
        }
        
        public Report build() {
            // Validation
            if (title == null || title.trim().isEmpty()) {
                throw new IllegalStateException("Report title is required");
            }
            if (sections.isEmpty()) {
                throw new IllegalStateException("Report must have at least one section");
            }
            return new Report(this);
        }
    }
}

// Usage examples
public class ReportGenerator {
    public static void main(String[] args) {
        // Simple report
        Report simpleReport = new Report.Builder("Monthly Sales")
            .author("Sales Team")
            .addSection("January: $10,000")
            .addSection("February: $12,000")
            .addSection("March: $15,000")
            .footer("Confidential")
            .build();
        
        // Complex report with all options
        Report complexReport = new Report.Builder("Q1 Financial Analysis")
            .author("Finance Department")
            .header("Quarterly Financial Report - Q1 2025")
            .addSection("Executive Summary: Strong quarter with 15% growth")
            .addSection("Revenue Analysis: Detailed breakdown by region")
            .addSection("Cost Analysis: Operating expenses increased 5%")
            .addSection("Projections: Q2 expected to maintain momentum")
            .footer("© 2025 Company Name. All Rights Reserved.")
            .format("PDF")
            .withCharts()
            .withTableOfContents()
            .watermark("DRAFT")
            .build();
        
        simpleReport.render();
        System.out.println("\n---\n");
        complexReport.render();
    }
}
```
