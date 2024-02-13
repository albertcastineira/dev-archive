# 🔐Basic Authentication System
---
## ⚠️ Requirements
- Springboot Project
- User JPA conected to a DB
---

### 1. User credentials & Auth Response
This are the core concepts of the user authentication. First we need to create this 2 files:

**AuthResponse.java**

```
// imports

public class AuthResponse {
    private String status;
    private String message;
    private String username;

    public AuthResponse(String status, String message, String username) {
        this.status = status;
        this.message = message;
        this.username = username;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

**UserCredentials.java**

```
// imports

@AllArgsConstructor
public class UserCredentials {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### 2: Adding methods to the User
The first thing we have to add is a way of searching the user by username and password. On the user repository we are going to add this method:

```
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {
    Optional<User> findByUsernameAndPassword(String username, String password);
}
```

Once we have it we need to create 2 methods on the user service:

```
public Optional<User> authenticateUser(String username, String password) {
        String hashedPassword = hashPassword(password);
        return userRepository.findByUsernameAndPassword(username, hashedPassword);
}

private String hashPassword(String password) {
    try {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hash = digest.digest(password.getBytes());
        StringBuilder hexString = new StringBuilder();

        for (byte b : hash) {
            String hex = Integer.toHexString(0xff & b);
            if (hex.length() == 1) hexString.append('0');
            hexString.append(hex);
        }

        return hexString.toString();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
        return null; // Handle this error appropriately in your application
    }
}
```

### 3: Creating auth endpoint
Now that we have all the needs we proceed to create a new endpoint to use this new methods to authenticate the user:

**api/users**

```
@PostMapping("/auth")
    public ResponseEntity<AuthResponse> userAuthentication(@RequestBody UserCredentials credentials) {
        String username = credentials.getUsername();
        String password = credentials.getPassword();

        Optional<User> authenticatedUser = userService.authenticateUser(username, password);

        if (authenticatedUser.isPresent()) {
            // User is authenticated, generate the authentication response
            User user = authenticatedUser.get();

            // Create the authentication response
            AuthResponse authResponse = new AuthResponse("success", "Login successful", user.getUsername());

            // Return the authentication response
            return ResponseEntity.ok(authResponse);
        } else {
            // Authentication failed
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(new AuthResponse("error", "Invalid credentials", null));
        }
}
```

### 4: Check the response from the server
The last part is checking the server response. If the login is succesfull it will return this on the endpoint: 

```http://localhost:8080/api/users/auth```

**JSON Endpoint Response**
```
// Status : 200 OK
{
    "status": "success",
    "message": "Login successful",
    "username": "Username"
}
```

> Warning: If the API has a security system remember to check the headers, or it will return a 401 Unauthorized Status
