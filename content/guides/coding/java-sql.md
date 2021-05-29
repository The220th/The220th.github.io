---
title: "Подключение SQL в Java без IDE"
date: 2021-04-14T21:49:58+03:00
draft: false
---
{{< toc >}}

## Шаг первый: Установка
Гайд предназначен, чтобы именно подключить MySQL, hibernate, и, возможно, jsch с нуля. Но это не руководство, как пользоваться этим вот всем. Постараюсь объяснять всё по ходу текста, но, встречая незнакомые слова, пробуйте их "загуглить". Возможно на изучение этого гайда уйдёт не один час.
Как СУБД будем использовать MySQL. Установка будет производиться на xubuntu 20.04 TLS, но для Windows тоже подойдёт. Надо просто сразу переходить к шагу 2, но узнать, как устанавливать MySQL на Windows, придётся самому.
``` bash
> sudo apt update
> sudo apt install mysql-server mysql-client
> mysql -V
```
В моём случае вывод такой: `mysql  Ver 14.14 Distrib 5.7.33, for Linux (x86_64) using  EditLine wrapper`. Т. е. MySQL версии 14.14
``` bash
> sudo systemctl status mysql
> sudo mysql_secure_installation
```

На первом шаге настраивается плагин валидации пароля. Чтобы его включить нажмите Y, или его можно не включать. Затем надо задать сложность пароля, который позволит установить этот плагин. Здесь 0 означает слабый пароль, а 2 - сложный. Когда плагин будет настроен введите пароль root (root для СУБД, а не системы) и подтвердите, что хотите использовать именно его.
Введите Y для отключения анонимного доступа к MySQL, затем ещё раз Y, чтобы запретить подключаться к базе от имени root удаленно.
Снова Y, чтобы удалить тестовую базу данных. Затем, обновите привилегии для пользователей.
После завершения настройки вы можете подключиться пользователем root к серверу баз данных из командной строки.
По умолчанию ваши созданные БД находятся в `/var/lib/mysql/`
``` bash
> sudo mysql -u root
```
Теперь создадим пользователя, тестовые БД и таблицу.
``` sql
-- Создание БД
CREATE DATABASE testDB;

use testDB;

--Создание таблицы books
CREATE TABLE `books` (
  `id` int(11) NOT NULL,
  `name` varchar(50) NOT NULL,
  `author` varchar(50) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

--Заполним таблицу какими-нибудь данными
INSERT INTO testDB.books (id, `name`, author)
    VALUES (1, 'Core Java, Volume I', 'Key Horstmann');
INSERT INTO testDB.books (id, `name`, author)
    VALUES (2, 'The Code Book', 'Simon Singh');

-- Создадим пользователя. Запомните его имя и пароль, нам они ещё пригодятся
CREATE USER 'testUser'@'ip' IDENTIFIED BY 'superUser.password2281337';
-- удаление пользователя: DROP USER 'testUser'@'ip';
-- вместо ip нужно вписать ваш белый ip или localhost. Если не указать @'ip', то будет @'%', это означает, что всё разрешено

-- Дадим созданному пользователю доступ к созданной БД
GRANT ALL PRIVILEGES ON testDB.* TO 'testUser'@'ip';

FLUSH PRIVILEGES;

-- Посмотрим, что всё получилось
SELECT user,host FROM mysql.user;

SHOW GRANTS FOR 'testUser'@'ip';

exit;
```
Зайдём от имени созданного пользователя.
``` bash
> mysql -u testUser -p [-h 'ip']
```
`[введите пароль]`
``` sql
-- Посмотрим, что имеем доступ к нашей таблице с книгами.
SELECT id, name, author FROM testDB.books;

-- Убедимся, что используется порт 3306
show variables like 'port';
-- по умолчанию должен быть именно порт 3306

quit;
```
## Шаг полтора: Если на удалённой машине

