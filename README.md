# Auto Dagger2

Auto Dagger2 is an annotation processor built on top of the Dagger2 annotation processor.  
It basically generates the component for you.

The goal is to reduce the boilerplate code required by Dagger2 when you have "empty" or simple components. It is usually the case in Android development.  

You can also mix manually written components with the ones generated by Auto Dagger2. Auto Dagger2 produces the human-readable code you would (hopefully) write yourself.


## Getting started

```java
@AutoComponent
@Singleton
public class ExampleApplication extends Application { 
}
```

It generates `ExampleApplicationComponent`

```java
@Component
@Singleton
public interface ExampleApplicationComponent { 
}
```

As you can see, the `@Singleton` annotation is applied to the generated component as well.


## API

### @AutoComponent

Annotate a class with `@AutoComponent` to generated an associated Component.
On the component, you can add dependencies, modules and superinterfaces.

```java
@AutoComponent(
    dependencies = ExampleApplication.class,
    modules = MainActivity.Module.class,
    superinterfaces = {ExampleApplication.class, GlobalComponent.class})
@Singleton
public class MainActivity extends Activity {
}
```

It generates `MainActivityComponent`

```java
@Component(
    dependencies = ExampleApplicationComponent.class,
    modules = MainActivity.Module.class
)
@Singleton
public interface MainActivityComponent extends ExampleApplicationComponent, GlobalComponent {
}
```


### @AutoInjector

`@AutoInjector` allows to add injector methods into a generated component.  

```java
@AutoInjector(MainActivity.class)
public class ObjectA {
}
```

It updates the `MainActivityComponent` by adding the following method:

```java
@Component(
    dependencies = ExampleApplicationComponent.class,
    modules = MainActivity.Module.class
)
@Singleton
public interface MainActivityComponent extends ExampleApplicationComponent, GlobalComponent {
  void inject(ObjectA objectA);
}
```

If you apply the `@AutoInjector` on the same class that has the `@AutoComponent` annotation, you can skip the value member:

```java
@AutoComponent(
    dependencies = ExampleApplication.class,
    modules = MainActivity.Module.class,
    superinterfaces = {ExampleApplication.class, GlobalComponent.class})
@AutoInjector
@Singleton
public class MainActivity extends Activity {
}
```

If your class have parameterized type, you can also specify it:

```java
@AutoInjector(value = MainActivity.class, parameterizedTypes = {String.class, String.class})
public class MyObject3<T, E> {
    private T t;
    private E e;
}
```


### @AutoExpose

`@AutoExpose` allows to expose a dependency within a generated component.  

```java
@AutoExpose(MainActivity.class)
@Singleton
public class SomeObject {

    @Inject
    public SomeObject() {
    }
}
```

It updates the `MainActivityComponent` by adding the following method:

```java
@Component(
    dependencies = ExampleApplicationComponent.class,
    modules = MainActivity.Module.class
)
@Singleton
public interface MainActivityComponent extends ExampleApplicationComponent, GlobalComponent {
  SomeObject someObject();
}
```

If you apply the `@AutoExpose` on the same class that has the `@AutoComponent` annotation, you can skip the value member:

```java
@AutoComponent(
    dependencies = ExampleApplication.class,
    modules = MainActivity.Module.class,
    superinterfaces = {ExampleApplication.class, GlobalComponent.class})
@AutoExpose
@Singleton
public class MainActivity extends Activity {
}
```

`@AutoExpose` can also expose dependency from a module's provider method:

```java
@dagger.Module
public class Module {
    @Provides
    @Singleton
    @AutoExpose(MainActivity.class)
    public SomeOtherObject providesSomeOtherObject() {
        return new SomeOtherObject();
    }
}
```

If your class have parameterized type, you can also specify it:

```java
@AutoExpose(value = MainActivity.class, parameterizedTypes = {String.class, String.class})
@Singleton
public class MyObject3<T, E> {
    private T t;
    private E e;

    @Inject
    public MyObject3() {
    }
}
```


## Reuse `@AutoComponent`

You can reuse `@AutoComponent` by creating an annotation that is itself annotated with `@AutoComponent`.

```java
@AutoComponent(
        dependencies = MyApp.class,
        superinterfaces = {HasDependenciesOne.class, HasDependenciesTwo.class},
        modules = StandardModule.class
)
public @interface StandardActivityComponent { }
```

You can then create an auto component that reuse directly that annotation.  
It will adds to the already defined dependencies, modules and superinterfaces.

```java
@AutoComponent(
        modules = SixthActivity.Module.class,
        includes = StandardActivityComponent.class)
@Singleton
public class SixthActivity extends Activity { }
```

You can also directly annotate the class:

```java
@StandardActivityComponent
@Singleton
public class SixthActivity extends Activity { }
```


## Scope

Whenever you use `@AutoComponent`, you also need to annotate the class with a dagger scope annotation (an annotation that is itself annotated with `@Scope`).
Auto Dagger2 will detect this annotation, and will apply it on the generated component.

If you don't provide scope annotation, the generated component will be unscoped.


## Installation

Beware that the groupId changed to **com.github.lukaspili.autodagger2**

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
		classpath 'com.android.tools.build:gradle:1.1.3'
		classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

dependencies {
    apt 'com.github.lukaspili.autodagger2:autodagger2-compiler:1.1'
    compile 'com.github.lukaspili.autodagger2:autodagger2:1.1'

    apt 'com.google.dagger:dagger-compiler:2.0.1'
    compile 'com.google.dagger:dagger:2.0.1'
    provided 'javax.annotation:javax.annotation-api:1.3.2' // Android only
}
```


## Status

Stable API.  

Auto Dagger2 was extracted from Auto Mortar to work as a standalone library.  
You can find more about Auto Mortar here:
[https://github.com/lukaspili/Auto-Mortar](https://github.com/lukaspili/Auto-Mortar)


## Author

- Lukasz Piliszczuk ([@lukaspili](https://twitter.com/lukaspili))


## License

Auto Dagger2 is released under the MIT license. See the LICENSE file for details.