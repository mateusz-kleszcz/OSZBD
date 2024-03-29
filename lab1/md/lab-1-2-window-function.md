
# SQL - Funkcje okna (Window functions) 

# Lab 1-2

---
**Imię i nazwisko:**
Jacek Budny, Mateusz Kleszcz

--- 


Celem ćwiczenia jest zapoznanie się z działaniem funkcji okna (window functions) w SQL, analiza wydajności zapytań i porównanie z rozwiązaniami przy wykorzystaniu "tradycyjnych" konstrukcji SQL

Swoje odpowiedzi wpisuj w miejsca oznaczone jako:

```sql
-- wyniki ...
```

Ważne/wymagane są komentarze.

Zamieść kod rozwiązania oraz zrzuty ekranu pokazujące wyniki, (dołącz kod rozwiązania w formie tekstowej/źródłowej)

Zwróć uwagę na formatowanie kodu

---

## Oprogramowanie - co jest potrzebne?

Do wykonania ćwiczenia potrzebne jest następujące oprogramowanie:
- MS SQL Server - wersja 2019, 2022
- PostgreSQL - wersja 15/16
- SQLite
- Narzędzia do komunikacji z bazą danych
	- SSMS - Microsoft SQL Managment Studio
	- DtataGrip lub DBeaver
-  Przykładowa baza Northwind
	- W wersji dla każdego z wymienionych serwerów

Oprogramowanie dostępne jest na przygotowanej maszynie wirtualnej

## Dokumentacja/Literatura

- Kathi Kellenberger,  Clayton Groom, Ed Pollack, Expert T-SQL Window Functions in SQL Server 2019, Apres 2019
- Itzik Ben-Gan, T-SQL Window Functions: For Data Analysis and Beyond, Microsoft 2020

- Kilka linków do materiałów które mogą być pomocne
	 - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-ver16
	- https://www.sqlservertutorial.net/sql-server-window-functions/
	- https://www.sqlshack.com/use-window-functions-sql-server/
	- https://www.postgresql.org/docs/current/tutorial-window.html
	- https://www.postgresqltutorial.com/postgresql-window-function/
	-  https://www.sqlite.org/windowfunctions.html
	- https://www.sqlitetutorial.net/sqlite-window-functions/

- Ikonki używane w graficznej prezentacji planu zapytania w SSMS opisane są tutaj:
	- [https://docs.microsoft.com/en-us/sql/relational-databases/showplan-logical-and-physical-operators-reference](https://docs.microsoft.com/en-us/sql/relational-databases/showplan-logical-and-physical-operators-reference)

---
# Zadanie 1 - obserwacja

Wykonaj i porównaj wyniki następujących poleceń.

```sql
select avg(unitprice) avgprice
from products p;

select avg(unitprice) over () as avgprice
from products p;

select categoryid, avg(unitprice) avgprice
from products ppro
group by categoryid

select avg(unitprice) over (partition by categoryid) as avgprice
from products p;
```

Jaka jest są podobieństwa, jakie różnice pomiędzy grupowaniem danych a działaniem funkcji okna?

```
Grupowanie danych zwraca zagregowane wyniki, podczas gdy funkcje okna zwracają wyniki osobno dla
wszystkich kolumn. Czas wykonania zapytania dla funkcji okna jak i dla grupowania danych jest 
zbliżony. 
```

---
# Zadanie 2 - obserwacja

Wykonaj i porównaj wyniki następujących poleceń.

```sql
--1)

select p.productid, p.ProductName, p.unitprice,
       (select avg(unitprice) from products) as avgprice
from products p
where productid < 10

--2)
select p.productid, p.ProductName, p.unitprice,
       avg(unitprice) over () as avgprice
from products p
where productid < 10
```


Jaka jest różnica? Czego dotyczy warunek w każdym z przypadków?

```
Funkcja okna wykonuje się po klauzuli where, dlatego zwraca średnią cenę produktów jedynie 
z id mniejszym niż 10. Jeżeli użyjemy podzaptania, to średnia zostanie policzona z całej tabeli, 
a następnie zostaną wyświetlone produkty z id mniejszym niż 10.
```

Napisz polecenie równoważne 
- 1) z wykorzystaniem funkcji okna. Napisz polecenie równoważne 
```sql
select top 9 p.productid, p.ProductName, p.unitprice,
      avg(unitprice) over () as avgprice
from products p
order by ProductID
```
- 2) z wykorzystaniem podzapytania
```sql
select p.productid, p.ProductName, p.unitprice,
       (select avg(unitprice) from products where productid < 10) as avgprice
from products p
where productid < 10
```

