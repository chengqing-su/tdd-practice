# Inventory Management System

## Problem Description

Build a comprehensive inventory management system with products, warehouses, stock movements, suppliers, and automated reordering. Focus on transactional consistency, event-driven architecture, and complex business rules.

## Requirements

### Domain Model
- `Product` - Items in inventory with SKU, name, description, category
- `Warehouse` - Storage locations with capacity and location details
- `Inventory` - Stock levels per product per warehouse
- `StockMovement` - Track all inventory changes (IN, OUT, TRANSFER)
- `Supplier` - Product suppliers with contact information
- `PurchaseOrder` - Orders placed with suppliers
- `ReorderRule` - Automatic reordering configuration

### Business Rules
- Stock levels must never go negative
- All stock movements must be recorded for audit
- Automatic reordering when stock falls below threshold
- Multi-warehouse stock allocation
- FIFO/LIFO inventory valuation methods
- Reserved stock for pending orders

### Key Features
- Real-time stock tracking
- Automated reorder point management
- Stock transfer between warehouses
- Low stock alerts and notifications
- Inventory valuation reports
- Supplier performance tracking

## Project Setup

### Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>kafka</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## Domain Entities

### Product Entity
```java
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false, length = 50)
    @NotBlank(message = "SKU is required")
    private String sku;

    @NotBlank(message = "Product name is required")
    @Size(max = 200, message = "Product name cannot exceed 200 characters")
    private String name;

    @Size(max = 1000, message = "Description cannot exceed 1000 characters")
    private String description;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "supplier_id")
    private Supplier primarySupplier;

    @DecimalMin(value = "0.0", message = "Unit cost must be non-negative")
    @Column(precision = 10, scale = 2)
    private BigDecimal unitCost;

    @DecimalMin(value = "0.0", message = "Unit price must be non-negative")
    @Column(precision = 10, scale = 2)
    private BigDecimal unitPrice;

    @Min(value = 0, message = "Minimum stock level must be non-negative")
    private Integer minimumStock = 0;

    @Min(value = 0, message = "Reorder quantity must be positive")
    private Integer reorderQuantity = 0;

    private boolean active = true;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    // constructors, getters, setters...
}
```

### Inventory Entity
```java
@Entity
@Table(name = "inventory", uniqueConstraints = {
    @UniqueConstraint(columnNames = {"product_id", "warehouse_id"})
})
public class Inventory {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "warehouse_id", nullable = false)
    private Warehouse warehouse;

    @Min(value = 0, message = "Available quantity cannot be negative")
    @Column(nullable = false)
    private Integer availableQuantity = 0;

    @Min(value = 0, message = "Reserved quantity cannot be negative")
    @Column(nullable = false)
    private Integer reservedQuantity = 0;

    @Min(value = 0, message = "In transit quantity cannot be negative")
    @Column(nullable = false)
    private Integer inTransitQuantity = 0;

    @LastModifiedDate
    private LocalDateTime lastUpdated;

    // constructors, getters, setters...

    public Integer getTotalQuantity() {
        return availableQuantity + reservedQuantity + inTransitQuantity;
    }
}
```

### Stock Movement Entity
```java
@Entity
@Table(name = "stock_movements")
public class StockMovement {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id", nullable = false)
    private Product product;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "warehouse_id", nullable = false)
    private Warehouse warehouse;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private MovementType movementType;

    @NotNull(message = "Quantity is required")
    private Integer quantity;

    @DecimalMin(value = "0.0", message = "Unit cost must be non-negative")
    @Column(precision = 10, scale = 2)
    private BigDecimal unitCost;

    @Size(max = 500, message = "Reference cannot exceed 500 characters")
    private String reference; // PO number, sales order, etc.

    @Size(max = 1000, message = "Notes cannot exceed 1000 characters")
    private String notes;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "created_by")
    private User createdBy;

    @CreatedDate
    private LocalDateTime createdAt;

    // constructors, getters, setters...
}

enum MovementType {
    PURCHASE_RECEIPT, SALES_SHIPMENT, STOCK_ADJUSTMENT,
    TRANSFER_IN, TRANSFER_OUT, RETURN_TO_SUPPLIER,
    CUSTOMER_RETURN, DAMAGE_WRITE_OFF
}
```

## Service Layer with Transactions