Пару слов: Соединение с базой данных MySQL по умолчанию БЕЗ ШИФРОВАНИЯ. Возможно 4 варианта (скорее всего больше) решения этой проблемы: 
* Шифровать всё в самой программе Java
* Подключить и настроить MySQL SSL
* Использовать VPN
* И, наверное, самый разумный: подключаться к БД через SSH. Но тогда переходите сразу к [соответствующему шагу](#бонус-подключение-через-ssh)

По умолчанию подключиться к СУБД можно только из `localhost`. Исправляем:

1) В конфигурации `/etc/mysql/mysql.conf.d/mysqld.cnf` находим `"bind-address = 127.0.0.1"`. Меняем его на `"bind-address = 0.0.0.0"`.

2) Перезапускаем сервис:
``` bash
> sudo systemctl restart mysql
```

3) Убеждаемся, что всё работает:
``` bash
sudo ss -tulpn | grep mysql
```
## Шаг второй: Тестовая Java программа
JDBC - это стандарт взаимодействия Java с СУБД, API. Можно сказать, что это интерфейс, который потом должны реализовывать.  
mysql-connector - это "драйвер" для коннекта к СУБД. Или штука, которая реализовывает интерфейс JDBC. Это нужно, чтобы подключаться к СУБД MySQL.  
Это на пальцах и не формально, всё сложнее)  
Не путайте с JPA. Подробнее про это лучше почитать тут: https://habr.com/ru/post/265061/, или поищите в интернете: "orm, jpa и jdbc"  

1) Находим где-нибудь mysql-connector-java-x.x.xx.jar. Например так:

Перейдите на сайт: https://downloads.mysql.com/archives/c-j/

Выбираем `Platform Independent` и скачиваем zip или tar archive. Там и будет `mysql-connector-java-x.x.xx.jar` (вместо x.x.xx должны быть циферки).
В моём случае - это `mysql-connector-java-8.0.23.jar`

2) Закидываем JAR-файл в директорию, содержащуюся в `CLASSPATH`. Например, в корень проекта. Но лучше создать папку `./lib/` и закинуть туда, потом укажем Java, где его искать (в пункте 4).

