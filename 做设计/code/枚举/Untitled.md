

```java
public enum ReplayStatus {
    /**
     * 执行中
     */
    PROCESSING(0, "执行中"),
    FINISH(1, "已完成"),
    FAILED(2, "已失败");

    private Integer status;

    private String desc;

    ReplayStatus(Integer status, String desc) {
        this.status = status;
        this.desc = desc;
    }
    private static final Map<Integer, ReplayStatus> CACHED = new HashMap<>(4);

    static {
        for (ReplayStatus status : values()) {
            CACHED.put(status.getStatus(), status);
        }
    }

    public static ReplayStatus of(Integer status) {
        return CACHED.get(status);
    }
  
      public static boolean check(int code) {
        return Objects.nonNull(Arrays.stream(ShipmentTypeEnum.values())
                .filter(item -> Objects.equals(item.getCode(), code))
                .findFirst()
                .orElse(null));
    }
}

```

