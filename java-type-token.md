## 运行时 java 范型参数的传递

参考资料与文档摘录：

[spring framework ParameterizedTypeReference javadoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/ParameterizedTypeReference.html)

[上面的javadoc中提到的Neal Gafter 的博客 *Super Type Tokens*](https://gafter.blogspot.nl/2006/12/super-type-tokens.html)

java.lang.Class类的方法getGenericSuperclass的javadoc：
<blockquote>
<p>
Returns the Type representing the direct superclass of the entity (class, interface, primitive type or void) represented by this Class.
</p>
<p>
If the superclass is a parameterized type, the Type object returned must accurately reflect the actual type parameters used in the source code. The parameterized type representing the superclass is created if it had not been created before. See the declaration of ParameterizedType for the semantics of the creation process for parameterized types. If this Class represents either the Object class, an interface, a primitive type, or void, then null is returned. If this object represents an array class then the Class object representing the Object class is returned.
</p>
</blockquote>

java.lang.reflect.ParameterizedType#getActualTypeArguments javadoc:
<blockquote>
<p>
Returns an array of Type objects representing the actual type arguments to this type.
</p>
<p>
Note that in some cases, the returned array be empty. This can occur if this type represents a non-parameterized type nested within a parameterized type.
</p>
</blockquote>
