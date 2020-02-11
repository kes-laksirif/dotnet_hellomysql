# dotnet_hellomysql

Creating a Database with Code First in EF Core

The Code First approach enables you to define an entity model in code, create a database from the model, and then add data to the database. The data added by the application is also retrieved by the application using MySQL Connector/NET.

The following example shows the process of creating a database from existing code. Although this example uses the C# language, you can execute it on Windows, macOS, or Linux.

    Create a console application for this example.

        Initialize a valid .NET Core project and console application using the .NET Core command-line interface (CLI) and then switch to the newly created folder (mysqlefcore).

        dotnet new console –o mysqlefcore

cd mysqlefcore

Add the MySql.Data.EntityFrameworkCore package to the application using the CLI as follows:

dotnet add package MySql.Data.EntityFrameworkCore --version 6.10.8

Alternatively, you can use the Package Manager Console in Visual Studio to add the package.

Install-Package MySql.Data.EntityFrameworkCore -Version 6.10.8

Note

The version (for example, 6.10.8) must match the actual Connector/NET version you are using. For current version information, see Table 8.2, “Supported versions of Entity Framework Core”.

Restore dependencies and project-specific tools that are specified in the project file as follows:

dotnet restore

Create the model and run the application.

The model in this EF Core example will be used by the console application. It consists of two entities related to a book library, which will be configured in the LibraryContext class (or database context).

    Create a new file named LibraryModel.cs and then add the following Book and Publisher classes to the mysqlefcore namespace.

    namespace mysqlefcore
    {
      public class Book
      {
        public string ISBN { get; set; }
        public string Title { get; set; }
        public string Author { get; set; }
        public string Language { get; set; }   
        public int Pages { get; set; }
        public virtual Publisher Publisher { get; set; }
      }

      public class Publisher
      {
        public int ID { get; set; }
        public string Name { get; set; }
        public virtual ICollection<Book> Books { get; set; }
      }
    }

Create a new file named LibraryContext.cs and add the code that follows. Replace the generic connection string with one that is appropriate for your MySQL server configuration.

The LibraryContext class contains the entities to use and it enables the configuration of specific attributes of the model, such as Key, required columns, references, and so on.

using Microsoft.EntityFrameworkCore;
using MySQL.Data.EntityFrameworkCore.Extensions;

namespace mysqlefcore
{
  public class LibraryContext : DbContext
  {
    public DbSet<Book> Book { get; set; }

    public DbSet<Publisher> Publisher { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
      optionsBuilder.UseMySQL("server=localhost;database=library;user=user;password=password");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      base.OnModelCreating(modelBuilder);

      modelBuilder.Entity<Publisher>(entity =>
      {
        entity.HasKey(e => e.ID);
        entity.Property(e => e.Name).IsRequired();
      });

      modelBuilder.Entity<Book>(entity =>
      {
        entity.HasKey(e => e.ISBN);
        entity.Property(e => e.Title).IsRequired();
        entity.HasOne(d => d.Publisher)
          .WithMany(p => p.Books);
      });
    }
  }
}

Insert the following code into the existing Program.cs file, replacing the default C# code.

using Microsoft.EntityFrameworkCore;
using System;
using System.Text;

namespace mysqlefcore
{
  class Program
  {
    static void Main(string[] args)
    {
      InsertData();
      PrintData();
    }

    private static void InsertData()
    {
      using(var context = new LibraryContext())
      {
        // Creates the database if not exists
        context.Database.EnsureCreated();

        // Adds a publisher
        var publisher = new Publisher
        {
          Name = "Mariner Books"
        };
        context.Publisher.Add(publisher);

        // Adds some books
        context.Book.Add(new Book
        {
          ISBN = "978-0544003415",
          Title = "The Lord of the Rings",
          Author = "J.R.R. Tolkien",
          Language = "English",
          Pages = 1216,
          Publisher = publisher
        });
        context.Book.Add(new Book
        {
          ISBN = "978-0547247762",
          Title = "The Sealed Letter",
          Author = "Emma Donoghue",
          Language = "English",
          Pages = 416,
          Publisher = publisher
        });

        // Saves changes
        context.SaveChanges();
      }
    }

