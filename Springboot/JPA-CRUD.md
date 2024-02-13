# 📦JPA CRUD
---
## ⚠️ Requirements
- Working database ( SQL )
- Springboot project
- Dependencies: 
  - javax.persistence-api
  - hibernate-core
  - mysql-connector-java

---

### 1. Database Connection
First of all we need to make sure that on our springboot application.properties file we have:
```
# DataSource settings
spring.datasource.url=jdbc:mysql://localhost:PORT/DATABASE
spring.datasource.username=USER
spring.datasource.password=PASSWORD

# Hibernate settings
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# Logging
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=trace
```
We will assume that you already have a table with data, in our case it will be the User table.

### 2. Project structure
The typical folder structure for file organization on a JPA CRUD application is:

- 📁 YourProject
  - 📁 controller
  📄 YourEntityController.java
  - 📁 service
  📄 YourEntityService.java
  - 📁 repository
  📄 YourEntityRepository.java
  - 📁 model
  📄 YourEntity.java

### 3. Creating the files
In this case we will make a users CRUD:

**User.java**
```
// imports

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String email;

    // Constructors, getters, and setters
    public User() {}

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }

    // Getters and setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

**UserRepository.java**

```
// imports

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // You can define custom query methods here if needed
}
```

**UserService.java**

```
// imports

@Service
public class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    public Optional<User> getUserById(Long id) {
        return userRepository.findById(id);
    }

    public User createUser(User user) {
        return userRepository.save(user);
    }

    public User updateUser(Long id, User newUser) {
        return userRepository.findById(id)
                .map(user -> {
                    user.setUsername(newUser.getUsername());
                    user.setEmail(newUser.getEmail());
                    return userRepository.save(user);
                })
                .orElse(null);
    }

    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

**UserController.java**

```
// imports

@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public Optional<User> getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }

    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User newUser) {
        return userService.updateUser(id, newUser);
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }
}
```

### 4. Result
Once we made all the steps, we need to check if the endpoint is working. Make sure that the springboot application start is correct and has no errors or warnings.

Lets make a GET to "http://localhost:8080/api/users".

If the result shows all the users on the table you have a basic CRUD working.