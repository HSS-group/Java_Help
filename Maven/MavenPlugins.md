### Maven plugins

Плагины - это способ расширить функциональность Maven
В простейшем случае запустить плагин просто, например:

~~~
mvn org.apache.maven.plugins:maven-checkstyle-plugin:check
mvn maven-checkstyle-plugin:check
mvn checkstyle:check
~~~

В данном примере вызывается плагин с groupId "org.apache.maven.plugins", artifactId "maven-checkstyle-plugin", последней
версией и целью (goal) "check".

Цель - это действие, которое плагин может выполнить. Целей может быть несколько.

#### Объявление плагина в pom.xml

Объявление плагина похоже на объявление зависимости. Так же, как и зависимости плагины идентифицируется с помощью GAV (
groupId, artifactId, version). Объявление плагина в pom.xml позволяет зафиксировать версию плагина, задать ему
необходимые параметры, привязать к фазам.

~~~
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>2.6</version>
</plugin>
~~~

После того как плагин объявлен, его можно настроить так, чтобы он автоматически запускался в нужный момент. Это делается
с помощью привязки плагина к фазе сборки проекта. В данном примере плагин запустится в фазе проекта package:

~~~
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
~~~

Плагин maven-archetype-plugin предназначен для того чтобы по существующему шаблону
создавать новые проекты. Создать проект с помощью maven-archetype-plugin достаточно просто - достаточно набрать:

~~~
mvn archetype:generate
~~~

Дальше выбрать шаблон из списка, ответить на дополнительные вопросы - и плагин сгенерирует проект.

maven-compiler-plugin - Компилятор - основной плагин который используется практически во всех проектах. Он доступен по
умолчанию, но текущая версия Java 5. В следующем примере в конфигурации используется версия java 1.9 (source - версия
языка, на котором написана программа. А
target - версия Java машины которая будет этот код запускать). И указано, что кодировка исходного кода программы UTF-8.

~~~
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>2.0.2</version>
    <configuration>
        <source>1.9</source>
        <target>1.9</target>
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>
~~~