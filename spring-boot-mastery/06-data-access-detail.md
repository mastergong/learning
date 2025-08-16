# Spring Data JPA: การจัดการข้อมูลแบบมืออาชีพด้วย Spring Boot

## 🎯 จุดประสงค์การเรียนรู้

เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจแนวคิดของ ORM และการทำงานของ Hibernate ใน Spring Boot
- กำหนด Entity, Repository และความสัมพันธ์ระหว่าง Entity อย่างถูกต้อง
- สร้าง Query ด้วยเทคนิคต่างๆ ของ Spring Data JPA
- ใช้ Transactions, Caching และกลไกการจัดการ N+1 Queries
- ปรับแต่ง Performance ของ JPA ในสภาพแวดล้อมการทำงานจริง

## 🌟 Spring Data JPA และ Hibernate

### **Spring Data JPA คืออะไร?**

Spring Data JPA เป็น subproject ของ Spring Data ที่ช่วยให้นักพัฒนาสามารถทำงานกับ relational databases ได้ง่ายขึ้น โดยลดโค้ดสำหรับ data access layer ด้วยการใช้แนวคิด repository pattern

```java
// ก่อนใช้ Spring Data JPA
public class UserDaoImpl implements UserDao {
    private EntityManager entityManager;
    
    @Autowired
    public UserDaoImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
    
    @Override
    public User findById(Long id) {
        return entityManager.find(User.class, id);
    }
    
    @Override
    public List<User> findAll() {
        return entityManager.createQuery("SELECT u FROM User u", User.class).getResultList();
    }
    
    @Override
    public void save(User user) {
        entityManager.persist(user);
    }
    
    @Override
    public void update(User user) {
        entityManager.merge(user);
    }
    
    @Override
    public void delete(User user) {
        entityManager.remove(user);
    }
}

// ด้วย Spring Data JPA
public interface UserRepository extends JpaRepository<User, Long> {
    // ได้รับฟังก์ชันพื้นฐานโดยอัตโนมัติ:
    // findById, findAll, save, delete, etc.
    
    // ฟังก์ชันที่กำหนดเอง
    List<User> findByEmail(String email);
}
```

### **Hibernate และ ORM**

Hibernate เป็น Object-Relational Mapping (ORM) framework ที่ใช้กันอย่างแพร่หลายใน Java และเป็น JPA provider หลักที่ Spring Boot ใช้โดย default

```
┌─────────────────────────┐
│   Spring Application    │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│     Spring Data JPA     │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│     JPA (Interface)     │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│ Hibernate (JPA Provider)│
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│      JDBC Driver        │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│        Database         │
└─────────────────────────┘
```

**ประโยชน์ของ ORM:**
- ลดโค้ด boilerplate สำหรับ data access
- จัดการความแตกต่างระหว่าง object model และ relational model
- มีระบบ caching เพื่อเพิ่มประสิทธิภาพ
- จัดการ database connection และ transaction
- ทำให้สามารถเปลี่ยน database ได้ง่าย

## 📋 Entity และ Relationships

### **Entity คืออะไร?**

Entity เป็น Plain Old Java Object (POJO) ที่แทน table ใน database โดยใช้ annotations เพื่อกำหนดความสัมพันธ์กับ database

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "first_name", length = 50, nullable = false)
    private String firstName;
    
    @Column(name = "last_name", length = 50, nullable = false)
    private String lastName;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_at")
    private Date createdAt;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "user_type")
    private UserType userType;
    
    // Getters and setters, equals, hashCode, toString
}
```

### **Annotations สำคัญสำหรับ Entity**

#### **@Entity และ @Table**
```java
@Entity // ระบุว่าคลาสนี้เป็น JPA entity
@Table(
    name = "products", // ชื่อตารางใน database
    schema = "inventory", // schema ที่ตารางอยู่
    uniqueConstraints = { // unique constraints
        @UniqueConstraint(columnNames = {"sku", "store_id"})
    },
    indexes = { // indexes เพื่อเพิ่มประสิทธิภาพการค้นหา
        @Index(name = "idx_product_name", columnList = "name")
    }
)
public class Product {
    // Entity properties
}
```

#### **@Id และ @GeneratedValue**
```java
@Id // Primary key
@GeneratedValue(strategy = GenerationType.IDENTITY) // Auto-increment
private Long id;

// หรือใช้ UUID เป็น primary key
@Id
@GeneratedValue(generator = "UUID")
@GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
@Column(updatable = false, nullable = false)
private UUID id;

