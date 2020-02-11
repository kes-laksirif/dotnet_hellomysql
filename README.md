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