# Zadanie 3

Baza: Northwind, tabela: products

Napisz polecenie, które zwraca: id produktu, nazwę produktu, cenę produktu, średnią cenę wszystkich produktów.

Napisz polecenie z wykorzystaniem z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

W SSMS włącz dwie opcje: Include Actual Execution Plan oraz Include Live Query Statistics

![w:700](_img/window-1.png)

W DataGrip użyj opcji Explain Plan/Explain Analyze

![w:700](_img/window-2.png)


![w:700](_img/window-3.png)


podzapytanie - 88ms - 115ms
```sql
select p.ProductID, p.ProductName, p.UnitPrice,
    (select avg(UnitPrice) from products) as avgprice
from products p
```
join - 110ms - 156ms
```sql
select p.ProductID, p.ProductName, p.UnitPrice,
    (select avg(UnitPrice) from products) as avgprice
from products p
cross join (select avg(UnitPrice) as avgprice from products) as prod
```
okna - 82ms - 112ms
```sql
select p.ProductID, p.ProductName, p.UnitPrice, avg(unitprice) over () as avgprice
from products p;
```

```
Powyższe zapytania testowaliśmy w obrębie MS SQL Server.
Czasy wykonania poszczególnych zapytań różnią się na tyle nieznacznie, że może być to błąd pomiaru.
Największe czasy uzyskujemy przy użyciu zapytania z JOIN'em. 
Jest to też zapytanie najbardziej skomplikowane do napisania.
```

```
Przetestowaliśmy również działanie funkcji okna w 3 SZBD.
Czasy różniły się znacznie: dla MS SQL Server uzyskaliśmy czas 130ms, w Postgressie 80ms 
i w SQLite 68ms.
Po przeanalizowaniu zapytań za pomocą "Explain Plain" w poszczególnych systemach, 
zauważyliśmy na diagramie, że MS SQL Server wykonuje znacznie więcej operacji w ramach 
wywołania funkcji (m. in. ma operację INNER JOIN, skanowanie indeksów i agregację), 
dlatego może być najwolniejsza.
```


# Zadanie 4

Baza: Northwind, tabela products

Napisz polecenie, które zwraca: id produktu, nazwę produktu, cenę produktu, średnią cenę produktów w kategorii, do której należy dany produkt. Wyświetl tylko pozycje (produkty) których cena jest większa niż średnia cena.

Napisz polecenie z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj zapytania. Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
podzapytanie -79ms
select p.ProductID, p.ProductName, p.UnitPrice, 
    (select avg(unitprice) from Products where p.CategoryID = CategoryID) as avgprice
from Products p 
where p.UnitPrice > (select avg(unitprice) from Products where p.CategoryID = CategoryID)
```
join - 74ms
```sql
SELECT p.productid, p.ProductName, p.unitprice, avgprice
FROM products p
LEFT JOIN (SELECT CategoryID, avg(unitprice) AS avgprice FROM products group by CategoryID) AS prod
ON prod.CategoryID = p.CategoryID
WHERE p.UnitPrice > avgprice
```
okna - 82ms
```sql
with t as (
    select p.ProductID, p.ProductName, p.UnitPrice, 
    avg(unitprice) over (partition by CategoryID) as avgprice from products p
) select * from t where t.UnitPrice > t.avgprice
```

```
Wszystkie serwery wykonały podzapytania w identycznym czasie. Porównanie czasu dla tak małych danych
nie ma większego sensu. Każda z funkcji wymagała użycia sumarycznie przynajmniej 2 zapytań, 
co wynika z tego, że warunek, który sprawdzamy, był zawarty w klauzuli WHERE, która wykona się 
jako pierwsza.
Tym razem nie zaobserwowaliśmy różnic w czasie pomiędzy różnymi SZBD 
```

---
# Zadanie 5 - przygotowanie

Baza: Northwind

Tabela products zawiera tylko 77 wiersz. Warto zaobserwować działanie na większym zbiorze danych.

Wygeneruj tabelę zawierającą kilka milionów (kilkaset tys.) wierszy

Stwórz tabelę o następującej strukturze:

Skrypt dla SQL Srerver

```sql
create table product_history(
   id int identity(1,1) not null,
   productid int,
   productname varchar(40) not null,
   supplierid int null,
   categoryid int null,
   quantityperunit varchar(20) null,
   unitprice decimal(10,2) null,
   quantity int,
   value decimal(10,2),
   date date,
 constraint pk_product_history primary key clustered
    (id asc )
)
```

Wygeneruj przykładowe dane:

Dla 30000 iteracji, tabela będzie zawierała nieco ponad 2mln wierszy (dostostu ograniczenie do możliwości swojego komputera)

Skrypt dla SQL Srerver

```sql
declare @i int  
set @i = 1  
while @i <= 30000  
begin  
    insert product_history  
    select productid, ProductName, SupplierID, CategoryID,   
         QuantityPerUnit,round(RAND()*unitprice + 10,2),  
         cast(RAND() * productid + 10 as int), 0,  
         dateadd(day, @i, '1940-01-01')  
    from products  
    set @i = @i + 1;  
