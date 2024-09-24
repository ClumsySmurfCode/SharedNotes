In **Spring Data JPA**, **derived query methods** allow you to automatically generate queries from the method names of your repository interface. These queries follow a predefined naming convention, and Spring will parse the method name to generate the corresponding SQL query.

### Syntax of Derived Query Methods

The method names are constructed based on the entity's property names and query keywords. The pattern is:
```
findBy<Property><Condition>(Parameters)
```

- `findBy`, `getBy`, `queryBy`, `readBy`, and `countBy` are common prefixes used to start the query method.
- `<Property>` is the entity’s field you want to query.
- `<Condition>` is the condition or operation you want to apply to the property (e.g., `Equals`, `Between`, `Like`, etc.).

### Common Keywords in Derived Queries:

1. **findBy**: Retrieves a record or records based on a field value.
   - Example: `findByFirstName(String firstName)`
     - This method will generate a query like `SELECT * FROM entity WHERE first_name = ?`.

2. **and / or**: Combines multiple conditions.
   - Example: `findByFirstNameAndLastName(String firstName, String lastName)`
     - This will generate a query like `SELECT * FROM entity WHERE first_name = ? AND last_name = ?`.

3. **Is / Equals**: Optional to use for readability.
   - Example: `findByFirstNameEquals(String firstName)`
     - Equivalent to `findByFirstName`.

4. **LessThan / GreaterThan / Between**: For numeric or date range comparisons.
   - Example: `findByAgeLessThan(Integer age)`
     - This will generate a query like `SELECT * FROM entity WHERE age < ?`.
   - Example: `findByDateBetween(LocalDate start, LocalDate end)`
     - This will generate `SELECT * FROM entity WHERE date BETWEEN ? AND ?`.

5. **Like / Containing / StartingWith / EndingWith**: For string pattern matching.
   - Example: `findByFirstNameLike(String firstName)`
     - This will generate a query like `SELECT * FROM entity WHERE first_name LIKE ?`.
   - Example: `findByFirstNameContaining(String firstName)`
     - This will generate `SELECT * FROM entity WHERE first_name LIKE %?%`.
   - Example: `findByFirstNameStartingWith(String prefix)`
     - This will generate `SELECT * FROM entity WHERE first_name LIKE ?%`.

6. **True / False**: For boolean fields.
   - Example: `findByActiveTrue()`
     - This will generate `SELECT * FROM entity WHERE active = true`.

7. **OrderBy**: For sorting the result.
   - Example: `findByLastNameOrderByFirstNameAsc(String lastName)`
     - This will generate a query like `SELECT * FROM entity WHERE last_name = ? ORDER BY first_name ASC`.

8. **In / NotIn**: For checking membership in a collection.
   - Example: `findByAgeIn(Collection<Integer> ages)`
     - This will generate `SELECT * FROM entity WHERE age IN ( ? )`.

9. **IsNull / IsNotNull**: For null checks.
   - Example: `findByLastNameIsNull()`
     - This will generate `SELECT * FROM entity WHERE last_name IS NULL`.

10. **Distinct**: For removing duplicate results.
    - Example: `findDistinctByLastName(String lastName)`
      - This will generate `SELECT DISTINCT * FROM entity WHERE last_name = ?`.

### Example: Derived Query Methods in Practice

Assume you have an entity called `User`:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String firstName;
    private String lastName;
    private Integer age;
    private Boolean active;
    private LocalDate createdDate;

    // Getters and setters...
}
```

Now, create a **repository interface** that extends `JpaRepository`:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // 1. Find users by first name
    List<User> findByFirstName(String firstName);

    // 2. Find users by first name and last name
    List<User> findByFirstNameAndLastName(String firstName, String lastName);

    // 3. Find users by age less than
    List<User> findByAgeLessThan(Integer age);

    // 4. Find users whose first name starts with a prefix
    List<User> findByFirstNameStartingWith(String prefix);

    // 5. Find users who are active
    List<User> findByActiveTrue();

    // 6. Find users created between two dates
    List<User> findByCreatedDateBetween(LocalDate startDate, LocalDate endDate);

    // 7. Find users ordered by last name in ascending order
    List<User> findByFirstNameOrderByLastNameAsc(String firstName);

    // 8. Find users where last name is null
    List<User> findByLastNameIsNull();
}
```

### Advanced Queries:
1. **Query with Limiting Results**:
   - Example: `findTop3ByOrderByAgeDesc()`
     - This will fetch the top 3 oldest users.
   
2. **Derived Count Queries**:
   - Example: `countByActiveTrue()`
     - This will count all users that are active.

