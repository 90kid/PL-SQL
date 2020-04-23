--------------------------------------------------------------------------------
------------------------------------- DROP -------------------------------------
--------------------------------------------------------------------------------
DROP VIEW KartaOcen        CASCADE CONSTRAINTS;
DROP VIEW PlanZajec        CASCADE CONSTRAINTS;
DROP VIEW ListaBudynkow    CASCADE CONSTRAINTS;
DROP VIEW ListaWykladowcow CASCADE CONSTRAINTS;
DROP VIEW ListaStudentow   CASCADE CONSTRAINTS;

DROP TABLE Ocena        CASCADE CONSTRAINTS;
DROP TABLE Zajecia      CASCADE CONSTRAINTS;
DROP TABLE Charakter    CASCADE CONSTRAINTS;
DROP TABLE Przedmiot    CASCADE CONSTRAINTS;

DROP TABLE Budynek      CASCADE CONSTRAINTS;
DROP TABLE Sala         CASCADE CONSTRAINTS;

DROP TABLE TytulNaukowy CASCADE CONSTRAINTS;
DROP TABLE Wykladowca   CASCADE CONSTRAINTS;

DROP TABLE Student      CASCADE CONSTRAINTS;
DROP TABLE Grupa        CASCADE CONSTRAINTS;
DROP TABLE Kierunek     CASCADE CONSTRAINTS;

DROP TABLE Adres        CASCADE CONSTRAINTS;
commit;

--------------------------------------------------------------------------------
--------------------------------- CREATE TABLE ---------------------------------
--------------------------------------------------------------------------------

CREATE TABLE Adres(
  ID_Adres        number(4) NOT NULL,
  Ulica           varchar(30),
  NrBudynku       number(4),
  NrLokalu        number(4),
  KodPocztowy     varchar(6),
  Miasto          varchar(20),
  
  CONSTRAINT PK_Adres PRIMARY KEY (ID_Adres)
);


--------------------------------------------------------------------------------
--Wykladowcy

CREATE TABLE TytulNaukowy(
  ID_Tytul        number(4) NOT NULL,
  Nazwa           varchar(25),
  
  CONSTRAINT PK_Tytul PRIMARY KEY (ID_TYTUL)
);

CREATE TABLE Wykladowca(
  ID_Wykladowca   number(4) NOT NULL,
  Imie            varchar(20),
  Nazwisko        varchar(30),
  ID_Adres        number(4),
  ID_Tytul        number(4),    
  
  CONSTRAINT PK_Wykladowca PRIMARY KEY (ID_Wykladowca),
  CONSTRAINT FK_AdresW     FOREIGN KEY (ID_Adres) REFERENCES Adres(ID_Adres),
  CONSTRAINT FK_Tytul      FOREIGN KEY (ID_Tytul) REFERENCES TytulNaukowy(ID_Tytul)
);


--------------------------------------------------------------------------------
--Studenci

CREATE TABLE Kierunek(
  ID_Kierunek     number(4) NOT NULL,
  Nazwa           varchar(40),
  
  CONSTRAINT PK_Kierunek PRIMARY KEY (ID_Kierunek)
);

CREATE TABLE Grupa(
  ID_Grupa        number(4) NOT NULL,
  Nazwa           varchar(10),
  ID_Kierunek     number(4),
  
  
  CONSTRAINT PK_Grupa    PRIMARY KEY (ID_Grupa),
  CONSTRAINT FK_Kierunek FOREIGN KEY (ID_Kierunek) REFERENCES Kierunek(ID_Kierunek)
);

CREATE TABLE Student(
  ID_Student      number(4) NOT NULL,
  Imie            varchar(20),
  Nazwisko        varchar(30),
  ID_Adres        number(4),
  NrAlbumu        number(10), 
  ID_Grupa        number(4),
  
  CONSTRAINT PK_Student  PRIMARY KEY (ID_Student),
  CONSTRAINT FK_AdresS   FOREIGN KEY (ID_Adres) REFERENCES Adres(ID_Adres),
  CONSTRAINT FK_GrupaS   FOREIGN KEY (ID_Grupa) REFERENCES Grupa(ID_Grupa),
  UNIQUE (NrAlbumu)
);