// หรือใช้ Sequence
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "product_seq")
@SequenceGenerator(name = "product_seq", sequenceName = "product_sequence", allocationSize = 1)
private Long id;
```

#### **@Column**
```java
@Column(
    name = "product_name", // ชื่อคอลัมน์ใน database
    nullable = false, // ไม่อนุญาตให้เป็น null
    length = 100, // ความยาวสูงสุด (สำหรับ String)
    unique = true, // ต้องมีค่าไม่ซ้ำกัน
    insertable = true, // สามารถใช้ในคำสั่ง insert
    updatable = true, // สามารถใช้ในคำสั่ง update
    precision = 10, // ความแม่นยำทั้งหมด (สำหรับตัวเลข)
    scale = 2 // ทศนิยม (สำหรับตัวเลข)
)
private String name;
```

#### **@Temporal, @Enumerated และ @Lob**
```java
// จัดการกับวันที่และเวลา
@Temporal(TemporalType.TIMESTAMP) // DATE, TIME, หรือ TIMESTAMP
@Column(name = "created_at")
private Date createdAt;

// จัดการกับ Java 8 Date/Time API
@Column(name = "created_at")
private LocalDateTime createdAt;

// จัดการกับ enum
@Enumerated(EnumType.STRING) // หรือ EnumType.ORDINAL
private Status status;

// จัดการกับข้อมูลขนาดใหญ่
@Lob // สำหรับ Large Object (CLOB หรือ BLOB)
private String description; // CLOB

@Lob
private byte[] image; // BLOB
```

### **ความสัมพันธ์ระหว่าง Entities**

#### **One-to-One**
```java
// ด้าน owning (User)
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // ความสัมพันธ์แบบ one-to-one กับ UserProfile
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private UserProfile profile;
    
    // Getters and setters
}

// ด้าน non-owning (UserProfile)
@Entity
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(mappedBy = "profile")
    private User user;
    
    // Getters and setters
}
```

#### **One-to-Many / Many-to-One**
```java
// ด้าน "one" (Author)
@Entity
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    // ความสัมพันธ์แบบ one-to-many กับ Book
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Book> books = new ArrayList<>();
    
    // Helper methods
    public void addBook(Book book) {
        books.add(book);
        book.setAuthor(this);
    }
    
    public void removeBook(Book book) {
        books.remove(book);
        book.setAuthor(null);
    }
    
    // Getters and setters
}

// ด้าน "many" (Book)
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    // ความสัมพันธ์แบบ many-to-one กับ Author
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id")
    private Author author;
    
    // Getters and setters
}
```

#### **Many-to-Many**
```java
// ด้านแรก (Student)
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    // ความสัมพันธ์แบบ many-to-many กับ Course
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
    
    // Helper methods
    public void addCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
    
    public void removeCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }
    
    // Getters and setters
}

// ด้านที่สอง (Course)
@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    // ความสัมพันธ์แบบ many-to-many กับ Student
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
    
    // Getters and setters
}
```

### **Cascade Types และ Fetch Types**

#### **Cascade Types**
Cascade เป็นการกำหนดว่าเมื่อมีการดำเนินการกับ entity หนึ่ง จะส่งผลให้เกิดการดำเนินการเดียวกันกับ entity ที่มีความสัมพันธ์หรือไม่

```java
@OneToMany(cascade = CascadeType.ALL) // ทุกการดำเนินการจะถูก cascade
private List<Comment> comments;

@OneToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE}) // เฉพาะ persist และ merge
private List<Comment> comments;
```

**Cascade Types:**
- `CascadeType.ALL` - ทุกการดำเนินการ
- `CascadeType.PERSIST` - การบันทึก entity
- `CascadeType.MERGE` - การอัปเดต entity
- `CascadeType.REMOVE` - การลบ entity
- `CascadeType.REFRESH` - การรีเฟรช entity
- `CascadeType.DETACH` - การ detach entity

#### **Fetch Types**
Fetch type กำหนดว่าเมื่อโหลด entity หลัก จะโหลด entity ที่มีความสัมพันธ์ทันทีหรือไม่

```java
@OneToMany(fetch = FetchType.LAZY) // โหลดเมื่อมีการเรียกใช้เท่านั้น
private List<Comment> comments;

@OneToOne(fetch = FetchType.EAGER) // โหลดทันทีพร้อมกับ entity หลัก
private UserProfile profile;
```

**Fetch Types:**
- `FetchType.LAZY` - โหลดเมื่อมีการเรียกใช้เท่านั้น (ค่าเริ่มต้นสำหรับ @OneToMany และ @ManyToMany)
- `FetchType.EAGER` - โหลดทันทีพร้อมกับ entity หลัก (ค่าเริ่มต้นสำหรับ @OneToOne และ @ManyToOne)

#### **orphanRemoval**
การกำหนดว่าเมื่อ entity ลูกถูกลบออกจากความสัมพันธ์ (เช่น ถูกลบออกจาก collection) แล้วจะลบออกจาก database ด้วยหรือไม่

```java
@OneToMany(mappedBy = "post", orphanRemoval = true)
private List<Comment> comments;
```

## 🔍 Repository Interfaces

### **JpaRepository และลำดับชั้น**

Spring Data JPA มีลำดับชั้นของ interfaces ที่จัดเตรียมความสามารถในการเข้าถึงข้อมูล:

```
┌───────────────────┐
│   Repository<T,ID>│ - Marker interface
└─────────┬─────────┘
          │
