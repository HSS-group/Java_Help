### Maven build profiles
Профили сборки - это настройки которые могут быть использованы для перезаписи
стандартных значений сборки т.е можно настроить сборку для различных окружений
(Dev | Prod)

#### Типы профилей 
* Per Project - определяется в самом POM 
* Per user - Определяется в настройках Maven – xml файл (%USER_HOME%/.m2/settings.xml)
* Global - Определяется в глобальных настройках – xml файл (%M2_HOME%/conf/settings.xml)

#### Активация профилей
* Явно (Explicitly)
* Неявно (Implicitly)
* На основе операционной системы
* На основе system properties
* По наличию файлов

*Профили настраиваются в файле pom.xml с помощью элементов activeProfiles / profiles и 
запускаются различными методами. Профили изменяют файл pom.xml во время сборки и используются 
для передачи параметров различным целевым окружениям, например,
в директорию сервера базы данных в продакшн, разработку и тестирования.*

Явная активация профиля:
~~~
<settings>
  ...
  <activeProfiles>
    <activeProfile>profile-1</activeProfile>
  </activeProfiles>
  ...
</settings>
~~~

Неявная:
~~~
<profiles>
  <profile>
    <activation>
      <jdk>1.4</jdk>
    </activation>
    ...
  </profile>
</profiles>
~~~

Конфигурация выше активирует профиль если версия JDK начинается с 1.4


На основе операционной системы
~~~
<profiles>
  <profile>
    <activation>
      <os>
        <name>Windows XP</name>
        <family>Windows</family>
        <arch>x86</arch>
        <version>5.1.2600</version>
      </os>
    </activation>
    ...
  </profile>
</profiles>
~~~
Активируется в зависимости от обнаруженной операционной системы


На основе system properties
~~~
<profiles>
  <profile>
    <activation>
      <property>
        <name>debug</name>
      </property>
    </activation>
    ...
  </profile>
</profiles>
~~~

По наличию файлов
~~~
<profiles>
  <profile>
    <activation>
      <file>
        <missing>target/generated-sources/axistools/wsdl2java/org/apache/maven</missing>
      </file>
    </activation>
    ...
  </profile>
</profiles>
~~~

Профиль выше будет активирован если сгенерированный файл отсутствует

Ниже приведена команда для деактивации профилей:
~~~
mvn groupId:artifactId:goal -P -profile-1,-profile-2,-?profile-3
~~~