3. **Not Equal Queries**:
   - Example: `findByAgeNot(Integer age)`
     - This will fetch users whose age is not equal to the given value.

### Using Derived Queries:

Once you have defined these methods in the `UserRepository`, you can simply call them in your service layer or controller, and Spring Data JPA will handle the query generation:

```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;

    public List<User> getUsersByFirstName(String firstName) {
        return userRepository.findByFirstName(firstName);
    }

    public List<User> getActiveUsers() {
        return userRepository.findByActiveTrue();
    }

    public List<User> getUsersCreatedBetween(LocalDate start, LocalDate end) {
        return userRepository.findByCreatedDateBetween(start, end);
    }
}
```

----------------------

# Detail explanation 

Certainly! Let's delve deeper into **Spring Data JPA derived query methods**, exploring advanced features, intricate naming conventions, best practices, and practical examples to provide a comprehensive understanding.

---

## Table of Contents

1. [Introduction to Derived Query Methods](#introduction)
2. [Detailed Naming Conventions](#naming-conventions)
   - [Property Expressions](#property-expressions)
   - [Logical Operators](#logical-operators)
   - [Comparison Operators](#comparison-operators)
   - [Sorting and Limiting Results](#sorting-and-limiting)
   - [Special Keywords](#special-keywords)
3. [Combining Multiple Keywords](#combining-keywords)
4. [Understanding How Spring Parses Method Names](#how-spring-parses)
5. [Advanced Examples](#advanced-examples)
6. [Pagination and Sorting](#pagination-sorting)
7. [Return Types](#return-types)
8. [Comparing Derived Queries with @Query Annotation](#derived-vs-query)
9. [Best Practices and Pitfalls](#best-practices)
10. [Performance Considerations](#performance)
11. [Custom Repository Methods](#custom-repository)
12. [Practical Implementation](#practical-implementation)
13. [Conclusion](#conclusion)

---

<a name="introduction"></a>
## 1. Introduction to Derived Query Methods

**Derived query methods** in Spring Data JPA allow you to define repository interface methods whose names follow specific patterns. Spring analyzes these method names, interprets the intended query, and automatically generates the corresponding SQL or JPQL queries at runtime.

This approach minimizes boilerplate code, enhances readability, and leverages the power of Spring Data JPA to simplify data access layers.

---

<a name="naming-conventions"></a>
## 2. Detailed Naming Conventions

Understanding the naming conventions is crucial for crafting effective derived query methods. The method names are parsed based on specific keywords and property expressions. Here's a detailed breakdown:

### a. Property Expressions

**Property expressions** refer to the entity's fields you want to query. They can be simple or nested (for related entities).

- **Simple Property**: Direct attribute of the entity.
  - Example: `findByFirstName`

- **Nested Property**: Attribute of a related entity.
  - Example: `findByAddress_City` (Assuming `User` has an `Address` entity with a `city` field)

**Note**: Use underscores (`_`) or camel case to navigate nested properties.

### b. Logical Operators

Logical operators allow combining multiple conditions.

- **And**: Both conditions must be true.
  - Example: `findByFirstNameAndLastName`

- **Or**: At least one condition must be true.
  - Example: `findByFirstNameOrLastName`

- **Not**: Negates a condition.
  - Example: `findByAgeNot`

### c. Comparison Operators

Comparison operators define the relationship between the property and the value.

- **Is, Equals**: Equality check.
  - Example: `findByStatusIs`, `findByStatusEquals`

- **Between**: Range between two values.
  - Example: `findByAgeBetween`

- **LessThan, LessThanEqual**: Less than or less than or equal to.
  - Example: `findByAgeLessThan`, `findByAgeLessThanEqual`

- **GreaterThan, GreaterThanEqual**: Greater than or greater than or equal to.
  - Example: `findByAgeGreaterThan`, `findByAgeGreaterThanEqual`

- **Like, NotLike**: Pattern matching using SQL `LIKE`.
  - Example: `findByNameLike`, `findByNameNotLike`

- **StartingWith, EndingWith, Containing**: Specialized `LIKE` operations.
  - Example: `findByNameStartingWith`, `findByNameEndingWith`, `findByNameContaining`

- **In, NotIn**: Membership in a collection.
  - Example: `findByStatusIn`, `findByStatusNotIn`

- **IsNull, IsNotNull**: Null checks.
  - Example: `findByMiddleNameIsNull`, `findByMiddleNameIsNotNull`

- **True, False**: Boolean checks.
  - Example: `findByActiveTrue`, `findByVerifiedFalse`

### d. Sorting and Limiting Results

- **OrderBy**: Specifies sorting.
  - Example: `findByLastNameOrderByFirstNameAsc`

- **Top, First**: Limits the number of results.
  - Example: `findTop3ByOrderByAgeDesc`, `findFirstByOrderByCreatedDate`

### e. Special Keywords

- **Distinct**: Removes duplicate results.
  - Example: `findDistinctByLastName`

- **IgnoreCase**: Case-insensitive queries.
  - Example: `findByFirstNameIgnoreCase`

---

<a name="combining-keywords"></a>
## 3. Combining Multiple Keywords

You can combine multiple keywords in a single method name to create complex queries. Here's how:

### Example 1: Multiple Logical Operators

```java
List<User> findByFirstNameAndLastNameOrEmail(String firstName, String lastName, String email);
```

**Interpreted Query**:
```
SELECT u FROM User u WHERE (u.firstName = ?1 AND u.lastName = ?2) OR u.email = ?3
```

**Note**: Parentheses are used to group conditions based on operator precedence.

### Example 2: Using Comparison and Logical Operators

```java
List<Order> findByTotalAmountGreaterThanAndStatusIn(Double amount, List<String> statuses);
```

**Interpreted Query**:
```
SELECT o FROM Order o WHERE o.totalAmount > ?1 AND o.status IN (?2)
```

### Example 3: Nested Properties with Sorting

```java
List<Employee> findByDepartment_NameAndAgeLessThanOrderByJoiningDateDesc(String departmentName, Integer age);
```

**Interpreted Query**:
```
SELECT e FROM Employee e WHERE e.department.name = ?1 AND e.age < ?2 ORDER BY e.joiningDate DESC
```

### Example 4: Combining Distinct and Like

```java
List<Product> findDistinctByNameContaining(String keyword);
```

**Interpreted Query**:
```
SELECT DISTINCT p FROM Product p WHERE p.name LIKE %?1%
```

### Example 5: Ignoring Case with Or

```java
List<Customer> findByFirstNameIgnoreCaseOrLastNameIgnoreCase(String firstName, String lastName);
```

**Interpreted Query**:
```
SELECT c FROM Customer c WHERE LOWER(c.firstName) = LOWER(?1) OR LOWER(c.lastName) = LOWER(?2)
```

---

<a name="how-spring-parses"></a>
## 4. Understanding How Spring Parses Method Names

Spring Data JPA uses **method name parsing** to derive the JPQL or SQL queries. Here's an overview of the process:

1. **Prefix Identification**: The method prefix (`findBy`, `getBy`, `readBy`, `countBy`, etc.) indicates the type of operation.

2. **Property Extraction**: Following the prefix, Spring identifies the properties involved in the query by matching them to the entity's fields. It supports nested properties via underscores or camel case.

3. **Operator Recognition**: Spring recognizes operators like `And`, `Or`, `Between`, `LessThan`, etc., and translates them into corresponding JPQL/SQL clauses.

4. **Keyword Handling**: Special keywords like `OrderBy`, `Distinct`, `IgnoreCase`, etc., are interpreted to modify the query behavior.

5. **Parameter Binding**: The method parameters are bound to the query's placeholders based on their order and type.

6. **Query Generation**: After parsing, Spring generates the JPQL or SQL query, which is then executed against the database.

**Behind the Scenes**:

- Spring Data uses **Parser Frameworks** and **Method Metadata** to parse method names.
- **Repositories** are dynamically proxied using **JDK Dynamic Proxies** or **CGLIB**.
- **Query Execution**: The parsed query is executed using **JPA's EntityManager**.

**Limitations**:

- Complex queries with multiple nested conditions can lead to very long and unreadable method names.
- Certain advanced SQL features (like subqueries, joins beyond simple associations) are not easily expressible via method names.

---

<a name="advanced-examples"></a>
## 5. Advanced Examples

Let's explore more complex scenarios to solidify understanding.

### Example 1: Query with Multiple Conditions and Sorting

```java
List<Employee> findByDepartment_NameAndSalaryGreaterThanOrAgeLessThanOrderByLastNameAscFirstNameDesc(String departmentName, Double salary, Integer age);
```

**Interpreted Query**:
```
SELECT e FROM Employee e 
WHERE (e.department.name = ?1 AND e.salary > ?2) OR e.age < ?3 
ORDER BY e.lastName ASC, e.firstName DESC
```

### Example 2: Query with Nested Properties and Distinct

Assuming `Book` entity has a relation to `Author` entity.

```java
List<Book> findDistinctByAuthor_LastNameAndTitleContainingIgnoreCase(String lastName, String titleKeyword);
```

**Interpreted Query**:
```
SELECT DISTINCT b FROM Book b 
WHERE b.author.lastName = ?1 AND LOWER(b.title) LIKE %LOWER(?2)%
```

### Example 3: Query with Between and In Operators

```java
List<Order> findByOrderDateBetweenAndStatusIn(LocalDate startDate, LocalDate endDate, List<String> statuses);
```

**Interpreted Query**:
```
SELECT o FROM Order o 
WHERE o.orderDate BETWEEN ?1 AND ?2 
AND o.status IN (?3)
```

### Example 4: Query with Null Checks and Ordering

```java
List<User> findByEmailIsNullOrderByCreatedDateDesc();
```

**Interpreted Query**:
```
SELECT u FROM User u 
WHERE u.email IS NULL 
ORDER BY u.createdDate DESC
```

### Example 5: Count Query

```java
Long countByStatusAndCreatedDateAfter(String status, LocalDate date);
```

**Interpreted Query**:
```
SELECT COUNT(u) FROM User u 
WHERE u.status = ?1 AND u.createdDate > ?2
```

---

<a name="pagination-sorting"></a>
## 6. Pagination and Sorting

Spring Data JPA supports **pagination** and **sorting** inherently, allowing methods to return `Page<T>` or `Slice<T>` instead of `List<T>`. Additionally, `Sort` parameters can be passed to dynamically sort results.

### Using Pageable for Pagination

**Repository Method**:

```java
Page<User> findByLastName(String lastName, Pageable pageable);
```

**Service Layer**:

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public Page<User> getUsersByLastName(String lastName, int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("firstName").ascending());
        return userRepository.findByLastName(lastName, pageable);
    }
}
```

**Controller Layer**:

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/users")
    public Page<User> getUsers(
            @RequestParam String lastName,
            @RequestParam int page,
            @RequestParam int size) {
        return userService.getUsersByLastName(lastName, page, size);
    }
}
```

### Dynamic Sorting with Sort Parameter

**Repository Method**:

```java
List<Product> findByCategory(String category, Sort sort);
```

**Service Layer**:

```java
public List<Product> getProductsByCategory(String category, String sortBy, String direction) {
    Sort sort = direction.equalsIgnoreCase("asc") ? Sort.by(sortBy).ascending() : Sort.by(sortBy).descending();
    return productRepository.findByCategory(category, sort);
}
```

**Controller Layer**:

```java
@GetMapping("/products")
public List<Product> getProducts(
        @RequestParam String category,
        @RequestParam String sortBy,
        @RequestParam String direction) {
    return productService.getProductsByCategory(category, sortBy, direction);
}
```

**Notes**:

- **Pageable**: Combines pagination and sorting information.
- **Page**: Includes the content as well as metadata like total pages, total elements, etc.
- **Slice**: Similar to `Page` but doesn't provide total count information.

---

<a name="return-types"></a>
## 7. Return Types

Spring Data JPA repository methods can return various types based on the requirement:

### Common Return Types

1. **Single Entity or Optional**

   - **Entity**: Returns a single entity. Throws `IncorrectResultSizeDataAccessException` if multiple results are found.
     ```java
     User findByEmail(String email);
     ```

   - **Optional<Entity>**: Handles the possibility of `null` gracefully.
     ```java
     Optional<User> findByEmail(String email);
     ```

2. **Collection Types**

   - **List<Entity>**: Returns a list of entities. Preserves order.
     ```java
     List<User> findByLastName(String lastName);
     ```

   - **Set<Entity>**: Returns a set of entities. Ensures uniqueness.
     ```java
     Set<User> findByActiveTrue();
     ```

   - **Collection<Entity>**: Generic collection interface.
     ```java
     Collection<User> findByAgeGreaterThan(Integer age);
     ```

3. **Page and Slice**

   - **Page<Entity>**: Includes paginated data with additional metadata.
     ```java
     Page<User> findByLastName(String lastName, Pageable pageable);
     ```

   - **Slice<Entity>**: Similar to `Page` but without total count.
     ```java
     Slice<User> findByLastName(String lastName, Pageable pageable);
     ```

4. **Stream<Entity>**

   - **Stream<Entity>**: Allows processing results as a Java 8 stream. Requires transactional context.
     ```java
     @Transactional(readOnly = true)
     Stream<User> findByCreatedDateAfter(LocalDate date);
     ```

5. **Custom Projections**

   - **DTOs**: Return specific fields mapped to a Data Transfer Object.
     ```java
     List<UserDTO> findByLastName(String lastName);
     ```

     ```java
     public interface UserDTO {
         String getFirstName();
         String getLastName();
     }
     ```

6. **Count Queries**

   - **Long**: Returns the count of matching entities.
     ```java
     Long countByStatus(String status);
     ```

   - **Integer**: Alternative count type.
     ```java
     Integer countByAgeGreaterThan(Integer age);
     ```

### Choosing the Right Return Type

- **Single Entity**: Use when expecting exactly one result or `Optional` when zero or one.
- **Collection Types**: Use for multiple results, with `List` being the most common.
- **Page/Slice**: Use for paginated results, especially in REST APIs.
- **Stream**: Use for processing large datasets efficiently.
- **Projections**: Use when only specific fields are needed, enhancing performance by reducing data retrieval.

---

<a name="derived-vs-query"></a>
## 8. Comparing Derived Queries with `@Query` Annotation

While **derived query methods** are powerful, sometimes you need more control over the query logic. This is where the `@Query` annotation comes into play.

### Derived Query Methods

- **Pros**:
  - Minimal boilerplate.
  - Automatically handled by Spring Data.
  - Easy to read for simple queries.
  - Type-safe and refactor-friendly.

- **Cons**:
  - Method names can become excessively long for complex queries.
  - Limited to the expressiveness of method naming conventions.
  - Hard to handle complex joins or subqueries.

### `@Query` Annotation

The `@Query` annotation allows you to define custom JPQL or SQL queries directly on repository methods.

- **Syntax**:
  ```java
  @Query("SELECT u FROM User u WHERE u.email = ?1")
  User findByEmailAddress(String email);
  ```

- **Using Native SQL**:
  ```java
  @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
  User findByEmailAddressNative(String email);
  ```

- **Named Parameters**:
  ```java
  @Query("SELECT u FROM User u WHERE u.firstName = :firstName AND u.lastName = :lastName")
  List<User> findByFullName(@Param("firstName") String firstName, @Param("lastName") String lastName);
  ```

- **Advanced Features**:
  - **Joins**: Handle complex relationships.
  - **Aggregations**: Perform calculations like COUNT, SUM, etc.
  - **Subqueries**: Nest queries within queries.
  - **Native Queries**: Use database-specific SQL features.

### When to Use Each

- **Use Derived Queries When**:
  - The query is simple and can be expressed clearly through method names.
  - You prefer type-safe, concise code.
  - You want to leverage Spring Data's query generation capabilities.

- **Use `@Query` When**:
  - The query is too complex for derived method names.
  - You need to optimize performance with custom JPQL or SQL.
  - You require specific database features not supported by JPQL.
  - You want to execute bulk operations or perform native queries.

### Example Comparison

**Derived Query Method**:

```java
List<User> findByLastNameAndAgeGreaterThan(String lastName, Integer age);
```

**Equivalent `@Query` Annotation**:

```java
@Query("SELECT u FROM User u WHERE u.lastName = :lastName AND u.age > :age")
List<User> findUsersByLastNameAndAge(@Param("lastName") String lastName, @Param("age") Integer age);
```

---

<a name="best-practices"></a>
## 9. Best Practices and Pitfalls

### Best Practices

1. **Keep Method Names Readable**:
   - Avoid overly long method names.
   - Use meaningful property names and logical groupings.

2. **Leverage Projections**:
   - Use DTOs or interface-based projections to fetch only necessary fields, improving performance.

3. **Use Optional for Single Results**:
   - Return `Optional<T>` to handle potential `null` values gracefully.

4. **Combine Derived Queries with `@Query`**:
   - Use derived queries for simple scenarios and `@Query` for complex ones.

5. **Utilize Pagination and Sorting**:
   - Implement `Pageable` and `Sort` to handle large datasets efficiently.

6. **Consistent Naming Conventions**:
   - Stick to a consistent naming strategy for repository methods to enhance readability and maintainability.

7. **Use `@Param` for Clarity**:
   - When using `@Query`, use named parameters for better readability.

8. **Document Repository Methods**:
   - Add JavaDoc comments to explain the purpose and behavior of complex query methods.

### Common Pitfalls

1. **Overly Complex Method Names**:
   - Extremely long and convoluted method names can become hard to read and maintain.

2. **Incorrect Property Names**:
   - Typos or incorrect property names in method names can lead to runtime errors.

3. **Ignoring Case Sensitivity**:
   - Be explicit about case sensitivity using `IgnoreCase` to avoid unexpected results.

4. **Assuming Fetch Strategies**:
   - Derived queries do not automatically manage fetch strategies. Use `@EntityGraph` or `JOIN FETCH` in `@Query` for eager loading.

5. **Misusing Logical Operators**:
   - Incorrect placement of `And` and `Or` can alter the intended logic. Use parentheses in `@Query` when necessary.

6. **Neglecting Indexing**:
   - Complex queries with multiple conditions may require appropriate database indexing to perform efficiently.

7. **Handling Nulls Improperly**:
   - Not accounting for `null` values can lead to unexpected query behavior.

8. **Ignoring Transaction Management**:
   - Ensure that repository methods that modify data are appropriately transactional.

---

<a name="performance"></a>
## 10. Performance Considerations

While derived query methods simplify development, it's essential to consider their performance implications:

### 1. **Query Optimization**

- **Selective Fields**: Fetch only required fields using projections to reduce data transfer and processing time.
  
- **Fetch Strategies**: Manage `EAGER` and `LAZY` fetch types appropriately to prevent unnecessary data loading or `N+1` select issues.

- **Indexes**: Ensure that frequently queried fields are indexed in the database to speed up lookups.

### 2. **Avoiding Unnecessary Queries**

- **Caching**: Implement caching mechanisms (e.g., second-level cache with Hibernate) to minimize repetitive database access.

- **Batch Operations**: For bulk inserts or updates, use batch processing to reduce the number of database calls.

### 3. **Pagination**

- **Use Pageable**: Implement pagination to handle large datasets efficiently, avoiding memory overflows and slow response times.

### 4. **Limit Derived Query Complexity**

- **Simplify Queries**: Keep derived queries simple. For complex logic, use `@Query` or custom repository implementations to optimize performance.

- **Monitor Query Execution**: Use tools like Hibernate's SQL logging or database profilers to monitor and optimize query performance.

### 5. **Handling Large Result Sets**

- **Streaming**: Use `Stream<T>` with appropriate transaction management to process large datasets without loading everything into memory.

- **Pagination**: Break down large result sets into manageable pages.

### 6. **Leveraging Native Queries When Necessary**

- **Performance Tuning**: In some cases, native SQL queries can be optimized better for performance than JPQL.

- **Database-Specific Features**: Utilize database-specific features for performance gains when using native queries.

---

<a name="custom-repository"></a>
## 11. Custom Repository Methods

For scenarios where derived queries and `@Query` annotations are insufficient, **custom repository methods** provide a way to implement complex logic.

### Steps to Create Custom Repository Methods

1. **Define a Custom Repository Interface**:

   ```java
   public interface UserRepositoryCustom {
       List<User> findUsersByCustomCriteria(String criteria);
   }
   ```

2. **Implement the Custom Interface**:

   ```java
   public class UserRepositoryCustomImpl implements UserRepositoryCustom {

       @PersistenceContext
       private EntityManager entityManager;

       @Override
       public List<User> findUsersByCustomCriteria(String criteria) {
           String jpql = "SELECT u FROM User u WHERE u.customField LIKE :criteria";
           return entityManager.createQuery(jpql, User.class)
                               .setParameter("criteria", "%" + criteria + "%")
                               .getResultList();
       }
   }
   ```

   **Naming Convention**: The implementation class should have the same name as the custom interface with `Impl` suffix.

3. **Extend the Custom Interface in the Repository**:

   ```java
   public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {
       // Derived query methods or @Query methods
   }
   ```

4. **Usage in Service Layer**:

   ```java
   @Service
   public class UserService {

       @Autowired
       private UserRepository userRepository;

       public List<User> getUsersByCustomCriteria(String criteria) {
           return userRepository.findUsersByCustomCriteria(criteria);
       }
   }
   ```

### Advantages

- **Flexibility**: Implement any custom logic using the `EntityManager` or other Spring components.
- **Separation of Concerns**: Keeps complex query logic separate from repository interface definitions.
- **Reusability**: Custom methods can be reused across different repositories if needed.

### Using Spring Data Specifications

Another approach for dynamic and complex queries is using **Spring Data JPA Specifications**.

- **Specification Interface**: Defines the criteria for queries.
  
  ```java
  public interface UserSpecification extends Specification<User> {
      // Custom criteria
  }
  ```

- **Using Specifications in Repository**:

  ```java
  List<User> findAll(Specification<User> spec);
  ```

- **Building Specifications**:

  ```java
  public class UserSpecifications {

      public static Specification<User> hasLastName(String lastName) {
          return (root, query, builder) -> builder.equal(root.get("lastName"), lastName);
      }

      public static Specification<User> hasAgeGreaterThan(Integer age) {
          return (root, query, builder) -> builder.greaterThan(root.get("age"), age);
      }
  }
  ```

- **Combining Specifications**:

  ```java
  Specification<User> spec = Specification.where(UserSpecifications.hasLastName("Doe"))
                                         .and(UserSpecifications.hasAgeGreaterThan(25));
  List<User> users = userRepository.findAll(spec);
  ```

**Benefits**:

- **Dynamic Querying**: Build queries dynamically based on runtime conditions.
- **Composability**: Combine multiple specifications using logical operators.

---

<a name="practical-implementation"></a>
## 12. Practical Implementation

Let's walk through a more elaborate, practical example that combines multiple aspects of derived query methods, pagination, sorting, and projections.

### Scenario

Suppose you have a **`Product`** entity with the following fields:

- `id` (Long)
- `name` (String)
- `description` (String)
- `price` (Double)
- `category` (String)
- `available` (Boolean)
- `createdDate` (LocalDate)
- `manufacturer` (Manufacturer)

And a **`Manufacturer`** entity:

- `id` (Long)
- `name` (String)
- `country` (String)

### Entities

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private Double price;
    private String category;
    private Boolean available;
    private LocalDate createdDate;

    @ManyToOne
    private Manufacturer manufacturer;

    // Getters and Setters
}

@Entity
public class Manufacturer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String country;

    // Getters and Setters
}
```

### Repository Interface

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    // 1. Find products by category and availability
    List<Product> findByCategoryAndAvailable(String category, Boolean available);

    // 2. Find top 5 most expensive products in a category
    List<Product> findTop5ByCategoryOrderByPriceDesc(String category);

    // 3. Find products with name containing keyword, case-insensitive
    List<Product> findByNameContainingIgnoreCase(String keyword);

    // 4. Count products by manufacturer country
    Long countByManufacturer_Country(String country);

    // 5. Find products created between dates and available
    List<Product> findByCreatedDateBetweenAndAvailable(LocalDate start, LocalDate end, Boolean available);

    // 6. Find products with price less than and sort by name ascending
    List<Product> findByPriceLessThanOrderByNameAsc(Double price);

    // 7. Find distinct categories of available products
    @Query("SELECT DISTINCT p.category FROM Product p WHERE p.available = true")
    List<String> findDistinctAvailableCategories();

    // 8. Find products by multiple categories using In
    List<Product> findByCategoryIn(List<String> categories);

    // 9. Find products by manufacturer name and price greater than
    List<Product> findByManufacturer_NameAndPriceGreaterThan(String manufacturerName, Double price);

    // 10. Find products where description is not null
    List<Product> findByDescriptionIsNotNull();
}
```

### Projection Interface

Suppose you want to retrieve only the product name and price.

```java
public interface ProductNamePriceProjection {
    String getName();
    Double getPrice();
}
```

**Repository Method**:

```java
List<ProductNamePriceProjection> findByCategory(String category);
```

### Service Layer

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> getAvailableProductsByCategory(String category) {
        return productRepository.findByCategoryAndAvailable(category, true);
    }

    public List<Product> getTopExpensiveProducts(String category, int top) {
        // Using Pageable for dynamic top 'n' results
        Pageable pageable = PageRequest.of(0, top, Sort.by("price").descending());
        return productRepository.findByCategory(category, pageable).getContent();
    }

    public List<Product> searchProductsByName(String keyword) {
        return productRepository.findByNameContainingIgnoreCase(keyword);
    }

    public Long countProductsByManufacturerCountry(String country) {
        return productRepository.countByManufacturer_Country(country);
    }

    public List<Product> getProductsByDateRange(LocalDate start, LocalDate end) {
        return productRepository.findByCreatedDateBetweenAndAvailable(start, end, true);
    }

    public List<Product> getAffordableProducts(Double maxPrice) {
        return productRepository.findByPriceLessThanOrderByNameAsc(maxPrice);
    }

    public List<String> getDistinctAvailableCategories() {
        return productRepository.findDistinctAvailableCategories();
    }

    public List<Product> getProductsByCategories(List<String> categories) {
        return productRepository.findByCategoryIn(categories);
    }

    public List<Product> getProductsByManufacturerAndPrice(String manufacturer, Double price) {
        return productRepository.findByManufacturer_NameAndPriceGreaterThan(manufacturer, price);
    }

    public List<Product> getProductsWithDescription() {
        return productRepository.findByDescriptionIsNotNull();
    }

    public List<ProductNamePriceProjection> getProductNamesAndPrices(String category) {
        return productRepository.findByCategory(category);
    }
}
```

### Controller Layer

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping("/available")
    public List<Product> getAvailableProducts(@RequestParam String category) {
        return productService.getAvailableProductsByCategory(category);
    }

    @GetMapping("/top-expensive")
    public List<Product> getTopExpensiveProducts(@RequestParam String category, @RequestParam int top) {
        return productService.getTopExpensiveProducts(category, top);
    }

    @GetMapping("/search")
    public List<Product> searchProducts(@RequestParam String keyword) {
        return productService.searchProductsByName(keyword);
    }

    @GetMapping("/count-by-country")
    public Long countByCountry(@RequestParam String country) {
        return productService.countProductsByManufacturerCountry(country);
    }

    @GetMapping("/date-range")
    public List<Product> getProductsByDateRange(@RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate start,
                                               @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate end) {
        return productService.getProductsByDateRange(start, end);
    }

    @GetMapping("/affordable")
    public List<Product> getAffordableProducts(@RequestParam Double maxPrice) {
        return productService.getAffordableProducts(maxPrice);
    }

    @GetMapping("/distinct-categories")
    public List<String> getDistinctAvailableCategories() {
        return productService.getDistinctAvailableCategories();
    }

    @GetMapping("/by-categories")
    public List<Product> getProductsByCategories(@RequestParam List<String> categories) {
        return productService.getProductsByCategories(categories);
    }

    @GetMapping("/manufacturer-price")
    public List<Product> getProductsByManufacturerAndPrice(@RequestParam String manufacturer,
                                                            @RequestParam Double price) {
        return productService.getProductsByManufacturerAndPrice(manufacturer, price);
    }

    @GetMapping("/with-description")
    public List<Product> getProductsWithDescription() {
        return productService.getProductsWithDescription();
    }

    @GetMapping("/names-prices")
    public List<ProductNamePriceProjection> getProductNamesAndPrices(@RequestParam String category) {
        return productService.getProductNamesAndPrices(category);
    }
}
```

### Testing the Endpoints

Assuming the application is running on `localhost:8080`, here are some example requests:

1. **Get Available Products by Category**:
   ```
   GET http://localhost:8080/api/products/available?category=Electronics
   ```

2. **Get Top 5 Expensive Products in a Category**:
   ```
   GET http://localhost:8080/api/products/top-expensive?category=Electronics&top=5
   ```

3. **Search Products by Name Keyword**:
   ```
   GET http://localhost:8080/api/products/search?keyword=phone
   ```

4. **Count Products by Manufacturer Country**:
   ```
   GET http://localhost:8080/api/products/count-by-country?country=USA
   ```

5. **Get Products Created Between Dates**:
   ```
   GET http://localhost:8080/api/products/date-range?start=2023-01-01&end=2023-12-31
   ```

6. **Get Affordable Products**:
   ```
   GET http://localhost:8080/api/products/affordable?maxPrice=500
   ```

7. **Get Distinct Available Categories**:
   ```
   GET http://localhost:8080/api/products/distinct-categories
   ```

8. **Get Products by Multiple Categories**:
   ```
   GET http://localhost:8080/api/products/by-categories?categories=Electronics,Books,Toys
   ```

9. **Get Products by Manufacturer and Price**:
   ```
   GET http://localhost:8080/api/products/manufacturer-price?manufacturer=Sony&price=1000
   ```

10. **Get Products with Description**:
    ```
    GET http://localhost:8080/api/products/with-description
    ```

11. **Get Product Names and Prices by Category**:
    ```
    GET http://localhost:8080/api/products/names-prices?category=Electronics
    ```

### Expected Responses

- **List of Products**: JSON array containing product details.
- **List of Strings**: JSON array of category names.
- **Count**: A single number representing the count.
- **Projections**: JSON array with only `name` and `price` fields.

---

<a name="conclusion"></a>
## 13. Conclusion

**Spring Data JPA derived query methods** offer a powerful, flexible, and concise way to interact with the database without writing explicit queries. By adhering to naming conventions and understanding the underlying mechanisms, developers can efficiently implement complex data retrieval logic.

However, it's essential to balance readability and maintainability, especially when dealing with complex queries. Combining derived methods with `@Query` annotations, custom repository implementations, and specifications can provide the necessary flexibility to handle diverse data access requirements.

**Key Takeaways**:

- **Understand Naming Conventions**: Mastering method naming is crucial for leveraging derived queries effectively.
- **Use Projections and Pagination**: Optimize performance and manage large datasets efficiently.
- **Combine with Other Features**: Use `@Query`, custom repositories, and specifications for advanced scenarios.
- **Follow Best Practices**: Maintain readability, ensure proper indexing, and handle transactions appropriately.
- **Monitor Performance**: Regularly assess and optimize query performance to ensure application responsiveness.