end;  
  
update product_history  
set value = unitprice * quantity  
where 1=1;
```


Skrypt dla Postgresql

```sql
create table product_history(
   id int generated always as identity not null  
       constraint pkproduct_history
            primary key,
   productid int,
   productname varchar(40) not null,
   supplierid int null,
   categoryid int null,
   quantityperunit varchar(20) null,
   unitprice decimal(10,2) null,
   quantity int,
   value decimal(10,2),
   date date
);
```

Wygeneruj przykładowe dane:

Skrypt dla Postgresql

```sql
do $$  
begin  
  for cnt in 1..30000 loop  
    insert into product_history(productid, productname, supplierid,   
           categoryid, quantityperunit,  
           unitprice, quantity, value, date)  
    select productid, productname, supplierid, categoryid,   
           quantityperunit,  
           round((random()*unitprice + 10)::numeric,2),  
           cast(random() * productid + 10 as int), 0,  
           cast('1940-01-01' as date) + cnt  
    from products;  
  end loop;  
end; $$;  
  
update product_history  
set value = unitprice * quantity  
where 1=1;
```


Wykonaj polecenia: `select count(*) from product_history`,  potwierdzające wykonanie zadania

```sql
Dla MS SQL Servera - 1s414ms
Dla Postgres - 354ms
```

---
# Zadanie 6

Baza: Northwind, tabela product_history

To samo co w zadaniu 3, ale dla większego zbioru danych

Napisz polecenie, które zwraca: id pozycji, id produktu, nazwę produktu, cenę produktu, średnią cenę produktów w kategorii do której należy dany produkt. Wyświetl tylko pozycje (produkty) których cena jest większa niż średnia cena.

Napisz polecenie z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj zapytania. Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

- Podzapytanie
```sql
select p.id, p.ProductID, p.ProductName, p.UnitPrice,
       (select avg(UnitPrice) from product_history) as avgprice
from product_history p
```
![w:700](_img/screen1.png)

- join
```sql
select p.id, p.ProductID, p.ProductName, p.UnitPrice,
    (select avg(UnitPrice) from product_history) as avgprice
from product_history p
    cross join (select avg(UnitPrice) as avgprice from product_history) as prod
```
![w:700](_img/screen2.png)

- okna
```sql
select p.id, p.ProductID, p.ProductName, p.UnitPrice, avg(unitprice) over () as avgprice
from product_history p;
```
![w:700](_img/screen3.png)

```
Zapytanie przy użyciu podzapytań MS SQL Server - 1s911ms
Zapytanie przy użyciu podzapytań Postgresa - 120ms
Zapytanie przy użyciu podzapytań SQLite - 72ms

Zapytanie przy użyciu join MS SQL Server - 1s976ms
Zapytanie przy użyciu join Postgresa - 1s650ms
Zapytanie przy użyciu join SQLite - 776ms

Zapytanie przy użyciu funkcji okna MS SQL Server - 1s833ms
Zapytanie przy użyciu funkcji okna Postgresa - 1s429ms
Zapytanie przy użyciu funkcji okna SQLite - 1s755ms

