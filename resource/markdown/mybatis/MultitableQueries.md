一对多 collection 标签

```xml
<resultMap>
    <result property="" column=""/>
    
    <collection property="" ofType="" column="">
        <id property="" column="" jdbcType=""/>
		<result property="" column="" jdbcType=""/>
    </collection>
</resultMap>
```





多对一 association 标签

```xml
<resultMap>
    <result property="" column=""/>
    
    <association property="" ofType="" column="">
        <id property="" column="" jdbcType=""/>
		<result property="" column="" jdbcType=""/>
    </association>
</resultMap>
```