```java
@Service
@Transactional
public class InventoryService {

    private final InventoryRepository inventoryRepository;
    private final StockMovementRepository stockMovementRepository;
    private final ApplicationEventPublisher eventPublisher;

    public InventoryService(InventoryRepository inventoryRepository,
                           StockMovementRepository stockMovementRepository,
                           ApplicationEventPublisher eventPublisher) {
        this.inventoryRepository = inventoryRepository;
        this.stockMovementRepository = stockMovementRepository;
        this.eventPublisher = eventPublisher;
    }

    public void receiveStock(Long productId, Long warehouseId, Integer quantity,
                            BigDecimal unitCost, String reference) {
        validatePositiveQuantity(quantity);

        Inventory inventory = getOrCreateInventory(productId, warehouseId);

        // Update inventory
        inventory.setAvailableQuantity(inventory.getAvailableQuantity() + quantity);
        inventoryRepository.save(inventory);

        // Record stock movement
        recordStockMovement(inventory.getProduct(), inventory.getWarehouse(),
                          MovementType.PURCHASE_RECEIPT, quantity, unitCost, reference);

        // Publish event for downstream processing
        eventPublisher.publishEvent(new StockReceivedEvent(productId, warehouseId, quantity));
    }

    public void reserveStock(Long productId, Long warehouseId, Integer quantity, String reference) {
        validatePositiveQuantity(quantity);

        Inventory inventory = findInventory(productId, warehouseId)
            .orElseThrow(() -> new InsufficientStockException("No inventory found"));

        if (inventory.getAvailableQuantity() < quantity) {
            throw new InsufficientStockException(
                String.format("Insufficient stock. Available: %d, Required: %d",
                             inventory.getAvailableQuantity(), quantity));
        }

        // Move stock from available to reserved
        inventory.setAvailableQuantity(inventory.getAvailableQuantity() - quantity);
        inventory.setReservedQuantity(inventory.getReservedQuantity() + quantity);
        inventoryRepository.save(inventory);

        // Record movement for audit trail
        recordStockMovement(inventory.getProduct(), inventory.getWarehouse(),
                          MovementType.STOCK_ADJUSTMENT, -quantity, null, reference);

        eventPublisher.publishEvent(new StockReservedEvent(productId, warehouseId, quantity));
    }

    public void shipStock(Long productId, Long warehouseId, Integer quantity, String reference) {
        validatePositiveQuantity(quantity);

        Inventory inventory = findInventory(productId, warehouseId)
            .orElseThrow(() -> new InsufficientStockException("No inventory found"));

        if (inventory.getReservedQuantity() < quantity) {
            throw new InsufficientStockException(
                String.format("Insufficient reserved stock. Reserved: %d, Required: %d",
                             inventory.getReservedQuantity(), quantity));
        }

        // Remove from reserved
        inventory.setReservedQuantity(inventory.getReservedQuantity() - quantity);
        inventoryRepository.save(inventory);

        // Record shipment
        recordStockMovement(inventory.getProduct(), inventory.getWarehouse(),
                          MovementType.SALES_SHIPMENT, -quantity, null, reference);

        eventPublisher.publishEvent(new StockShippedEvent(productId, warehouseId, quantity));
    }

    public void transferStock(Long productId, Long fromWarehouseId, Long toWarehouseId,
                             Integer quantity, String reference) {
        validatePositiveQuantity(quantity);

        // Check source warehouse has enough stock
        Inventory fromInventory = findInventory(productId, fromWarehouseId)
            .orElseThrow(() -> new InsufficientStockException("Source inventory not found"));

        if (fromInventory.getAvailableQuantity() < quantity) {
            throw new InsufficientStockException("Insufficient stock for transfer");
        }

        // Reduce from source
        fromInventory.setAvailableQuantity(fromInventory.getAvailableQuantity() - quantity);
        inventoryRepository.save(fromInventory);

        // Add to destination
        Inventory toInventory = getOrCreateInventory(productId, toWarehouseId);
        toInventory.setAvailableQuantity(toInventory.getAvailableQuantity() + quantity);
        inventoryRepository.save(toInventory);

        // Record movements
        recordStockMovement(fromInventory.getProduct(), fromInventory.getWarehouse(),
                          MovementType.TRANSFER_OUT, -quantity, null, reference);
        recordStockMovement(toInventory.getProduct(), toInventory.getWarehouse(),
                          MovementType.TRANSFER_IN, quantity, null, reference);

        eventPublisher.publishEvent(new StockTransferredEvent(productId, fromWarehouseId, toWarehouseId, quantity));
    }

    @Transactional(readOnly = true)
    public List<Inventory> getLowStockItems() {
        return inventoryRepository.findLowStockItems();
    }

    @Transactional(readOnly = true)
    public InventoryReport generateInventoryReport(Long warehouseId) {
        List<Inventory> inventoryItems = warehouseId != null
            ? inventoryRepository.findByWarehouseId(warehouseId)
            : inventoryRepository.findAll();

        BigDecimal totalValue = inventoryItems.stream()
            .map(this::calculateInventoryValue)
            .reduce(BigDecimal.ZERO, BigDecimal::add);

        return InventoryReport.builder()
            .warehouseId(warehouseId)
            .totalItems(inventoryItems.size())
            .totalValue(totalValue)
            .generatedAt(LocalDateTime.now())
            .inventoryItems(inventoryItems)
            .build();
    }

    private Inventory getOrCreateInventory(Long productId, Long warehouseId) {
        return findInventory(productId, warehouseId)
            .orElseGet(() -> createNewInventory(productId, warehouseId));
    }

    private Optional<Inventory> findInventory(Long productId, Long warehouseId) {
        return inventoryRepository.findByProductIdAndWarehouseId(productId, warehouseId);
    }

    private Inventory createNewInventory(Long productId, Long warehouseId) {
        Inventory inventory = new Inventory();
        inventory.setProduct(new Product(productId));
        inventory.setWarehouse(new Warehouse(warehouseId));
        inventory.setAvailableQuantity(0);
        inventory.setReservedQuantity(0);
        inventory.setInTransitQuantity(0);
        return inventory;
    }

    private void recordStockMovement(Product product, Warehouse warehouse, MovementType type,
                                   Integer quantity, BigDecimal unitCost, String reference) {
        StockMovement movement = new StockMovement();
        movement.setProduct(product);
        movement.setWarehouse(warehouse);
        movement.setMovementType(type);
        movement.setQuantity(Math.abs(quantity)); // Store positive quantities
        movement.setUnitCost(unitCost);
        movement.setReference(reference);
        stockMovementRepository.save(movement);
    }

    private void validatePositiveQuantity(Integer quantity) {
        if (quantity == null || quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
    }

    private BigDecimal calculateInventoryValue(Inventory inventory) {
        // Simplified - in real world, would use FIFO/LIFO/weighted average
        return inventory.getProduct().getUnitCost()
            .multiply(new BigDecimal(inventory.getTotalQuantity()));
    }
}
```

