# SOLID Principle
## S: Single-Responsibility Principle
Setiap class hanya boleh memiliki satu dan hanya satu tanggungjawab. Bukan berarti satu fungsi, tapi satu tanggungjawab. Semua fungsi harus berkaitan dengan tanggungjawabnya.

Contoh:

```C#
public class UserService  
{  
   public void Register(string email, string password)  
   {  
      if (!ValidateEmail(email))  
         throw new ValidationException("Email is not an email");  
         var user = new User(email, password);  
  
         SendEmail(new MailMessage("mysite@nowhere.com", email) { Subject="HEllo foo" });  
   }
   public virtual bool ValidateEmail(string email)  
   {  
     return email.Contains("@");  
   }  
   public bool SendEmail(MailMessage message)  
   {  
     _smtpClient.Send(message);  
   }  
}
```

Contoh diatas tidak menunjukkan Single-Responsibility, ValidateEmail dan SendEmail tidak ada hubungannya dengan UserService.

```C#
public class UserService  
{  
    EmailService _emailService;  
    DbContext _dbContext;  
    public UserService(EmailService aEmailService, DbContext aDbContext)  
    {  
        _emailService = aEmailService;  
        _dbContext = aDbContext;  
    }  
    public void Register(string email, string password)  
    {  
        if (!_emailService.ValidateEmail(email))  
            throw new ValidationException("Email is not an email");  
            var user = new User(email, password);  
            _dbContext.Save(user);  
            emailService.SendEmail(new MailMessage("myname@mydomain.com", email) {Subject="Hi. How are you!"});  
    
    }  
}  

public class EmailService  
{  
    SmtpClient _smtpClient;  
    public EmailService(SmtpClient aSmtpClient)  
    {  
        _smtpClient = aSmtpClient;  
    }  
    public bool virtual ValidateEmail(string email)  
    {  
        return email.Contains("@");  
    }  
    public bool SendEmail(MailMessage message)  
    {  
        _smtpClient.Send(message);  
    }  
}   
```

## O: Open/Closed Principle
Objek harus terbuka untuk _extension_ namun tertutup untuk _modification_.

**_Open for Extension_**, Fungsionalitas baru bisa ditambahkan ketika ada kebutuhan.

**_Closed for Modification_**, Class sudah dbuat dan sudah di tes, jangan diubah selain untuk fungsionalitas.

Contoh:

Kita memiliki class Rectangle.

```C#
public class Rectangle{  
   public double Height {get;set;}  
   public double Wight {get;set; }  
}
```

Kita perlu menghitung total luas dari beberapa Rectangle. Ini adalah tanggungjawab Class baru (sesuai SRP) yaitu AreaCalculator.

```C#
public class AreaCalculator {  
   public double TotalArea(Rectangle[] arrRectangles)  
   {  
      double area;  
      foreach(var objRectangle in arrRectangles)  
      {  
         area += objRectangle.Height * objRectangle.Width;  
      }  
      return area;  
   }  
}
```

Sekarang, kita memiliki class Circle.

```C#
public class Circle{  
   public double Radius {get;set;}  
}
```

Cara menghitung luas Circle berbeda dengan Rectangle, maka kita bisa tambahkan **if** di AreaCalculator.

```C#
public class AreaCalculator  
{  
   public double TotalArea(object[] arrObjects)  
   {  
      double area = 0;  
      Rectangle objRectangle;  
      Circle objCircle;  
      foreach(var obj in arrObjects)  
      {  
         if(obj is Rectangle)  
         {    
            area += obj.Height * obj.Width;  
         }  
         else  
         {  
            objCircle = (Circle)obj;  
            area += objCircle.Radius * objCircle.Radius * Math.PI;  
         }  
      }  
      return area;  
   }  
}
```

Namun, bagaimana kalau kita mau menambahkan Segitiga? Kita **perlu merubah lagi AreaCalculator**. Setiap kali ada bentuk baru, AreaCalculator perlu dirubah. Hal ini berarti melanggar **Closed for Modification** karena _kita tidak menambah fungsi baru selain menghitung luas, hanya memperkenalkan bentuk baru, dan itu perlu mengubah class_.

