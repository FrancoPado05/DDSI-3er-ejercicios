# Ejercicio para aprender Spring Boot: Gestión de Biblioteca

Crea una aplicación Spring Boot para gestionar libros de una biblioteca usando la estructura Model-Repository-Service-Controller.

---

## 📝 Enunciado

### Objetivo
Construir una API RESTful simple para gestionar tareas, donde puedas crear, listar, actualizar y eliminar tareas.

### Estructura requerida
1. **Model**: Define la entidad.
2. **Repository**: Interface para operaciones de base de datos.
3. **Service**: Lógica de negocio.
4. **Controller**: Manejo de peticiones HTTP.

src 

└── main

└── java

└── com.ejemplo.todo

├── model

│   └── Tarea.java

├── repository

│   └── TareaRepository.java

├── service

│   └── TareaService.java

├── controller

│   └── TareaController.java

└── TodoApplication.java


---

## 🧩 Ejemplos de ayuda (para otros contextos)

### Ejemplo 1: Model (Entidad "Product")
```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    private Double price;
    
    // Getters y setters
}
```

### 2. Repository (Operaciones para "Book")
```java
public interface BookRepository extends JpaRepository<Book, Long> {
    
    // Ejemplo 1: Buscar libros por autor
    List<Book> findByAuthor(String author);
    
    // Ejemplo 2: Buscar libros publicados después de un año específico
    List<Book> findByPublicationYearGreaterThan(Integer year);
    
    // Ejemplo 3: Buscar por título (ignore mayúsculas/minúsculas)
    List<Book> findByTitleContainingIgnoreCase(String keyword);
}
```
### 🛠️ Service (Lógica para "Book" - Explicación detallada)

**Objetivo**:  
Gestionar las reglas de negocio y coordinar las operaciones entre el Controller y el Repository.

```java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;  // Inyección del Repository

    // -------------------------------
    // 1. Crear libro con validación
    // -------------------------------
    public Book createBook(Book book) {
        // Validación personalizada: el año no puede ser futuro
        if (book.getPublicationYear() > Year.now().getValue()) {
            throw new IllegalArgumentException("❌ El año de publicación no puede ser futuro");
        }
        return bookRepository.save(book);  // Delegar al Repository
    }

    // ---------------------------------
    // 2. Buscar libros por rango de año
    // ---------------------------------
    public List<Book> getBooksByYearRange(int startYear, int endYear) {
        // Lógica compleja: combinar dos métodos del Repository
        List<Book> books = bookRepository.findByPublicationYearGreaterThanEqual(startYear);
        return books.stream()
                .filter(book -> book.getPublicationYear() <= endYear)
                .collect(Collectors.toList());
    }

    // --------------------------
    // 3. Eliminar libro por ID
    // --------------------------
    public void deleteBook(Long id) {
        if (!bookRepository.existsById(id)) {
            throw new RuntimeException("⚠️ Libro no encontrado (ID: " + id + ")");
        }
        bookRepository.deleteById(id);
    }
}
```
```java
@RestController
@RequestMapping("/api/books")
public class BookController {
    
    @Autowired
    private BookService bookService;

    // POST: Crear libro
    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody Book book) {
        return ResponseEntity.status(HttpStatus.CREATED).body(bookService.createBook(book));
    }

    // GET: Obtener todos los libros o filtrar por año
    @GetMapping
    public List<Book> getBooks(
        @RequestParam(required = false) Integer year
    ) {
        if (year != null) {
            return bookService.getBooksPublishedAfter(year);
        }
        return bookService.getAllBooks();
    }

    // DELETE: Eliminar libro por ID
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        bookService.deleteBook(id);
        return ResponseEntity.noContent().build();
    }
}
```