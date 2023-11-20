import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;


class Product {
  private int id;
  private String name;
  private double price;
  private String description;

  
}

// Observer pattern
interface AuthObserver {
  void onUserAuthenticated(String username);
}

class AuthenticationSubject {
  private List<AuthObserver> observers = new ArrayList<>();

  public void addObserver(AuthObserver observer) {
    observers.add(observer);
  }

  public void notifyObservers(String username) {
    for (AuthObserver observer : observers) {
      observer.onUserAuthenticated(username);
    }
  }
}

// Command pattern
interface Command {
  void execute();
}

class AddProductCommand implements Command {
  private JewelryShop shop;
  private Product product;

  public AddProductCommand(JewelryShop shop, Product product) {
    this.shop = shop;
    this.product = product;
  }

  @Override
  public void execute() {
    shop.addProduct(product);
  }
}

// Singleton pattern for Database Connection
public class JewelryShop {
  private static JewelryShop instance;

  private JewelryShop() {
    
  }

  public static JewelryShop getInstance() {
    if (instance == null) {
      instance = new JewelryShop();
    }
    return instance;
  }

  // Database connection information
  private static final String DB_URL = "jdbc:mysql://localhost:3306/jewelry_shop";
  private static final String DB_USERNAME = "your_username";
  private static final String DB_PASSWORD = "your_password";

  // SQL queries
  private static final String SELECT_ALL_PRODUCTS_QUERY = "SELECT * FROM products";
  private static final String INSERT_PRODUCT_QUERY = "INSERT INTO products (name, price, description) VALUES (?, ?, ?)";
  private static final String DELETE_PRODUCT_QUERY = "DELETE FROM products WHERE id = ?";

  // Authentication Subject
  private AuthenticationSubject authenticationSubject = new AuthenticationSubject();

  // Establish a connection to the database
  private Connection getConnection() throws SQLException {
    return DriverManager.getConnection(DB_URL, DB_USERNAME, DB_PASSWORD);
  }

  
  public List<Product> getAllProducts() {
    List<Product> products = new ArrayList<>();

    try (Connection conn = getConnection();
        PreparedStatement stmt = conn.prepareStatement(SELECT_ALL_PRODUCTS_QUERY);
        ResultSet rs = stmt.executeQuery()) {
      while (rs.next()) {
        Product product = new Product();
        product.setId(rs.getInt("id"));
        product.setName(rs.getString("name"));
        product.setPrice(rs.getDouble("price"));
        product.setDescription(rs.getString("description"));
        products.add(product);
      }
    } catch (SQLException e) {
      e.printStackTrace();
    }

    return products;
  }

  // Adding a new product to the shop
  public void addProduct(Product product) {
    try (Connection conn = getConnection(); PreparedStatement stmt = conn.prepareStatement(INSERT_PRODUCT_QUERY)) {
      stmt.setString(1, product.getName());
      stmt.setDouble(2, product.getPrice());
      stmt.setString(3, product.getDescription());
      stmt.executeUpdate();
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }

  // Remove a product from the shop
  public void deleteProduct(int productId) {
    try (Connection conn = getConnection(); PreparedStatement stmt = conn.prepareStatement(DELETE_PRODUCT_QUERY)) {
      stmt.setInt(1, productId);
      stmt.executeUpdate();
    } catch (SQLException e) {
      e.printStackTrace();
    }
  }

  // Factory method for creating products
  public Product createProduct(String name, double price, String description) {
    Product product = new Product();
    product.setName(name);
    product.setPrice(price);
    product.setDescription(description);
    return product;
  }

  // Observer pattern for authentication
  public void addObserver(AuthObserver observer) {
    authenticationSubject.addObserver(observer);
  }

  // Main method for Registration part
  public static void main(String[] args) throws SQLException {
    Scanner sc = new Scanner(System.in);
    System.out.println("Hi, Are u signed?");
    System.out.println("If you have signed, enter: YES. If not, enter: NO.");
    String answer = sc.nextLine();

    JewelryShop jewelryShop = JewelryShop.getInstance();

    if (answer.equals("YES")) {
      System.out.println("Login");
      String login = sc.nextLine();
      System.out.println("Password");
      String password = sc.nextLine();

      String query = "SELECT * FROM Login WHERE Phone = " + login + " AND Password = " + password;
      Statement st = jewelryShop.getConnection().createStatement();
      boolean ans = st.execute(query);

      if (ans) {
        // authentication successful
        jewelryShop.authenticationSubject.notifyObservers(login);
      } else {
        System.out.println("Login or password is invalid");
      }
    } else {
      System.out.println("Name");
      String name = sc.nextLine();
      System.out.println("Surname");
      String surname = sc.nextLine();
      System.out.println("Phone");
      String phone = sc.nextLine();

   
      Product newProduct = jewelryShop.createProduct("New Product", 99.99, "Description of the new product");
      Command addProductCommand = new AddProductCommand(jewelryShop, newProduct);
      addProductCommand.execute();
    }
  }
}
