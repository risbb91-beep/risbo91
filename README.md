1. Nova baza i kopiranje podataka Kreirati novu bazu pod nazivom PrviKolokvijum2025. Iz restorovane baze kopirati podatke iz tabela iz šeme Purchasing 
u novokreiranu bazu PrviKolokvijum2025 koristeći SELECT INTO naredbe.

USE PrviKolokvijum2025;
GO
CREATE SCHEMA Purchasing;
GO

SELECT * INTO PrviKolokvijum2025.Purchasing.Vendor
FROM AdventureWorks2016.Purchasing.Vendor;

-- Kopiranje tabele PurchaseOrderHeader
SELECT * INTO PrviKolokvijum2025.Purchasing.PurchaseOrderHeader
FROM AdventureWorks2016.Purchasing.PurchaseOrderHeader;

-- Kopiranje tabele PurchaseOrderDetail
SELECT * INTO PrviKolokvijum2025.Purchasing.PurchaseOrderDetail
FROM AdventureWorks2016.Purchasing.PurchaseOrderDetail;

-- Kopiranje tabele ProductVendor
SELECT * INTO PrviKolokvijum2025.Purchasing.ProductVendor
FROM AdventureWorks2016.Purchasing.ProductVendor;

-- Kopiranje tabele ShipMethod
SELECT * INTO PrviKolokvijum2025.Purchasing.ShipMethod
FROM AdventureWorks2016.Purchasing.ShipMethod;
GO

2. U restorovanoj bazi kreirati view Sales.EmployeeSales koji prikazuje BusinessEntityID,
ime i prezime zaposlenog, naziv teritorije i SalesQuota gde je SalesQuota veći ili jednak od 300000. 
Tabele potrebne za upit: Sales.SalesPerson, Person.Person i Sales.SalesTerritory

  . Kreiranje nove baze i šeme
U Object Explorer-u (leva strana), desni klik na folder Databases -> New Database.
Upiši ime PrviKolokvijum2025 i klikni OK.
Proširi novu bazu, desni klik na folder Security -> Schemas -> New Schema.
Upiši Purchasing i klikni OK.

  . Kopiranje podataka (Import/Export Wizard)
Umesto SELECT INTO koda, možeš koristiti čarobnjak koji je sigurniji:
Desni klik na bazu AdventureWorks2016 -> Tasks -> Export Data....
Source: Izaberi "SQL Server Native Client" i bazu AdventureWorks2016.
Destination: Ponovo "SQL Server Native Client", ali izaberi tvoju novu bazu PrviKolokvijum2025.
Izaberi opciju "Copy data from one or more tables or views".
Na listi štikliraj samo tabele koje pored imena imaju [Purchasing].
Klikni Next do kraja i Finish.

  . Ažuriranje podataka (Edit Rows)
Ako ne želiš da pišeš UPDATE upit:
Nađi tabelu koju treba da menjaš u Object Explorer-u.
Desni klik na nju -> Edit Top 200 Rows.
Samo klikni u polje (kolonu) koju želiš da promeniš, upiši novu vrednost (1, 2 ili 3) i pritisni Enter na tastaturi. Promena je odmah sačuvana u bazi.

  . Kreiranje View-a (View Designer)
Ovo je vizuelni način da spojiš tabele:
U bazi AdventureWorks2016, desni klik na folder Views -> New View.
U prozoru koji se otvori, dodaj tabele: SalesPerson, Person i SalesTerritory.
Povezivanje: Ako SSMS nije sam povukao linije, klikni mišem na BusinessEntityID u jednoj tabeli i "prevuci" ga na isti taj ID u drugoj.
Štikliraj kolone: Izaberi BusinessEntityID, FirstName, LastName, Name (iz teritorije) i SalesQuota.
Filter: U koloni Filter pored SalesQuota upiši >= 300000.
Klikni na ikonicu diskete (Save) i nazovi view EmployeeSales.

  . Database Dijagram
Desni klik na Database Diagrams -> New Database Diagram.
Selektuj sve tabele iz jedne šeme (npr. Purchasing) držeći taster Shift.
Klikni Add, pa Close.
SSMS će sam rasporediti tabele i nacrtati relacije. Sačuvaj dijagram pod imenom te šeme.

3.	U restorovanoj bazi kreirati stored proceduru Sales.OrdersByCustomer koja prima parametre @CustomerID tipa INT I @MinTotal tipa Money I za 
zadatog kupca vraća njegove porudžbine gde je Subtotal veće ili jednako od @MinTotal. Tabela potrebna za upit: Sales.SalesOrderHeader

sql
CREATE PROCEDURE Sales.OrdersByCustomer
    @CustomerID INT,
    @MinTotal MONEY