## Event-Driven Architecture

### Domain Events
```java
public class StockReceivedEvent {
    private final Long productId;
    private final Long warehouseId;
    private final Integer quantity;
    private final LocalDateTime timestamp;

    public StockReceivedEvent(Long productId, Long warehouseId, Integer quantity) {
        this.productId = productId;
        this.warehouseId = warehouseId;
        this.quantity = quantity;
        this.timestamp = LocalDateTime.now();
    }

    // getters...
}

@Component
public class ReorderEventListener {

    private final ReorderService reorderService;
    private final NotificationService notificationService;

    public ReorderEventListener(ReorderService reorderService,
                               NotificationService notificationService) {
        this.reorderService = reorderService;
        this.notificationService = notificationService;
    }

    @EventListener
    @Async
    public void handleStockShipped(StockShippedEvent event) {
        // Check if reorder is needed
        reorderService.checkReorderPoint(event.getProductId(), event.getWarehouseId());
    }

    @EventListener
    @Async
    public void handleLowStock(LowStockEvent event) {
        // Send notification
        notificationService.sendLowStockAlert(event);
    }
}
```

## TDD Testing Examples

### 1. Repository Tests
```java
@DataJpaTest
class InventoryRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private InventoryRepository inventoryRepository;

    @Test
    void shouldFindLowStockItems() {
        // Given
        Product product1 = createProduct("PROD-001", 10); // min stock 10
        Product product2 = createProduct("PROD-002", 5);  // min stock 5
        Warehouse warehouse = createWarehouse("WH-001");

        entityManager.persistAndFlush(product1);
        entityManager.persistAndFlush(product2);
        entityManager.persistAndFlush(warehouse);

        // Low stock inventory (below minimum)
        Inventory lowStock = createInventory(product1, warehouse, 5); // below min of 10
        // Normal stock inventory
        Inventory normalStock = createInventory(product2, warehouse, 10); // above min of 5

        entityManager.persistAndFlush(lowStock);
        entityManager.persistAndFlush(normalStock);

        // When
        List<Inventory> lowStockItems = inventoryRepository.findLowStockItems();

        // Then
        assertThat(lowStockItems).hasSize(1);
        assertThat(lowStockItems.get(0).getProduct().getSku()).isEqualTo("PROD-001");
    }

    @Test
    void shouldCalculateTotalQuantityCorrectly() {
        // Given
        Product product = createProduct("PROD-001", 10);
        Warehouse warehouse = createWarehouse("WH-001");

        entityManager.persistAndFlush(product);
        entityManager.persistAndFlush(warehouse);

        Inventory inventory = new Inventory();
        inventory.setProduct(product);
        inventory.setWarehouse(warehouse);
        inventory.setAvailableQuantity(100);
        inventory.setReservedQuantity(20);
        inventory.setInTransitQuantity(15);

        entityManager.persistAndFlush(inventory);

        // When
        Optional<Inventory> found = inventoryRepository.findByProductIdAndWarehouseId(
            product.getId(), warehouse.getId());

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getTotalQuantity()).isEqualTo(135); // 100 + 20 + 15
    }

    private Product createProduct(String sku, Integer minimumStock) {
        Product product = new Product();
        product.setSku(sku);
        product.setName("Test Product " + sku);
        product.setMinimumStock(minimumStock);
        product.setReorderQuantity(50);
        product.setUnitCost(new BigDecimal("10.00"));
        return product;
    }

    private Warehouse createWarehouse(String code) {
        Warehouse warehouse = new Warehouse();
        warehouse.setCode(code);
        warehouse.setName("Test Warehouse");
        return warehouse;
    }

    private Inventory createInventory(Product product, Warehouse warehouse, Integer quantity) {
        Inventory inventory = new Inventory();
        inventory.setProduct(product);
        inventory.setWarehouse(warehouse);
        inventory.setAvailableQuantity(quantity);
        inventory.setReservedQuantity(0);
        inventory.setInTransitQuantity(0);
        return inventory;
    }
}
```