3) Создадим файл `JavaToMySQL.java`. Пробуем написать программу:

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JavaToMySQL {

    // JDBC URL, username and password of MySQL server
    // Обратите внимание на [ip]. Укажите ваш белый ip или localhost
    private static final String url = "jdbc:mysql://[ip]:3306/testDB";
        // Помните, мы делали пользователя. Здесь его login и password
    private static final String user = "testUser";
    private static final String password = "superUser.password2281337";

    // JDBC variables for opening and managing connection
    private static Connection con;
    private static Statement stmt;
    private static ResultSet rs;

    public static void main(String args[]) {
        // Сам запрос к СУБД
        String query = "select count(*) from books";

        try {
            // opening database connection to MySQL server
            con = DriverManager.getConnection(url, user, password);

            // getting Statement object to execute query
            stmt = con.createStatement();

            // executing SELECT query
            rs = stmt.executeQuery(query);

            while (rs.next()) {
                int count = rs.getInt(1);
                System.out.println("Total number of books in the table : " + count);
            }

        } catch (SQLException sqlEx) {
            sqlEx.printStackTrace();
        } finally {
        //Знаю, лучше использовать оператор try с ресурсами, чтобы не делать всего этого. Здесь так шакально, чтобы показать, что нужно это вот всё вообще-то закрывать.
            //close connection ,stmt and resultset here
            try { con.close(); } catch(SQLException se) { /*can't do anything */ }
            try { stmt.close(); } catch(SQLException se) { /*can't do anything */ }
            try { rs.close(); } catch(SQLException se) { /*can't do anything */ }
        }
    }
}
```

4) Самое интересное. Компиляция с драйвером и запуск. Начальные условия:  

Мы находимся в папке `.`  
`JavaToMySQL.java` находится в папке `.`  
`mysql-connector-java-x.x.xx.jar` находится в `./lib/`  
Тогда компиляция: 
``` bash
> javac -encoding utf-8 -cp .:./lib/mysql-connector-java-x.x.xx.jar JavaToMySQL.java
```
Вместо `x.x.xx` должны быть циферки. Флаг `-encoding utf-8` означает, что раскодировать исходники компилятору нужно с помощью `utf-8`, `-cp ...` показывает компилятору, где искать классы и jar-ники. И теперь запуск:
``` bash
> java -cp .:./lib/mysql-connector-java-x.x.xx.jar JavaToMySQL 
```
Должно вывести: `"Total number of books in the table : 2"`

Если нет, то проверьте, что вы правильно ввели все данные для подключения к СУБД, что вы правильно указали все jar-ники, что вы используйте одинаковые версии java и javac. Может быть файл `mysql-connector-java-x.x.xx.jar` загрузился битым или неполностью, проверьте это. Если это всё не помогло, то подождите, пока не пройдёт полнолуние, и попробуйте снова.

Следующим шагом подключимся к СУБД с помощью SSH. Если вам это не нужно, то сразу переходите [к следующему шагу](#шаг-третий-hibernate-pizza-time)

## Бонус: Подключение через SSH

Как было уже сказано, убедитесь, что в `/etc/mysql/mysql.conf.d/mysqld.cnf` написано именно `"bind-address = 127.0.0.1"`, а не `"bind-address = 0.0.0.0"`.

### Вариант 1: Всё ручками
Итак ситуация следующая. Имеем сервер с белым IP `serverIP`, где настроен SQL сервер на порте 3306.
Если у вас Windows 7 или ниже, то используйте puttySSH или переходите сразу к [следующему пункту](#вариант-2-мы-же-используем-java-так-ведь)

Вводим команду, чтобы открыть SSH туннель:  
``` bash
> ssh -fN -L 1234:localhost:3306 username@serverIP
```
или если вы используете ключи (`ssh-keygen`)
``` bash
> ssh -L 1234:localhost:3306 username@serverIP &
```
Теперь нужно в программе подключаться к порту 1234, а не 3306.

Затем всё что необходимо поменять в `JavaToMySQL.java` - это строку в коде:  
``` java
private static final String url = "jdbc:mysql://[ip]:3306/testDB";
```
на  
``` java
private static final String url = "jdbc:mysql://localhost:1234/testDB";
```

Если хотим "уничтожить" SSG туннель:
``` bash
> ps aux | grep 1234
```
Найдите там PID нашего процесса, что-то похожее на это:
``` bash
the220th   13427  0.0  0.0  14612   732 ?        Ss   [время]   0:00 ssh -fN -L 1234:localhost:3306 username@serverIP
```
В данной ситуации нужное нам число — это `13427`

Уничтожаем:
``` bash
> kill 13427    #(если не поможет, то: sudo kill -9 13427)
```

### Вариант 2: Мы же используем java, так ведь?

Чтобы сделать то же самое с Java, вы можете использовать [JSch](http://www.jcraft.com/jsch/). В гайде будет `jsch-0.1.55.jar`. Сейчас ситуация та же, что и в 4-ом пункте [на шаге два](#шаг-второй-тестовая-java-программа), но в `./lib/` будет ещё лежать `jsch-0.1.55.jar`.

Модифицируем программу. Создадим UpdateMySqlDatabase.java  
Что изменилось:  
Добавилась функция:
`makeTunnelSSH()` 
и поменялся порт в строке:
`private static final String url = "jdbc:mysql://[ip]:1234/testDB";` (было 3306)

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;

/**
 * Simple Java program to connect to MySQL database running on localhost and
 * running SELECT and INSERT query to retrieve and add data.
 * @author Javin Paul
 */
public class UpdateMySqlDatabase {

    // JDBC URL, username and password of MySQL server
    private static final String url = "jdbc:mysql://[ip]:1234/testDB";
    private static final String user = "testUser";
    private static final String password = "superUser.password2281337";

    // JDBC variables for opening and managing connection
    private static Connection con;
    private static Statement stmt;
    private static ResultSet rs;

	private static Session session;
    private static int lport;
    private static String rhost;
    private static int rport;

    public static void main(String args[]) {
    	makeTunnelSSH();

        String query = "select count(*) from books";

        try {
            // opening database connection to MySQL server
            con = DriverManager.getConnection(url, user, password);

            // getting Statement object to execute query
            stmt = con.createStatement();

            // executing SELECT query
            rs = stmt.executeQuery(query);

            while (rs.next()) {
                int count = rs.getInt(1);
                System.out.println("Total number of books in the table : " + count);
            }

        } catch (SQLException sqlEx) {
            sqlEx.printStackTrace();
        } finally {
            //close connection ,stmt and resultset here
            try { con.close(); } catch(SQLException se) { /*can't do anything */ }
            try { stmt.close(); } catch(SQLException se) { /*can't do anything */ }
            try { rs.close(); } catch(SQLException se) { /*can't do anything */ }
            session.disconnect();
        }
    }

    public static void makeTunnelSSH(){
        String user = "Имя пользователя, который user из >ssh user@host";
        String password = "Пароль этого пользователя";
        String host = "host из >ssh user@host. Белый ip крч";
        int port=22; //порт для ssh соединения с host
        try
            {
            JSch jsch = new JSch();
            session = jsch.getSession(user, host, port);
            lport = 1234;
            rhost = "localhost";
            rport = 3306;
            session.setPassword(password);
            session.setConfig("StrictHostKeyChecking", "no");
            System.out.println("Establishing Connection...");
            session.connect();
            int assinged_port=session.setPortForwardingL(lport, rhost, rport);
            System.out.println("localhost:"+assinged_port+" -> "+rhost+":"+rport);
            }
        catch(Exception e){System.err.print(e);}
    }
}
```