┌─────────▼─────────┐
│CrudRepository<T,ID>│ - CRUD operations
└─────────┬─────────┘
          │
┌─────────▼──────────────┐
│PagingAndSortingRepository<T,ID>│ - Paging and sorting
└─────────┬──────────────┘
          │
┌─────────▼─────────┐
│JpaRepository<T,ID>│ - JPA specific operations
└───────────────────┘
```

```java
// Repository ทั่วไป
public interface UserRepository extends JpaRepository<User, Long> {
    // ได้รับเมธอดมากมายจาก JpaRepository:
    // - save(), findById(), findAll(), delete(), count() ฯลฯ
}
```

### **Method Naming Conventions**

Spring Data JPA สนับสนุนการสร้าง query methods โดยใช้ naming conventions

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // ค้นหาตาม property หนึ่งค่า
    User findByEmail(String email);
    
    // ค้นหาตาม property หลายค่า
    List<User> findByLastName(String lastName);
    
    // ค้นหาตาม property และเรียงลำดับ
    List<User> findByLastNameOrderByFirstNameAsc(String lastName);
    
    // ค้นหาโดยใช้ And
    List<User> findByFirstNameAndLastName(String firstName, String lastName);
    
    // ค้นหาโดยใช้ Or
    List<User> findByFirstNameOrLastName(String firstName, String lastName);
    
    // ค้นหาโดยใช้ Between
    List<User> findByAgeBetween(int startAge, int endAge);
    
    // ค้นหาโดยใช้ LessThan
    List<User> findByAgeLessThan(int age);
    
    // ค้นหาโดยใช้ GreaterThan
    List<User> findByAgeGreaterThan(int age);
    
    // ค้นหาโดยใช้ Like
    List<User> findByEmailLike(String pattern);
    
    // ค้นหาโดยใช้ NotLike
    List<User> findByEmailNotLike(String pattern);
    
    // ค้นหาโดยใช้ StartingWith
    List<User> findByFirstNameStartingWith(String prefix);
    
    // ค้นหาโดยใช้ EndingWith
    List<User> findByLastNameEndingWith(String suffix);
    
    // ค้นหาโดยใช้ Containing
    List<User> findByEmailContaining(String infix);
    
    // ค้นหาโดยใช้ In
    List<User> findByAgeIn(Collection<Integer> ages);
    
    // ค้นหาโดยใช้ NotIn
    List<User> findByAgeNotIn(Collection<Integer> ages);
    
    // ค้นหาโดยใช้ True/False
    List<User> findByActiveTrue();
    List<User> findByActiveFalse();
    
    // ค้นหาโดยใช้ IgnoreCase
    List<User> findByFirstNameIgnoreCase(String firstName);
    
    // นับโดยใช้ Count
    long countByLastName(String lastName);
    
    // เช็คว่ามีหรือไม่โดยใช้ Exists
    boolean existsByEmail(String email);
    
    // ลบโดยใช้ Delete
    void deleteByLastName(String lastName);
    
    // Top/First N results
    List<User> findTop3ByOrderByAgeDesc();
    
    // First result
    User findFirstByOrderByLastNameAsc();
    
    // Distinct results
    List<User> findDistinctByLastName(String lastName);
}
```

### **@Query Annotation**

เมื่อ method naming conventions ไม่เพียงพอ คุณสามารถใช้ `@Query` annotation เพื่อเขียน query แบบกำหนดเองได้