Podobnie jak w poprzednich zadaniach, widać zachowaną regułę, według której MS SQL Server radzi sobie 
najwolniej (szczególnie widoczne jest to w przypadku podzapytań), a SQLite najlepiej. Warto jednak 
zauważyć, że w przypadku funkcji okna, niezależnie od zastosowanego SZBD uzyskaliśmy prawie zawsze 
czas ok. 1.5s czyli znacznie wolniej niż w przypadku podzapytań dla Postgrea i SQLite'a. Może to 
oznaczać, że używanie funkcji okna nie sprawdzi się w przypadku danych, których nie grupujemy.
```


---
# Zadanie 7

Baza: Northwind, tabela product_history

Lekka modyfikacja poprzedniego zadania

Napisz polecenie, które zwraca: id pozycji, id produktu, nazwę produktu, cenę produktu oraz
-  średnią cenę produktów w kategorii do której należy dany produkt.
-  łączną wartość sprzedaży produktów danej kategorii (suma dla pola value)
-  średnią cenę danego produktu w roku którego dotyczy dana pozycja
- łączną wartość sprzedaży produktów danej kategorii (suma dla pola value)

Napisz polecenie z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj zapytania. W przypadku funkcji okna spróbuj użyć klauzuli WINDOW.

Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

- podzapytania, na postgresie trzeba zamienić year na extract(year from ...)
```sql
select p.id, p.ProductID, p.ProductName, p.UnitPrice,
    (select avg(UnitPrice) from product_history where CategoryID = p.CategoryID) as avgprice,
    (select sum(UnitPrice) from product_history where CategoryID = p.CategoryID) as sumprice,
    (select avg(UnitPrice) from product_history 
        where ProductID = p.ProductID and year(date) = year(p.date)) as yearavgprice
from product_history p
```
- join
```sql
select p.id, p.ProductID, p.ProductName, p.UnitPrice,
    avg(p2.UnitPrice) as avgprice,
    sum(p2.UnitPrice)as avgprice,
    avg(p3.UnitPrice) as avgYear
from product_history p
    join product_history p2 on p2.CategoryID = p.CategoryID
    join product_history p3 on p3.productid = p.productid and year(p.date) = year(p3.date)
group by p.id, p.ProductID, p.ProductName, p.UnitPrice
```
- okna
```sql
select p.id, p.ProductID, p.ProductName, p.UnitPrice,
    avg(unitprice) over (partition by CategoryID) as avgprice,
    sum(unitprice) over (partition by CategoryID) as sumprice,
    avg(unitprice) over (partition by ProductID, year(p.date)) as yearavgprice
from product_history p
```
![w:700](_img/screen7-3.png)

```
Zapytanie przy użyciu podzapytań MS SQL Server - 3s226ms
Zapytanie przy użyciu podzapytań Postgresa - nie dało się wykonać w sensownym czasie
Zapytanie przy użyciu podzapytań SQLite - nie dało się wykonać w sensownym czasie

Zapytanie przy użyciu joina nie dało się przetworzyć na żadnym systemie 

Zapytanie przy użyciu funkcji okna MS SQL Server - 3s331ms
Zapytanie przy użyciu funkcji okna Postgresa - 7s82ms
Zapytanie przy użyciu funkcji okna SQLite - 5s453ms

Dzięki powyższym wynikom jesteśmy w statnie zauważyć przewagę systemu MS SQL Server nad innymi. 
Pomimo, że system ten sprawdzał się gorzej w przypadku mniej złożonych zapytań, tak w tym przypadku 
jako jedyny poradził sobie np. z podzapytaniami. Sprawdzał się on też najszybciej ze wszystkich 
systemów.
Funkcje okna, choć trochę wolniejsze od podzapytań, okazały się najbardziej niezawodnym, a przy tym 
łatwym do napisania sposobem.
Operacja z joinem jest już zbyt skomplikowana, aby za jej pomocą przetwarzać zapytania.
```

---
# Zadanie 8 - obserwacja

Funkcje rankingu, `row_number()`, `rank()`, `dense_rank()`

Wykonaj polecenie, zaobserwuj wynik. Porównaj funkcje row_number(), rank(), dense_rank()

```sql 
select productid, productname, unitprice, categoryid,  
    row_number() over(partition by categoryid order by unitprice desc) as rowno,  
    rank() over(partition by categoryid order by unitprice desc) as rankprice,  
    dense_rank() over(partition by categoryid order by unitprice desc) as denserankprice  
