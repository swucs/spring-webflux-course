# MongoDB Docker Setup

```yaml
version: '3.1'

services:

  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    volumes:
      - /root/docker_volumes/mongo:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 1234

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 38081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: 1234
      ME_CONFIG_MONGODB_URL: mongodb://root:1234@mongo:27017/
```

http://localhost:38081/

비밀번호는 console에 찍힌다. (admin/pass)


# Initial Data Setup

```java
@Service
public class DataSetupService implements CommandLineRunner {

    @Autowired
    private ProductService service;

    @Override
    public void run(String... args) throws Exception {
        ProductDto p1 = new ProductDto("4k-tv", 1000);
        ProductDto p2 = new ProductDto("slr-camera", 750);
        ProductDto p3 = new ProductDto("iphone", 800);
        ProductDto p4 = new ProductDto("headphone", 100);

        Flux.just(p1, p2, p3, p4)
                .concatWith(newProducts())
                .flatMap(p -> this.service.insertProduct(Mono.just(p)))
                .subscribe(System.out::println);

    }

    private Flux<ProductDto> newProducts(){
        return Flux.range(1, 1000)
                .delayElements(Duration.ofSeconds(2))
                .map(i -> new ProductDto("product-" + i, ThreadLocalRandom.current().nextInt(10, 100)));
    }

}
```