Компилируем её:
``` bash
javac -encoding utf-8 -cp .:./lib/mysql-connector-java-x.x.xx.jar:./lib/jsch-0.1.55.jar UpdateMySqlDatabase.java
```
Запускаем:
``` bash
java -cp .:./lib/mysql-connector-java-x.x.xx.jar:./lib/jsch-0.1.55.jar UpdateMySqlDatabase
```
Если всё правильно сделать, то вывод должен быть тот же:  
`Total number of books in the table : 2`

Забавно, что пока запущена эта программа, то туннель создан и существовать, пока работает программа. Если закомментировать строку
`session.disconnect();`  
то программа не завершит свою работу, а значит туннель будет по-прежнему открыт
 

## Шаг третий: Hibernate pizza time
Что за hibernate? Есть СУБД и java. Надо это как-то соединить. JDBC - это как раз и делает:  
`java <-> JDBC <-> СУБД`  
Но многим кажется, что это сложно. Придумали hibernate:  
`java <-> hibernate <-> JDBC <-> СУБД`  
А на самом деле вот так:  
`java <-> hibernate <-> JDBC и ещё много всего <-> СУБД`  

Т. е. hibernate использует ещё и то, что реализует JDBC (т. е. mysql-connector), для подключения к БД. Он даёт возможность использовать СУБД "без знания" SQL, всю эту работу (составление запросов, подключение к БД, проведение транзакций, удаление и создание записей в БД) hiberbate берёт на себя, но за счёт снижения (достаточно сильно) скорости работы программы. Нам же остаётся просто работать с объектами как раньше. Hibernate должен упрощать работу с БД, но это не значит, что можно не знать SQL язык. Также это не значит, что можно не уметь работать с JDBC.

Но между hibernate и JDBC большая разница.

`ORM` - технология (шаблон) проектирования, которая(ый) связывает `БД` с `ООП`

`JPA` - это стандарт для `ORM`

`JDBC` - это стандарт для доступа к `БД`

`mysql-connector` - реализует `JDBC`

`hibernate` - реализует `JPA`

Надеюсь, теперь стало лучше с пониманием, что за hibernate. Теперь к делу.