from products;
```

```sql
row_number - numer wiersza
rank - ranking elementów, z uwzględnieniem remisów (1, 2, 2, 4)
dense_rank - raknking elementów, bez uwzględniania remisów (1, 2, 2, 3)
```

Zadanie

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna

```sql
select productid, productname, unitprice, categoryid,
       (select count(*) from products p1
            where p.CategoryID = p1.CategoryID
              and (p.UnitPrice < p1.UnitPrice 
              or (p.UnitPrice = p1.UnitPrice and p.ProductID > p1.ProductID)))
                + 1 as myrowno,
       (select count(*) from products p1
            where p1.CategoryID = p.CategoryID
              and p.UnitPrice < p1.UnitPrice)
           + 1 as myrankprice,
       (select count(*) from
            (select distinct UnitPrice 
             from products p1 
             where p1.CategoryID = p.CategoryID and p.UnitPrice < p1.UnitPrice) as less)
                + 1 as mydenserankprice
from products p order by CategoryID, UnitPrice desc, ProductID;
```
![w:700](_img/screen8.png)

---
# Zadanie 9

Baza: Northwind, tabela product_history

Dla każdego produktu, podaj 4 najwyższe ceny tego produktu w danym roku. Zbiór wynikowy powinien zawierać:
- rok
- id produktu
- nazwę produktu
- cenę
- datę (datę uzyskania przez produkt takiej ceny)
- pozycję w rankingu

Uporządkuj wynik wg roku, nr produktu, pozycji w rankingu

```sql
with t as (
    select year(ph.Date) as year, p.ProductID, p.ProductName, ph.UnitPrice,
        rank() over (partition by p.ProductID, year(ph.Date)
            order by ph.UnitPrice desc) as rank
    from Products p
        join product_history ph on ph.ProductID = p.ProductID
)
select * from t
where rank <= 4
order by Year, ProductID, rank;
```

![w:700](_img/screen9-1.png)

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)


```sql
with t as (
    select year(ph.Date) as year, p.ProductID, p.ProductName, ph.UnitPrice,
           (select top 4 count (*) + 1
            from product_history ph1
            where p.ProductID = ph1.productid 
                and year(ph.date) = year(ph1.date) 
                and ph1.unitprice > ph.unitprice
            ) as rank
    from Products p
             join product_history ph on ph.ProductID = p.ProductID
)
select * from t
where rank <= 4
order by Year, ProductID, rank;
```

```
Po 30 krotnym :) zmniejszeniu tabeli wynikowej udało nam się uzyskać wynik 6s264ms dla MS SQL Servera 
(dla pozostałych SZBD nie udało się uzyskać wyniku nawet wtedy). Funkcje okna poradziy sobie z 
podanym zadaniem w 136ms. Oprócz tego rozwiązanie z funkcjami okna jest zdecydowanie dużo prostsze do 
napisania.
```

---
# Zadanie 10 - obserwacja

Funkcje `lag()`, `lead()`

Wykonaj polecenia, zaobserwuj wynik. Jak działają funkcje `lag()`, `lead()`

```sql
select productid, productname, categoryid, date, unitprice,  
       lag(unitprice) over (partition by productid order by date)   
as previousprodprice,  
       lead(unitprice) over (partition by productid order by date)   
as nextprodprice  
from product_history  
where productid = 1 and year(date) = 2022  
order by date;  
  
with t as (select productid, productname, categoryid, date, unitprice,  
                  lag(unitprice) over (partition by productid   
order by date) as previousprodprice,  
                  lead(unitprice) over (partition by productid   
order by date) as nextprodprice  
           from product_history  
           )  
select * from t  
where productid = 1 and year(date) = 2022  
order by date;
```

```
Lag pokazuje wartość poprzedniego rekordu, lead następnego.