AS
BEGIN
    SELECT 
        SalesOrderID, 
        OrderDate, 
        CustomerID, 
        SubTotal
    FROM Sales.SalesOrderHeader
    WHERE CustomerID = @CustomerID 
      AND SubTotal >= @MinTotal;
END;

4. Napraviti Cross Database upit koji kombinuje podatke iz restorovane baze i baze PrviKolokvijum2025. 
Upit prikazuje Ime i prezime zaposlenog, Način dostave i sumirani TotalDue grupisano po godini narudžbine i
ostalim neophodnim atributima. Upit ograničiti za osobe kojima je ime ’Linda’ i gde je sumirani TotalDue veći od 600000
Tabele potrebne za upit: PurchaseOrderHeader (iz baze PrviKolokvijum2025), ShipMethod (iz baze PrviKolokvijum2025), Person.Person (Iz restorovane baze)

SELECT 
    p.FirstName, 
    p.LastName, 
    sm.Name AS ShipMethodName,
    YEAR(poh.OrderDate) AS OrderYear,
    SUM(poh.TotalDue) AS TotalSum
FROM PrviKolokvijum2025.Purchasing.PurchaseOrderHeader AS poh
INNER JOIN PrviKolokvijum2025.Purchasing.ShipMethod AS sm 
    ON poh.ShipMethodID = sm.ShipMethodID
INNER JOIN AdventureWorks2016.Person.Person AS p 
    ON poh.EmployeeID = p.BusinessEntityID
WHERE p.FirstName = 'Linda'
GROUP BY 
    p.FirstName, 
    p.LastName, 
    sm.Name, 
    YEAR(poh.OrderDate)
HAVING SUM(poh.TotalDue) > 600000;

II kolok
1. Napraviti database dijagram koji će sadržati sve tabele iz baze.

   Pozicioniraj se: U Object Explorer-u (leva strana), proširi tvoju bazu (npr. PrviKolokvijum2025 ili AdventureWorks2016).
Pokreni dijagram: Desni klik na folder Database Diagrams -> New Database Diagram.
Ako te pita da li želiš da instaliraš podršku za dijagrame, klikni Yes.
Dodaj sve tabele:
Otvoriće se prozor Add Table.
Klikni na prvu tabelu na listi.
Drži taster Shift na tastaturi i klikni na poslednju tabelu na listi (tako ćeš selektovati sve).
Klikni na dugme Add.
Sačekaj: SQL Serveru će trebati par sekundi (ili minuta, ako ima stotine tabela) da ih sve iscrta i poveže linijama (relacijama).
Organizuj: Kada završi, tabele će verovatno biti razbacane. Možeš uraditi desni klik na prazan prostor u dijagramu i
izabrati Arrange Tables da ih SSMS automatski lepo složi.
Sačuvaj: Klikni na disketu (Save) u gornjem meniju i daj naziv dijagramu (npr. "KompletnaSema").

2. Dati su sledeći zahtevi na osnovu kojih je potrebno napraviti model skladišta podataka pomoću
   Visio programa (Visio fajl sa dijagramom sačuvati na lokaciji D:/MIS Jun 2022/ime-prezime.vsdx):
a.	Pronaći informaciju koje su to najčešće tarife u kojima se građani voze.
b.	Najpozivaniji dobavljač i prikazati kao Pie chart.
c.	Vožnje po mesecima I dobavljačima.

Pokreni Visio.
Izaberi kategoriju Software and Database (ili kucaj u pretrazi "Database").
Odaberi šablon Crow's Foot Database Notation (ovaj šablon koristi standardne "nožice" za relacije).
Klikni na Create (ili Metric Units pa Create).

  a. Kreiranje tabela Dimenzija (Sporedne tabele)
Iz palete sa leve strane (Shapes) prevuci oblik Entity na sredinu ekrana.
Tabela DimTarifa:
Klikni na naslov "Entity" i promeni u DimTarifa.
Dodaj kolone (Attributes): TarifaKey (stavi PK pored), NazivTarife, CenaPoKM.
Tabela DimDobavljac:
Prevuci novi Entity, nazovi ga DimDobavljac.
Kolone: DobavljacKey (PK), NazivDobavljaca, KontaktTelefon.
Tabela DimDatum:
Prevuci novi Entity, nazovi ga DimDatum.
Kolone: DatumKey (PK), Dan, Mesec, Godina, Kvartal.
  
  b. Kreiranje tabele Činjenica (Centralna tabela)
Ova tabela mora biti u sredini jer se sve ostale spajaju sa njom.
Tabela FactVoznje:
Prevuci Entity, nazovi ga FactVoznje.
Primarni ključ: VoznjaID (PK).
Strani ključevi (veza sa dimenzijama): TarifaKey, DobavljacKey, DatumKey.
Mere (Brojevi): BrojPoziva, UkupnaCena, TrajanjeVoznje.
  
  c. Povezivanje (Relacije)