    private static void PrintData()
    {
      // Gets and prints all books in database
      using (var context = new LibraryContext())
      {
        var books = context.Book
          .Include(p => p.Publisher);
        foreach(var book in books)
        {
          var data = new StringBuilder();
          data.AppendLine($"ISBN: {book.ISBN}");
          data.AppendLine($"Title: {book.Title}");
          data.AppendLine($"Publisher: {book.Publisher.Name}");
          Console.WriteLine(data.ToString());
        }
      }
    }
  }
}

Use the following CLI commands to restore the dependencies and then run the application.

dotnet restore

dotnet run

The output from running the application is represented by the following example:

ISBN: 978-0544003415
Title: The Lord of the Rings
Publisher: Mariner Books

ISBN: 978-0547247762
Title: The Sealed Letter
Publisher: Mariner Books

#MySQL installation guide

How To Install MySQL on Ubuntu 18.04
PostedApril 27, 2018 852.8k views Ubuntu MySQL Ubuntu 18.04 Databases

    3072b0699d3add2e0d0f31b97397dd68302c8842

    By Mark Drake

    Become an author

Not using Ubuntu 18.04? Choose a different version:

    CentOS 7
    Ubuntu 16.04
    Ubuntu 14.04
    Automated: Docker
    request
    Automated: Bash
    request
    Automated: Ansible
    request
    CentOS 8
    request
    Debian 9
    request
    Debian 8
    request
    Debian 10
    request
    View All

A previous version of this tutorial was written by Hazel Virdó
Introduction

MySQL is an open-source database management system, commonly installed as part of the popular LAMP (Linux, Apache, MySQL, PHP/Python/Perl) stack. It uses a relational database and SQL (Structured Query Language) to manage its data.

The short version of the installation is simple: update your package index, install the mysql-server package, and then run the included security script.

    sudo apt update
    sudo apt install mysql-server
    sudo mysql_secure_installation

This tutorial will explain how to install MySQL version 5.7 on an Ubuntu 18.04 server. However, if you’re looking to update an existing MySQL installation to version 5.7, you can read this MySQL 5.7 update guide instead.
Prerequisites

To follow this tutorial, you will need:

    One Ubuntu 18.04 server set up by following this initial server setup guide, including a non-root user with sudo privileges and a firewall.

Step 1 — Installing MySQL

On Ubuntu 18.04, only the latest version of MySQL is included in the APT package repository by default. At the time of writing, that’s MySQL 5.7

To install it, update the package index on your server with apt:

    sudo apt update

Then install the default package:

    sudo apt install mysql-server

This will install MySQL, but will not prompt you to set a password or make any other configuration changes. Because this leaves your installation of MySQL insecure, we will address this next.
Step 2 — Configuring MySQL

For fresh installations, you’ll want to run the included security script. This changes some of the less secure default options for things like remote root logins and sample users. On older versions of MySQL, you needed to initialize the data directory manually as well, but this is done automatically now.

Run the security script:

    sudo mysql_secure_installation

This will take you through a series of prompts where you can make some changes to your MySQL installation’s security options. The first prompt will ask whether you’d like to set up the Validate Password Plugin, which can be used to test the strength of your MySQL password. Regardless of your choice, the next prompt will be to set a password for the MySQL root user. Enter and then confirm a secure password of your choice.

From there, you can press Y and then ENTER to accept the defaults for all the subsequent questions. This will remove some anonymous users and the test database, disable remote root logins, and load these new rules so that MySQL immediately respects the changes you have made.

To initialize the MySQL data directory, you would use mysql_install_db for versions before 5.7.6, and mysqld --initialize for 5.7.6 and later. However, if you installed MySQL from the Debian distribution, as described in Step 1, the data directory was initialized automatically; you don’t have to do anything. If you try running the command anyway, you’ll see the following error:
Output

mysqld: Can't create directory '/var/lib/mysql/' (Errcode: 17 - File exists)
. . .
2018-04-23T13:48:00.572066Z 0 [ERROR] Aborting

Note that even though you’ve set a password for the root MySQL user, this user is not configured to authenticate with a password when connecting to the MySQL shell. If you’d like, you can adjust this setting by following Step 3.
Step 3 — (Optional) Adjusting User Authentication and Privileges

In Ubuntu systems running MySQL 5.7 (and later versions), the root MySQL user is set to authenticate using the auth_socket plugin by default rather than with a password. This allows for some greater security and usability in many cases, but it can also complicate things when you need to allow an external program (e.g., phpMyAdmin) to access the user.

