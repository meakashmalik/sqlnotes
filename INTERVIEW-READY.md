# Kal interview + machine test — ready file

Aaj jo sawal aaye, wahi yahan **ek jagah**. Har topic: 30-second jawab → example → extra sawal.

Machine test me C# program **Console App** me type karna: `dotnet new console` ya Visual Studio → Console.

Neeche **sample data** + har query/program ka **output** hai. Interview me “result kya aayega?” isi se bolo.

---

## Sample data (SQL — yeh tables assume karo)

**tblcountry**

| cid | cname |
|-----|--------|
| 1 | India |
| 2 | Pakistan |
| 3 | usa |
| 4 | Brazil |

Brazil ka **koi user nahi**.

**tblusers**

| uid | name | gender | salary | country |
|-----|------|--------|--------|---------|
| 1 | alok | male | 13000 | 1 |
| 2 | monika | female | 15000 | 2 |
| 3 | javed | male | 11000 | 3 |
| 4 | seema | female | 17000 | 2 |
| 5 | madhu | female | 19000 | NULL |

`madhu` ki country nahi — LEFT/FULL me dikhegi, INNER me nahi.

**Users** (SELF JOIN)

| uid | uname | umanager |
|-----|--------|----------|
| 1 | alok | 3 |
| 2 | amit | 4 |
| 3 | javed | 1 |
| 4 | priya | 1 |

---

## Index