Kita bisa memindahkan logic untuk menghitung luas dari AreaCalculator kedalam Class untuk masing-masing bentuk.

```C#
public class Rectangle  
{  
   public double Height {get;set;}  
   public double Width {get;set;}  
   public override double Area()  
   {  
      return Height * Width;  
   }  
}  
public class Circle  
{  
   public double Radius {get;set;}  
   public override double Area()  
   {  
      return Radius * Radius * Math.PI;  
   }  
}  
```

Karena setiap bentuk sudah tahu cara menghitung luasnya masing-masing, maka AreaCalculator menjadi lebih simple.

```C#
public class AreaCalculator  
{  
   public double TotalArea(Shape[] arrShapes)  
   {  
      double area=0;  
      foreach(var objShape in arrShapes)  
      {  
         area += objShape.Area();  
      }  
      return area;  
   }  
}
```

Namun, bagaimana AreaCalculator tahu bahwa Class yang ada di _arrShapes_ adalah sebuah bentuk bangun datar? Kita perlu memastikan hal tersebut dengan mengimplementasi Abstract Class.

```C#
public abstract class Shape  
{  
   public abstract double Area();  
} 
```

Sehingga Class bentuk bangun datar bisa mengacu pada Abstract Class. Hal ini membangun sebuah _standard_ untuk Class bentuk bangun datar.

```C#
public class Rectangle: Shape  
{  
   public double Height {get;set;}  
   public double Width {get;set;}  
   public override double Area()  
   {  
      return Height * Width;  
   }  
}  
public class Circle: Shape  
{  
   public double Radius {get;set;}  
   public override double Area()  
   {  
      return Radius * Radus * Math.PI;  
   }  
}
```

## L: Liskov Substitution
Setiap SubClass harus dapat menggantikan ParentClass. SubClass harus memiliki _behavior_ yang sama dengan ParentClass.

Contoh:

```C#
public class AreaCalculator  
{  
   public double Sum(Shape[] arrShapes)  
   {  
      double area=0;  
      foreach(var objShape in arrShapes)  
      {  
         area += objShape.Area();  
      }  
      return area;  
   }  
}

public class VolumeCalculator : AreaCalculator  
{  
   public double Sum(Shape[] arrShapes)  
   {  
      double volume=0;  
      foreach(var objShape in arrShapes)  
      {  
         volume += objShape.Volume();  
      } 

      int res = Convert.ToInt32(volume);
      return res;  
   }  
}
```

Class VolumeCalculator merupakan SubClass dari AreaCalculator. Keduanya memiliki fungsi **Sum** dengan bentuk return yang berbeda. Hal ini menunjukkan keduanya memiliki _behavior_ yang berbeda.

Contoh lain:
Kita membuat program untuk mengelola Text File. Kita perlu fungsionalitas untuk memuat dan menyimpan text kedalam file.

```C#
public class TextFile  
{  
   public string FilePath {get;set;}  
   public string FileText {get;set;}  
   public string LoadText()  
   {  
      /* Code to read text from text file */  
   }  
   public string SaveText()  
   {  
      /* Code to save text into text file */  
   }  
}  

public class TextFileManager  
{  
   public List<TextFile> lstTxtFiles {get;set}  
  
   public string GetTextFromFiles()  
   {  
      StringBuilder objStrBuilder = new StringBuilder();  
      foreach(var objFile in lstTxtFiles)  
      {  
         objStrBuilder.Append(objFile.LoadText());  
      }  
      return objStrBuilder.ToString();  
   }  
   public void SaveTextIntoFiles()  
   {  
      foreach(var objFile in lstTxtFiles)  
      {  
         objFile.SaveText();  
      }  
   }  
} 
```

Ternyata terdapat file yang hanya bisa dibaca dan tidak bisa dimodifikasi.

```C#
public class ReadOnlyTextFile: TextFile  
{  
   public string FilePath {get;set;}  
   public string FileText {get;set;}  
   public string LoadText()  
   {  
      /* Code to read text from text file */  
   }  
   public void SaveText()  
   {  
      /* Throw an exception when app flow tries to do save. */  
      throw new IOException("Can't Save");  
   }  
}
```

