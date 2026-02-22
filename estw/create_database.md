<button>ESTW</button>

<small>[Entwicklung](../develop.md) / [ESTW](estw.md) / [Files](files.md) / create_database.sql</small>

# create_database.sql

-- estw2 Datenbank
-- Verschlussplanprinzip 

-- äöüÄÖÜß

create database if not exists estw;
use estw;

create table projekte (
id int not null,
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id)
) engine=innodb default charset=latin1 row_format=default;
create unique index index1 on projekte (bezeichnung);

create table elementtypen (
id int not null,
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id)
) engine=innodb default charset=latin1 row_format=default;
create unique index index2 on elementtypen (bezeichnung);

create table elemente (
id int not null auto_increment,
projekt_id int not null,
typ_id int not null,
unterelementart int not null default 1,
hauptelement_id int default 0 comment 'Ist ein Zusatzelement, wenn nicht 0',
bezeichnung varchar(50) not null,
beschreibung varchar(254),
PZSP ENUM('N','J') comment 'Zielsperre',
PR ENUM('N','J') comment 'Rangierstraßen zulässig',
PZ ENUM('N','J') comment 'Zugstraßen zulässig',
PSSL ENUM('N','J') comment 'Stellsperre Links',
PSSR ENUM('N','J') comment 'Stellsperre Rechts',
PRS ENUM('N','J') comment 'Rangierstraßenstart zulässig',
PRZ ENUM('N','J') comment 'Rangierstraßenziel zulässig',
PZS ENUM('N','J') comment 'Zugstraßenstart zulässig',
PZZ ENUM('N','J') comment 'Zugstraßenziel zulässig',
lupe1x int not null comment 'Lupe 1: X-Koordinate',
lupe1y int not null comment 'Lupe 1: Y-Koordinate',
rotation int not null default 0 comment 'Rotation in 90° Schritten (0=0°, 1=90°, 2=180°, 3=270°)',
mirror ENUM('N','J') not null default 'N' comment 'Spiegeln',
primary key (id),
check (rotation IN (0, 1, 2, 3)),
foreign key (projekt_id) references projekte (id) on update cascade,
foreign key (typ_id) references elementtypen (id) on update cascade,
foreign key (hauptelement_id) references elemente (id)  -- nur ids erlaubt, die in elemente.id existieren
) engine=innodb default charset=latin1 row_format=default;
create unique index index3 on elemente (projekt_id,typ_id,bezeichnung);
create unique index index4 on elemente (projekt_id,lupe1x,lupe1y);

create table fahrstrassen (
id int not null auto_increment,
projekt_id int not null,
typ_id int not null default 1 comment 'Typ der Fahrstraße, 1 = Rangierstraße, 2 = Zugfahrstraße',
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id),
check (typ_id IN (1, 2)),
foreign key (projekt_id) references projekte (id) on update cascade
) engine=innodb default charset=latin1 row_format=default;
create unique index index5 on fahrstrassen (projekt_id, bezeichnung);

create table fahrstrassenelementtypen (
id int not null,
bezeichnung varchar(50) not null,
beschreibung varchar(254),
primary key (id)
) engine=innodb default charset=latin1 row_format=default;
create unique index index6 on fahrstrassenelementtypen (bezeichnung);

create table fahrstrassenelemente (
id int not null auto_increment,
fahrstrasse_id int not null,
element_id int not null,
fahrstrassenelementtyp_id int not null,
sollstellung int not null,
primary key (id),
foreign key (fahrstrasse_id) references fahrstrassen (id) on update cascade,
foreign key (element_id) references elemente (id) on update cascade,
foreign key (fahrstrassenelementtyp_id) references fahrstrassenelementtypen (id) on update cascade
) engine=innodb default charset=latin1 row_format=default;
create unique index index7 on fahrstrassenelemente (fahrstrasse_id, element_id);

-- Trigger, der prüft, ob die projekt_id von elemente und fahrstrassen übereinstimmt
-- beim Insert/Update in fahrstrassenelemente
DROP TRIGGER IF EXISTS check_projekt_id_on_insert;
DROP TRIGGER IF EXISTS check_projekt_id_on_update;

DELIMITER %%