W drugim zapytaniu where zadziała dopiero po wykonaniu with i dlatego będziemy mieli wartość dla 
pierwszego produktu, w przypadku pierwszego zapytania where wykona się od razu i pierwszy rekord 
będzie miał wartość null. Obydwa zapytania zwracają null dla ostatniego rekordu.
```


Zadanie

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
select ph.productid, ph.productname, ph.categoryid, ph.date, ph.unitprice,
    (select ph1.unitprice
    from product_history ph1
    where ph1.productid = ph.productid and ph1.date = dateadd(day, -1, ph.date)
    ) as previousprodprice,
    (select ph1.unitprice
     from product_history ph1
     where ph1.productid = ph.productid and ph1.date = dateadd(day, 1, ph.date)
     ) as nextprodprice
from product_history ph
where ph.productid = 1 and year(ph.date) = 2022
order by ph.date;
```

![w:700](_img/screen10.png)

```
Zapytanie z funkcją okna wykonywało się 276ms, a zapytanie z selectem 9s217ms. Dodatkowo, 
wykorzystaliśmy tutaj fakt, że w kolumnie date znajdują się daty co 1 dzień, jeżeli którejś daty by 
brakowało to zapytanie nie zadziałałoby. Rozwiązaniem mogłoby być użycie funkcji okna row_number (ale 
jeżeli zakładamy że ich nie używamy, to musielibyśmy zrobić selecta z countem, co prawdopodobnie 
sprawiłoby że nie uzyskalibyśmy wyników w sensownym czasie)
Dla Postgresa i SQLitea zapytanie się nie wykonało nawet na zmniejszonym zbiorze wynikowym.
```

---
# Zadanie 11

Baza: Northwind, tabele customers, orders, order details

Napisz polecenie które wyświetla inf. o zamówieniach

Zbiór wynikowy powinien zawierać:
- nazwę klienta, nr zamówienia,
- datę zamówienia,
- wartość zamówienia (wraz z opłatą za przesyłkę),
- nr poprzedniego zamówienia danego klienta,
- datę poprzedniego zamówienia danego klienta,
- wartość poprzedniego zamówienia danego klienta.

```sql
with t as(
    select C.ContactName, O.OrderID, O.OrderDate, 
           sum(OD.UnitPrice * OD.Quantity * (1 - OD.Discount)) + O.Freight as OrderValue,
           lag(O.OrderID) over ( partition by C.CustomerID order by O.OrderDate) as PrevOrderID,
           lag(O.OrderDate) over ( partition by C.CustomerID order by O.OrderDate) as PrevOrderDate
    from Orders O
    join [Order Details] OD
    on O.OrderID = OD.OrderID
    join Customers C
    on O.CustomerID = C.CustomerID
    group by O.OrderID, O.Freight, O.OrderDate, C.CustomerID, C.ContactName)
select t.*, PD.OrderValue
from t
        left join (select O.OrderID, 
                        sum(OD.UnitPrice * OD.Quantity * (1 - OD.Discount)) + O.Freight as OrderValue
                   from Orders O
                   join dbo.[Order Details] OD
                   on O.OrderID = OD.OrderID
                   group by O.OrderID, O.Freight) as PD
        on t.PrevOrderID = PD.OrderID
```

![w:700](_img/ex11_result.png)

`Do uzyskania dostęp do danych o poprzednim zamówieniu świetnie nadaje się funkcja lag()`

---
# Zadanie 12 - obserwacja

Funkcje `first_value()`, `last_value()`

Wykonaj polecenia, zaobserwuj wynik. Jak działają funkcje `first_value()`, `last_value()`. Skomentuj uzyskane wyniki. Czy funkcja `first_value` pokazuje w tym przypadku najdroższy produkt w danej kategorii, czy funkcja `last_value()` pokazuje najtańszy produkt? Co jest przyczyną takiego działania funkcji `last_value`. Co trzeba zmienić żeby funkcja last_value pokazywała najtańszy produkt w danej kategorii

```sql
select productid, productname, unitprice, categoryid,  
    first_value(productname) over (partition by categoryid   
order by unitprice desc) first,  
    last_value(productname) over (partition by categoryid   
order by unitprice desc) last  
from products  
order by categoryid, unitprice desc;
```
```
Można zauważyć że zapytanie w tej formie nie pokazuje poprawnie najtańszego produktu w danej 
kategorii. Wynika to z faktu że funkcje last_value(), oraz first_value() domyślnie wykorzystują 
zakres RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW, co w naszym przypadku powoduje niepoprawne 
wyświetlanie produktu najtańszego w danej kategorii. Poniżej poprawiona wersja zapytania:
```