Untuk itu, kita perlu mengubah fungsi TextFileManager dengan menambahkan sebuah kondisional.

```C#
public class TextFileManager  
{  
   public List<textFile? lstTxtFiles {get;set}  
   public string GetTextFromFiles()  
   {  
      StringBuilder objStrBuilder = new StringBuilder();  
      foreach(var objFile in lstTxtFiles)  
      {  
         objStrBuilder.Append(objFile.LoadText());  
      }  
      return objStrBuilder.ToString();  
   }  
   public void SaveTextIntoFiles()  
   {  
      foreach(var objFile in lstTxtFiles)  
      {  
         //Check whether the current file object is read-only or not.If yes, skip calling it's  
         // SaveText() method to skip the exception.  
  
         if(! objFile is ReadOnlyTextFile)  
         objFile.SaveText();  
      }  
   }  
}
```

Penambahan kondisional diatas menunjukkan bahwa kita tidak dapat menggunakan SubClass sebagai pengganti ParentClass, keduanya memiliki _behavior_ yang berbeda.

---

Solusi untuk permasalahan diatas dapat diselesaikan dengan penerapan interface.

```C#
public interface IReadableTextFile  
{  
    string LoadText();  
}  
public interface IWritableTextFile  
{  
    void SaveText();  
}  

public class ReadOnlyTextFile: IReadableTextFile  
{  
   public string FilePath {get;set;}  
   public string FileText {get;set;}  
   public string LoadText()  
   {  
      /* Code to read text from text file */  
   }  
}

public class TextFile: IWritableTextFile,IReadableTextFile  
{  
   public string FilePath {get;set;}  
   public string FileText {get;set;}  
   public string LoadText()  
   {  
      /* Code to read text from text file */  
   }  
   public void SaveText()  
   {  
      /* Code to save text into text file */  
   }  
}

public class TextFileManager  
{  
   public string GetTextFromFiles(List<IReadableTextFile> aLstReadableFiles)  
   {  
      StringBuilder objStrBuilder = new StringBuilder();  
      foreach(var objFile in aLstReadableFiles)  
      {  
         objStrBuilder.Append(objFile.LoadText());  
      }  
      return objStrBuilder.ToString();  
   }  
   public void SaveTextIntoFiles(List<IWritableTextFile> aLstWritableFiles)  
   {  
   foreach(var objFile in aLstWritableFiles)  
   {  
      objFile.SaveText();  
   }  
   }  
}
```

Disini GetTextFromFiles hanya akan menerima instance yang mengimplementasikan IReadableTextFile, yaitu TextFile dan ReadOnlyTextFile. Sedangkan SaveTextIntoFiles hanya akan menerima instance yang mengimplementasikan IWritableTextFile, yaitu TextFile.

## I: Interface Segregation Principle
Client tidak boleh dipaksa mengimplementasikan Interface yang tidak digunakannya, atau bergantung pada Method yang tidak digunakan.

Interface harus memiliki tujuan yang spesifik.

Contoh:

TeamLead bisa mendelegasikan tugas sekaligus mengerjakan tugas.
```C#
public Interface ILead  
{  
   void CreateSubTask();  
   void AssginTask();  
   void WorkOnTask();  
}  
public class TeamLead : ILead  
{  
   public void AssignTask()  
   {  
      //Code to assign a task.  
   }  
   public void CreateSubTask()  
   {  
      //Code to create a sub task  
   }  
   public void WorkOnTask()  
   {  
      //Code to implement perform assigned task.  
   }  
}
```

Sedangkan Manager bisa mendelegasikan tugas, namun tidak boleh mengerjakan tugas.
```C#
public class Manager: ILead  
{  
   public void AssignTask()  
   {  
      //Code to assign a task.  
   }  
   public void CreateSubTask()  
   {  
      //Code to create a sub task.  
   }  
   public void WorkOnTask()  
   {  
      throw new Exception("Manager can't work on Task");  
   }  
}
```