### 2. Service Tests with Transaction Verification
```java
@ExtendWith(MockitoExtension.class)
class InventoryServiceTest {

    @Mock
    private InventoryRepository inventoryRepository;

    @Mock
    private StockMovementRepository stockMovementRepository;

    @Mock
    private ApplicationEventPublisher eventPublisher;

    @InjectMocks
    private InventoryService inventoryService;

    @Test
    void shouldReceiveStockSuccessfully() {
        // Given
        Long productId = 1L;
        Long warehouseId = 1L;
        Integer quantity = 100;
        BigDecimal unitCost = new BigDecimal("15.50");
        String reference = "PO-12345";

        Inventory existingInventory = createInventory(productId, warehouseId, 50);

        when(inventoryRepository.findByProductIdAndWarehouseId(productId, warehouseId))
            .thenReturn(Optional.of(existingInventory));
        when(inventoryRepository.save(any(Inventory.class)))
            .thenAnswer(invocation -> invocation.getArgument(0));

        // When
        inventoryService.receiveStock(productId, warehouseId, quantity, unitCost, reference);

        // Then
        assertThat(existingInventory.getAvailableQuantity()).isEqualTo(150); // 50 + 100

        verify(inventoryRepository).save(existingInventory);
        verify(stockMovementRepository).save(argThat(movement ->
            movement.getMovementType() == MovementType.PURCHASE_RECEIPT &&
            movement.getQuantity().equals(quantity) &&
            movement.getUnitCost().equals(unitCost) &&
            movement.getReference().equals(reference)
        ));
        verify(eventPublisher).publishEvent(any(StockReceivedEvent.class));
    }

    @Test
    void shouldReserveStockWhenSufficientAvailable() {
        // Given
        Long productId = 1L;
        Long warehouseId = 1L;
        Integer quantityToReserve = 30;
        String reference = "SO-67890";

        Inventory inventory = createInventory(productId, warehouseId, 100);

        when(inventoryRepository.findByProductIdAndWarehouseId(productId, warehouseId))
            .thenReturn(Optional.of(inventory));

        // When
        inventoryService.reserveStock(productId, warehouseId, quantityToReserve, reference);

        // Then
        assertThat(inventory.getAvailableQuantity()).isEqualTo(70); // 100 - 30
        assertThat(inventory.getReservedQuantity()).isEqualTo(30);

        verify(inventoryRepository).save(inventory);
        verify(stockMovementRepository).save(any(StockMovement.class));
        verify(eventPublisher).publishEvent(any(StockReservedEvent.class));
    }

    @Test
    void shouldThrowExceptionWhenInsufficientStockForReservation() {
        // Given
        Long productId = 1L;
        Long warehouseId = 1L;
        Integer quantityToReserve = 150; // More than available

        Inventory inventory = createInventory(productId, warehouseId, 100);

        when(inventoryRepository.findByProductIdAndWarehouseId(productId, warehouseId))
            .thenReturn(Optional.of(inventory));

        // When & Then
        assertThatThrownBy(() ->
            inventoryService.reserveStock(productId, warehouseId, quantityToReserve, "SO-123"))
            .isInstanceOf(InsufficientStockException.class)
            .hasMessageContaining("Insufficient stock. Available: 100, Required: 150");

        // Verify no changes were made
        assertThat(inventory.getAvailableQuantity()).isEqualTo(100);
        assertThat(inventory.getReservedQuantity()).isEqualTo(0);
        verify(inventoryRepository, never()).save(any());
        verify(stockMovementRepository, never()).save(any());
        verify(eventPublisher, never()).publishEvent(any());
    }

    @Test
    void shouldTransferStockBetweenWarehouses() {
        // Given
        Long productId = 1L;
        Long fromWarehouseId = 1L;
        Long toWarehouseId = 2L;
        Integer quantity = 25;
        String reference = "TRANSFER-001";

        Inventory fromInventory = createInventory(productId, fromWarehouseId, 100);
        Inventory toInventory = createInventory(productId, toWarehouseId, 50);

        when(inventoryRepository.findByProductIdAndWarehouseId(productId, fromWarehouseId))
            .thenReturn(Optional.of(fromInventory));
        when(inventoryRepository.findByProductIdAndWarehouseId(productId, toWarehouseId))
            .thenReturn(Optional.of(toInventory));

        // When
        inventoryService.transferStock(productId, fromWarehouseId, toWarehouseId, quantity, reference);

        // Then
        assertThat(fromInventory.getAvailableQuantity()).isEqualTo(75); // 100 - 25
        assertThat(toInventory.getAvailableQuantity()).isEqualTo(75);   // 50 + 25

        verify(inventoryRepository, times(2)).save(any(Inventory.class));
        verify(stockMovementRepository, times(2)).save(any(StockMovement.class));
        verify(eventPublisher).publishEvent(any(StockTransferredEvent.class));
    }

    private Inventory createInventory(Long productId, Long warehouseId, Integer availableQuantity) {
        Inventory inventory = new Inventory();

        Product product = new Product();
        product.setId(productId);
        product.setSku("PROD-" + productId);
        inventory.setProduct(product);

        Warehouse warehouse = new Warehouse();
        warehouse.setId(warehouseId);
        warehouse.setCode("WH-" + warehouseId);
        inventory.setWarehouse(warehouse);

        inventory.setAvailableQuantity(availableQuantity);
        inventory.setReservedQuantity(0);
        inventory.setInTransitQuantity(0);

        return inventory;
    }
}
```

