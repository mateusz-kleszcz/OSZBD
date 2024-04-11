
# Indeksy,  optymalizator <br>Lab 5

<!-- <style scoped>
 p,li {
    font-size: 12pt;
  }
</style>  -->

<!-- <style scoped>
 pre {
    font-size: 8pt;
  }
</style>  -->


---

**Jacek Budny, Mateusz Kleszcz**

--- 

Celem ćwiczenia jest zapoznanie się z planami wykonania zapytań (execution plans), oraz z budową i możliwością wykorzystaniem indeksów (cz. 2.)

Swoje odpowiedzi wpisuj w miejsca oznaczone jako:

---
> Wyniki: 

```sql
--  ...
```

---

Ważne/wymagane są komentarze.

Zamieść kod rozwiązania oraz zrzuty ekranu pokazujące wyniki, (dołącz kod rozwiązania w formie tekstowej/źródłowej)

Zwróć uwagę na formatowanie kodu

## Oprogramowanie - co jest potrzebne?

Do wykonania ćwiczenia potrzebne jest następujące oprogramowanie
- MS SQL Server,
- SSMS - SQL Server Management Studio    
- przykładowa baza danych AdventureWorks2017.
    
Oprogramowanie dostępne jest na przygotowanej maszynie wirtualnej


## Przygotowanie  

Uruchom Microsoft SQL Managment Studio.
    
Stwórz swoją bazę danych o nazwie XYZ. 

```sql
create database lab5  
go  
  
use lab5  
go
```


## Dokumentacja/Literatura

Obowiązkowo:

