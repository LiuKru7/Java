# 🐶 Spring CacheManager Example: DogService

This file demonstrates how to implement caching using Spring's `CacheManager` in a simple `DogService`.

---

## 🧾 DogService with CacheManager

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class DogService {

    private final DogRepo dogRepo;
    private final CacheManager cacheManager;

    public Dog getDogById(final Long dogId) {
        // Get the cache named "dogs"
        Cache dogCache = cacheManager.getCache("dogs");

        if (dogCache != null) {
            // Try retrieving dog from cache
            Dog dogFromCache = dogCache.get(dogId, Dog.class);
            if (dogFromCache != null) {
                log.info("Retrieved dog from cache: {}", dogFromCache);
                return dogFromCache;
            }

            log.info("Retrieving dog from H2 database...");

            // If not in cache, retrieve from DB
            Dog dogFromDb = dogRepo.findById(dogId).orElseThrow(() ->
                    new RuntimeException("Dog not found with ID: " + dogId));

            // Store in cache for next time
            dogCache.put(dogId, dogFromDb);
            log.info("Putting dog into cache: {}", dogFromDb);

            return dogFromDb;
        }

        return null; // Fallback if cache is not available
    }

    public void clearCache(String cacheNameToClear) {
        // Manually clear a specific cache
        Cache cacheToClear = cacheManager.getCache(cacheNameToClear);
        if (cacheToClear != null) {
            cacheToClear.clear();
            log.warn("Cache {} was cleared!", cacheNameToClear);
        }
    }
}
```

---

## ⚙️ Cache Configuration

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        // Create an in-memory cache manager with "dogs" cache
        return new ConcurrentMapCacheManager("dogs");
    }
}
```