WorkOnTask tidak seharusnya ada di class Manager, namun kita mengimplementasikan Class ini berdasarkan ILead Interface.
Untuk itu, kita perlu memperbaiki rancangannya.

```C#
public interface IProgrammer  
{  
   void WorkOnTask();  
}

public interface ILead  
{  
   void AssignTask();  
   void CreateSubTask();  
}

public class Programmer: IProgrammer  
{  
   public void WorkOnTask()  
   {  
      //code to implement to work on the Task.  
   }  
}  

public class Manager: ILead  
{  
   public void AssignTask()  
   {  
      //Code to assign a Task  
   }  
   public void CreateSubTask()  
   {  
   //Code to create a sub taks from a task.  
   }  
}

public class TeamLead: IProgrammer, ILead  
{  
   public void AssignTask()  
   {  
      //Code to assign a Task  
   }  
   public void CreateSubTask()  
   {  
      //Code to create a sub task from a task.  
   }  
   public void WorkOnTask()  
   {  
      //code to implement to work on the Task.  
   }  
}
```
## D: Dependency Inversion Principle
Entitas harus bergantung pada abstraksi, bukan pada implementasi(konkresi). Ini menyatakan bahwa modul tingkat tinggi tidak boleh bergantung pada modul tingkat rendah, tetapi harus bergantung pada abstraksi.

Contoh:
```C#
public class MySqlConnection {
    public Connection connect(){
        Connection conn = new Connection("MySQL");
        return conn;
    }
}

public class PasswordReminder{
    private Connection dbConnection;

    public PasswordReminder(MySQLConnection conn){
        dbConnection = conn.connect();
    }
}
```

MySqlConnection adalah _low-level module_ dan PasswordReminder adalah _high-level_. Hal diatas menyalahi prinsip karena PasswordReminder bergantung pada MySqlConnection (MySqlConnection adalah parameter required dalam constructor PasswordReminder).

PasswordReminder seharusnya tidak perlu tahu database apa yang digunakan. Untuk itu kita bisa menggunakan Interface, karena low-level maupun high-level seharusnya bergantung pada abstraksi.

```C#
public interface IDBConnection
{  
   Connection connect();
}

public class MySqlConnection: IDBConnection 
{
    public Connection connect(){
        Connection conn = new Connection("MySQL");
        return conn;
    }
}

public class MongoDbConnection: IDBConnection 
{
    public Connection connect(){
        Connection conn = new Connection("MongoDb");
        return conn;
    }
}

public class PasswordReminder{
    private Connection dbConnection;

    public PasswordReminder(IDBConnection conn){
        dbConnection = conn.connect();
    }
}
```

Hal ini memungkinkan PasswordReminder menerima **Connection** apapun selama dia menerapkan Interface IDBConnection, sehingga high-level dan low-level bergantung pada abstraksi.