--------------------------------------------------------------------------------
--Sale

CREATE TABLE Budynek(
  ID_Budynek      number(4) NOT NULL,
  Nazwa           varchar(50),
  ID_Adres        number(4),
  
  CONSTRAINT PK_Budynek  PRIMARY KEY (ID_Budynek),
  CONSTRAINT FK_AdresB   FOREIGN KEY (ID_Adres) REFERENCES Adres(ID_Adres)
);

CREATE TABLE Sala(
  ID_Sala         number(4) NOT NULL,
  ID_Budynek      number(4),
  KodSali         varchar(10),
  
  CONSTRAINT PK_Sala    PRIMARY KEY (ID_Sala),
  CONSTRAINT FK_Budyenk FOREIGN KEY (ID_Budynek) REFERENCES Budynek(ID_Budynek)
);


--------------------------------------------------------------------------------
--Przedmioty

CREATE TABLE Przedmiot(
  ID_Przedmiot    number(4) NOT NULL,
  Nazwa           varchar(50),
  Ects            number(1),
  
  CONSTRAINT PK_Przedmiot PRIMARY KEY (ID_Przedmiot)
);

CREATE TABLE Charakter(
  ID_Charakter    number(4) NOT NULL,
  Nazwa           varchar(20),

  CONSTRAINT PK_Charakter PRIMARY KEY (ID_Charakter)  
);


--------------------------------------------------------------------------------
--Zajecia

CREATE TABLE Zajecia(
  ID_Zajecia      number(4) NOT NULL,
  ID_Przedmiot    number(4),
  ID_Charakter    number(4),
  ID_Sala         number(4),
  ID_Wykladowca   number(4),
  ID_Grupa        number(4),
  DzienTyg        varchar(12),

  CONSTRAINT PK_Zajecia      PRIMARY KEY (ID_Zajecia),
  CONSTRAINT FK_PrzedmiotZ   FOREIGN KEY (ID_Przedmiot)  REFERENCES Przedmiot(ID_Przedmiot),
  CONSTRAINT FK_CharakterZ   FOREIGN KEY (ID_Charakter)  REFERENCES Charakter(ID_Charakter),
  CONSTRAINT FK_Sala         FOREIGN KEY (ID_Sala)       REFERENCES Sala(ID_Sala),
  CONSTRAINT FK_WykladowcaZ  FOREIGN KEY (ID_Wykladowca) REFERENCES Wykladowca(ID_Wykladowca),
  CONSTRAINT FK_Grupa        FOREIGN KEY (ID_Grupa)      REFERENCES Grupa(ID_Grupa)
);

CREATE TABLE Ocena(
  ID_Ocena        number(4) NOT NULL,
  ID_Student      number(4),
  ID_Zajecia      number(4),
  Ocena           number(4,1),
  Data            date,          
  
  CONSTRAINT PK_Ocena       PRIMARY KEY (ID_Ocena),
  CONSTRAINT FK_Student     FOREIGN KEY (ID_Student)    REFERENCES Student(ID_Student),
  CONSTRAINT FK_Zajecia     FOREIGN KEY (ID_Zajecia)    REFERENCES Zajecia(ID_Zajecia)
);
commit; 

