# Лабораторная работа №3

*по курсу: "DEV-J30. Программирование на платформе Java. Разработка многоуровневых приложений"*

## JDBC - Java Database Connectivity 

Java Database Connectivity API - промышленный стандарт сопряжения java-приложения с источниками данных, обеспечивающий абстрагирование приложения от деталей реализации технологии хранения данных на стороне системы управления базами данных (СУБД).

### Создание соединения с источником данных

Соединение с источником данных выполняется с использованием статического метода `getConnection()` класса `DriverManager`. Данный метод имеет три перегруженных варианта для различных сценариев использования. Упомянутые методы объявлены в классе `DriverManager` следующим образом.

```java
public class DriverManager {
    
    // ...
    
    /**
     * Attempts to establish a connection to the given database URL.
     * The <code>DriverManager</code> attempts to select an appropriate driver from
     * the set of registered JDBC drivers.
     *<p>
     * <B>Note:</B> If a property is specified as part of the {@code url} and
     * is also specified in the {@code Properties} object, it is
     * implementation-defined as to which value will take precedence.
     * For maximum portability, an application should only specify a
     * property once.
     *
     * @param url a database url of the form
     * <code> jdbc:<em>subprotocol</em>:<em>subname</em></code>
     * @param info a list of arbitrary string tag/value pairs as
     * connection arguments; normally at least a "user" and
     * "password" property should be included
     * @return a Connection to the URL
     * @exception SQLException if a database access error occurs or the url is
     * {@code null}
     * @throws SQLTimeoutException  when the driver has determined that the
     * timeout value specified by the {@code setLoginTimeout} method
     * has been exceeded and has at least tried to cancel the
     * current database connection attempt
     */
    @CallerSensitive
    public static Connection getConnection(String url, Properties info) throws SQLException {
        return (getConnection(url, info, Reflection.getCallerClass()));
    }

    /**
     * Attempts to establish a connection to the given database URL.
     * The <code>DriverManager</code> attempts to select an appropriate driver from
     * the set of registered JDBC drivers.
     *<p>
     * <B>Note:</B> If the {@code user} or {@code password} property are
     * also specified as part of the {@code url}, it is
     * implementation-defined as to which value will take precedence.
     * For maximum portability, an application should only specify a
     * property once.
     *
     * @param url a database url of the form
     * <code>jdbc:<em>subprotocol</em>:<em>subname</em></code>
     * @param user the database user on whose behalf the connection is being
     *   made
     * @param password the user's password
     * @return a connection to the URL
     * @exception SQLException if a database access error occurs or the url is
     * {@code null}
     * @throws SQLTimeoutException  when the driver has determined that the
     * timeout value specified by the {@code setLoginTimeout} method
     * has been exceeded and has at least tried to cancel the
     * current database connection attempt
     */
    @CallerSensitive
    public static Connection getConnection(String url, String user, String password) throws SQLException {
        Properties info = new Properties();

        if (user != null) {
            info.put("user", user);
        }
        if (password != null) {
            info.put("password", password);
        }

        return (getConnection(url, info, Reflection.getCallerClass()));
    }

    /**
     * Attempts to establish a connection to the given database URL.
     * The <code>DriverManager</code> attempts to select an appropriate driver from
     * the set of registered JDBC drivers.
     *
     * @param url a database url of the form
     *  <code> jdbc:<em>subprotocol</em>:<em>subname</em></code>
     * @return a connection to the URL
     * @exception SQLException if a database access error occurs or the url is
     * {@code null}
     * @throws SQLTimeoutException  when the driver has determined that the
     * timeout value specified by the {@code setLoginTimeout} method
     * has been exceeded and has at least tried to cancel the
     * current database connection attempt
     */
    @CallerSensitive
    public static Connection getConnection(String url) throws SQLException {
        java.util.Properties info = new java.util.Properties();
        return (getConnection(url, info, Reflection.getCallerClass()));
    }
    
    // ...
}
```

Исходя из вышеописанного можно сказать, что соединение с источником данных может быть установлено на основании унифицированного определителя ресурса. Унифицированный определитель ресурса (Uniform Resource Locator - URL)  должен быть сформирован в соответствии со стандартом [RFC 1738](https://tools.ietf.org/html/rfc1738).

![Формат записи URL-строки][url]

[url]: https://upload.wikimedia.org/wikipedia/commons/9/96/URI_syntax_diagram.png

Пример соединения с базой данных Java DB может выглядеть следующим образом:

```java
String url = "jdbc:derby://localhost/database-name";
Connection connection = DriverManager.getConnection(url);
```

Кроме определителя ресурса, может потребоваться также указать учётные данные пользователя, от имени которого выполняется соединение с источником данных. Учётные дынные могут быть переданы в метод `getConnection()` двумя способами: в виде строковых аргументов, или в составе объекта типа `Properties`.

По исходному коду метода `getConnection()` можно видеть, что если мы передадим учётные данные в строковом формате, они будут помещены в экземпляр типа `Properties` с ключами `user` и `password` соответственно.

Если мы храним конфигурацию приложения в виде файла свойств. То можно хранить эти параметры с соответствующими ключами и просто передавать экземпляр типа `Properties` в метод `getConnection()`.

В случае успешного завершения вызова метода `getConnection()` класса `DriverManager` мы получаем экземпляр типа `Connection`. В случае неудачи будет инициировано исключение типа `SQLException`. 

### Интерфейс `java.sql.Connection`

Экземпляр интерфейса `Connection` описывает установленное соединение. В интерфейсе `Connection` определены методы, позволяющие управлять установленным соединением и создавать экземпляры интерфейсов, предоставляющих возможность  выполнять запросы к источнику данных: `Statement`, `PreparedStatement` и `CallableStatement`.

В предложенной ниже таблице представлены некоторые методы интерфейса `Connection`. 

Название метода|Описание метода
---|---
`createStatement()`| Возвращает экземпляр реализации интерфейса `Statement` для текущего источника данных.
`prepareStatement()`| Возвращает экземпляр реализации интерфейса `PreparedStatement` для текущего источника данных.
`prepareCall()`| Возвращает экземпляр реализации интерфейса `CallableStatement` для текущего источника данных.
`close()`| Закрывает соединение.
`isClosed()`| Возвращает `true`, если соединение закрыто. В обратном случае - `false`.
`commit()`| Завершает транзакцию.
`rollback()`| Отменяет транзакцию.
`setAutoCommit()`| Устанавливает значение свойства `AutoCommit`.

> Настоящее описание к лабораторной работе будет расширено  и дополнено в ближайшее время.

Подробная информация может быть найдена в [официальной сайте Oracle](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html).