- [https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes)
- [https://docs.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide](https://docs.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide)
- [https://www.simple-talk.com/sql/performance/14-sql-server-indexing-questions-you-were-too-shy-to-ask/](https://www.simple-talk.com/sql/performance/14-sql-server-indexing-questions-you-were-too-shy-to-ask/)

Materiały rozszerzające:
- [https://www.sqlshack.com/sql-server-query-execution-plans-examples-select-statement/](https://www.sqlshack.com/sql-server-query-execution-plans-examples-select-statement/)

<div style="page-break-after: always;"></div>

# Zadanie 1 - Indeksy klastrowane I nieklastrowane

Skopiuj tabelę `Customer` do swojej bazy danych:

```sql
select * into customer from adventureworks2017.sales.customer
```

Wykonaj analizy zapytań:

```sql
select * from customer where storeid = 594  
  
select * from customer where storeid between 594 and 610
```

Zanotuj czas zapytania oraz jego koszt koszt:

---
> Wyniki: 
![[img/1-1.png | 500]]
![[img/1-2.png | 500]]
> Pierwsze zapytanie: 0.139158
> Drugie zapytanie: 0.139158


Dodaj indeks:

```sql
create  index customer_store_cls_idx on customer(storeid)
```

Jak zmienił się plan i czas? Czy jest możliwość optymalizacji?


---
> Wyniki: 
![[img/1-3.png | 500]]
![[img/1-5.png | 500]]
> Pierwsze zapytanie: 0.0065704
> Drugie zapytanie: 0.0507122

> Na planie widać dodatkową strukturę heap, plan jest bardziej skomplikowany od podstawowego. Można zauważyć nieznaczne obniżenie czasu i znaczne obniżenie kosztu. Warto zaznaczyć, że dla zapytań bez użycia indeksów, ich koszt był identyczny. Z kolei dla zapytań z użyciem indeksu nieklastrowego, drugie zapytanie miało znacznie większy koszt, co wynika z wielokrotnego odwoływania się do stworzonej struktury. Dalszą możliwością optymalizacji może być stworzenie indeksu klastrowego. 

Dodaj indeks klastrowany:

```sql
create clustered index customer_store_cls_idx on customer(storeid)
```

Czy zmienił się plan i czas? Skomentuj dwa podejścia w wyszukiwaniu krotek.

---
> Wyniki: 
![[img/1-7.png | 500]]
![[img/1-9.png | 500]]
> Pierwsze zapytanie: 0.0032831
> Drugie zapytanie: 0.0032996

> Koszt oraz czas wykonania znacznie zmniejszył się w stosunku do zastosowania indeksu nieklastrowego. Na planie również widać znaczne uproszczenie operacji. Nie ma dodatkowej struktury heap, jest tylko jedna operacja clustered index seek. Problemem jednak jest fakt, że możemy utworzyć tylko jedną taką strukturę na całą tabelę, dlatego indeks taki najlepiej nalożyć na często uzywane kolumny.


# Zadanie 2 – Indeksy zawierające dodatkowe atrybuty (dane z kolumn)

Celem zadania jest poznanie indeksów z przechowujących dodatkowe atrybuty (dane z kolumn)

Skopiuj tabelę `Person` do swojej bazy danych:

```sql
select businessentityid  
      ,persontype  
      ,namestyle  
      ,title  
      ,firstname  
      ,middlename  
      ,lastname  
      ,suffix  
      ,emailpromotion  
      ,rowguid  
      ,modifieddate  
into person  
from adventureworks2017.person.person
```
---

Wykonaj analizę planu dla trzech zapytań:

```sql
select * from [person] where lastname = 'Agbonile'  
  
select * from [person] where lastname = 'Agbonile' and firstname = 'Osarumwense'  
  
select * from [person] where firstname = 'Osarumwense'
```

Co można o nich powiedzieć?


---
> Wyniki: 
![[img/2-1.png | 500]]
![[img/2-2.png | 500]]
![[img/2-3.png | 500]]
> Kazde zapytanie ma taką samą strukturę i taki sam koszt.


Przygotuj indeks obejmujący te zapytania:

```sql
create index person_first_last_name_idx  
on person(lastname, firstname)
```

Sprawdź plan zapytania. Co się zmieniło?


---
> Wyniki: 
![[img/2-4.png | 500]]
![[img/2-5.png | 500]]
![[img/2-6.png | 500]]

> We wszystkich 3 zapytaniach pojawiła się pętla łącząca wyniki oraz skanowanie indeksów. Koszt wyraźnie zmalał.

Przeprowadź ponownie analizę zapytań tym razem dla parametrów: `FirstName = ‘Angela’` `LastName = ‘Price’`. (Trzy zapytania, różna kombinacja parametrów). 

Czym różni się ten plan od zapytania o `'Osarumwense Agbonile'` . Dlaczego tak jest?

---
> Wyniki: 
![[img/2-7.png | 500]]
![[img/2-8.png | 500]]
![[img/2-9.png | 500]]

> Tym razem róznica w wykonywanych operacjach oraz koszcie, pojawiła się tylko w przypadku drugiego zapytania. Wynika to z faktu, że w przypadku poprzedniego zapytania, mieliśmy tylko jeden zwracany rekord w każdym przypadku. W bazie istnieje jednak wiele osób o imieniu Angela lub nazwisku Price, co sprawia ze dane są mało specyficzne. W tym przypadku planer stwierdził, ze lepiej jest nie wykorzystywać utworzonego indeksu.


# Zadanie 3

Skopiuj tabelę `PurchaseOrderDetail` do swojej bazy danych:

```sql
select * into purchaseorderdetail from  adventureworks2017.purchasing.purchaseorderdetail
```

Wykonaj analizę zapytania:

```sql
select rejectedqty, ((rejectedqty/orderqty)*100) as rejectionrate, productid, duedate  
from purchaseorderdetail  
order by rejectedqty desc, productid asc
```

Która część zapytania ma największy koszt?

---
> Wyniki: 
![[img/3-1.png | 500]]

> Jak widać na załączonym diagramie, największy koszt generuje operacja sortowania danych, jest to az 87% całego kosztu zapytania. Koszt całego zapytania to 0.528317.

Jaki indeks można zastosować aby zoptymalizować koszt zapytania? Przygotuj polecenie tworzące index.


---

> Można zastosować indeks na kolumny, które są użyte podczas operacji sortowania. Ważne jest dodanie klauzuli include, w której zawarte są kolumny używane bezpośrednio w select. Bez nich indeks nie jest brany pod uwagę.

```sql
create index purchase1
on purchaseorderdetail (rejectedqty desc, productid asc) include (orderqty, duedate);
```

 Ponownie wykonaj analizę zapytania:

---
> Wyniki: 
![[img/3-2.png | 500]]
> Operacja skanowania i sortowania została zastąpiona przez pojedynczą operację Full Index Scan, a całkowity koszt wykonania zapytania wyniósł 0.0406, co jest znaczną poprawą w stosunku do braku indeksu (około 13 razy).

# Zadanie 4

Celem zadania jest porównanie indeksów zawierających wszystkie kolumny oraz indeksów przechowujących dodatkowe dane (dane z kolumn).

Skopiuj tabelę `Address` do swojej bazy danych:

```sql
select * into address from  adventureworks2017.person.address
```

W tej części będziemy analizować następujące zapytanie:

```sql
select addressline1, addressline2, city, stateprovinceid, postalcode  
from address  
where postalcode between n'98000' and n'99999'
```

```sql
create index address_postalcode_1  
on address (postalcode)  
include (addressline1, addressline2, city, stateprovinceid);  
go  
  
create index address_postalcode_2  
on address (postalcode, addressline1, addressline2, city, stateprovinceid);  
go
```


Czy jest widoczna różnica w zapytaniach? Jeśli tak to jaka? Aby wymusić użycie indeksu użyj `WITH(INDEX(Address_PostalCode_1))` po `FROM`:

> Wyniki: 
Bez uzycia indeksu
![[img/4-5.png | 500]]
![[img/4-1.png | 500]]
Z indeksem
![[img/4-4.png | 500]]
![[img/4-2.png | 500]]
> W przypadku zapytania z wykorzystaniem indeksów, kolumny są dodatkowo posortowane, według kolumny postalcode, adressline1, addresline2, city i stateprovinceid. Analizując plan zapytania mozemy zobaczyć, że Full Scan został zastąpiony Index Scanem.


Sprawdź rozmiar Indeksów:

```sql
select i.name as indexname, sum(s.used_page_count) * 8 as indexsizekb  
from sys.dm_db_partition_stats as s  
inner join sys.indexes as i on s.object_id = i.object_id and s.index_id = i.index_id  
where i.name = 'address_postalcode_1' or i.name = 'address_postalcode_2'  
group by i.name  
go
```


Który jest większy? Jak można skomentować te dwa podejścia do indeksowania? Które kolumny na to wpływają?


> Wyniki: 
![[img/4-3.png | 500]]

> Drugi indeks, jest nieznacznie większy. Wynika to z faktu, ze w przypadku pierwszego indeksu wykorzystana jest tylko kolumna postalcode, z kolei w drugim indeksie wszystkie. Jeżeli w zapytaniu będziemy używać wszystkich kolumn, to szybkość zrekompensuje koszty pamięciowe. W przypadku gdy np. w klauzuli WHERE wykorzystujemy w większości przypadków tylko postalcode, lepiej zastosować indeks pierwszy.


# Zadanie 5 – Indeksy z filtrami

Celem zadania jest poznanie indeksów z filtrami.

Skopiuj tabelę `BillOfMaterials` do swojej bazy danych:

```sql
select * into billofmaterials  
from adventureworks2017.production.billofmaterials
```


W tej części analizujemy zapytanie:

```sql
select productassemblyid, componentid, startdate  
from billofmaterials  
where enddate is not null  
    and componentid = 327  
    and startdate >= '2010-08-05'
```

Zastosuj indeks:

```sql
create nonclustered index billofmaterials_cond_idx  
    on billofmaterials (componentid, startdate)  
    where enddate is not null
```

Sprawdź czy działa. 

Przeanalizuj plan dla poniższego zapytania:

Czy indeks został użyty? Dlaczego?

> Wyniki: 
![[img/5-1.png | 500]]

> Indeks nie został użyty, co wynika z faktu, z małej róznorodności danych (dosyć często pojawiają się dane z takim samym componentid oraz startdate). Podobnie jak w zadaniu 2, wykorzystanie indeksu w tym przypadku nie powinno przyspieszyć zapytań.

Spróbuj wymusić indeks. Co się stało, dlaczego takie zachowanie?

```sql
select productassemblyid, componentid, startdate  
from billofmaterials WITH(INDEX (billofmaterials_cond_idx))
where enddate is not null  
    and componentid = 327  
    and startdate >= '2010-08-05'
```

![[img/5-2.png | 500]]

> Wyniki: 
> Po wymuszeniu indeksu koszt całego zapytania wzrósł 3 krotnie. Wynika to z opisanego wyżej problemu, dane często powtarzają się. Rozwiązaniem mogłoby być np. dodanie INCLUDE(productassemblyid)


---

Punktacja:

|         |     |
| ------- | --- |
| zadanie | pkt |
| 1       | 2   |
| 2       | 2   |
| 3       | 2   |
| 4       | 2   |
| 5       | 2   |
| razem   | 10  |
