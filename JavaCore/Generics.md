### Что такое generics(обобщения)

Рассмотрим пример без обобщений:

~~~
public class Box {
    private Object object;

    public void set(Object object) { this.object = object; }
    public Object get() { return object; }
}
~~~

Метод set принимает Object, это значит что мы можем передавать все что захотим(кроме примитивов). У нас
нет возможности проверить какой тип будет получен во время компиляции.Т.е одна часть кода может ожидать что нам вернется
Integer, а
другая, что String, в конечном итоге это будет порождать ошибки в runtime.

Версия класса с generics:

~~~
public class Box<T> {
    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
~~~

В данном случае мы определили тип T. При создании экземпляра Box мы можем указать значения какого типа он
будет принимать и отдавать(в данном случае это Integer). Теперь при попытке передачи методу set параметр
типа например String у нас будет ошибка компиляции.

~~~
 Box<Integer> box = new Box<>();
~~~

Существует определенное соглашение для имен параметров. Пример с документации

* E - Element (used extensively by the Java Collections Framework)
* K - Key
* N - Number
* T - Type
* V - Value
* S,U,V etc. - 2nd, 3rd, 4th types

Не путать! Foo<T> - T - это параметр типа, Foo<String> - String - это аргумент типа

#### Стирание типов

На этапе компиляции происходит "Type Erasure" так называемое стирание типов,т.е на стадии компиляции
происходит замена типа на Object либо на empty

~~~
List<String> list = new ArrayList<>();
        list.add("abc");
        list.add("cdv");
~~~

На самом деле код будет выглядеть так:

~~~
List list = new ArrayList<>();
        list.add((String) "abc");
        list.add((String) "cdv");
~~~

Как мы видим компилятор использует приведение типов.Так же он может использовать bridge methods.
Еще один пример:

~~~
class Person<T> {
    public int compareTo(T o) {
        return 0;
    }
}
~~~

Теперь попробуем с помощью рефлексии получить методы класса Person:

~~~
 Stream.of(Person.class.getDeclaredMethods()).forEach(System.out::println);
 
 Вывод:
 public int org.example.Person.compareTo(java.lang.Object)
~~~

Т.е мы видим, что до runtime у нас доходят информация о типах не доходит.

#### Bridge methods

Давайте рассмотрим такой пример:

~~~
class Person implements Comparable<Person> {

    @Override
    public int compareTo(Person o) {
        return 0;
    }
}
~~~

Может показаться,что если мы так же воспользуемся reflection api и достанем все методы нам вернется 1 метод.
Но что мы видим:

~~~
public int org.example.Person.compareTo(org.example.Person)
public int org.example.Person.compareTo(java.lang.Object)
~~~

На первый взгляд может показаться непонятным откуда берется 2й метод. Можно воспользоваться методом reflection
isBridge() который возвращает true, если метод был сгенерирован компилятором.

~~~
  Method method1 = Person.class.getMethod("compareTo", Person.class);
  Method method2 = Person.class.getMethod("compareTo", Object.class);
  System.out.println(method1.isBridge());
  System.out.println(method2.isBridge());
  
  Вывод:
  false
  true
~~~

Почему так происходит?
У нас есть интерфейс:

~~~
public interface Comparable<T> {
    public int compareTo(T o);
}
~~~

После стирания типов мы получаем:

~~~
public interface Comparable {
    public int compareTo(Object o);
}
~~~

При реализации данного интерфейса в классе Person:

~~~
class Person implements Comparable<Person> {

    @Override
    public int compareTo(Person o) {
        return 0;
    }
}
~~~

Заметим, что тут мы поставили аннотацию @Override(оначает, что данный метод интерфейса мы переопределяем, а в данном
случе реализуем его). Но можно заметить, что после стирания типов у нас не будет метода compareTo(Person p).
И поэтому для поддержания отношения наследования компилятор и добавляет данный bridge method.

~~~
    public int compareTo(Object o) {
        return compareTo((Person) o);
    }
~~~

#### Нельзя параметризовать:

* Классы, имеющие в предках Throwable
* Анонимные классы
* Enums

#### Wildcard

Рассмотрим пример у нас есть 3 класса:

~~~
abstract class Animal {

    public abstract void say();
}

class Dog extends Animal {
    @Override
    public void say() {
        System.out.println("bark");
    }

}

class Cat extends Animal {
    @Override
    public void say() {
        System.out.println("Meou!");
    }
}
~~~

Допустим,мы хотим написать метод который принимает как параметр список животных и у каждого из них вызывает
метод say()

~~~
   private static void show(List<Animal> animals) {
        for (Animal animal : animals)
            animal.say();
    }
~~~

К сожалению классический полиморфизм тут не работает. Например:

~~~
 List<Animal> list = new ArrayList<>();
        list.add(new Dog());
        list.add(new Cat());
        show(list);
~~~

При попытке передать список Dog компилятор не даст нам этого сделать,потому что на вход одижает только List<Animal>
Теперь мы можем задействовать wildcard который выражается в виде знака вопроса.

~~~
   private static void show(List<?> animals) {
        for (Animal animal : animals)
            animal.say();
    }
~~~

Но теперь мы рассматриваем список animals, как список каких то объектов и в данном случае это не сработает,
так как в for each цикле мы явно указываем тип Animal. Решить данную проблему можно заменив ? на ? extends
animal. Теперь мы указываем что передавать в метод можно либо списки Animal, либо любыми другими наследниками
класса Animal. Грубо говоря мы прописали ограничение:

~~~
Animal
Dog
Cat
~~~

А простой wildcard был:

~~~
Object
Animal
Dog
Cat
~~~

И последний вариант с wildcard это ключевое слово super. Например, изменив наш метод на:
~~~
    private static void show(List<? super Animal> animals) {
        for (Object animal : animals)
            ((Animal) animal).say();
    }
~~~
Теперь заменив extends на super мы получим ограничение в виде:
~~~
Object
Animal
~~~

Т.е теперь мы можем принимать списки типа Animal и выше по иерархии, но выше в данном примере только Object