```sql
select productid, productname, unitprice, categoryid,
       first_value(productname) over (partition by categoryid
           order by unitprice desc) first,
       last_value(productname) over (partition by categoryid
           order by unitprice desc RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) last
from products
order by categoryid, unitprice desc;
```

Zadanie

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
select productid, productname, unitprice, categoryid,
    (select top 1 ProductName 
     from Products p2 
     where p2.CategoryID = p.CategoryID 
     order by UnitPrice desc) as last,
    (select top 1 ProductName 
     from Products p2 
     where p2.CategoryID = p.CategoryID 
     order by UnitPrice) as first
from products p
order by categoryid, unitprice desc;
```

![w:700](_img/ex12_no_window_result.png)
`Plan zapytania z funkcją okna na serwerze PostgreSql`
![w:700](_img/ex12_psql_window_plan.png)
`Plan zapytania z funkcją okna na serwerze MS SQL`
![w:700](_img/ex12_sql_window_plan.png)
`Plan zapytania bez funkcji okna na serwerze MS SQL`
![w:700](_img/ex12_psql_window_plan.png)

```
Porównując plany zapytań ciężko dojść do wniosku które zapytanie jest bardziej optymalne. Różnica 
zauważalna jest podczas porównania kosztów zapytań w wersji z funkcją okna jest to 0.0173 a w wersji 
bez funkcji okna 0.254. Widać więc że wykorzystanie funkcji okna jest dobrym wyborem, zmniejszającym 
koszt wykonania zapytania.
```

---
# Zadanie 13

Baza: Northwind, tabele orders, order details

Napisz polecenie które wyświetla inf. o zamówieniach

Zbiór wynikowy powinien zawierać:
- Id klienta,
- nr zamówienia,
- datę zamówienia,
- wartość zamówienia (wraz z opłatą za przesyłkę),
- dane zamówienia klienta o najniższej wartości w danym miesiącu
	- nr zamówienia o najniższej wartości w danym miesiącu
	- datę tego zamówienia
	- wartość tego zamówienia
- dane zamówienia klienta o najwyższej wartości w danym miesiącu
	- nr zamówienia o najniższej wartości w danym miesiącu
	- datę tego zamówienia
	- wartość tego zamówienia

```sql
with t as
        (select C.CustomerID, O.OrderID, O.OrderDate, 
            sum(OD.UnitPrice * OD.Quantity * (1 - OD.Discount)) + O.Freight as OrderValue
        from Orders O
        join [Order Details] OD
        on O.OrderID = OD.OrderID
        join Customers C
        on O.CustomerID = C.CustomerID
        group by C.CustomerID, O.OrderID, O.OrderDate, O.Freight)
select *,
       first_value(concat(t.OrderID, ' ', t.OrderDate, ' ', t.OrderValue)) 
        over (partition by t.CustomerID, month(t.OrderDate) order by t.OrderValue) as MinValueOrder,
       first_value(concat(t.OrderID, ' ', t.OrderDate, ' ', t.OrderValue)) 
        over (partition by t.CustomerID, month(t.OrderDate) order by t.OrderValue desc) as MaxValueOrder
from t
```
![w:700](_img/ex13_result.png)

`W celu skrócenia zapytania dwukrotnie wykorzystano funkcję first_value() zmieniając kolejność segregowanie`

---
# Zadanie 14

Baza: Northwind, tabela product_history

Napisz polecenie które pokaże wartość sprzedaży każdego produktu narastająco od początku każdego miesiąca. Użyj funkcji okna

Zbiór wynikowy powinien zawierać:
- id pozycji
- id produktu
- datę
- wartość sprzedaży produktu w danym dniu
- wartość sprzedaży produktu narastające od początku miesiąca

```sql
with t as 
        (select PH.id, PH.productid, PH.date, PH.value
        from product_history PH)
select t.*, sum(t.value) 
                over(partition by t.productid, year(t.date), month(t.date) order by t.date) as ascVal
