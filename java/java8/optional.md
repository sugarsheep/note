- Optional.of(T)

  > 创建指定引用的Optional实例，若引用为null则**快速失败**
  >
  > 说明：of方法内部调用构造方法创建对象，构造方法中对参数进行了非空校验
  >
  > ```java
  > private Optional(T value) {
  >     this.value = Objects.requireNonNull(value);
  > }
  > ```