#### **JPQL Queries**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // JPQL query พื้นฐาน
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    User findByEmailAddress(String email);
    
    // JPQL กับตัวแปร named parameters
    @Query("SELECT u FROM User u WHERE u.firstName = :firstName AND u.lastName = :lastName")
    List<User> findByFirstNameAndLastName(@Param("firstName") String firstName, 
                                         @Param("lastName") String lastName);
    
    // Query ที่ซับซ้อน
    @Query("SELECT u FROM User u WHERE LOWER(u.email) LIKE LOWER(CONCAT('%', :keyword, '%')) " +
           "OR LOWER(u.firstName) LIKE LOWER(CONCAT('%', :keyword, '%')) " +
           "OR LOWER(u.lastName) LIKE LOWER(CONCAT('%', :keyword, '%'))")
    List<User> searchUsers(@Param("keyword") String keyword);
    
    // Query จากหลายตาราง
    @Query("SELECT u FROM User u JOIN u.roles r WHERE r.name = :roleName")
    List<User> findByRoleName(@Param("roleName") String roleName);
    
    // Query พร้อม aggregate function
    @Query("SELECT AVG(u.age) FROM User u WHERE u.active = true")
    Double getAverageAgeOfActiveUsers();
    
    // Query เพื่อ update
    @Modifying
    @Query("UPDATE User u SET u.active = :active WHERE u.lastLoginDate < :date")
    int updateUserActiveStatus(@Param("active") boolean active, @Param("date") LocalDate date);
    
    // Query เพื่อ delete
    @Modifying
    @Query("DELETE FROM User u WHERE u.lastLoginDate < :date")
    int deleteInactiveUsersSince(@Param("date") LocalDate date);
}
```

#### **Native SQL Queries**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Native SQL query
    @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
    User findByEmailNative(String email);
    
    // Native SQL query พร้อม named parameters
    @Query(value = "SELECT * FROM users WHERE first_name = :firstName AND last_name = :lastName", 
           nativeQuery = true)
    List<User> findByFirstNameAndLastNameNative(@Param("firstName") String firstName, 
                                               @Param("lastName") String lastName);
    
    // Native query พร้อม JOIN
    @Query(value = "SELECT u.* FROM users u " +
                  "JOIN user_roles ur ON u.id = ur.user_id " +
                  "JOIN roles r ON r.id = ur.role_id " +
                  "WHERE r.name = :roleName", 
           nativeQuery = true)
    List<User> findByRoleNameNative(@Param("roleName") String roleName);
    
    // Native query สำหรับ complex data
    @Query(value = "SELECT u.id, u.first_name, u.last_name, COUNT(o.id) as order_count " +
                  "FROM users u " +
                  "LEFT JOIN orders o ON u.id = o.user_id " +
                  "GROUP BY u.id " +
                  "HAVING COUNT(o.id) > :minOrders", 
           nativeQuery = true)
    List<Object[]> getUsersWithOrderCount(@Param("minOrders") int minOrders);
    
    // Native query กับ @Modifying
    @Modifying
    @Query(value = "UPDATE users SET active = :active WHERE last_login < :date", 
           nativeQuery = true)
    int updateUserActiveStatusNative(@Param("active") boolean active, @Param("date") LocalDate date);
}
```

### **Paging และ Sorting**

Spring Data JPA มีกลไกจัดการ pagination และ sorting อย่างง่าย

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Paging และ sorting กับ method naming
    Page<User> findByLastName(String lastName, Pageable pageable);
    
    // Sorting กับ method naming
    List<User> findByActiveTrue(Sort sort);
    
    // Paging และ sorting กับ @Query
    @Query("SELECT u FROM User u WHERE u.active = true")
    Page<User> findActiveUsers(Pageable pageable);
}
```

ตัวอย่างการใช้งาน:

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public Page<User> getUsersWithPagination(int page, int size) {
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findAll(pageable);
    }
    
    public Page<User> getUsersWithPaginationAndSorting(int page, int size, String sortBy, String direction) {
        Sort sort = direction.equalsIgnoreCase(Sort.Direction.ASC.name()) ?
                    Sort.by(sortBy).ascending() :
                    Sort.by(sortBy).descending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        return userRepository.findAll(pageable);
    }
    
    public Page<User> getActiveUsers(int page, int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("lastName").ascending());
        return userRepository.findActiveUsers(pageable);
    }
}
```

### **Specifications**

Specifications ช่วยให้คุณสร้าง dynamic queries ได้ง่าย

```java
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
    // ด้วยการ extends JpaSpecificationExecutor 
    // คุณได้รับเมธอด findAll(Specification)
}

// สร้าง Specifications
@Component
public class UserSpecifications {
    public static Specification<User> hasEmail(String email) {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.equal(root.get("email"), email);
    }
    
    public static Specification<User> hasLastName(String lastName) {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.equal(root.get("lastName"), lastName);
    }
    
    public static Specification<User> isActive() {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.isTrue(root.get("active"));
    }
    
    public static Specification<User> isOlderThan(int age) {
        return (root, query, criteriaBuilder) -> 
            criteriaBuilder.greaterThan(root.get("age"), age);
    }
}

// ใช้งาน Specifications
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public List<User> findActiveUsersWithLastName(String lastName) {
        return userRepository.findAll(
            Specification.where(UserSpecifications.isActive())
                         .and(UserSpecifications.hasLastName(lastName))
        );
    }
    
    public List<User> findActiveUsersOlderThan(int age) {
        return userRepository.findAll(
            Specification.where(UserSpecifications.isActive())
                         .and(UserSpecifications.isOlderThan(age))
        );
    }
}
```

### **Custom Repository Implementations**

บางครั้งคุณอาจต้องการเพิ่มเมธอดที่ซับซ้อนหรือไม่สามารถสร้างด้วย method naming conventions หรือ @Query annotation ได้