from t
```

![w:700](_img/ex14_window_result.png)

Spróbuj wykonać zadanie bez użycia funkcji okna. Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
select PH.id, PH.productid, PH.date, PH.value, 
    (select sum(PH2.value) 
    from product_history PH2 
    where PH2.date <= PH.date 
        and  PH2.productid = PH.productid 
        and year(PH2.date) = year(PH.date) 
        and month(PH2.date) = month(PH.date))
from product_history PH
order by PH.productid, PH.date
```

```
Zapytanie bez funkcji okna wykonuje się w znacznie dłuższym czasie. Różnicę widać w koszcie wykonania 
zapytań, 8112 bez wykorzystania funkcji okna oraz 26 wykorzystując funkcję okna. Pokazuje to 
przydatność tego mechanizmu. 
```

![w:700](_img/ex14_no_window_result.png)

```
Porównując plany zapytań można dojść do dwóch wniosków.
- Plany nie różnią się znacznie między różnymi serwerami.
- Zapytania z funkcją okna mają znacznie prostrzy plan, w porównaniu do zapytań bez funkcji okna.

Czas wykonania zapytań był najkrótszy na serwerze MS SQL. Dodatkowo 
```
`Plan zapytania z funkcją okna na serwerze PostgreSql`
![w:700](_img/ex14_psql_window_plan.png)
`Plan zapytania z funkcją okna na serwerze MS SQL`
![w:700](_img/ex14_sql_window_plan.png)
`Plan zapytania bez funkcji okna na serwerze MS SQL`
![w:700](_img/ex14_sql_no_window_plan.png)

---
# Zadanie 15

Wykonaj kilka "własnych" przykładowych analiz. Czy są jeszcze jakieś ciekawe/przydatne funkcje okna (z których nie korzystałeś w ćwiczeniu)? Spróbuj ich użyć w zaprezentowanych przykładach.

```sql
select
    c.contactname, o.ORDERID, o.ORDERDATE,
    (SUM(od.unitprice * od.quantity * (1 - od.discount)) + o.freight) as OrderValue,
    ntile(4) over (
        partition by YEAR(o.ORDERDATE), MONTH(o.ORDERDATE) 
        order by (sum(od.unitprice * od.quantity * (1 - od.discount)) + o.freight)) as Quartile
from
    orders o
        join
    customers c on o.CUSTOMERID = c.CUSTOMERID
        join
    [order details] od on o.ORDERID = od.ORDERID
group by
    o.ORDERID, o.ORDERDATE, o.FREIGHT, c.CONTACTNAME
```

```
Zapytanie to zwraca nazwę klienta, numer zamówienia, datę zamówienia, wartość zamówienia oraz 
kwartyl, w którym znajduje się wartość zamówienia dla danego miesiąca
```
![w:700](_img/ex15_1.png)


```sql
with t as (
    select
        o.CustomerID,
        c.ContactName,
        o.OrderID,
        (sum(od.UnitPrice * od.Quantity * (1 - od.Discount)) + o.Freight) as OrderValue,
        row_number() over (partition by o.CustomerID order by o.OrderDate desc ) as OrderRank
    from
        orders o
    join
        [order details] od on o.OrderID = od.OrderID
    join Customers c on o.CustomerID = c.CustomerID
    group by
        o.CustomerID, c.ContactName, o.OrderID, o.Freight, o.OrderDate
)
select t.CustomerID, t.ContactName, t.OrderID, t.OrderValue
from t
where OrderRank = 1;
```

```
Wykorzystujemy funkcję okna ROW_NUMBER() do przypisania kolejności zamówień dla każdego klienta na 
podstawie daty zamówienia, sortując malejąco po dacie zamówienia, a następnie wybieramy tylko te 
zamówienia, które mają wartość 1 w kolumnie OrderRank, co oznacza ostatnie zamówienie dla każdego 
klienta.
```

![w:700](_img/ex15_2.png)

Punktacja

|         |     |
| ------- | --- |
| zadanie | pkt |
| 1       | 0,5 |
| 2       | 0,5 |
| 3       | 1   |
| 4       | 1   |
| 5       | 0,5 |
| 6       | 2   |
| 7       | 2   |
| 8       | 0,5 |
| 9       | 2   |
| 10      | 1   |
| 11      | 2   |
| 12      | 1   |
| 13      | 2   |
| 14      | 2   |
| 15      | 2   |
| razem   | 20  |
