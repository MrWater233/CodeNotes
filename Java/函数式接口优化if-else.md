> 使用策略模式也能优化if-else
>
> 但是策略模式会导致策略实现类较多，不能从宏观的角度去了解各个逻辑
>
> 因此可以使用`map+函数式接口+lambda表达式`的方式去优化

# 基本写法

```java
@Service
public class RoomServiceImpl implements RoomService {
    /**
     * 函数式接口
     * Consumer<T> 消费型接口  void accept(T t);
     * Supplier<T> 供给型接口  T get();
     * Function<T,R> 函数型接口  R apply(T t);
     * Predicate<T,R> 断言型接口  boolean test(T t);
     */
    private Map<RoomType, Supplier<Double>> roomTypeDispatcher = new HashMap<>();

    /**
     * 在构造函数之后调用，对map进行初始化
     */
    @PostConstruct
    public void roomTypeDispatcherInit() {
        roomTypeDispatcher.put(RoomType.AROOM,() -> Double.valueOf(1200));
        roomTypeDispatcher.put(RoomType.BROOM,() -> Double.valueOf(1000));
        roomTypeDispatcher.put(RoomType.CROOM,() -> Double.valueOf(800));
    }

    @Override
    public double getRoomPrice(Room room) {
        Supplier<Double> result = roomTypeDispatcher.get(room.getType());
        return result.get();
    }
}
```

将逻辑都保存进一个map ，通过不同的key去获取

# 逻辑代码分离写法

当各个**策略的逻辑**过多时也可以将逻辑提取出一个公共`service`

```java
@Service
public class RoomUnitServiceImpl implements RoomUnitService {
    @Override
    public Double roomAPrice() {
        return Double.valueOf(1200);
    }

    @Override
    public Double roomBPrice() {
        return Double.valueOf(1000);
    }

    @Override
    public Double roomCPrice() {
        return Double.valueOf(800);
    }
}
```

```java
@Service
public class RoomServiceImpl implements RoomService {
    @Autowired
    RoomUnitService roomUnitService;
    
    /**
     * 函数式接口
     * Consumer<T> 消费型接口  void accept(T t);
     * Supplier<T> 供给型接口  T get();
     * Function<T,R> 函数型接口  R apply(T t);
     * Predicate<T,R> 断言型接口  boolean test(T t);
     */
    private Map<RoomType, Supplier<Double>> roomTypeDispatcher = new HashMap<>();

    /**
     * 在构造函数之后调用，对map进行初始化
     */
    @PostConstruct
    public void roomTypeDispatcherInit() {
        roomTypeDispatcher.put(RoomType.AROOM,() -> roomUnitService.roomAPrice());
        roomTypeDispatcher.put(RoomType.BROOM,() -> roomUnitService.roomBPrice());
        roomTypeDispatcher.put(RoomType.CROOM,() -> roomUnitService.roomCPrice());

    }

    @Override
    public double getRoomPrice(Room room) {
        Supplier<Double> result = roomTypeDispatcher.get(room.getType());
        return result.get();
    }
}
```