```java
// 1. สร้าง custom repository interface
public interface CustomUserRepository {
    List<User> findUsersWithComplexCriteria(String keyword, boolean active, List<String> roles);
}

// 2. สร้าง implementation
public class CustomUserRepositoryImpl implements CustomUserRepository {
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public List<User> findUsersWithComplexCriteria(String keyword, boolean active, List<String> roles) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> root = query.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        // เงื่อนไข keyword
        if (keyword != null && !keyword.isEmpty()) {
            String pattern = "%" + keyword.toLowerCase() + "%";
            Predicate keywordPredicate = cb.or(
                cb.like(cb.lower(root.get("firstName")), pattern),
                cb.like(cb.lower(root.get("lastName")), pattern),
                cb.like(cb.lower(root.get("email")), pattern)
            );
            predicates.add(keywordPredicate);
        }
        
        // เงื่อนไข active
        predicates.add(cb.equal(root.get("active"), active));
        
        // เงื่อนไข roles
        if (roles != null && !roles.isEmpty()) {
            Join<User, Role> rolesJoin = root.join("roles");
            predicates.add(rolesJoin.get("name").in(roles));
        }
        
        // รวมเงื่อนไข
        query.where(cb.and(predicates.toArray(new Predicate[0])));
        
        // distinct เพื่อป้องกันผลลัพธ์ซ้ำ
        query.distinct(true);
        
        return entityManager.createQuery(query).getResultList();
    }
}

// 3. รวม custom repository เข้ากับ JpaRepository
public interface UserRepository extends JpaRepository<User, Long>, CustomUserRepository {
    // ได้รับเมธอดทั้งจาก JpaRepository และ CustomUserRepository
}

// 4. ใช้งาน
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public List<User> searchUsers(String keyword, boolean active, List<String> roles) {
        return userRepository.findUsersWithComplexCriteria(keyword, active, roles);
    }
}
```

## 🛠️ Transactions Management

### **@Transactional Annotation**

การจัดการ transactions ใน Spring ทำได้ง่ายๆ ด้วย `@Transactional` annotation

```java
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private RoleRepository roleRepository;
    
    // ค่าเริ่มต้นคือ read-write transaction
    @Transactional
    public User registerUser(UserRegistrationDto dto) {
        User user = new User();
        user.setFirstName(dto.getFirstName());
        user.setLastName(dto.getLastName());
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setActive(true);
        
        // บันทึก user
        User savedUser = userRepository.save(user);
        
        // ถ้าเกิดข้อผิดพลาดที่นี่ ทุกอย่างจะ rollback
        Role userRole = roleRepository.findByName("ROLE_USER");
        savedUser.addRole(userRole);
        
        return savedUser;
    }
    
    // Transaction แบบอ่านอย่างเดียว
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    }
    
    // กำหนด isolation level
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void transferFunds(Long fromAccountId, Long toAccountId, BigDecimal amount) {
        // ...
    }
    
    // กำหนด propagation
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createAuditLog(String action, String detail) {
        // ...
    }
    
    // กำหนด timeout
    @Transactional(timeout = 30) // 30 วินาที
    public void runLongProcessingTask() {
        // ...
    }
    
    // กำหนดว่าต้อง rollback เมื่อเกิด exceptions ใด
    @Transactional(rollbackFor = {CustomException.class})
    public void processWithCustomExceptionHandling() {
        // ...
    }
    
    // กำหนดว่าไม่ต้อง rollback เมื่อเกิด exceptions ใด
    @Transactional(noRollbackFor = {NotFoundException.class})
    public void processSafely() {
        // ...
    }
}
```

### **Transaction Propagation Types**

Spring สนับสนุน propagation types หลายแบบเพื่อควบคุมพฤติกรรมของ transactions

```java
// REQUIRED - ใช้ transaction ที่มีอยู่หรือสร้างใหม่ถ้าไม่มี (ค่าเริ่มต้น)
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    // ใช้ transaction ที่มีอยู่หรือสร้างใหม่
    methodB(); // จะใช้ transaction เดียวกัน
}

// REQUIRES_NEW - สร้าง transaction ใหม่เสมอ และระงับ transaction ที่มีอยู่ชั่วคราว
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    // ทำงานใน transaction ใหม่เสมอ
}

// SUPPORTS - ใช้ transaction ที่มีอยู่ถ้ามี แต่ไม่สร้างใหม่ถ้าไม่มี
@Transactional(propagation = Propagation.SUPPORTS)
public void methodC() {
    // อาจทำงานใน transaction หรือไม่ก็ได้
}

// NOT_SUPPORTED - ไม่ใช้ transaction และระงับ transaction ที่มีอยู่
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void methodD() {
    // ทำงานโดยไม่มี transaction
}

// MANDATORY - ต้องมี transaction อยู่แล้ว ไม่เช่นนั้นจะเกิดข้อผิดพลาด
@Transactional(propagation = Propagation.MANDATORY)
public void methodE() {
    // ต้องถูกเรียกจากเมธอดที่มี transaction แล้ว
}

// NEVER - ต้องไม่มี transaction อยู่เลย ไม่เช่นนั้นจะเกิดข้อผิดพลาด
@Transactional(propagation = Propagation.NEVER)
public void methodF() {
    // ต้องไม่ถูกเรียกจากเมธอดที่มี transaction
}

// NESTED - สร้าง nested transaction ถ้ามี transaction อยู่แล้ว
@Transactional(propagation = Propagation.NESTED)
public void methodG() {
    // สร้าง savepoint ใน transaction ที่มีอยู่
    // หรือทำงานเหมือน REQUIRED ถ้าไม่มี transaction
}
```

