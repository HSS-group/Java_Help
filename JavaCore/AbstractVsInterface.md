### Фундаментальное отличие интерфейса от абстрактного класса

Фундаментальная разница заключается в том,что интерфейс определяет только поведение.Он не сообщает ничего про объект,
который будет его реализовывать.

Например, такое поведение как «движение» может быть применимо к разным типам объектов: машина, кот, котировки курса и т.
д.
Эти объекты не имеют ничего общего, кроме того, что они могут двигаться.

Абстрактный же класс описывает некий абстрактный объект (автомобиль, человека, кота и т. д.), а не только поведение.
Например:

~~~

abstract class Automobile {

    Engine engine;
    Wheels[] wheels;
    Gears[] gears;

    abstract void move();
    abstract void turn();
    abstract void accelerate();
    abstract void brake();
}
~~~

Можно сделать вывод Если нам нужно поведение — необходимо использовать интерфейс. Если речь про концептуальный объект —
мы должны использовать абстрактный класс.

### Технические отличия интерфейса от абстрактного класса

* При наследовании абстрактного класса используется ключевое слово "extends",a при реализации интерфейса "implements"
* Класс может одновременно и наследоваться от абстрактного класса (только одного) и реализовать один или множество
  интерфейсов.

#### Наличие конструктора

* Мы не можем создать экземпляр абстрактного класса,но мы можем определить в нем конструктор(В противном случае за нас
  это сделает компилятор, создав конструктор по умолчанию)
  Без него код просто не скомпилируется, поскольку при создании конкретного класса первым оператором будет неявный вызов
  super() конструктора суперкласса, в данном случае абстрактного.
* Для интерфейсов понятия «конструктор» не существует.

#### Типы переменных

* Все переменные в интерфейсах неявно являются public static final (т.е. константами). «final» подразумевает, что
  переменной обязательно должно быть присвоено значение во время инициализации.

~~~
public interface MyInterface {

    // эта строка не скомпилируется
    int value_1;   
                 
    int value_2 = 1;
    public final int value_3 = 1;
    static int value_4 = 1;
    public final static int value_5 = 1;
    static final int value_6 = 1;    
}
~~~

Поскольку value_1 не присвоено конкретное значение, а она является неявно final, код с такой строкой не скомпилируется.
Остальные строки не вызовут ошибок, т.к. public static final можно не указывать

* В абстрактных классах переменные могут быть любыми — абстрактность класса не накладывает ограничений.

#### Методы с реализацией

* Модификаторы доступа для абстрактных классов могут быть любыми. При этом, все методы, кроме абстрактных, должны иметь
  реализацию.

* В случае с интерфейсами модификаторы доступа могут быть только двух типов, public и private (последний — начиная с
  Java 9). Private может быть применим только к методам, имеющим реализацию, которые, в свою очередь, могут
  использоваться только
  методами по умолчанию, находящимися в интерфейсе. Класс, реализующий интерфейс, не будет иметь к ним доступ. Методы
  интерфейса без реализации являются неявно public, поэтому этот модификатор можно не писать.
* Для абстрактного класса все методы, кроме абстрактных, должны иметь реализацию
* Для интерфейсов, начиная с Java 8, вводится понятие метода по умолчанию. Такие методы, во-первых, должны иметь
  реализацию в интерфейсе, а во-вторых, помечены ключевым словом default. Они также являются неявно public. При этом они
  не должны в обязательном порядке иметь реализацию в реализующем интерфейс классе, ! но могут быть в нем
  переопределены !

#### Наследование

* Интерфейс не может реализовывать интерфейс, не может наследовать абстрактный класс, но может наследовать (используя
  ключевое слово extends) множество других интерфейсов.
* Абстрактный класс может наследовать как обычный класс, так и абстрактный. В обоих случаях это будет только один
  класс (в Java нет множественного наследования классов).
* Абстрактный класс также может реализовать до 65 535 интерфейсов (это связано с ограничением константы
  interfaces_count в структуре ClassFile).

#### Рекомендации к применению абстрактных классов и интерфейсов

В соответствии с рекомендациями Oracle абстрактный класс нужно использовать в следующих случаях:

* Необходимо выделить общий код между несколькими тесно связанными классами(Пояснение: это типовой рефакторинг, целью
  которого является устранение дублирования кода.)
* Мы ожидаем, что классы, расширяющие абстрактный класс, имеют много общих методов, полей или требуют модификаторов
  доступа, отличных от public (protected и private.Пояснение: ранее мы писали, что в интерфейсах методы, имеющие
  реализацию (помеченные ключевым словом default), являются неявно public. Если же метод, имеющий реализацию, помечен
  явно как private, то он не сможет быть использован в классах, реализующих этот интерфейс, а только в других методах
  интерфейса. Поэтому, если нам нужны методы с модификаторами доступа не public, мы должны использовать абстрактный
  класс.)
* Мы хотим объявить не static или не final поля для изменения состояния объекта(Пояснение: ранее мы писали, что все
  переменные в интерфейсах неявно являются public static final — из-за чего они не могут быть изменены.)

Для использования интерфейсов существуют следующие причины:

* Планируется, что несвязанные между собой классы будут реализовывать интерфейс. Например, интерфейсы Comparable и
  Cloneable реализуются многими несвязанными между собой классами.
* Мы хотим воспользоваться преимуществами множественного наследования типов