Скачаем всё необходимое. Заходим на [hibernate.org/orm/](https://hibernate.org/orm/) и скачиваем (жмякаем на большую синюю кнопку Download Zip archive). В моём случае `hibernate-release-5.4.28.Final.zip`. Весит, конечно, он изрядно, но благо нам оттуда нужны лишь файлы из `/hibernate-release-x.x.xx.Final/lib/required/`  

У меня были такие:
* antlr-2.7.7.jar
* FastInfoset-1.2.15.jar
* istack-commons-runtime-3.0.7.jar
* javax.activation-api-1.2.0.jar
* jaxb-api-2.3.1.jar
* jaxb-runtime-2.3.1.jar
* stax-ex-1.8.jar
* txw2-2.3.1.jar
* byte-buddy-1.10.17.jar
* classmate-1.5.1.jar
* dom4j-2.1.3.jar
* hibernate-commons-annotations-5.1.2.Final.jar
* jandex-2.2.3.Final.jar
* javassist-3.27.0-GA.jar
* javax.persistence-api-2.2.jar
* jboss-logging-3.4.1.Final.jar
* jboss-transaction-api_1.2_spec-1.1.1.Final.jar
* hibernate-core-5.4.28.Final.jar

Много тут всякого... Почти 15 МБ! Но это ведь целый framework, чего вы хотели?  
Ещё на момент создания гайда использую `mysql-connector-java-8.0.23.jar`  
Продолжаем:  
Задача запустить какой-нибудь код с использованием hibernate. Без IDE... Попробуем.  
Закинем ВСЕ эти jar-файлики в `./lib/`  
Первым делом создадим тестовую БД `studtable`, таблицу `student` и пользователя `suser`:  
``` sql
CREATE DATABASE studtable;

use studtable;

CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(45) DEFAULT NULL,
  `last_name` varchar(45) DEFAULT NULL,
  `email` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;

CREATE USER 'suser' IDENTIFIED BY 'suser123?Password';

GRANT ALL PRIVILEGES ON studtable.* TO 'suser';

FLUSH PRIVILEGES;

exit;
```
Теперь создадим  в директории `./` файлик `hibernate.cfg.xml` с содержимым:

``` xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <!-- JDBC Database connection settings -->
        <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://[ip]:3306/studtable</property>
        <property name="connection.username">suser</property>
        <property name="connection.password">suser123?Password</property>

        <!-- JDBC connection pool settings ... using built-in test pool -->
        <property name="connection.pool_size">1</property>

        <!-- Select our SQL dialect -->
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- Echo the SQL to stdout -->
        <property name="show_sql">true</property>

		<!-- Set the current session context -->
		<property name="current_session_context_class">thread</property>
 
    </session-factory>

</hibernate-configuration>
```
Чтобы hibernate указать, что объект должен соответствовать записи в БД, то:
- Создаём специальный xml файл. Но это устарело и не будем рассматривать
- Или аннорируем код (то, что начинается с символа `@`)
Рассмотрим аннотацию на примере Student.java:
``` java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

//Entity=Объект соответствует записи в таблице
//                                      Table=название этой таблице
@Entity
@Table(name="student")
public class Student {

    //Id=что это "primary key" объекта
    //GeneratedValue=способ генерации этого ключа
    //strategy=GenerationType.IDENTITY значит также как и в БД. Во время создания таблицы мы указали AUTO_INCREMENT, значит так и будет
    //Column(name="id") значит, что это поле соответствует колонке (полю) id
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="id")
	private int id;

	//Column(name="first_name") значит, что это поле соответствует колонке (полю) first_name
	@Column(name="first_name")
	private String firstName;

    //Column(name="last_name") значит, что это поле соответствует колонке (полю) last_name
	@Column(name="last_name")
	private String lastName;
	
    //Column(name="email") значит, что это поле соответствует колонке (полю) email
	@Column(name="email")
	private String email;
	
	public Student() {}
	public Student(String firstName, String lastName, String email) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
	}
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getFirstName() {
		return firstName;
	}
	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	@Override
	public String toString() {
		return "Student [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email + "]";
	}
}
```

И сам класс `CreateStudent.java` с подключением:
``` java
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class CreateStudent {

	public static void main(String[] args) {

		// create session factory
		SessionFactory factory = new Configuration()
					.configure("hibernate.cfg.xml")
					.addAnnotatedClass(Student.class)
					.buildSessionFactory();
		
		// create session
		Session session = factory.getCurrentSession();
		
		try {			
			// create a student object
			System.out.println("Creating new student object...");

            // Когда создавали factory, то указали addAnnotatedClass(Student.class). Обратите на это внимание, когда дойдёте до session.save(tempStudent) 
			Student tempStudent = new Student("Paul", "Wall", "blblbl@horse.hole");
			
			// start a transaction
			session.beginTransaction();
			
			// save the student object
			System.out.println("Saving the student...");
			session.save(tempStudent);
			
			// commit transaction
			session.getTransaction().commit();
			
			System.out.println("Done!");
		}
		finally {
			factory.close();
		}
	}

}
```

Попробуем скомпилировать:
``` bash
> javac -encoding utf-8 -cp .:./lib/mysql-connector-java-8.0.23.jar:./lib/antlr-2.7.7.jar:./lib/FastInfoset-1.2.15.jar:./lib/istack-commons-runtime-3.0.7.jar:./lib/javax.activation-api-1.2.0.jar:./lib/jaxb-api-2.3.1.jar:./lib/jaxb-runtime-2.3.1.jar:./lib/stax-ex-1.8.jar:./lib/txw2-2.3.1.jar:./lib/byte-buddy-1.10.17.jar:./lib/classmate-1.5.1.jar:./lib/dom4j-2.1.3.jar:./lib/hibernate-commons-annotations-5.1.2.Final.jar:./lib/jandex-2.2.3.Final.jar:./lib/javassist-3.27.0-GA.jar:./lib/javax.persistence-api-2.2.jar:./lib/jboss-logging-3.4.1.Final.jar:./lib/jboss-transaction-api_1.2_spec-1.1.1.Final.jar:./lib/hibernate-core-5.4.28.Final.jar CreateStudent.java
```
Фуф, сколько символов. Теперь пробуем запустить:
``` bash
> java -cp .:./lib/mysql-connector-java-8.0.23.jar:./lib/antlr-2.7.7.jar:./lib/FastInfoset-1.2.15.jar:./lib/istack-commons-runtime-3.0.7.jar:./lib/javax.activation-api-1.2.0.jar:./lib/jaxb-api-2.3.1.jar:./lib/jaxb-runtime-2.3.1.jar:./lib/stax-ex-1.8.jar:./lib/txw2-2.3.1.jar:./lib/byte-buddy-1.10.17.jar:./lib/classmate-1.5.1.jar:./lib/dom4j-2.1.3.jar:./lib/hibernate-commons-annotations-5.1.2.Final.jar:./lib/jandex-2.2.3.Final.jar:./lib/javassist-3.27.0-GA.jar:./lib/javax.persistence-api-2.2.jar:./lib/jboss-logging-3.4.1.Final.jar:./lib/jboss-transaction-api_1.2_spec-1.1.1.Final.jar:./lib/hibernate-core-5.4.28.Final.jar CreateStudent
```

Если не нужен весь этот мусор в консоли, то в файле `hibernate.cfg.xml` меняем строку `<property name="show_sql">true</property>` на `<property name="show_sql">false</property>`

Теперь проверьте, что запись в БД успешно создалась (ну или ждите пока пройдёт полнолуние)


Мы рассмотрели как установить mySQL (на Linux), как подключить ваш java проект к СУБД, как учесть то, что у вас может быть SQL сервер запущен на настоящем сервере, как создать SSH туннель с помощью стандартных средств и с помощью java, как подключить к вашему проекту hibernate. Много короче рассмотрели. Теперь, можно пользоваться java с SQL с помощью hibernate без IDE (осталось только выучить java, SQL, JDBC и hibernate). Хотя, если вы знаете, как работать в вашей рабочей среде, то сможете спокойно применить этот гайд и к вашей IDE.