### **Transaction Isolation Levels**

Spring สนับสนุน isolation levels หลายระดับเพื่อจัดการกับ concurrent transactions

```java
// DEFAULT - ใช้ค่าเริ่มต้นของ database
@Transactional(isolation = Isolation.DEFAULT)
public void methodA() {
    // ...
}

// READ_UNCOMMITTED - อ่านข้อมูลที่ยังไม่ commit ได้ (เร็วที่สุดแต่อาจเกิดปัญหา dirty reads)
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public void methodB() {
    // ...
}

// READ_COMMITTED - อ่านได้เฉพาะข้อมูลที่ commit แล้ว (ป้องกัน dirty reads แต่อาจเกิด non-repeatable reads)
@Transactional(isolation = Isolation.READ_COMMITTED)
public void methodC() {
    // ...
}

// REPEATABLE_READ - รับประกันว่าข้อมูลที่อ่านไปแล้วจะไม่เปลี่ยนแปลง (ป้องกัน non-repeatable reads แต่อาจเกิด phantom reads)
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void methodD() {
    // ...
}

// SERIALIZABLE - รับประกันว่า transactions จะทำงานเหมือนทำทีละ transaction (ปลอดภัยที่สุดแต่ช้าที่สุด)
@Transactional(isolation = Isolation.SERIALIZABLE)
public void methodE() {
    // ...
}
```

### **Programmatic Transaction Management**

นอกจาก @Transactional คุณสามารถจัดการ transaction แบบ programmatic ได้

```java
@Service
public class UserService {
    @Autowired
    private PlatformTransactionManager transactionManager;
    @Autowired
    private UserRepository userRepository;
    @Autowired
    private RoleRepository roleRepository;
    
    public User registerUser(UserRegistrationDto dto) {
        // กำหนดค่า transaction
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        def.setIsolationLevel(TransactionDefinition.ISOLATION_REPEATABLE_READ);
        def.setTimeout(30); // 30 วินาที
        
        // เริ่ม transaction
        TransactionStatus status = transactionManager.getTransaction(def);
        
        try {
            User user = new User();
            user.setFirstName(dto.getFirstName());
            user.setLastName(dto.getLastName());
            user.setEmail(dto.getEmail());
            user.setPassword(passwordEncoder.encode(dto.getPassword()));
            user.setActive(true);
            
            // บันทึก user
            User savedUser = userRepository.save(user);
            
            // เพิ่ม role
            Role userRole = roleRepository.findByName("ROLE_USER");
            savedUser.addRole(userRole);
            
            // commit transaction ถ้าทุกอย่างเรียบร้อย
            transactionManager.commit(status);
            
            return savedUser;
        } catch (Exception e) {
            // rollback transaction ถ้าเกิดข้อผิดพลาด
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```

## 🚀 Performance Optimization

### **Caching in JPA**

Hibernate มีกลไกการ cache หลายระดับเพื่อเพิ่มประสิทธิภาพ

#### **First-Level Cache (Session Cache)**
Cache นี้ทำงานโดยอัตโนมัติในระดับ Hibernate Session

```java
@Service
public class ProductService {
    @Autowired
    private EntityManager entityManager;
    
    @Transactional
    public void demonstrateFirstLevelCache() {
        // ครั้งแรกจะ hit database
        Product product = entityManager.find(Product.class, 1L);
        
        // ครั้งที่สองจะใช้จาก first-level cache โดยไม่ hit database
        Product sameProduct = entityManager.find(Product.class, 1L);
        
        // ทั้งสอง references ชี้ไปที่ object เดียวกัน
        System.out.println(product == sameProduct); // true
    }
}
```

#### **Second-Level Cache**
Cache นี้ทำงานระหว่าง sessions หลายๆ session ต้องเปิดใช้งานก่อน

```java
// ใน application.properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
spring.jpa.properties.hibernate.javax.cache.provider=org.ehcache.jsr107.EhcacheCachingProvider
spring.jpa.properties.hibernate.cache.use_query_cache=true

// Entity ที่ต้องการ cache
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    
    private String name;
    
    @Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
    @OneToMany(mappedBy = "product")
    private List<Review> reviews;
    
    // ...
}

// ใน query
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    @QueryHints({@QueryHint(name = "org.hibernate.cacheable", value = "true")})
    List<Product> findByCategory(String category);
}
```

#### **Query Cache**
Cache สำหรับผลลัพธ์ของ queries

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    @QueryHints({@QueryHint(name = "org.hibernate.cacheable", value = "true")})
    List<Product> findTop10ByOrderBySoldCountDesc();
}