Reference:
1. [c-sharpcorner](https://www.c-sharpcorner.com/UploadFile/damubetha/solid-principles-in-C-Sharp/)
2. [Digitalocean Community](https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design)


# When to Use
## Kapan Menggunakan Abstract Class atau Interface
### Abstract Class
- Tidak bisa di inisialisasi
- Dirancang untuk pewarisan sifat ke SubClass
- Method yg sudah dibuat di Abstract Class bisa langsung digunakan (apabila method konkrit) atau di _override_ (apabila method abstrak/tidak memiliki body)
- Abstract Class bisa memiliki Constructor
- Class hanya bisa mengimplementasi satu Abstract Class

Gunakan Abstract Class ketika:
Memiliki beberapa Concrete Method yang sama dan beberapa Abstract Method yang bisa di Override oleh Derived Class.
```C#
public abstract class Shape  
{  
   public abstract double Area();
   public string Hello(){
       return "Hello";
   }
} 

public class Rectangle: Shape  
{  
   public double Height {get;set;}  
   public double Width {get;set;}  
   public override double Area()  
   {  
      return Height * Width;  
   }  
}  
public class Circle: Shape  
{  
   public double Radius {get;set;}  
   public override double Area()  
   {  
      return Radius * Radus * Math.PI;  
   }  
}
```
Hal diatas menunjukkan bahwa setiap Class bangun datar bisa meng*override* fungsi **Area()** sesuai dengan caranya masing-masing dalam menghitung luas bidang, dan setiap Class bangun datar memiliki fungsi **Hello()** yang akan mengembalikan string "Hello".

### Interface
- Tidak bisa di inisialisasi
- Sebuah kontrak. Method yang ada di Interface harus diimplementasikan SELURUHNYA oleh Class yang mengimplementasikan Interface
- Hanya berisi deklarasi method, tidak memiliki body
- Hanya berisi deklarasi property, Tidak bisa memiliki data
- Class bisa mengimplementasi lebih dari satu interface

Gunakan Interface ketika:
Memastikan bahwa Class yang mengimplementasi Interface harus memiliki **semua Method dan Property yang dijanjikan**.
```C#
public interface IDBConnection
{  
   Connection connect();
}

public class MySqlConnection: IDBConnection 
{
    public Connection connect(){
        Connection conn = new Connection("MySQL");
        return conn;
    }
}

public class MongoDbConnection: IDBConnection 
{
    public Connection connect(){
        Connection conn = new Connection("MongoDb");
        return conn;
    }
}

public class PasswordReminder{
    private Connection dbConnection;

    public PasswordReminder(IDBConnection conn){
        dbConnection = conn.connect();
    }
}
```
Contoh diatas ketika PasswordReminder menggunakan IDBConnection sebagai required parameter di Constructornya. PasswordReminder butuh memastikan bahwa objek yang masuk didalam parameternya harus memiliki fungsi **connect()** yang akan mengembalikan objek **Connection**, maka dari itu dibuat Interface IDBConnection.

Referensi:
[InfoWorld](https://www.infoworld.com/article/2928719/when-to-use-an-abstract-class-vs-interface-in-csharp.html)

## Kapan Menggunakan Public, Private, Protected
- Public : member (Method ataupun Variabel) bisa diakses oleh code lain di Assembly yang sama
- Private : member hanya bisa diakses oleh code di dalam Class yang sama
- Protected : member bisa diakses oleh code di dalam Class yang sama, atau dari Derived Class.

Referensi:
[Microsoft Documentation](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)

## Kapan Mengunakan Static
### Static Class
Static Class akan di _load_ oleh .NET saat program yang memanggil Class tersebut dijalankan. Maka dari itu: 
- Static Class tidak bisa di inisialisasi
- Static Class tidak bisa mengakses apapun diluar class itu sendiri -- dengan kata lain, _Sealed_
- Tidak bisa memiliki Constructor

Static class hanya memproses input di argumen fungsinya (bila ada) dan mengembalikan output sebagai return.

```C#
public static class TemperatureConverter
{
    public static double CelsiusToFahrenheit(string temperatureCelsius)
    {
        double celsius = Double.Parse(temperatureCelsius);
        double fahrenheit = (celsius * 9 / 5) + 32;

        return fahrenheit;
    }

    public static double FahrenheitToCelsius(string temperatureFahrenheit)
    {
        double fahrenheit = Double.Parse(temperatureFahrenheit);
        double celsius = (fahrenheit - 32) * 5 / 9;

        return celsius;
    }
}

Console.WriteLine(TemperatureConverter.CelsiusToFahrenheit("35")); //return 95
Console.WriteLine(TemperatureConverter.FahrenheitToCelsius("75")); //return 23.889
```
### Static Member
- Static Member (seperti Variable) bisa dipanggil oleh Class lain meskipun Classnya belum dibuat
- Static Member dipanggil menggunakan Class Name, bukan Instance Name
- Static Member hanya ada satu buah, meskipun terdapat banyak instance yang memanggil
- Static Member digunakan untuk menyimpan value yang digunakan di banyak instance

```C#
public class Automobile
{
    public static int NumberOfWheels = 4;

    public static int SizeOfGasTank
    {
        get
        {
            return 15;
        }
    }

    public static void Drive() { }

    public static event EventType RunOutOfGas;

    // Other non-static fields and properties...
}

Automobile.Drive();
int i = Automobile.NumberOfWheels;
```

Referensi:
[Microsoft Documentation](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members)