Iz palete Shapes uzmi alat Relationship.
Pravilo: Uvek spajaš PK iz dimenzije sa FK u Fact tabeli.
Klikni na jedan kraj konektora i zakači ga za DimTarifa, a drugi kraj za FactVoznje.
Uradi isto za DimDobavljac i DimDatum.
Rezultat treba da izgleda kao zvezda (Star Schema).
  
  d. Dodavanje Pie Chart-a (Zahtev pod b)
Pošto Visio dozvoljava grafičke elemente:
U gornjem meniju idi na Insert -> Chart (ako tvoja verzija ima tu opciju) ili...
Sa leve strane u pretrazi oblika kucaj "Pie chart".
Prevuci kružni dijagram pored tabele DimDobavljac.
Dva puta klikni na njega da uneseš legende (npr. "Dobavljač A 40%", "Dobavljač B 60%").
  
  e. Čuvanje fajla (Tačna lokacija)
Idi na File -> Save As.
Klikni na Browse.
Navigiraj do D:/MIS Jun 2022/.
U polje za ime upiši tvoje ime-prezime.vsdx.
Klikni Save.
Pro savet za kolokvijum: Ako primetiš da ti se linije relacija "lome" neuredno, desni klik na liniju -> Straight Connector da bi dijagram izgledao profesionalnije.


3. Napraviti u okviru postojeće restorovane baze u SQL serveru virtuelne tabele koje će odgovarati modelu skladišta podataka

 Naravno, kreiranje virtuelnih tabela (View-ova) putem GUI-ja je zapravo vizuelno slaganje slagalice. Evo kako da to uradiš korak po korak u SQL Server Management Studio:
  a. Kreiranje virtuelne tabele činjenica (vFactVoznje)
U Object Explorer-u, proširi bazu, desni klik na folder Views -> New View....
U prozoru koji iskoči, dodaj tabelu koja sadrži glavne podatke (npr. PurchaseOrderHeader ako simuliramo nabavku/vožnje). Klikni Add, pa Close.
Štikliraj kolone: Izaberi PurchaseOrderID, VendorID, ShipMethodID, OrderDate i TotalDue.
Promena naziva (Aliasing): U donjoj tabeli (kolona Alias), pored originalnih imena upiši ona koja si planirao u modelu:
PurchaseOrderID -> VoznjaID
VendorID -> DobavljacKey
ShipMethodID -> TarifaKey
Dodavanje godine/meseca: U prazan red u koloni Column upiši YEAR(OrderDate) i daj mu Alias Godina.
Klikni na Save (ikonica diskete) i nazovi view vFactVoznje.
  
  b. Kreiranje virtuelne dimenzije (vDimDobavljac)
Ponovo desni klik na Views -> New View....
Dodaj tabelu Vendor.
Štikliraj BusinessEntityID (Alias: DobavljacKey) i Name (Alias: NazivDobavljaca).
Sačuvaj kao vDimDobavljac.
  
  c. Kreiranje virtuelne dimenzije (vDimTarifa)
Desni klik na Views -> New View....
Dodaj tabelu ShipMethod (koja nam služi kao izvor za tarife/načine dostave).
Štikliraj ShipMethodID (Alias: TarifaKey) i Name (Alias: NazivTarife).
Sačuvaj kao vDimTarifa.
  
  d.U Object Explorer-u, desni klik na folder Views -> New View....
Dodaj tabelu koja ima datume (npr. SalesOrderHeader ili PurchaseOrderHeader). Klikni Add, pa Close.
U donjoj tabeli (gde piše Column, Alias, Table...), ručno unesi sledeće redove:
Column	Alias	Objašnjenje
OrderDate	DatumKey	Osnovna kolona za povezivanje
DAY(OrderDate)	Dan	Izvlači dan (1-31)
MONTH(OrderDate)	Mesec	Izvlači redni broj meseca
YEAR(OrderDate)	Godina	Izvlači godinu (npr. 2022)
DATEPART(qq, OrderDate)	Kvartal	Izvlači kvartal (1-4)
Ukloni duplikate: Pošto jedan datum može imati više narudžbina, desni klik na prazan prostor u gornjem delu dizajnera -> Properties. U listi pronađi stavku Distinct Values i postavi je na Yes.
Klikni na Save i nazovi view vDimDatum