CREATE TRIGGER check_projekt_id_on_insert
BEFORE INSERT ON fahrstrassenelemente
FOR EACH ROW
BEGIN
    DECLARE element_pid INT DEFAULT NULL;
    DECLARE fahrstrasse_pid INT DEFAULT NULL;

    SELECT e.projekt_id
      INTO element_pid
      FROM elemente e
     WHERE e.id = NEW.element_id
     LIMIT 1;

    SELECT f.projekt_id
      INTO fahrstrasse_pid
      FROM fahrstrassen f
     WHERE f.id = NEW.fahrstrasse_id
     LIMIT 1;

    IF element_pid IS NULL
       OR fahrstrasse_pid IS NULL
       OR element_pid <> fahrstrasse_pid
    THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'projekt_id mismatch (oder element/fahrstrasse nicht gefunden)';
    END IF;
END%%

CREATE TRIGGER check_projekt_id_on_update
BEFORE UPDATE ON fahrstrassenelemente
FOR EACH ROW
BEGIN
    DECLARE element_pid INT DEFAULT NULL;
    DECLARE fahrstrasse_pid INT DEFAULT NULL;

    SELECT e.projekt_id
      INTO element_pid
      FROM elemente e
     WHERE e.id = NEW.element_id
     LIMIT 1;

    SELECT f.projekt_id
      INTO fahrstrasse_pid
      FROM fahrstrassen f
     WHERE f.id = NEW.fahrstrasse_id
     LIMIT 1;

    IF element_pid IS NULL
       OR fahrstrasse_pid IS NULL
       OR element_pid <> fahrstrasse_pid
    THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'projekt_id mismatch (oder element/fahrstrasse nicht gefunden)';
    END IF;
END%%

DELIMITER ;



-- Testdaten einfügen
insert into projekte (id, bezeichnung) values
(1, 'Testbahnhof'),
(2, 'Erlangen NER'),
(3, 'Chemnitz Hbf'),
(4, 'Projekt D'),
(5, 'Projekt E'),
(99, 'Test');

insert into elementtypen (id, bezeichnung, beschreibung) values
(1, 'G', 'Gleis'),
(2, 'W', 'Weiche'),
(3, 'S', 'Signal'),
(4, 'B', 'Blindziel'),
(5, 'A', 'Auflöseelement'),
(7, 'Z', 'Zusatzelement'),
(99, 'T', 'Test');

insert into elemente (projekt_id, typ_id, bezeichnung, lupe1x, lupe1y) values
(1, 1, 'Gleis A', 0, 0),
(1, 2, 'Weiche B', 1, 0),
(1, 3, 'Signal C', 2, 0),
(1, 6, 'Prellbock D', 3, 0),
(1, 1, 'Gleis E', 4, 0),
(1, 2, 'Weiche F', 5, 0),

-- Project 2 Tracks (Gleise)
(2, 1, '27G01', 0, 1),
(2, 1, '27G02', 1, 1),
(2, 1, '27G03', 2, 1),
(2, 1, '27G04', 3, 1),
(2, 1, '27G05', 4, 1),

-- Project 2 Signals
(2, 3, '27A', 0, 2),
(2, 3, '27B', 1, 2),
(2, 3, '27C', 2, 2),
(2, 3, '27D', 3, 2),
(2, 3, '27E', 4, 2),
(2, 3, '27F', 5, 2),
(2, 3, '27G', 6, 2),
(2, 3, '27N1', 7, 2),
(2, 3, '27N2', 8, 2),

-- Project 2 Switches (Weichen)
(2, 2, '27W01', 0, 3),
(2, 2, '27W02', 1, 3),

-- Project 3 Tracks (Gleise)
(3, 1, '35G01', 0, 4),
(3, 1, '35G02', 1, 4),
(3, 1, '35G03', 2, 4),
(3, 1, '35G04', 3, 4),
(3, 1, '35G05', 4, 4),

-- Project 3 Signals
(3, 3, '35A', 0, 5),
(3, 3, '35B', 1, 5),
(3, 3, '35C', 2, 5),
(3, 3, '35D', 3, 5),
(3, 3, '35E', 4, 5),
(3, 3, '35F', 5, 5),
(3, 3, '35G', 6, 5),
(3, 3, '35N1', 7, 5),
(3, 3, '35N2', 8, 5),

-- Project 3 Switches (Weichen)
(3, 2, '35W01', 0, 6),
(3, 2, '35W02', 1, 6),
(99, 99, 'Test Element', 0, 99);

