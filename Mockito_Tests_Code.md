# 🧪 Mockito Testing Guide – Tenant & Property Example

This document explains how to use Mockito and Spring Boot test features for unit and integration testing of a Tenant & Property management system.

---

## 🔍 Service Unit Tests – TenantAndPropertyServiceImplTest.java

Tests business logic in the service layer using mocks.

```java
@ExtendWith(MockitoExtension.class)
class TenantAndPropertyServiceImplTest {

    // Mocking dependencies
    @Mock private PropertyRepository propertyRepository;
    @Mock private TenantRepository tenantRepository;
    @Mock private PropertyMapper propertyMapper;
    @Mock private TenantMapper tenantMapper;

    // Injecting mocks into the service
    @InjectMocks private TenantAndPropertyServiceImpl testService;

    @Test
    void testAddProperty() {
        // Arrange
        Property property = TestData.testProperty();
        PropertyDTO dto = TestData.testPropertyDTO();
        when(propertyMapper.propertyDtoToProperty(dto)).thenReturn(property);
        when(propertyRepository.save(property)).thenReturn(property);
        when(propertyMapper.propertyToPropertyDto(property)).thenReturn(dto);

        // Act
        PropertyDTO result = testService.addProperty(dto);

        // Assert
        assertEquals(dto, result);
    }

    @Test
    void testAddTenant() {
        Long id = 1L;
        Property property = TestData.testProperty();
        Tenant tenant = TestData.testTenant();
        TenantDTO dto = TestData.testTenantDTO();

        when(propertyRepository.findById(id)).thenReturn(Optional.of(property));
        when(tenantMapper.tenantDtoToTenant(dto)).thenReturn(tenant);
        when(tenantRepository.save(tenant)).thenReturn(tenant);
        when(tenantMapper.tenantToTenantDto(tenant)).thenReturn(dto);

        TenantDTO result = testService.addTenant(id, dto);
        assertEquals(dto, result);
    }

    @Test
    void testDeleteTenantThrowsException() {
        when(tenantRepository.existsById(1L)).thenReturn(false);
        assertThrows(TenantOrPropertyNotFoundException.class, () -> testService.deleteTenant(1L));
    }

    @Test
    void testDeletePropertyThrowsException() {
        when(propertyRepository.existsById(1L)).thenReturn(false);
        assertThrows(TenantOrPropertyNotFoundException.class, () -> testService.deleteProperty(1L));
    }

    // Additional tests cover getAll, update, and delete scenarios
}
```

---

## 🌐 Controller Integration Tests – TenantAndPropertyControllerTest.java