Prednost GUI pristupa:
Dok biraš kolone, u donjem delu ekrana SSMS automatski piše SQL kod umesto tebe. Ako pogrešiš, samo odštikliraj kolonu ili promeni alias u tabeli.
Kako da ih vidiš u dijagramu?
Kada odeš na Database Diagrams -> New Diagram, možeš dodati ove View-ove isto kao i obične tabele (samo klikni na tab Views u prozoru za dodavanje).
 Tako ćeš vizuelno potvrditi da tvoje virtuelne tabele čine Star Schema koju si nacrtao u Visio-u.

4. Pokrenuti PowerBI, povezati se na prethodno restorovanu bazu, odnosno na njene virtuelne tabele i kreirati izveštaje za zahteve opisane u zadatku 3

Povezivanje na bazu
Otvori Power BI Desktop.
Klikni na Get Data -> SQL Server.
U polje Server upiši naziv svog servera (obično . ili localhost ili naziv tvog računara).
U polje Database upiši naziv restorovane baze.
U prozoru koji se otvori, nemoj birati obične tabele, već pronađi i štikliraj tvoje virtuelne tabele (View-ove): 
vFactVoznje, vDimDobavljac, vDimTarifa i vDimDatum.
Klikni Load.
  
  2. Kreiranje izveštaja (Zahtevi iz zadatka 3)
 
 a) Najčešće tarife u kojima se građani voze
U panelu Visualizations (desno), odaberi Clustered Column Chart (stubični dijagram).
Iz vDimTarifa prevuci NazivTarife u polje X-axis.
Iz vFactVoznje prevuci VoznjaID u polje Y-axis.
Klikni na strelicu pored VoznjaID u polju Y-axis i proveri da li je izabrano Count (da bi izbrojao vožnje po tarifi).
 
 b) Najpozivaniji dobavljač (Pie Chart)
Klikni na prazan prostor na strani, pa u panelu Visualizations odaberi Pie Chart.
Iz vDimDobavljac prevuci NazivDobavljaca u polje Legend.
Iz vFactVoznje prevuci BrojPoziva (ili VoznjaID) u polje Values.
Power BI će automatski prikazati procente koji dobavljač ima najviše poziva.
 
 c) Vožnje po mesecima i dobavljačima
Odaberi Line Chart (linijski dijagram) ili Stacked Column Chart.
Iz vDimDatum prevuci Mesec (ili DatumKey) u polje X-axis.
Iz vDimDobavljac prevuci NazivDobavljaca u polje Legend (ovo će napraviti posebnu liniju za svakog dobavljača).
Iz vFactVoznje prevuci VoznjaID u polje Y-axis (podesi na Count).
  
  3. Sređivanje modela (Relationship view)
Ako grafikoni ne prikazuju tačne podatke, klikni na ikonicu Model view (skroz levo, treća ikonica odozgo):
Proveri da li su View-ovi povezani. Ako nisu, prevuci DobavljacKey iz vDimDobavljac na DobavljacKey u vFactVoznje.
Uradi isto za TarifaKey i DatumKey.


*** dopuna : Pozicionirati se u restorovanu bazu i u tabeli XXX ažurirati sledeće podatke
a.	Kolona A na 1
b.	Kolona B na 2
c.	Kolona C na 3  u gui?

U Object Explorer-u (sa leve strane), proširi tvoju bazu.
Proširi folder Tables.
Pronađi tabelu koju treba da ažuriraš (tvoja tabela XXX).
Korak 2: Otvaranje režima za izmenu (Edit)
Desni klik na tu tabelu.
Izaberi opciju Edit Top 200 Rows.
Napomena: Ako tabela ima više od 200 redova, a ti treba da nađeš neki specifičan, klikni na ikonicu "SQL" na gornjoj traci, dopiši WHERE uslov i klikni Execute.
Korak 3: Ažuriranje ćelija
Sada ćeš videti tabelu kao mrežicu (grid).
Pronađi red koji želiš da izmeniš.
Klikni u ćeliju ispod Kolone A i prosto upiši 1.
Klikni u ćeliju ispod Kolone B i upiši 2.
Klikni u ćeliju ispod Kolone C i upiši 3.
Korak 4: Čuvanje izmena
U SSMS-u ne postoji dugme "Save" za ove izmene. One se čuvaju automatski:
Kada završiš sa kucanjem u jednom redu, samo pritisni taster Enter ili klikni u bilo koji drugi red.
Ako vidiš crvenu ikonicu uzvičnika pored reda dok kucaš, to znači da izmena još nije potvrđena u bazi. 
Kada klikneš van tog reda, ona nestaje – što znači da je podatak uspešno upisan.
Važno: Ako ti se pojavi greška prilikom klika na drugi red, to znači da vrednosti koje si uneo (1, 2, 3) 
nisu usklađene sa tipom podataka te kolone (npr. pokušavaš da upišeš broj u polje koje prihvata samo slova).


   