--------------------------------------------------------------------------------
------------------------------------ VIEWS -------------------------------------
--------------------------------------------------------------------------------
CREATE VIEW KartaOcen (NrAlbumu, Nazwisko, Imie, Grupa, Kierunek, Przedmiot, Charakter, ECTS, Prowadzacy, Ocena, Data) AS
  SELECT Student.NrAlbumu, Student.Nazwisko, Student.Imie,
         Grupa.Nazwa, Kierunek.Nazwa,
         Przedmiot.Nazwa, Przedmiot.ECTS, Charakter.Nazwa,
         Wykladowca.Nazwisko, Ocena.Ocena, Ocena.Data
    FROM Ocena, Zajecia, Student, Przedmiot, Charakter, Wykladowca, Grupa, Kierunek
    WHERE Ocena.ID_Zajecia = Zajecia.ID_Zajecia
          AND Ocena.ID_Student = Student.ID_Student
          AND Zajecia.ID_Grupa = Grupa.ID_Grupa
          AND Grupa.ID_Kierunek = Kierunek.ID_Kierunek
          AND Student.ID_Grupa = Grupa.ID_Grupa
          AND Zajecia.ID_Przedmiot = Przedmiot.ID_Przedmiot
          AND Zajecia.ID_Charakter = Charakter.ID_Charakter
          AND Zajecia.ID_Wykladowca= Wykladowca.ID_Wykladowca
    ORDER BY Kierunek.Nazwa, Grupa.Nazwa, Student.Nazwisko, Przedmiot.Nazwa;
      
      
CREATE VIEW ListaBudynkow (Nazwa, Ulica, NrBudynku, NrLokalu, KodPocztowy, Miasto) AS
  SELECT Budynek.Nazwa, Adres.Ulica, Adres.NrBudynku, Adres.NrLokalu, Adres.KodPocztowy, Adres.Miasto
  FROM Budynek, Adres
  WHERE Budynek.ID_Adres = Adres.ID_Adres;
  

CREATE VIEW ListaWykladowcow (Nazwisko, Imie, TytulNaukowy, Ulica, NrBudynku, NrLokalu, KodPocztowy, Miasto) AS
  SELECT Wykladowca.Nazwisko, Wykladowca.Imie,
         TytulNaukowy.Nazwa,
         Adres.Ulica, Adres.NrBudynku, Adres.NrLokalu, Adres.KodPocztowy, Adres.Miasto
  FROM Wykladowca, TytulNaukowy, Adres
  WHERE Wykladowca.ID_Tytul = TytulNaukowy.ID_Tytul
        AND Wykladowca.ID_Adres = Adres.ID_Adres
  ORDER BY Wykladowca.Nazwisko;


CREATE VIEW ListaStudentow (NrAlbumu, Nazwisko, Imie, Ulica, NrBudynku, NrLokalu, KodPocztowy, Miasto, Grupa, Kierunek) AS
  SELECT Student.NrAlbumu, Student.Nazwisko, Student.Imie, 
         Adres.Ulica, Adres.NrBudynku, Adres.NrLokalu, Adres.KodPocztowy, Adres.Miasto,
         Grupa.Nazwa, Kierunek.Nazwa 
  FROM  Student, Adres, Grupa, Kierunek
  WHERE Student.ID_Grupa = Grupa.ID_Grupa 
        AND Grupa.ID_Kierunek = Kierunek.ID_Kierunek
        AND Student.ID_Adres = Adres.ID_Adres
  ORDER BY Student.Nazwisko;
  
  
CREATE VIEW PlanZajec (DzienTyg, Przedmiot, Charakter, Prowadzacy, Grupa, Kierunek, Sala, Budynek ) AS
  SELECT Zajecia.DzienTyg, Przedmiot.Nazwa, Charakter.Nazwa, 
       Wykladowca.Nazwisko, Grupa.Nazwa, Kierunek.Nazwa, 
       Sala.KodSali, Budynek.Nazwa
  FROM Zajecia, Przedmiot, Charakter, Sala, Wykladowca, Grupa, Kierunek, Budynek
  WHERE Zajecia.Id_Przedmiot = Przedmiot.Id_Przedmiot
        AND Zajecia.ID_Charakter = Charakter.ID_Charakter
        AND Zajecia.ID_Sala = Sala.Id_Sala
        AND Sala.ID_Budynek = Budynek.Id_Budynek
        AND Zajecia.ID_Wykladowca = Wykladowca.ID_Wykladowca
        AND Zajecia.Id_Grupa = Grupa.ID_Grupa
        AND Grupa.Id_Kierunek = Kierunek.Id_Kierunek
  ORDER BY Kierunek.Nazwa, Grupa.Nazwa, Zajecia.Dzientyg, Charakter.Nazwa;