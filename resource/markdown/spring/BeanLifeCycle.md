<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">SpringBean的生命周期</h3>

#### 一、传统 Bean 的生命周期
1. new实例化；
2. 可使用了
3. 无引用时，GC回收♻️。

#### 二、Servlet 的生命周期
1. 实例化Servlet对象；
2. init初始化对象；
3. 相应客户端请求service()（doGet()与doPost()）；
4. destroy()终止/销毁。

#### 三、Spring Bean的生命周期
1. 实例化对象；
2. 填充属性值及引用；
3. 调用 `BeanNameAware` 的 `setBeanName(String name)` 设置 bean 的 id；
4. 调用 `BeanFactoryAware` 的 `setBeanFactory(BeanFactory beanFactory)` 设置 `BeanFactory` Bean工厂；
5. 同上：`ApplicationContextAware` `setApplicationContext(ApplicationContext applicationContext)`；
6. 如果实现 `BeanPostProcessor`，则 调用 postProcessBeforeInitialization() 初始化前的后置处理方法
7. 如果实现了 `InitializingBean` 接口，则使用 `afterPropertiesSet()` 来初始化属性
8. 如果实现 `BeanPostProcessor`，则 调用 postProcessAfterInitialization() 初始化后的后置处理方法
9. 此时，bean 就可以使用了
10. `DisposableBean`接口 `destroy()` 销毁bean。不过在Spring5.0开始，DisposableBean.destroy() 已经是过时的方法了，可直接使用 close()。