### 3. Integration Tests with Event Publishing
```java
@SpringBootTest
@ActiveProfiles("test")
@TestMethodOrder(OrderAnnotation.class)
class InventoryManagementIntegrationTest {

    @Autowired
    private InventoryService inventoryService;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private WarehouseRepository warehouseRepository;

    @Autowired
    private InventoryRepository inventoryRepository;

    @Autowired
    private StockMovementRepository stockMovementRepository;

    @MockBean
    private ApplicationEventPublisher eventPublisher;

    private Product testProduct;
    private Warehouse testWarehouse;

    @BeforeEach
    void setUp() {
        stockMovementRepository.deleteAll();
        inventoryRepository.deleteAll();
        productRepository.deleteAll();
        warehouseRepository.deleteAll();

        testProduct = createAndSaveProduct();
        testWarehouse = createAndSaveWarehouse();
    }

    @Test
    @Order(1)
    void shouldCompleteFullInventoryLifecycle() {
        // Step 1: Receive stock
        inventoryService.receiveStock(
            testProduct.getId(),
            testWarehouse.getId(),
            100,
            new BigDecimal("10.00"),
            "PO-001"
        );

        Optional<Inventory> inventory = inventoryRepository
            .findByProductIdAndWarehouseId(testProduct.getId(), testWarehouse.getId());

        assertThat(inventory).isPresent();
        assertThat(inventory.get().getAvailableQuantity()).isEqualTo(100);

        // Verify stock movement recorded
        List<StockMovement> movements = stockMovementRepository.findAll();
        assertThat(movements).hasSize(1);
        assertThat(movements.get(0).getMovementType()).isEqualTo(MovementType.PURCHASE_RECEIPT);
        assertThat(movements.get(0).getQuantity()).isEqualTo(100);

        // Step 2: Reserve stock
        inventoryService.reserveStock(
            testProduct.getId(),
            testWarehouse.getId(),
            30,
            "SO-001"
        );

        inventory = inventoryRepository
            .findByProductIdAndWarehouseId(testProduct.getId(), testWarehouse.getId());

        assertThat(inventory.get().getAvailableQuantity()).isEqualTo(70);
        assertThat(inventory.get().getReservedQuantity()).isEqualTo(30);

        // Step 3: Ship reserved stock
        inventoryService.shipStock(
            testProduct.getId(),
            testWarehouse.getId(),
            30,
            "SHIP-001"
        );

        inventory = inventoryRepository
            .findByProductIdAndWarehouseId(testProduct.getId(), testWarehouse.getId());

        assertThat(inventory.get().getAvailableQuantity()).isEqualTo(70);
        assertThat(inventory.get().getReservedQuantity()).isEqualTo(0);

        // Verify all movements recorded
        movements = stockMovementRepository.findAll();
        assertThat(movements).hasSize(3); // Receipt, reservation, shipment

        // Verify events were published
        verify(eventPublisher, times(3)).publishEvent(any());
    }

    @Test
    void shouldPreventNegativeStock() {
        // Given - No initial stock

        // When & Then
        assertThatThrownBy(() ->
            inventoryService.reserveStock(testProduct.getId(), testWarehouse.getId(), 10, "SO-001"))
            .isInstanceOf(InsufficientStockException.class);

        // Verify no inventory record created
        Optional<Inventory> inventory = inventoryRepository
            .findByProductIdAndWarehouseId(testProduct.getId(), testWarehouse.getId());
        assertThat(inventory).isEmpty();

        // Verify no stock movements recorded
        List<StockMovement> movements = stockMovementRepository.findAll();
        assertThat(movements).isEmpty();
    }

    private Product createAndSaveProduct() {
        Product product = new Product();
        product.setSku("TEST-PROD-001");
        product.setName("Test Product");
        product.setDescription("Test Description");
        product.setUnitCost(new BigDecimal("10.00"));
        product.setUnitPrice(new BigDecimal("15.00"));
        product.setMinimumStock(10);
        product.setReorderQuantity(50);
        return productRepository.save(product);
    }

    private Warehouse createAndSaveWarehouse() {
        Warehouse warehouse = new Warehouse();
        warehouse.setCode("TEST-WH-001");
        warehouse.setName("Test Warehouse");
        warehouse.setAddress("123 Test Street");
        return warehouseRepository.save(warehouse);
    }
}
```