// หรือด้วย EntityManager
@Service
public class ProductService {
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<Product> getTopProducts() {
        return entityManager.createQuery("SELECT p FROM Product p ORDER BY p.soldCount DESC", Product.class)
            .setHint("org.hibernate.cacheable", true)
            .setMaxResults(10)
            .getResultList();
    }
}
```

### **Handling N+1 Query Problem**

ปัญหา N+1 เกิดขึ้นเมื่อ JPA โหลด entities แล้วต้องทำ query เพิ่มสำหรับแต่ละความสัมพันธ์

#### **การใช้ Join Fetch**
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    // โดยปกติจะเกิดปัญหา N+1 ถ้า comments ใช้ lazy loading
    List<Post> findByAuthor(String author);
    
    // ใช้ JOIN FETCH เพื่อโหลด comments พร้อมกันใน query เดียว
    @Query("SELECT p FROM Post p JOIN FETCH p.comments WHERE p.author = :author")
    List<Post> findByAuthorWithComments(@Param("author") String author);
}

// หรือด้วย EntityManager
@Service
public class PostService {
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<Post> getPostsWithComments(String author) {
        return entityManager.createQuery(
            "SELECT DISTINCT p FROM Post p " +
            "JOIN FETCH p.comments " +
            "WHERE p.author = :author", Post.class)
            .setParameter("author", author)
            .getResultList();
    }
}
```

#### **การใช้ EntityGraph**
```java
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    // ใช้ @EntityGraph เพื่อระบุความสัมพันธ์ที่ต้องการโหลดพร้อมกัน
    @EntityGraph(attributePaths = {"comments"})
    List<Post> findByAuthor(String author);
    
    // สามารถระบุความสัมพันธ์หลายระดับได้
    @EntityGraph(attributePaths = {"comments", "comments.user"})
    Optional<Post> findById(Long id);
}

// หรือกำหนด EntityGraph ใน Entity
@Entity
@NamedEntityGraph(
    name = "Post.withCommentsAndUsers",
    attributeNodes = {
        @NamedAttributeNode(value = "comments", subgraph = "comments")
    },
    subgraphs = {
        @NamedSubgraph(
            name = "comments",
            attributeNodes = @NamedAttributeNode("user")
        )
    }
)
public class Post {
    // ...
}

// และใช้ใน Repository
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    @EntityGraph(value = "Post.withCommentsAndUsers")
    List<Post> findByAuthor(String author);
}
```

#### **การใช้ Batch Fetching**
```java
// ใน Entity
@Entity
public class Post {
    @Id
    private Long id;
    
    @BatchSize(size = 20)
    @OneToMany(mappedBy = "post")
    private List<Comment> comments;
}

// หรือตั้งค่าใน application.properties
spring.jpa.properties.hibernate.default_batch_fetch_size=20
```

### **Optimizing Queries**

#### **การใช้ Projections**
เมื่อไม่ต้องการข้อมูล entity ทั้งหมด ใช้ projections เพื่อดึงเฉพาะข้อมูลที่ต้องการ

```java
// สร้าง interface projection
public interface UserSummary {
    Long getId();
    String getFirstName();
    String getLastName();
    String getEmail();
}

// ใช้ใน Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findByLastName(String lastName);
    
    // หรือด้วย @Query
    @Query("SELECT u.id as id, u.firstName as firstName, u.lastName as lastName, u.email as email " +
           "FROM User u WHERE u.active = true")
    List<UserSummary> findActiveUsers();
}

// Class-based projection
public class UserSummaryDto {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    
    public UserSummaryDto(Long id, String firstName, String lastName, String email) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }
    
    // Getters...
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT new com.example.dto.UserSummaryDto(u.id, u.firstName, u.lastName, u.email) " +
           "FROM User u WHERE u.active = true")
    List<UserSummaryDto> findActiveUsersWithDto();
}
```

#### **การใช้ Native Queries เมื่อจำเป็น**
```java
@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
    // Native query ที่ซับซ้อนเกินกว่าที่ JPQL จะทำได้อย่างมีประสิทธิภาพ
    @Query(value = 
        "SELECT o.* FROM orders o " +
        "JOIN order_items oi ON o.id = oi.order_id " +
        "JOIN products p ON oi.product_id = p.id " +
        "WHERE p.category = :category " +
        "GROUP BY o.id " +
        "HAVING SUM(oi.quantity * p.price) > :minAmount " +
        "ORDER BY o.created_at DESC",
        nativeQuery = true)
    List<Order> findComplexOrdersData(@Param("category") String category, 
                                     @Param("minAmount") BigDecimal minAmount);
}
```

#### **การใช้ Pagination เพื่อลด Memory Usage**
```java
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    
    public void processAllOrders() {
        int page = 0;
        int pageSize = 100;
        Page<Order> orders;
        
        do {
            // โหลดข้อมูลทีละหน้า
            orders = orderRepository.findAll(PageRequest.of(page, pageSize));
            
            // ประมวลผลข้อมูล
            for (Order order : orders.getContent()) {
                processOrder(order);
            }
            
            // เพิ่มหน้า
            page++;
        } while (orders.hasNext());
    }
    
    private void processOrder(Order order) {
        // ประมวลผล order
    }
}
```