In order to use a password to connect to MySQL as root, you will need to switch its authentication method from auth_socket to mysql_native_password. To do this, open up the MySQL prompt from your terminal:

    sudo mysql

Next, check which authentication method each of your MySQL user accounts use with the following command:

    SELECT user,authentication_string,plugin,host FROM mysql.user;

Output
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             |                                           | auth_socket           | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *CC744277A401A7D25BE1CA89AFF17BF607F876FF | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
4 rows in set (0.00 sec)

In this example, you can see that the root user does in fact authenticate using the auth_socket plugin. To configure the root account to authenticate with a password, run the following ALTER USER command. Be sure to change password to a strong password of your choosing, and note that this command will change the root password you set in Step 2:

    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

Then, run FLUSH PRIVILEGES which tells the server to reload the grant tables and put your new changes into effect:

    FLUSH PRIVILEGES;

Check the authentication methods employed by each of your users again to confirm that root no longer authenticates using the auth_socket plugin:

    SELECT user,authentication_string,plugin,host FROM mysql.user;

Output
+------------------+-------------------------------------------+-----------------------+-----------+
| user             | authentication_string                     | plugin                | host      |
+------------------+-------------------------------------------+-----------------------+-----------+
| root             | *3636DACC8616D997782ADD0839F92C1571D6D78F | mysql_native_password | localhost |
| mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | mysql_native_password | localhost |
| debian-sys-maint | *CC744277A401A7D25BE1CA89AFF17BF607F876FF | mysql_native_password | localhost |
+------------------+-------------------------------------------+-----------------------+-----------+
4 rows in set (0.00 sec)

You can see in this example output that the root MySQL user now authenticates using a password. Once you confirm this on your own server, you can exit the MySQL shell:

    exit

Alternatively, some may find that it better suits their workflow to connect to MySQL with a dedicated user. To create such a user, open up the MySQL shell once again:

    sudo mysql

Note: If you have password authentication enabled for root, as described in the preceding paragraphs, you will need to use a different command to access the MySQL shell. The following will run your MySQL client with regular user privileges, and you will only gain administrator privileges within the database by authenticating:

    mysql -u root -p

From there, create a new user and give it a strong password:

    CREATE USER 'sammy'@'localhost' IDENTIFIED BY 'password';

Then, grant your new user the appropriate privileges. For example, you could grant the user privileges to all tables within the database, as well as the power to add, change, and remove user privileges, with this command:

    GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'localhost' WITH GRANT OPTION;

Note that, at this point, you do not need to run the FLUSH PRIVILEGES command again. This command is only needed when you modify the grant tables using statements like INSERT, UPDATE, or DELETE. Because you created a new user, instead of modifying an existing one, FLUSH PRIVILEGES is unnecessary here.

Following this, exit the MySQL shell:

    exit

Finally, let’s test the MySQL installation.
Step 4 — Testing MySQL

Regardless of how you installed it, MySQL should have started running automatically. To test this, check its status.

    systemctl status mysql.service

You’ll see output similar to the following:
Output

● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: en
   Active: active (running) since Wed 2018-04-23 21:21:25 UTC; 30min ago
 Main PID: 3754 (mysqld)
    Tasks: 28
   Memory: 142.3M
      CPU: 1.994s
   CGroup: /system.slice/mysql.service
           └─3754 /usr/sbin/mysqld

If MySQL isn’t running, you can start it with sudo systemctl start mysql.

For an additional check, you can try connecting to the database using the mysqladmin tool, which is a client that lets you run administrative commands. For example, this command says to connect to MySQL as root (-u root), prompt for a password (-p), and return the version.

    sudo mysqladmin -p -u root version

You should see output similar to this:
Output

mysqladmin  Ver 8.42 Distrib 5.7.21, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version      5.7.21-1ubuntu1
Protocol version    10
Connection      Localhost via UNIX socket
UNIX socket     /var/run/mysqld/mysqld.sock
Uptime:         30 min 54 sec

Threads: 1  Questions: 12  Slow queries: 0  Opens: 115  Flush tables: 1  Open tables: 34  Queries per second avg: 0.006

This means MySQL is up and running.
Conclusion

You now have a basic MySQL setup installed on your server. Here are a few examples of next steps you can take:

    Implement some additional security measures
    Relocate the data directory
    Manage your MySQL servers with SaltStack