Tests the full HTTP request-response cycle using `MockMvc`.

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class TenantAndPropertyControllerTest {

    @Autowired private MockMvc mockMvc;
    @Autowired private TenantAndPropertyService service;
    @Autowired private PropertyRepository propertyRepository;

    @Test
    void testAddProperty() throws Exception {
        PropertyDTO dto = TestData.testPropertyDTO();
        String json = new ObjectMapper().writeValueAsString(dto);

        propertyRepository.deleteAll();

        mockMvc.perform(post("/api/property")
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
            .andExpect(status().isOk())
            .andExpect(MockMvcResultMatchers.jsonPath("$.address").value(dto.getAddress()))
            .andExpect(MockMvcResultMatchers.jsonPath("$.rentAmount").value(dto.getRentAmount()));
    }

    @Test
    void testAddTenant() throws Exception {
        PropertyDTO property = TestData.testPropertyDTO();
        PropertyDTO savedProperty = service.addProperty(property);

        TenantDTO tenant = TestData.testTenantDTO();
        String json = new ObjectMapper().writeValueAsString(tenant);

        mockMvc.perform(post("/api/tenant/" + savedProperty.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(json))
            .andExpect(status().isOk())
            .andExpect(MockMvcResultMatchers.jsonPath("$.name").value(tenant.getName()));
    }

    @Test
    void testGetAllProperties() throws Exception {
        propertyRepository.deleteAll();
        PropertyDTO dto = TestData.testPropertyDTO();
        service.addProperty(dto);

        mockMvc.perform(MockMvcRequestBuilders.get("/api/property"))
            .andExpect(MockMvcResultMatchers.jsonPath("$.[0].address").value(dto.getAddress()))
            .andExpect(MockMvcResultMatchers.jsonPath("$.[0].rentAmount").value(dto.getRentAmount()));
    }

    @Test
    void testGetAllTenants() throws Exception {
        PropertyDTO property = TestData.testPropertyDTO();
        TenantDTO tenant = TestData.testTenantDTO();
        property.setTenants(List.of(tenant));

        propertyRepository.deleteAll();
        service.addProperty(property);

        mockMvc.perform(MockMvcRequestBuilders.get("/api/tenant"))
            .andExpect(MockMvcResultMatchers.jsonPath("$.[0].name").value(tenant.getName()));
    }

    @Test
    @Transactional
    void testDeleteTenant() throws Exception {
        propertyRepository.deleteAll();

        PropertyDTO property = TestData.testPropertyDTO();
        PropertyDTO saved = service.addProperty(property);

        TenantDTO tenant = TestData.testTenantDTO();
        Long tenantId = service.addTenant(saved.getId(), tenant).getId();

        mockMvc.perform(MockMvcRequestBuilders.delete("/api/tenant/" + tenantId))
            .andExpect(status().isOk());
    }
}
```

---

## ✅ Summary

- ✅ **Mockito** helps isolate logic for reliable unit testing.
- ✅ **MockMvc** enables full request simulation for integration tests.
- ✅ Best practice: test service/business logic and REST controller endpoints separately.

---


---

## 🧩 Related Code for Testing Context

These are the key classes being tested in the previous examples.

---

### 🧠 Service Implementation – TenantAndPropertyServiceImpl.java

Contains the core business logic for managing properties and tenants.

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class TenantAndPropertyServiceImpl implements TenantAndPropertyService {

    private final PropertyRepository propertyRepository;
    private final TenantRepository tenantRepository;
    private final PropertyMapper propertyMapper;
    private final TenantMapper tenantMapper;

    @Value("${error.propertyNotFound}")
    private String propertyNotFoundMessage;
    @Value("${error.tenantNotFound}")
    private String tenantNotFoundMessage;

    @Override
    public PropertyDTO addProperty(final PropertyDTO propertyDTO) {
        Property property = propertyRepository.save(propertyMapper.propertyDtoToProperty(propertyDTO));
        return propertyMapper.propertyToPropertyDto(property);
    }

    @Override
    public TenantDTO addTenant(final Long propertyId, TenantDTO tenantDTO) {
        Optional<Property> propertyOptional = propertyRepository.findById(propertyId);
        if (propertyOptional.isPresent()) {
            Property property = propertyOptional.get();
            Tenant tenant = tenantMapper.tenantDtoToTenant(tenantDTO);
            tenant.setProperty(property);
            tenant = tenantRepository.save(tenant);
            return tenantMapper.tenantToTenantDto(tenant);
        }
        throw new TenantOrPropertyNotFoundException(propertyNotFoundMessage);
    }

    @Override
    public List<PropertyDTO> getAllProperties() {
        return propertyRepository.findAll().stream()
                .map(propertyMapper::propertyToPropertyDto)
                .toList();
    }

    @Override
    public List<TenantDTO> getAllTenants() {
        return tenantRepository.findAll().stream()
                .map(tenantMapper::tenantToTenantDto)
                .toList();
    }

    @Override
    public void deleteTenant(final Long id) {
        if (!tenantRepository.existsById(id))
            throw new TenantOrPropertyNotFoundException(tenantNotFoundMessage);
        tenantRepository.deleteById(id);
    }

    @Override
    public void deleteProperty(final Long id) {
        if (!propertyRepository.existsById(id))
            throw new TenantOrPropertyNotFoundException(propertyNotFoundMessage);
        propertyRepository.deleteById(id);
    }

    @Override
    public PropertyDTO updateProperty(final PropertyDTO propertyDTO, final Long id) {
        Property property = propertyRepository.findById(id)
                .orElseThrow(() -> new TenantOrPropertyNotFoundException(propertyNotFoundMessage));

        if (propertyDTO.getAddress() != null) property.setAddress(propertyDTO.getAddress());
        if (propertyDTO.getRentAmount() != null) property.setRentAmount(propertyDTO.getRentAmount());

        return propertyMapper.propertyToPropertyDto(propertyRepository.save(property));
    }

    @Override
    public TenantDTO updateTenant(final TenantDTO tenantDTO, final Long id) {
        Tenant tenant = tenantRepository.findById(id)
                .orElseThrow(() -> new TenantOrPropertyNotFoundException(tenantNotFoundMessage));

        if (tenantDTO.getName() != null) tenant.setName(tenantDTO.getName());

        return tenantMapper.tenantToTenantDto(tenantRepository.save(tenant));
    }
}
```

---

### 🌐 REST Controller – TenantAndPropertyController.java

Handles API endpoints for frontend/backend communication.

```java
@RestController
@RequestMapping("/api/")
@RequiredArgsConstructor
public class TenantAndPropertyController {

    private final TenantAndPropertyService service;

    @PostMapping("/property")
    public ResponseEntity<PropertyDTO> addProperty(@Valid @RequestBody PropertyDTO propertyDTO) {
        return new ResponseEntity<>(service.addProperty(propertyDTO), HttpStatus.OK);
    }

    @PostMapping("/tenant/{propertyId}")
    public ResponseEntity<TenantDTO> addTenant(@PathVariable Long propertyId, @Valid @RequestBody TenantDTO tenantDTO) {
        return new ResponseEntity<>(service.addTenant(propertyId, tenantDTO), HttpStatus.OK);
    }

    @GetMapping("/property")
    public ResponseEntity<List<PropertyDTO>> getAllProperties() {
        return new ResponseEntity<>(service.getAllProperties(), HttpStatus.OK);
    }

    @GetMapping("/tenant")
    public ResponseEntity<List<TenantDTO>> getAllTenants() {
        return new ResponseEntity<>(service.getAllTenants(), HttpStatus.OK);
    }

    @DeleteMapping("/tenant/{id}")
    public ResponseEntity<?> deleteTenant(@PathVariable Long id) {
        service.deleteTenant(id);
        return new ResponseEntity<>("Deleted successfully id: " + id, HttpStatus.OK);
    }

    @DeleteMapping("/property/{id}")
    public ResponseEntity<?> deleteProperty(@PathVariable Long id) {
        service.deleteProperty(id);
        return new ResponseEntity<>("Deleted successfully id: " + id, HttpStatus.OK);
    }

    @PutMapping("/property/{id}")
    public ResponseEntity<?> updateProperty(@PathVariable Long id, @Valid @RequestBody PropertyDTO propertyDTO) {
        return new ResponseEntity<>(service.updateProperty(propertyDTO, id), HttpStatus.OK);
    }

    @PutMapping("/tenant/{id}")
    public ResponseEntity<?> updateTenant(@PathVariable Long id, @Valid @RequestBody TenantDTO tenantDTO) {
        return new ResponseEntity<>(service.updateTenant(tenantDTO, id), HttpStatus.OK);
    }
}
```