**SQL**
1. [JOINs](#1-sql-joins)
2. [UNION vs UNION ALL](#2-union-vs-union-all)
3. [VIEW](#3-view)
4. [GROUP BY](#4-group-by)

**C# / OOP / MVC**
5. [OOP](#5-oops--inheritance-abstraction-encapsulation)
6. [Interface](#6-interface)
7. [Abstract class](#7-abstract-class)
8. [Abstract method](#8-abstract-method)
9. [ViewData, ViewBag, TempData](#9-viewdata-viewbag-tempdata)
10. [Variables in C#](#10-variables-in-c)
11. [2 numbers SUM](#11-machine-test--2-numbers-sum)
12. [2 numbers SUBTRACT](#12-machine-test--2-numbers-subtract)
13. [1 to 10 print](#13-machine-test--1-to-10-print)

---

## 1. SQL JOINs

**30-second:** Do tables ko **matching column** se jodna. Users ki `country` = country table ka `cid`.

**Kyun:** user ke saath country **naam** chahiye, sirf number `1, 2, 3` nahi.

```sql
-- INNER JOIN = sirf jo dono me match
SELECT U.name, C.cname
FROM tblusers U
INNER JOIN tblcountry C ON U.country = C.cid;

-- LEFT  = saare users + match (na mile to country NULL)
SELECT * FROM tblusers U LEFT JOIN tblcountry C ON U.country = C.cid;

-- RIGHT = saari countries + match (Brazil ka user NULL)
SELECT * FROM tblusers U RIGHT JOIN tblcountry C ON U.country = C.cid;

-- FULL  = dono ke saare
SELECT * FROM tblusers U FULL JOIN tblcountry C ON U.country = C.cid;

-- CROSS = har user × har country
SELECT * FROM tblusers CROSS JOIN tblcountry;

-- SELF  = same table do baar (employee + manager)
SELECT E.uname AS Emp, M.uname AS Manager
FROM Users E
JOIN Users M ON M.uid = E.umanager;
```

**Output — INNER** (`name`, `cname`) — madhu nahi (country NULL):

| name | cname |
|------|--------|
| alok | India |
| monika | Pakistan |
| javed | usa |
| seema | Pakistan |

**Output — LEFT** (short): 4 matched + madhu with `cname = NULL`.

**Output — RIGHT:** 4 matched + **Brazil** with user columns `NULL`.

**Output — FULL:** madhu (no country) **aur** Brazil (no user).

**Output — CROSS:** 5 users × 4 countries = **20 rows**.

**Output — SELF JOIN:**

| Emp | Manager |
|-----|---------|
| alok | javed |
| amit | priya |
| javed | alok |
| priya | alok |

| Type | Bolna |
|------|--------|
| INNER | sirf common |
| LEFT | left ke saare |
| RIGHT | right ke saare |
| FULL | dono ke saare |
| CROSS | N × M |
| SELF | ek table, do alias |

**Follow-up:** unmatched users?

```sql
SELECT U.*
FROM tblusers U
LEFT JOIN tblcountry C ON U.country = C.cid
WHERE C.cid IS NULL;
```

**Output:**

| uid | name | gender | salary | country |
|-----|------|--------|--------|---------|
| 5 | madhu | female | 19000 | NULL |

**Machine test tip:** `ON` pe column names clear rakho. Same naam ho to `U.country = C.cid`. Alias `name1`, `name2`.

---

## 2. UNION vs UNION ALL

**30-second:** Do `SELECT` ke result **ek ke neeche** lana. Dono me **same number of columns** + compatible types.

| | UNION | UNION ALL |
|--|--------|-----------|
| Duplicates | **hataata** hai | **saare** rows rakhta hai |
| Speed | slow (sort/distinct) | fast |
| Kab | unique list chahiye | ginti / saari rows chahiye |

Pehle alag-alag:

`WHERE country = 1` → `alok`  
`WHERE country = 2` → `monika`, `seema`

```sql
-- India + Pakistan users ke naam (unique)
SELECT name FROM tblusers WHERE country = 1
UNION
SELECT name FROM tblusers WHERE country = 2;

-- saari rows, naam repeat ho to bhi
SELECT name FROM tblusers WHERE country = 1
UNION ALL
SELECT name FROM tblusers WHERE country = 2;
```

Yahan teeno naam alag hain, isliye UNION aur UNION ALL **same 3 rows**: alok, monika, seema.

Duplicate dikhane ke liye:

```sql
SELECT name FROM tblusers WHERE gender = 'male'
UNION
SELECT name FROM tblusers WHERE country = 1;

SELECT name FROM tblusers WHERE gender = 'male'
UNION ALL
SELECT name FROM tblusers WHERE country = 1;
```

Male = `alok`, `javed`. Country 1 = `alok`.

**Output UNION** (unique):

| name |
|------|
| alok |
| javed |

**Output UNION ALL** (alok do baar):

| name |
|------|
| alok |
| javed |
| alok |

Column count same:

```sql
-- ERROR: pehle 1 column, doosre 2
-- SELECT name FROM tblusers
-- UNION
-- SELECT name, salary FROM tblusers;

SELECT name, salary FROM tblusers WHERE country = 1
UNION ALL
SELECT name, salary FROM tblusers WHERE country = 2;
```

**JOIN vs UNION:** JOIN = columns **side by side** (row wider). UNION = rows **neeche** (zyada rows).

---

## 3. VIEW

**30-second:** saved `SELECT`. Virtual table — data copy nahi, query save. `SELECT * FROM view_name`.

**Kyun:** salary chhupana, lambi JOIN ko chhota naam, security.

```sql
CREATE VIEW vw_UserDetails
AS
SELECT
    U.uid,
    U.name AS UserName,
    U.gender,
    C.cname AS CountryName
FROM tblusers U
JOIN tblcountry C ON U.country = C.cid;

SELECT * FROM vw_UserDetails;
SELECT * FROM vw_UserDetails WHERE CountryName = 'India';

-- Output SELECT *  (salary nahi — hide):
-- uid | UserName | gender | CountryName
-- 1   | alok     | male   | India
-- 2   | monika   | female | Pakistan
-- 3   | javed    | male   | usa
-- 4   | seema    | female | Pakistan
-- madhu INNER JOIN me nahi

-- Output WHERE India:
-- 1 | alok | male | India

ALTER VIEW vw_UserDetails
AS
SELECT U.uid, U.name AS UserName, C.cname AS CountryName
FROM tblusers U
JOIN tblcountry C ON U.country = C.cid;

-- DROP VIEW vw_UserDetails;
```

| View | Table |
|------|--------|
| Query save | Data store |
| Parameters nahi (`WHERE` use) | |
| Simple view me kabhi UPDATE | Full CRUD table pe |

**Follow-up:** View vs SP? View = `SELECT` dikhana. SP = `EXEC`, parameters, INSERT/UPDATE.

---

## 4. GROUP BY

**30-second:** rows ko **teams** me baanto, phir `COUNT / SUM / AVG / MIN / MAX`.

**WHERE** = group se **pehle** rows filter.  
**HAVING** = group ke **baad** team filter.

```sql
SELECT gender, COUNT(*) AS Kitne
FROM tblusers
GROUP BY gender;

SELECT country, COUNT(*) AS Kitne, AVG(salary) AS AvgSal
FROM tblusers
GROUP BY country;

SELECT C.cname, COUNT(*) AS Kitne, AVG(U.salary) AS AvgSal
FROM tblusers U
JOIN tblcountry C ON U.country = C.cid
GROUP BY C.cname;

-- sirf woh gender jahan 3 se zyada
SELECT gender, COUNT(*) AS Kitne
FROM tblusers
GROUP BY gender
HAVING COUNT(*) > 3;
```

**Output — gender count** (5 users: 2 male, 3 female):

| gender | Kitne |
|--------|-------|
| male | 2 |
| female | 3 |

**Output — by country** (madhu country NULL — SQL Server me alag NULL group):

| country | Kitne | AvgSal |
|---------|-------|--------|
| 1 | 1 | 13000 |
| 2 | 2 | 16000 |
| 3 | 1 | 11000 |
| NULL | 1 | 19000 |

**Output — cname** (INNER, madhu nahi):

| cname | Kitne | AvgSal |
|--------|-------|--------|
| India | 1 | 13000 |
| Pakistan | 2 | 16000 |
| usa | 1 | 11000 |

**Output — HAVING COUNT(*) > 3:** empty (kisi gender me 3 se zyada nahi).  
Is data pe `HAVING COUNT(*) >= 3` → female, 3.

Order: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`

**Galat:** `SELECT name, COUNT(*) FROM tblusers GROUP BY gender;`  
`name` na GROUP BY me hai na aggregate me.

---

## 5. OOPS — inheritance, abstraction, encapsulation

**4 pillars (bolna):** Encapsulation, Abstraction, Inheritance, Polymorphism.

### Encapsulation

Data **hide** + controlled access. `private` field, `public` property.

```csharp
class Employee
{
    private int salary;

    public int Salary
    {
        get { return salary; }
        set
        {
            if (value >= 0)
                salary = value;
        }
    }
}

Employee e = new Employee();
e.Salary = 13000;   // OK
e.Salary = -5;      // reject
Console.WriteLine(e.Salary);
```

**Output:**

```text
13000
```

(`-5` set nahi hua.)

**Bolna:** `e.salary = -5` na ho sake.

### Abstraction

Zaroori API dikhao, andar ka kaam chhupao. `car.Start()` — spark/fuel nahi dikhte.

```csharp
class Car
{
    public void Start()
    {
        CheckFuel();
        Spark();
        Console.WriteLine("Started");
    }
    private void CheckFuel() { }
    private void Spark() { }
}

Car c = new Car();
c.Start();
```

**Output:**

```text
Started
```

(`CheckFuel` / `Spark` print nahi — chhupi hui.)

| Encapsulation | Abstraction |
|---------------|-------------|
| Data hide | Complexity hide |
| `private` + property | simple method / abstract / interface |

### Inheritance

Child `:` Parent — common code reuse. Manager **is-an** Employee.

```csharp
class Employee
{
    public string Name;
    public void Work() { Console.WriteLine(Name + " working"); }
}

class Manager : Employee
{
    public int Bonus;
    public void Lead() { Console.WriteLine(Name + " leads"); }
}

Manager m = new Manager();
m.Name = "alok";
m.Work();
m.Lead();
```

**Output:**

```text
alok working
alok leads
```

C#: **ek hi class** inherit. Kai contracts → **interface**.

### Polymorphism (agar poochhein)

Same call, alag object, alag kaam. `virtual` + `override`, ya interface.

```csharp
Payment p = new UpiPayment();
p.Pay(500);   // UPI wali body
```

---

## 6. Interface

**30-second:** **contract** — methods ka naam. Body class deti hai. Ek class **kai** interfaces le sakti hai.

**Kyun:** File log / DB log — caller `ILogger` use kare, implementation badlo to baaki code same.

```csharp
interface ILogger
{
    void Log(string msg);
}

class FileLogger : ILogger
{
    public void Log(string msg)
    {
        Console.WriteLine("File: " + msg);
    }
}

class DbLogger : ILogger
{
    public void Log(string msg)
    {
        Console.WriteLine("DB: " + msg);
    }
}

ILogger log = new FileLogger();
log.Log("saved");
log = new DbLogger();
log.Log("saved");
```

**Output:**

```text
File: saved
DB: saved
```

`new ILogger()` **nahi**. Prefix `I`.

---

## 7. Abstract class

**30-second:** incomplete class. `new Payment()` **error**. Shared code + force methods.

```csharp
abstract class Payment
{
    public abstract void Pay(int amount);   // child override

    public void Receipt()                   // common
    {
        Console.WriteLine("Receipt");
    }
}

class UpiPayment : Payment
{
    public override void Pay(int amount)
    {
        Console.WriteLine("UPI " + amount);
    }
}

Payment p = new UpiPayment();  // variable parent, object child
p.Pay(500);
p.Receipt();
```

**Output:**

```text
UPI 500
Receipt
```

`new Payment()` compile error.

---

## 8. Abstract method

**30-second:** sirf signature, **body nahi**. Child **`override` zaroori**. Sirf **abstract class** me.

```csharp
public abstract void Pay(int amt);
```

Normal class me abstract method **compile error**.

---

### Abstract class vs Interface (zaroor poochhenge)

| | Abstract class | Interface |
|--|----------------|-----------|
| `new` | nahi | nahi |
| Inherit | **ek** parent class | **kai** interfaces |
| Fields / constructor | haan | traditionally nahi |
| Shared code | aasan | C# 8+ default optional |
| Socho | is-a + common code | can-do / plugin |

```csharp
class WhatsApp : MessageService, INotify, IEncrypt { }
```

---

## 9. ViewData, ViewBag, TempData

ASP.NET MVC — Controller se **View** ko data.

| | ViewData | ViewBag | TempData |
|--|----------|---------|----------|
| Type | `ViewDataDictionary` (string key) | `dynamic` | dictionary + **next request** |
| Cast | haan `as string` | nahi (runtime) | haan |
| Life | **isi** request | isi request | **redirect** ke baad ek baar |
| Null | key miss = null | miss = null | read ke baad generally clear |

**ViewData**

```csharp
// Controller
public ActionResult Index()
{
    ViewData["Title"] = "Users";
    ViewData["Count"] = 10;
    return View();
}
```

```html
<!-- View -->
<h1>@ViewData["Title"]</h1>
<p>Count: @ViewData["Count"]</p>
```

**Browser output:**

```text
Users
Count: 10
```

**ViewBag** — wahi data, property jaisa (wrapper over ViewData).

```csharp
ViewBag.Title = "Users";
ViewBag.Count = 10;
```

```html
<h1>@ViewBag.Title</h1>
```

**TempData** — `RedirectToAction` ke baad message.

```csharp
public ActionResult Save()
{
    // save user...
    TempData["Msg"] = "User saved";
    return RedirectToAction("Index");
}

public ActionResult Index()
{
    ViewBag.Msg = TempData["Msg"];  // ek baar milta hai
    return View();
}
```

```html
@if (ViewBag.Msg != null)
{
    <p>@ViewBag.Msg</p>
}
```

Save ke baad Index page:

```text
User saved
```

Dobara refresh pe message **gayab** (TempData ek baar).

**Bolna:** same page → ViewBag/ViewData. Redirect / “saved successfully” → **TempData**. Session lamba; yeh short.

---

## 10. Variables in C#

**Variable** = naam diya hua box memory me.

```csharp
int age = 25;                 // value type
string name = "alok";         // reference, text
double salary = 13000.50;
bool ok = true;
int? country = null;          // nullable
var x = 10;                   // compiler type decide (int)
const int Max = 100;          // badal nahi sakte
```

| Category | Examples | Assign |
|----------|----------|--------|
| Value | `int`, `bool`, `struct` | **copy** |
| Reference | `class`, `array`, `string` | **address** |

```csharp
int a = 10;
int b = a;
b = 20;
Console.WriteLine(a);  // 10  copy
Console.WriteLine(b);  // 20

Employee e1 = new Employee();
e1.Name = "temp";
Employee e2 = e1;
e2.Name = "alok";
Console.WriteLine(e1.Name);  // alok — same object
```

**Output:**

```text
10
20
alok
```

Scope: method ke andar local. `public int Salary` = field. Property alag.

Machine test: type likho, `Convert.ToInt32(Console.ReadLine())` se input.

---

## 11. Machine test — 2 numbers SUM

**Poora program — yahi type karna.**

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Pehla number: ");
        int a = Convert.ToInt32(Console.ReadLine());

        Console.Write("Doosra number: ");
        int b = Convert.ToInt32(Console.ReadLine());

        int sum = a + b;
        Console.WriteLine("Sum = " + sum);
    }
}
```

**Screen (input 10 aur 20):**

```text
Pehla number: 10
Doosra number: 20
Sum = 30
```

Bina input (agar jaldi chahiye):

```csharp
int a = 10, b = 20;
Console.WriteLine(a + b);   // 30
```

Method se (agar class poochhein):

```csharp
static int Add(int x, int y)
{
    return x + y;
}
```

---

## 12. Machine test — 2 numbers SUBTRACT

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.Write("Pehla number: ");
        int a = Convert.ToInt32(Console.ReadLine());

        Console.Write("Doosra number: ");
        int b = Convert.ToInt32(Console.ReadLine());

        int diff = a - b;
        Console.WriteLine("Difference = " + diff);
    }
}
```

**Screen (input 50 aur 20):**

```text
Pehla number: 50
Doosra number: 20
Difference = 30
```

---

## 13. Machine test — 1 to 10 print

**for (sabse common)**

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 1; i <= 10; i++)
        {
            Console.WriteLine(i);
        }
    }
}
```

**Output:**

```text
1
2
3
4
5
6
7
8
9
10
```

`Console.Write(i + " ");` → `1 2 3 4 5 6 7 8 9 10`

**while** (agar poochhein)

```csharp
int i = 1;
while (i <= 10)
{
    Console.WriteLine(i);
    i++;
}
```

Ek line me: `Console.Write(i + " ");`

Ulta 10 to 1: `for (int i = 10; i >= 1; i--)`

---

## Kal subah 10 minute — ratta nahi, bolna

```text
JOIN     = do tables ON key; INNER common, LEFT saari left
UNION    = rows neeche, unique     UNION ALL = duplicates bhi
VIEW     = saved SELECT, virtual table
GROUP BY = teams + COUNT/SUM; WHERE pehle, HAVING baad
OOP      = encapsulate private+property; abstract Start();
           inherit Child : Parent; poly same call alag kaam
Interface = contract, kai OK, no new
Abstract  = no new, override zaroori, shared code OK
ViewBag   = isi request, dynamic
ViewData  = isi request, string key + cast
TempData  = redirect ke baad ek baar (message)
Variable  = int a = 5;  var / const
SUM       = a + b     SUB = a - b     1-10 = for i=1; i<=10; i++
```

SQL machine test tables: `tblusers` + `tblcountry` — JOIN + GROUP BY likh ke aana.

C# machine test: upar wale 3 programs **bina dekhe** type karo ek baar aaj raat.

Zyada depth: `SQL-INTERVIEW.md`, `csharp/CSHARP-INTERVIEW.md`.
