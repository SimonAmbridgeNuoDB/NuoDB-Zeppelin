# NuoDB Analytics Using Zeppelin Notebooks, SparkSQL, Python and NuoSQL

Apache Zeppelin is a completely open web-based notebook that enables interactive data analytics, data ingestion, data exploration, visualisation, sharing and collaboration.

![Image description](zeppelin-chart-example.png)

This project will demonstrate how to run analytic queries and data visualisation against NuoDB using Zeppelin Notebooks with SparkSQL, Python and NuoSQL.

We will also set up basic authentication using Apache Shiro to control access to the notebooks that you create.


![Image description](nuosql-notebook-2.png)

This demo will use the following technologies:

- NuoDB distributed SQL cloud-native relational database
- Apache Zeppelin
- Apache Shiro
- NuoSQL
- Spark
- Scala
- Python

## Pre-requisites


* A NuoDB database to run your queries against.
	* Hardware and software requirements for NuoDB are listed here - http://doc.nuodb.com/Latest/Default.htm#System-Requirements.htm
	* Installation and Deployment options are described here - http://doc.nuodb.com/Latest/Default.htm#Deployment-models.htm
	* You can download the NuoDB binaries for various platforms here - https://www.nuodb.com/dev-center/community-edition-download
* A machine on which to install Zeppelin.
	* This could be one of the NuoDB Transaction Engines (TE's), or a separate machine/cloud instance, or even a laptop. 
	* Ideally the Zeppelin server should be in close proximity to the data source in order to reduce latency.

<BR>
NB This exercise was completed in a Linux environment. NuoDB is also supported on Windows, but only for development purposes.

## Installation

### Download the NuoDB JDBC Driver

Download the NuoDB JDBC driver - you can download the latest NuoDB JDBC driver from Maven Central directly from here - https://repo1.maven.org/maven2/com/nuodb/jdbc/nuodb-jdbc/20.1.0/nuodb-jdbc-20.1.0.jar

Place the JDBC driver in a location where it can be accessed by Zeppelin.


### Install Apache Zeppelin

On the machine where Zeppelin will run, download the Binary package with all interpreters at https://zeppelin.apache.org/download.html

There is also a Docker image available, or the option to build from source.

* Install guide: https://zeppelin.apache.org/docs/0.8.1/quickstart/install.html

* Documentation: http://zeppelin.apache.org/docs/0.8.1/index.html

In this example I'll deploy the Zeppelin binaries from the tarball distribution.

Download the Zeppelin archive to a working directory.

```
$ ls
zeppelin-0.8.1-bin-all.tgz
```

Using your user account (you don't need to do this as root), uncompress the archive and go into the extracted Zeppelin directory: 
```
$ tar xvf zeppelin-0.8.1-bin-all.tgz

$ cd zeppelin-0.8.1-bin-all

$ ls
LICENSE			bin			lib			logs			spark-warehouse
NOTICE			conf			licenses		notebook		zeppelin-web-0.8.1.war
README.md		interpreter		local-repo		run
```
Use sudo to start the Zeppelin daemon:
```
$ sudo bin/zeppelin-daemon.sh start
Zeppelin start                                             [  OK  ]
```

Go to the Zeppelin server home page at http://[your-zeppelin-host]:8080

Where [your-zeppelin-host] is the name or the IP address of the machine where Zeppelin is running.

At this point you will see the main Zeppelin page, where you can run the Zeppelin tutorials, import new notebooks, link to the documentation, etc.

![Image description](zeppelin-start-page.png)

<BR>
If you want to stop Zeppelin you can use the stop option:

```
$ bin/zeppelin-daemon.sh stop
Zeppelin stopped                                           [  OK  ]
```
<BR>

### Install The NuoDB Python driver

On the machine where Zeppelin is running, download the NuoDB Python driver:

```
[Ec2-user_host-1 ~]$ curl -L https://github.com/nuodb/nuodb-python/archive/master.tar.gz | tar xz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   126    0   126    0     0    362      0 --:--:-- --:--:-- --:--:--   363
  0     0    0  351k    0     0   268k      0 --:--:--  0:00:01 --:--:-- 1041k
```

Install the NuoDB Python driver:


```
[Ec2-user_host-1 ~]$ cd nuodb-python*

[Ec2-user_host-1 ~]$ nuodb-python-master]$ sudo python setup.py install
```

<BR>

## Run Python code against a NuoDB database using a Zeppelin notebook

The first notebook example will use Python to query the NuoDB database.

From the Zeppelin main page, on the menu bar click Notebook -> Create New Note
* Give your notebook a meaningful name, and select your default interpreter.
* The default interpreter is Spark, but in this case we will be using Python, so change the drop-down selection accordingly.

![Image description](new-notebook-python.png)

Click Create.

In the notebook itself, go to the first blank paragraph and type:
```
import pynuodb
```

Your screen should look like this:

![Image description](python-notebook-1.png)

<BR>
Then click the play icon on the right-hand side of the paragraph to run the instruction.

![Image description](zeppelin-buttons.png)

<BR>
When the instruction completes (very quickly) you will get a message below it, like this:

```
Took 3 sec. Last updated by anonymous at August 06 2019, 5:46:03 PM.
```

<BR>
<B>NB</B> If you had selected Spark as your default interpreter you would have had to invoke the Python interpreter in each cell like this to override the default:

```
%python
import pynuodb
```

<BR>
<B>NB</B> 
You can change the default interpreter in a running notebook - use the cog symbol at the top right of the notebook to modify the notebook interpreter bindings.

![Image description](zeppelin-cog.png)

<BR>
Now let's demonstrate how to connect to NuoDB using Python code running inside the notebook.

This Python code sample will connect to the database, drop the test table in the "user" schema if it already exists, then create the test table, insert some records and finally query them back.

Add a paragraph below the one you used to import the NuoDB Python library
* Create a new paragraph by hovering the mouse over the lower edge of the existing paragraph like this:
![Image description](zeppelin-new-para.png)


Paste in the block of code below and edit according to your connection details.

```
options = {"schema": "user"}
connect_kw_args = {'database': "your-db", 'host': "your-server", 'user': "dba", 'password': "dba", 'options': options}
```

Change the database connection details to match your environment and credentials - database name, host address or name, username and password, and schema name.

To demonstrate how a notebook works, we'll break the remainder of the code into sections to show how individual instructions, or sets of instructions, can be modified and re-run.

You should now have a new paragraph similar to the following example, so now click run: 

![Image description](python-notebook-2.png)

There will be no output, but the command will completed without error.


Next we'll create a couple of connection objects:

```
connection = pynuodb.connect(**connect_kw_args)
cursor = connection.cursor()
```

Click run and your output should look like this:

![Image description](python-notebook-3.png)

Finally, to prove it all works, run the main part of the program:

```
try:
    stmt_drop = "DROP TABLE IF EXISTS names"
    cursor.execute(stmt_drop)

    stmt_create = """
    CREATE TABLE names (
        id BIGINT NOT NULL GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
        name VARCHAR(30) DEFAULT '' NOT NULL,
        age INTEGER DEFAULT 0
    )"""
    cursor.execute(stmt_create)

    names = (('Greg', 17,), ('Marsha', 16,), ('Jan', 14,))
    stmt_insert = "INSERT INTO names (name, age) VALUES (?, ?)"
    cursor.executemany(stmt_insert, names)

    connection.commit()

    age_limit = 15
    stmt_select = "SELECT id, name FROM names where age > ? ORDER BY id"
    cursor.execute(stmt_select, (age_limit,))
    print("Results:")
    for row in cursor.fetchall():
        print("%d | %s" % (row[0], row[1]))

finally:
    cursor.execute(stmt_drop)
    cursor.close()
    connection.close()
```
<BR>

After pasting in the code above, click the play/run icon. Your output will look like this:

```
Results:
1 | Greg
2 | Marsha

Took 1 sec. Last updated by anonymous at August 06 2019, 6:11:47 PM.
```

Now let's look at an example that return data from the sample Hockey database.

This code block starts with the same connection configuration that must be edited to reflect your environment. 
```
options = {"schema": "user"}
connect_kw_args = {'database': "your-db", 'host': "your-server", 'user': "dba", 'password': "dba", 'options': options}

connection = pynuodb.connect(**connect_kw_args)
cursor = connection.cursor()

try:

    querystr = "SELECT p.firstname,p.lastname,p.firstnhl,p.lastnhl,s.teamid,s.stint,gamesplayed FROM players p LEFT OUTER JOIN scoring s ON p.playerid = s.playerid AND p.firstnhl = s.year AND s.position = 'G' WHERE p.firstnhl = 2011 AND s.gamesplayed IS NOT NULL ORDER BY LASTNAME,FIRSTNAME,TEAMID"
    cursor.execute(querystr)

    for row in cursor.fetchall():
        print row[0], row[1],row[2],row[3],row[4],row[5],row[6]

finally:
    cursor.close()
    connection.close()


print
d.disconnect()
```

When you click the play/run icon you'll see the following sample data from the Hockey schema returned from NuoDB:

```
Brian Foster 2011 2011 FLO 1 1
Matt Hackett 2011 2011 MIN 1 12
Shawn Hunwick 2011 2011 CBS 1 1
Leland Irving 2011 2011 CAL 1 7
Mike Murphy 2011 2011 CAR 1 2
Anders Nilsson 2011 2011 NYI 1 4
Jussi Rynnas 2011 2011 TOR 1 2
Ben Scrivens 2011 2011 TOR 1 12
Iiro Tarkki 2011 2011 AND 1 1
Brad Thiessen 2011 2011 PIT 1 5
Allen York 2011 2011 CBS 1 11

```

We now have a working sample Python notebook that can access data in NuoDB!

<BR>

## Run SparkSQL, Scala and Java instructions against a NuoDB database in a Zeppelin notebook

This example will use SparkSQL and some light Scala to query the NuoDB database.

First we need to point the Spark interpreter to our NuoDB JDBC driver.

### Add the NuoDB JDBC driver to the Spark Shell artifact list.

Click the drop down next to anonymous in the top right corner of the page and select Intepreters.

![Image description](zeppelin-menu.png)

Scroll down to the Spark definition. 

Click the edit button on the right hand side of the Spark interpreter section. Scroll down the Spark definition and add the artefact describing the location of the NuoDB JDBC driver that you downloaded previously onto the server where Zeppelin is running.

For example

```
Dependencies
artifact
/home/ec2-user/nuodb-jdbc-20.0.0.jar
```
<BR>

### Create The Spark Notebook

Create a new blank workbook (Notebook -> Create New Note) and give it a title but this time leave the default interpreter set to Spark.

In the new empty cell, past the following library imports and run the cell contents by clicking the play icon as you did previously:

```
import org.apache.commons.io.IOUtils
import java.net.URL
import java.nio.charset.Charset

import org.apache.log4j.{Level, Logger}
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.{SQLContext, SaveMode}

import java.util.Properties;
import java.io._

import java.sql.DriverManager;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
```
This might take a few seconds the first time that you run this command as it takes time for the Spark engine to initialise.

In a new cell, paste the following and edit the paremeters to match your environment, then click run.

```
val jdbcHostname = "your-host"
val jdbcPort = 48004
val jdbcDatabase = "your-db"
val jdbcUsername = "dba"
val jdbcPassword = "dba"
val jdbcSchema   = "user"
```

In the next cell create the val containing the JDBC URL and click run.

```
val jdbcUrl = s"jdbc:com.nuodb://${jdbcHostname}:${jdbcPort}/${jdbcDatabase}"
```
You'll see output like this:

```
jdbcUrl: String = jdbc:com.nuodb://<your-hostname>:48004/your-db
```


In the next cell, create the JDBC properties object and click run.

```
val connProperties = new Properties()
connProperties.put("user", s"${jdbcUsername}")
connProperties.put("password", s"${jdbcPassword}")
connProperties.put("schema", s"${jdbcSchema}")
```

You'll see this:

```
connProperties: java.util.Properties = {user=dba, password=dba, schema=user}
res45: Object = null
```

Now that you have a working connection, you can query the database. In the next cell we create a dataframe containing data from the Hockey sample schema. 
Click run.

```
val player1 = spark.read
  .format("jdbc")
  .option("url",jdbcUrl+"?user=dba&password=dba&schema=user")
  .option("dbtable", "players")
  .load()

```

You now have a dataframe called player1 containing the data from the players table in NuoDB.

In a new cell run the following command to print the dataframe schema:

```
player1.printSchema()
```

Your output will look like this:
```
root
 |-- PLAYERID: string (nullable = false)
 |-- FIRSTNAME: string (nullable = true)
 |-- LASTNAME: string (nullable = true)
 |-- HEIGHT: integer (nullable = true)
 |-- WEIGHT: integer (nullable = true)
 |-- FIRSTNHL: integer (nullable = false)
 |-- LASTNHL: integer (nullable = false)
 |-- POSITION: string (nullable = true)
 |-- BIRTHYEAR: integer (nullable = true)
 |-- BIRTHMON: integer (nullable = true)
 |-- BIRTHDAY: integer (nullable = true)
 |-- BIRTHCOUNTRY: string (nullable = true)
 |-- BIRTHSTATE: string (nullable = true)
 |-- BIRTHCITY: string (nullable = true)
```

How many records? Use this:
```
player1.count()

res48: Long = 7520
```

Finally, to actually view the data:
```
player1.show(5)
```
The output will be:
```
+---------+---------+----------+------+------+--------+-------+--------+---------+--------+--------+------------+----------+------------+
| PLAYERID|FIRSTNAME|  LASTNAME|HEIGHT|WEIGHT|FIRSTNHL|LASTNHL|POSITION|BIRTHYEAR|BIRTHMON|BIRTHDAY|BIRTHCOUNTRY|BIRTHSTATE|   BIRTHCITY|
+---------+---------+----------+------+------+--------+-------+--------+---------+--------+--------+------------+----------+------------+
|aaltoan01|    Antti|     Aalto|    73|   210|    1997|   2000|       C|     1975|       3|       4|     Finland|         0|Lappeenranta|
|abbeybr01|    Bruce|     Abbey|    73|   185|       0|      0|       D|     1951|       8|      18|      Canada|        ON|     Toronto|
|abbotge01|   George|    Abbott|    67|   153|    1943|   1943|       G|     1911|       8|       3|      Canada|        ON|    Synenham|
|abbotre01|      Reg|    Abbott|    71|   164|    1952|   1952|       C|     1930|       2|       4|      Canada|        MB|    Winnipeg|
|abdelju01|   Justin|Abdelkader|    73|   195|    2007|   2011|       L|     1987|       2|      25|         USA|        MI|    Muskegon|
+---------+---------+----------+------+------+--------+-------+--------+---------+--------+--------+------------+----------+------------+
only showing top 5 rows
```


<BR>

## Run SQL instructions against a NuoDB database in a Zeppelin notebook


The final exercise will create and use a custom NuoSQL interpreter to allow SQL commands to be run interactively against a NuoDB database using a Zeppelin notebook.

There is no default NuoDB interpreter provided, but we can easily create one.


### Create The NuoSQL Interpreter

Click the drop-down menu next to Anonymous in the top right corner of the page.

Select the Interpreter option.

Click the Create button.

Give the new interpreter a name, e.g. NuoSQL

Select jdbc from the list of options under Interpreter Group

A list of properties and values for the JDBC connection will be displayed. 

Change the following:

```
default.user		<your-user>
default.password	<your-password>
default.driver		com.nuodb.jdbc.Driver
default.url		jdbc:com.nuodb://<your-server:48004/<your-db>
```

At the bottom of the definition under Dependencies add the JDBC driver e.g.

```
Dependencies
artifact
/home/ec2-user/nuodb-jdbc-20.0.0.jar
```

![Image description](nuodb-interpreter-config.png)

Click Save.

### Create The NuoSQL Workbook ###

Now create a new blank workbook as you did previously, but this time select NuoSQL as the default interpreter.

In the new empty cell, past the following and run the cell:

```
select * from system.connections
```

We can see the data queried from NuoDB using SQL:

![Image description](nuosql-notebook-1.png)

<BR>
<BR>

## Set Up User Security with Apache Shiro

By default, all users log into Zeppelin as anonymous, which is fine if you're developing on your own machine. But if you want to share with others, it's nice to have some control over who has access and what permissions they have.

Apache Shiro is a powerful and easy-to-use Java security framework that performs authentication, authorisation, cryptography, and session management. 

Read more here: https://zeppelin.apache.org/docs/0.6.2/security/shiroauthentication.html


When you connect to Apache Zeppelin, you will be asked to enter your credentials. Once you're logged in you have access to all notes including other user's notes.
<BR>

### Creating Roles & Users in Apache Shiro


Set up basic authentication in the shiro.ini file:

```
$ cp conf/shiro.ini.template conf/shiro.ini
```

```
$ vi conf/shiro.ini
```

The default users and passwords, role assignments:
```
admin = password1, admin
user1 = password2, role1, role2
user2 = password3, role3
user3 = password4, role2
```

The default roles:
```
[roles]
role1 = *
role2 = *
role3 = *
admin = *
```

### Turn On authc Security ###

Turn on auth security at the bottom of the file:
```
#/** = anon
/** = authc
```

### Configure Zeppelin for Apache Shiro ###

Configure the zeppelin-site.xml file:

```
$ cp zeppelin-site.xml.template zeppelin-site.xml
$ vi zeppelin-site.xml
```

Turn off anonymous logins - set true to false:
```
<property>
  <name>zeppelin.anonymous.allowed</name>
  <value>false</value>
  <description>Anonymous user allowed by default</description>
</property>
```

### Restart The Zeppelin Daemon & Test ###

Restart the Zeppelin daemon:

```
$ bin/zeppelin-daemon.sh restart
Zeppelin stop                                              [  OK  ]
Zeppelin start                                             [  OK  ]
```

Reload the main page at http://[your-server]:8080

You'll notice a new login button in the top right corner:

![Image description](zeppelin-login-1.png)

Click Login and enter the credentials defined above in shiro.ini, e.g. admin/password1

![Image description](zeppelin-login-2.png)

That's it - you now have basic security enabled. You can now play around with roles and permissions on notebooks.

<BR>







