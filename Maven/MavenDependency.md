### Все(почти) о зависимостях

Dependencies в Maven спасают нас от ситуаций:

* Допустим, ты работаешь в команде работая над проектом нам нужна
  библиотека (назовем ее библиотека "A" версии "1.1") все просто, подумаешь ты и решишь скачать например
  jar "A" какой-то библиотеки и добавишь его в classpath вручную. Далее говоришь своим
  коллегам, что для работы проекта нужна библиотека "A" версии "1.1". Пока звучит нормально,
  но обычно в проекте намного больше чем одна библиотека! При большом количестве библиотек
  возникнет ситуация, когда кто-то добавит библиотеку другой версии(Обратная совместимость? Не слышал!)
  проект может банально не скомпилироваться.
* Теперь представь что одна библиотека зависит от другой (-_-)

С maven ты можешь сконфигурировать зависимости своего проекта например указав зависимость

~~~
     <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot</artifactId>
            <version>3.0.4</version>
        </dependency>
~~~

Если зависимости нет в Maven local .m2 папке, тогда maven выгружает зависимость из
Maven Central. С этой зависимостью у нас подгружаются все необходимые jar файлы

![img.png](\images\img.png)

#### Транзитивные зависимости

Есть 2 вида зависимостей:

* Direct (Прямые) - зависимости которые мы определяем в блоке <dependencies/>
* Transitive - зависимости которые зависят от наших прямых зависимостей

В скриншоте выше как раз таки показаны транзитивные зависимости нашей зависимости "spring-boot"
В с помощью тега <exclusions> мы можем прописать какие из транзитивных зависимостей
мы не хотим видеть в classpath

#### В maven в блоке exclusions можно запрещать вообще любые транзитивные зависимости указав символ '*'

~~~
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot</artifactId>
            <version>3.0.4</version>
            <exclusions>
                <exclusion>
                    <groupId>*</groupId>
                    <artifactId>*</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
~~~

Допустим у нас есть такое древо зависимостей:

~~~
  A
  ├── B
  │   └── C
  │       └── D 2.0
  └── E
      └── D 1.0
~~~

Можно задаться вопросом,какая из зависимостей D будет выбрана. Адекватный человек предпочел бы
сказать что зависимость у которой версия более новая! Но для maven решили так, что
в таких ситуациях зависимость будет выбрана та,которая ближе к нашему проекту в
древе зависимостей.

Пример:

Допустим у нас есть зависимость та же зависимость:

~~~
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot</artifactId>
            <version>3.0.4</version>
        </dependency>
~~~

Для наглядности будем использовать inteliij maven dependency diagram вместо mvn dependency:tree
Вот наши транзитивные зависимости:

![img_1.png](\images\img_1.png)

Далее попробуем передавить транзитивую зависимость spring-aop,для этого явно укажем ее
в root pom.xml файле.В итоге блок с зависимостями выглядит так:

~~~
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot</artifactId>
            <version>3.0.4</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.3.15</version>
        </dependency>
    </dependencies>
~~~

Теперь взглянем на maven dependency diagram:

![img_2.png](\images\img_2.png)
Как мы видим произошел конфликт зависимостей.В результате зависимости которые
"подтянулись" будут spring-aop(5.3.15) и spring-beans(5.3.15) т.к путь от них,до
нашего root проекта короче чем от таких же зависимостей, но версии(6.0.6).

С этим все понятно,но как быть с транзитивной зависимостью spring-core.На диаграмме
они находятся на одной глубине.Но обратившись к документации на эту тему,все становится
понятно(Note that if two dependency versions are at the same depth in the dependency tree, the first declaration wins.)
В двух словах:Мы объявили spring-boot зависимость первую и поэтому разрешение зависимостей
будет в сторону spring-core(6.0.6).

#### Dependency management

Выше мы рассмотрели встроенный способ разрешения зависимостей. Еще одним способом разрешения зависимостей в Maven
является
Dependency management - Это блок в котором мы объявляем зависимости какой версии нам понадобятся,
если кто-то их запросит.

Еще одним преимуществом Dependency management являтется то,что при работе с многомодульным
приложением,мы можем один раз прописать dependency managment блок в parent(reactor) pom'e
и в будущем использовать зависимости без указания версий в дочерних модулях.

Может показаться, что работая с тем же spring приложением нам придется расписывать огромный
Dependency management блок вручную, подбирая совместимые между собой версии, но Pivotal
об этом позаботились и добавили готовый pom со всеми совместимыми версиями spring проектов
Например, заимпортировав:

~~~
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.spring.platform</groupId>
                <artifactId>platform-bom</artifactId>
                <version>Cairo-SR8</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
~~~

Данный блок dependencyManagement мы получим огромный список совместимых зависимостей.

### Dependency Scope

Области видимости зависимостей нужны для определения того,когда зависимость включается
в classpath.В maven есть 6 видов scope:

* compile - Является scope по умолчанию,если не указан явно и доступны во всех classpath-ах проекта,также
  распространяются на зависимые проекты
* provided - Необходимо использовать этот scope когда мы хотим предоставить зависимости в
  runtime.Т.е нам нужен JAR для компиляции,но в runtime у нас уже есть JAR предоставленный
  JDK или контейнером.
* runtime - Обозначает,что зависимость не нужна для компиляции,но предназначена для runtime
  хороший пример:

~~~
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
    <scope>runtime</scope>
</dependency>
~~~

* test - Зависимость нужна только для тестирования
* system - Очень похож на scope provided,за исключением того,что мы должны напрямую указать
  jar

~~~
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>custom-dependency</artifactId>
    <version>1.3.2</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/libs/custom-dependency-1.3.2.jar</systemPath>
</dependency>
~~~

* import - данный тип scope поддерживается только если тип зависимости указан как pom
  в секции dependencyManagement,данный импорт говорит нам о том,что данная зависимость должна
  быть заменена действующим списком зависимостей которые в ней объявлены
