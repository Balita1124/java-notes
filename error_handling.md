Here’s an example of code review focusing on proper usage of **`try-catch-finally`**, **`try-with-resources`**, and **exception handling** practices. We will review both an example of *bad* code and a *good* refactor that adheres to the guidelines.

### **Bad Example**

```java
public class FileService {

    public void readFile(String filePath) {
        FileReader reader = null;
        try {
            reader = new FileReader(filePath);
            int data;
            while ((data = reader.read()) != -1) {
                System.out.print((char) data);
            }
        } catch (IOException e) {
            // Swallowing exception, nothing is logged or handled
        } finally {
            try {
                if (reader != null) {
                    reader.close(); // Closing resources manually
                }
            } catch (IOException e) {
                // Exception ignored, nothing logged
            }
        }
    }
}
```

### **Review**:
1. **Swallowed Exception**: In the `catch` block, the exception is caught but no logging or handling is performed, so it's completely ignored.
2. **Improper Resource Management**: The `FileReader` resource is manually closed in the `finally` block, which is error-prone and requires additional exception handling.
3. **Exception in `finally` Block**: If `reader.close()` fails, the exception is swallowed again in the nested `try-catch` block in the `finally` clause.
4. **No Custom Exceptions**: There’s no use of custom exceptions that could provide more meaningful context or specific error information.

### **Good Example (Refactor)**

```java
import java.io.FileReader;
import java.io.IOException;
import java.util.logging.Level;
import java.util.logging.Logger;

public class FileService {

    private static final Logger LOGGER = Logger.getLogger(FileService.class.getName());

    public void readFile(String filePath) {
        // Use try-with-resources for automatic resource management
        try (FileReader reader = new FileReader(filePath)) {
            int data;
            while ((data = reader.read()) != -1) {
                System.out.print((char) data);
            }
        } catch (IOException e) {
            // Proper logging of the exception
            LOGGER.log(Level.SEVERE, "Error reading file at " + filePath, e);

            // Throw a custom exception with context
            throw new FileProcessingException("Failed to read file: " + filePath, e);
        }
    }
}

// Custom exception class
public class FileProcessingException extends RuntimeException {
    public FileProcessingException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### **Review & Improvements**:
1. **Proper Exception Logging**: The exception is logged using `Logger` and includes both a meaningful message and the exception stack trace (`e`).
   - `LOGGER.log(Level.SEVERE, "Error reading file", e)` logs the exception at `SEVERE` level, giving useful diagnostic information.
   
2. **Try-with-Resources**: Instead of manually closing the `FileReader`, the `try-with-resources` construct is used, which automatically closes the resource when done, even if an exception occurs.
   - `try (FileReader reader = new FileReader(filePath)) { ... }` ensures that the resource is properly closed without needing the manual `finally` block.
   
3. **Avoid Swallowing Exceptions**: The exceptions are not swallowed anymore. They are both logged and thrown, allowing the error to propagate or be handled appropriately by the calling code.

4. **Custom Exception**: A custom exception (`FileProcessingException`) is thrown with a clear message. This improves the clarity of error handling and provides better feedback when things go wrong.
   - `throw new FileProcessingException("Failed to read file: " + filePath, e)` allows better encapsulation of the error, giving context and wrapping the original `IOException` for more granular handling later.

### **Why This is Better**:
- **Better Exception Handling**: Exceptions are logged immediately and thrown as custom exceptions, preventing silent failures.
- **Readability**: Using `try-with-resources` makes the code cleaner and reduces the potential for resource leaks.
- **Maintainability**: The custom exception provides clearer, context-specific error messages, which are easier to track and troubleshoot during development and in production.

---

This code review example demonstrates how applying best practices in exception handling and resource management leads to more reliable, readable, and maintainable code.