### **Stateless vs Stateful Interaction**

#### **การใช้ Detached Entities เพื่อลด Memory Usage**
```java
@Service
public class ProductService {
    @PersistenceContext
    private EntityManager entityManager;
    
    public void updateProductsStockLevel() {
        int page = 0;
        int pageSize = 100;
        List<Product> products;
        
        do {
            // โหลดข้อมูลทีละหน้าด้วย JPQL
            products = entityManager.createQuery("SELECT p FROM Product p", Product.class)
                .setFirstResult(page * pageSize)
                .setMaxResults(pageSize)
                .getResultList();
            
            for (Product product : products) {
                updateStockLevel(product);
                
                // เมื่อประมวลผลเสร็จ detach entity เพื่อประหยัด memory
                entityManager.detach(product);
            }
            
            page++;
        } while (!products.isEmpty());
    }
    
    @Transactional
    public void updateStockLevel(Product product) {
        // คำนวณ stock level ใหม่
        int newStockLevel = calculateNewStockLevel(product);
        
        // อัปเดต entity
        Product managedProduct = entityManager.find(Product.class, product.getId());
        managedProduct.setStockLevel(newStockLevel);
        
        // ไม่ต้อง call merge เพราะ entity อยู่ใน managed state ใน transaction นี้
    }
    
    private int calculateNewStockLevel(Product product) {
        // คำนวณ stock level ใหม่
        return product.getStockLevel();
    }
}
```

## 📝 แบบฝึกหัด

### **แบบฝึกหัดที่ 1: สร้าง Entity และความสัมพันธ์**

สร้างระบบการจองห้องพัก โดยมี entities ต่อไปนี้:
- Hotel - มี name, address, rating
- Room - มี number, type, price, availability
- Booking - มี check-in date, check-out date, status
- Guest - มี name, email, phone
- Review - มี rating, comment, date

กำหนดความสัมพันธ์:
- Hotel มี Rooms หลายห้อง (one-to-many)
- Guest สามารถทำ Bookings หลายรายการ (one-to-many)
- Room สามารถถูกจองโดยหลาย Bookings ในเวลาที่ต่างกัน (one-to-many)
- Guest สามารถเขียน Reviews สำหรับ Hotel ได้ (many-to-many)

### **แบบฝึกหัดที่ 2: สร้าง Repository Interface**

สร้าง repository interfaces สำหรับ entities ในแบบฝึกหัดที่ 1 พร้อมเพิ่ม query methods ต่อไปนี้:
- HotelRepository: findByRatingGreaterThanEqual, findByNameContaining
- RoomRepository: findByHotelIdAndAvailabilityTrue, findByTypeAndPriceBetween
- BookingRepository: findByGuestAndCheckOutDateGreaterThanEqual, findByRoomAndStatusIn
- GuestRepository: findByEmailContaining, existsByEmail
- ReviewRepository: findByHotelAndRatingGreaterThanEqual, countByHotel

### **แบบฝึกหัดที่ 3: สร้าง JPQL และ Native Queries**

สำหรับ HotelRepository เพิ่ม queries ต่อไปนี้:
- JPQL query หา hotels ที่มี availability สำหรับ date range ที่กำหนด
- JPQL query หา hotels พร้อม average rating
- Native query หา hotels ที่มี bookings มากที่สุด 10 อันดับแรก
- Native query หา revenue ของแต่ละ hotel ในช่วงเวลาที่กำหนด

### **แบบฝึกหัดที่ 4: แก้ปัญหา N+1 Queries**

ปรับปรุง queries ต่อไปนี้เพื่อแก้ปัญหา N+1:
- โหลด hotels พร้อม rooms ทั้งหมด
- โหลด guest พร้อม bookings ทั้งหมด
- โหลด hotel พร้อม reviews และ guests ที่เขียน reviews

### **แบบฝึกหัดที่ 5: สร้าง Service Layer กับ Transaction Management**

สร้าง BookingService ที่มีเมธอดต่อไปนี้:
- createBooking - สร้าง booking ใหม่และอัปเดต availability ของห้อง
- cancelBooking - ยกเลิก booking และอัปเดต availability ของห้อง
- transferBooking - ย้าย booking จากห้องหนึ่งไปอีกห้อง

ทุกเมธอดต้องมี transaction management ที่เหมาะสม

## 🔍 อ้างอิงเพิ่มเติม

- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/)
- [JPA Specification](https://download.oracle.com/otndocs/jcp/persistence-2_2-mrel-spec/index.html)
- [Spring Transaction Management](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)

---

**[⬅️ กลับไปที่บทที่ 5: Web Development](./05-web-development.md) | [➡️ ไปที่บทที่ 7: Security Basics](./07-security-basics.md)**