## Learning Goals

- Complex business rule implementation
- Transactional consistency across multiple entities
- Event-driven architecture patterns
- Inventory management domain knowledge
- Concurrent access and data integrity
- Audit trail implementation
- Performance optimization for high-volume operations

## TDD Steps

1. **Red**: Write repository tests for inventory queries
2. **Green**: Implement custom repository methods
3. **Red**: Write service tests for stock operations
4. **Green**: Implement transactional service methods
5. **Red**: Write tests for business rule violations
6. **Green**: Add validation and error handling
7. **Red**: Write event publishing tests
8. **Green**: Implement event-driven features
9. **Red**: Write integration tests for full workflows
10. **Green**: Ensure transactional consistency
11. **Refactor**: Optimize performance and cleanup

## Advanced Topics

- Optimistic locking for concurrent stock updates
- Event sourcing for complete audit trails
- CQRS for read/write separation
- Saga pattern for distributed transactions
- Performance testing with high concurrency

---

## 问题描述

构建带产品、仓库、库存变动、供应商和自动补货的综合库存管理系统。专注于事务一致性、事件驱动架构和复杂业务规则。

## 学习目标

- 复杂业务规则实现
- 多实体间的事务一致性
- 事件驱动架构模式
- 库存管理领域知识
- 并发访问和数据完整性
- 审计跟踪实现
- 高容量操作的性能优化