insert into fahrstrassen (id, projekt_id, typ_id, bezeichnung) values
(1, 1, 2, 'Zugstraße 1'),
(2, 1, 1, 'Rangierfahrstraße 1'),
(3, 1, 2, 'Zugstraße 2'),
(4, 1, 1, 'Rangierfahrstraße 2'),
(5, 1, 2, 'Zugstraße 3'),
(6, 2, 2, '27A.27N2'),
(7, 2, 2, '27B.27N2'),
(8, 2, 2, '27C.27N2'),
(9, 2, 2, '27D.27N2'),
(10, 2, 2, '27E.27N2'),
(11, 2, 2, '27A.27N1'),
(12, 2, 2, '27B.27N1'),
(13, 2, 2, '27C.27N1'),
(14, 2, 2, '27D.27N1'),
(15, 2, 2, '27E.27N1'),
(16, 2, 1, '27A-27N1'),
(17, 2, 1, '27B-27N1'),
(18, 2, 1, '27C-27N1'),
(19, 2, 1, '27A-27N2'),
(20, 2, 1, '27B-27N2'),
(21, 2, 1, '27C-27N2'),
(22, 2, 1, '27L01-27N1'),
(23, 2, 1, '27L02-27N1'),
(24, 2, 1, '27L03-27N1'),
(25, 2, 1, '27L01-27N2'),
(26, 2, 1, '27L02-27N2'),
(27, 2, 1, '27L03-27N2'),
(28, 2, 1, '27A-27M01'),
(29, 2, 1, '27B-27M01'),
(30, 2, 1, '27C-27M01'),
(31, 2, 1, '27A-27M02'),
(32, 2, 1, '27B-27M02'),
(33, 2, 1, '27C-27M02'),
(34, 2, 1, '27L01-27M01'),
(35, 2, 1, '27L02-27M01'),
(36, 2, 1, '27L03-27M01'),
(37, 2, 1, '27L01-27M02'),
(38, 2, 1, '27L02-27M02'),
(39, 2, 1, '27L03-27M02');

insert into fahrstrassenelementtypen (id, bezeichnung, beschreibung) values
(1, 'FWE', 'Fahrwegelement'),
(2, 'FSS', 'Fahrstraßenstart'),
(3, 'FSZ', 'Fahrstraßenziel'),
(4, 'AUF', 'Auflöseelement'),
(5, 'FSE', 'Flankenschutzelement'),
(6, 'VWE', 'Vorwegelement'),
(7, 'UWE', 'Unterwegselement');

insert into fahrstrassenelemente (fahrstrasse_id, element_id, fahrstrassenelementtyp_id, sollstellung) values
(6, 7, 1, 0),
(6, 8, 2, 0),
(6, 9, 3, 0),
(6, 10, 4, 0),
(6, 11, 5, 0),
(6, 12, 6, 0),
(6, 13, 7, 0),
(7, 7, 1, 0),
(7, 8, 2, 0),
(7, 9, 3, 0),
(7, 10, 4, 0),
(7, 11, 5, 0),
(7, 12, 6, 0),
(7, 13, 7, 0);
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G01L', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 4, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G02', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 1, 4, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G03', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 2, 4, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G04', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 3, 4, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G05', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 4, 4, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35A', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35B', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 1, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35C', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 2, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35D', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 3, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35E', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 4, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35F', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 5, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35G', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 6, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35N13', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 7, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35N2', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 8, 5, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 2, NULL, '35W01', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 6, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 2, NULL, '35W02', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 1, 6, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (99, 99, NULL, 'Test Element', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 0, 99, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 6, NULL, '35P01', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 1, 1, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 4, NULL, '38T01', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 2, 1, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G11', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 5, 1, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 1, NULL, '35G01', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 3, 1, 0, 'N');
INSERT INTO `elemente` (`projekt_id`, `typ_id`, `hauptelement_id`, `bezeichnung`, `beschreibung`, `PFZS`, `PR`, `PRS`, `PRZ`, `PSSL`, `PSSR`, `PZ`, `PZS`, `PZZ`, `lupe1x`, `lupe1y`, `rotation`, `mirror`) VALUES (4, 3, NULL, '35N1', NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, NULL, 4, 1, 0, 'N');

```
