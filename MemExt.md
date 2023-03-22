MemExt
Obsah
    � �vod 
    � Realizace zapojen� 
        ? Popis zapojen� 
        ? Instalace modulu 
    � Pou��v�n� modulu MemExt 
        ? P��klady nastaven� str�nkov�n� 
        ? Inicializace str�nkov�n� 
        ? Postup p�i programov�n� FLASH 
        ? Programov�n� FLASH mimo MZ800 
        ? Obsah pam�ti FLASH 

�vod
Modul MemExt umo�n� pou��vat v MZ800 a� 1 MB opera�n� pam�ti. Obsahuje str�nkovac� syst�m, kter� p�ev�d� 16-ti bitovou adresu mikroprocesoru Z80 na 20-ti bitovou adresu pam�ti. 
      Slovn��ek pojm�:
    � Logick� pam� - to je pam� kterou zn�me jako fyzickou RAM z MZ800 p�ed instalac� MemExt. M� velikost 64kB a m��e m�t adresu 0-FFFFh. 
    � Fyzick� pam� - to je celkov� pam�. Jej� velikost je 1MB a m��e m�t adresu 0-FFFFFh. Je rozd�len� na dv� ��sti 512 kB RAM a 512 kB FLASH. 
    � Str�nkovac� pam� - je prost�edn�kem mezi logickou a fyzickou pam�t�. Jej� velikost je pouh�ch 16 bajt�. 
    � Str�nkov�n� - je hlavn� �innost� modulu MemExt. Je to p�evod 16-ti bitov� adresy na 20-ti bitovou. Ve skute�nosti ov�em nejvy��� 4 adresn� bity (A12-A15) adresuj� jednu z 16-ti bun�k str�nkovac� pam�ti a jej� 8-mi bitov� v�stup pak dopln� 12 ni���ch adresn�ch bit� (A0-A11) na 20-ti bitovou adresu. 
    � Str�nkovac� adresa - m� tento tvar "BB:AAA" kde BB /0-FFh/ je obsah str�nkovac� pam�ti, "AAA" /0-FFFh/ jsou spodn� nestr�nkovan� bity adresy. 
    � Str�nka - m� velikost 4 kB. Logick� pam� m� 16 str�nek, fyzick� pam� pak 256 z toho 128 v RAM a 128 ve FLASH. 
    � Mapov�n� - je mapov�n� pam�t� VRAM a PROM pomoc� I/O port� 0E0h-0E4h. Mapov�n� je nad�azeno str�nkov�n�. Pokud p�imapujeme ��st FLASH budu ji ozna�ovat jako PROM, proto�e do mapovan� FLASH nen� mo�n� z�pis. Pro programov�n� FLASH je nutn� pou��t str�nkov�n�. MemExt roz�i�uje vlastnosti mapov�n� PROM na mo�nost volit ze dvou 16 kB ��sti FLASH p�ep�na�em MZ700/MZ800. 
Nastaven� str�nkovac� pam�ti se prov�d� z�pisem na I/O port E7h, kde se 8 bit� datov� sb�rnice zapisuje do bu�ky ur�en� nejvy���mi 4-mi bity 16-ti bitov� adresn� sb�rnice, nastaven� tedy mus� b�t provedeno instrukc� OUT (C), r kdy vy��� 4 bity registru B ur�uj� adresu ve str�nkovac� pam�ti. Zapojen� neumo��uje str�nkovac� pam� programov� ��st. 

Popis zapojen�
    � Adresn� dekod�r I/O portu E7h - Je tvo�en obvody LS138 a LS10 (3 hradla).
V�stupem je pin 15 obvodu LS138 a nab�v� log "0" p�i z�pisu do I/O portu E7h. 
    � Logika m�du str�nkov�n� / mapov�n� - Sest�v� pouze z obvodu LS157.
Vstupem S (pin 1 LS157) je sign�l /CSROM (p�vodn� /CS pro pam� PROM). Mux s v�stupem Q3 (pin 9) funguje jako invertor sign�lu /CSROM a jeho aktivita zakazuje str�nkovac� pam� a generuje sign�l /CS pro pam� FLASH. Pokud je vstup /CSROM aktivn� (log "0") plat�:
- sign�l /ME (memory enabled) u obvod� 7489 je v log "1",
- sign�l /CS u pam�ti FLASH je v log "0",
- B12 = A12 (mux s v�stupem Q1 pin 4),
- B13 = A13 (mux s v�stupem Q4 pin 12),
- B14, B15 = log "0" (v�stupy str�nkovac� pam�ti v log "1" a invertor),
- B16 = stav p�ep�na�e, m�d MZ700 = log "1", m�d MZ800 = log "0", (mux s v�stupem Q2 pin 7),
- B17, B18 = log "1" (v�stupy str�nkovac� pam�ti v log "1"). 
    � Str�nkovac� pam� - Je tvo�en dv�ma obvody 7489, p�ti invertory obvodu LS04, osmi odpory 1k (obvod 7489 m� na v�stupu otev�en� kolektor)
Pam� je neaktivn� jedin� v p��pad� aktivity sign�lu /CSROM (m�d mapov�n�). Jinak pam� neust�le generuje podle bit� A12 - A15 adresn� sb�rnice sign�ly B12 - B19. V p��pad� z�pisu do I/O portu E7h je aktivn� sign�l /WE (pin 3) a bity datov� sb�tnice se zap�� do bu�ky str�nkovac� pam�ti vybran� bity A12 - A15 adresn� sb�rnice 
    � Logika v�b�ru RAM / FLASH - Sest�v� z obvodu LS02 (4 hradla) a invertoru obvodu LS04.
Vstupem je sign�l /CAS, kter� aktivoval p�vodn� pam�ov� �ipy 4164 - nyn� vstupuje na piny 6 a 8 obvodu LS02. Druh�m rozhodovac�m sign�lem je nejvy��� adresn� v�stup str�nkovac� pam�ti B19 - ten rozhodne zda bude aktivov�na pam� RAM nebo FLASH. Pam� FLASH m��e b�t tak� aktivovan� sign�lem /CSROM. 
    � Pam�ti FLASH/RAM - Obvody 29F040 a 628512.
Tady snad jedin� �e 628512 nem� uspo��dan� piny jako 29F040 tak jsou adresn� vodi�e v zapojen� pon�kud p�eh�zen�, ale vzhledem k tomu �e je to RAM tak to snad ani nevad�. 

Instalace modulu
    � Nejprve je t�eba si p�ipravit pomocn� datov� konektor se sedmi vodi�i a p�ip�jet je podle obr�zku mb.jpg na z�kladn� desku. Po�ad� vodi�� na pomocn�m konektoru (od zadn� ��sti MZ800) je:
A14
/WR
/RD
/IORQ
A15
7/8
/CAS. 
    � Druh�m krokem je odstaven� st�vaj�c�ch �ip� RAM 4164. To je realizov�no p�eru�en�m sign�lu CAS. Podle obr�zku mb.jpg najd�te nad obvodem 8253 �ty�i odpory v �ad� (na m�m obr�zku je �tvrt� odpor nahrazen propojkou, kterou jsem tam dal kv�li testov�n� abych nemusel st�le p�jet). Vyp�jejte no�i�ku �tvrt�ho odporu sm��uj�c� k obvodu 74LS125 a p�ipojte ji na +5V. 
    � Te� vyndejte z patice pam� PROM a m�sto n� vlo�te modul a p�ipojte pomocn� datov� konektor. 
    � Mu�ete zkusit co to ud�l�, kdy� se to zapne. 

P��klady nastaven� str�nkov�n�
Ovl�d�n� str�nkovac�ho mechanizmu je velice jednoduch� tak jen p�r p��klad�: 
      A� logickou pam� 2000-2FFF tvo�� str�nka 00 (prvn� str�nka RAM) 
    � LD BC, 20E7 - B=adresa str�nkovac� pam�ti 2, C=port str�nkov�n� 
    � LD A, 00 - A=str�nka 
    � OUT (C), A
A� logickou pam� 9000-9FFF tvo�� str�nka 7F (posledn� str�nka RAM) 
    � LD BC, 90E7 - B=adresa str�nkovac� pam�ti 9, C=port str�nkov�n� 
    � LD A, 7F - A=str�nka 
    � OUT (C), A
A� logickou pam� 4000-4FFF tvo�� str�nka 80 (prvn� str�nka FLASH) 
    � LD BC, 40E7 - B=adresa str�nkovac� pam�ti 4, C=port str�nkov�n� 
    � LD A, 80 - A=str�nka 
    � OUT (C), A
A� logickou pam� C000-CFFF tvo�� str�nka FF (posledn� str�nka FLASH) 
    � LD BC, C0E7 - B=adresa str�nkovac� pam�ti C, C=port str�nkov�n� 
    � LD A, FF - A=str�nka 
    � OUT (C), A
P�edstavme si, �e m�me odmapovan� v�echny PROM a SHARP je v m�du MZ-800 a chceme pomoc� str�nkov�n� prov�st ekvivalent p��kazu OUT (E4), A. 
    � LD BC, 00E7 - B=adresa str�nkovac� pam�ti 0, C=port str�nkov�n� 
    � LD A, 80 - A=str�nka 
    � OUT (C), A 
    � LD B, E0 - B=adresa str�nkovac� pam�ti E 
    � LD A, 82 - A=str�nka (str�nka 81 je znakov� gener�tor) 
    � OUT (C), A 
    � LD B, F0 - B=adresa str�nkovac� pam�ti F 
    � INC A - A=str�nka 83 
    � OUT (C), A 

Inicializace str�nkov�n�
Proto�e HW inicializace str�nkov�n� by byla p��li� slo�it� je nutn� �prava BIOSu. N�sleduj�c� program nastav� str�nkovac� pam� takto: logick� pam� 0-FFFh bude m�t p�imapovanou str�nku 00, logick� pam� 1000-1FFFh bude m�t p�imapovanou str�nku 01, ..., logick� pam� F000-FFFFh bude m�t p�imapovanou str�nku 0F.
0000h
JP 748h
C3 48 07
�prava po��te�n�ho skoku
0748h
LD BC, 0E7h
01 E7 00
B=0 adresa str�nkovac� pam�ti, C=IO port str�nkov�n�
074Bh
LD E, Bh
58
E=0 hodnota str�nkovan�
074Ch
OUT (C), E
ED 59
ulo� hodnotu pro str�nkovan�
074Eh
INC E
1C
zv��en� hodnoty pro str�nkovac� pam�
074Fh
LD A, B
78
zv��en� adresy str�nkovac� pam�ti
0750h
ADD A, 10h
C6 10

0752h
LD B, A
47

0753h
JR NZ, 074Ch
20 F7
smy�ka pob�� celkem 16x
0755h
JP E800h
C3 00 E8
pokra�ovat na IPL
Postup p�i programov�n� FLASH
Zad�n�:
1. m�m v logick� pam�ti na adrese 2000-2FFF n�co co chci dostat do FLASH do str�nky A3
2. str�nka A3 ve FLASH je bu� pr�zdn�, a nebo m� nezaj�m� co se stane s obsahem str�nek A0, A1, A2, A4 - AF
3. mohu si p�istr�nkovat FLASH na adresy 3000-3FFF (program a z�sobn�k je v logick� pam�ti n�kde jinde)
P�istr�nkov�n� FLASH
4000h
LD BC, 30E7h
B=30 (logick� pam� 3000-3FFF), C=port str�nkov�n�
4003h
LD A, 0A3h
A=A3 (str�nka FLASH A3)
4005h
OUT (C), A
nastav str�nkov�n�
Hlavn� smy�ka
4007h
LD HL, 2000h
HL=data adresa zdroje
400Ah
LD DE, 3000h
DE=data adresa c�le (programov�n�)
400Dh
LD A, 0F0h
RESET OBVODU FLASH - z�pis bajtu F0 kamkoli
400Fh
LD (DE), A

4010h
LD A, (DE)
�te bajt z FLASH
4011h
CP (HL)
a pokud to nen� stejn� s t�m co tam m�me d�t
sko��me na programov�n�
4012h
JR NZ,401Ch

4014h
INC HL
dal�� adresy
4015h
INC DE

4016h
LD A, H
test jestli to u� nen� v�e
4017h
CP 30h

4019h
JR NZ, 4010h

401Bh
RET

Programov�n� bajtu
401Ch
AND (HL)
kontrola zda bajt m��e b�t naprogramov�n:
nelze programovat log "1" na log "0"
pokud nelze mus� se sektor pam�ti FLASH nejprve vymazat
401Dh
CP (HL)

401Eh
JR NZ,403Bh

4020h
LD A, 0AAh
Sekvence programov�n� - krok 1
na adresu ve FLASH xx555 zapi� AA
4022h
LD (3555h), A

4025h
CPL
Sekvence programov�n� - krok 2
na adresu ve FLASH xx2AA zapi� 55
4026h
LD (32AAh), A

4029h
LD A, 0A0h
Sekvence programov�n� - krok 3
na adresu ve FLASH xx555 zapi� A0
402Bh
LD (3555h), A

402Eh
LD A, (HL)
Sekvence programov�n� - krok 4
na spr�vnou adresu zapi� spr�vn� bajt
402Fh
LD (DE), A

4030h
LD C, A
bajt dej i do C
4031h
LD A, (DE)
�ti bajt z FLASH 
4032h
CP C
pokud jsou tam spr�vn� data
konec programov�n� bajtu 
4033h
JR Z, 400Dh

4035h
BIT 5, A
pokud je konec �asov�ho limitu
konec programov�n� bajtu 
4037h
JR Z, 400Dh

4039h
JR 4031h
dal�� pr�chod �ten�m a testov�n�m 
Maz�n� sektoru FLASH (!!! pozor ma�e cel� sektor FLASH to je 16 str�nek !!!)
403Bh
LD A, 0AAh
Sekvence sektor erase - krok 1
na adresu ve FLASH xx555 zapi� AA
403Dh
LD (3555h), A

4040h
CPL
Sekvence sektor erase - krok 2
na adresu ve FLASH xx2AA zapi� 55
4041h
LD (32AAh), A

4044h
LD A, 80h
Sekvence sektor erase - krok 3
na adresu ve FLASH xx555 zapi� 80
4046h
LD (3555h), A

4049h
LD A, 0AAh
Sekvence sektor erase - krok 4
na adresu ve FLASH xx555 zapi� AA
404Bh
LD (3555h), A

404Eh
CPL
Sekvence sektor erase - krok 5
na adresu ve FLASH xx2AA zapi� 55
404Fh
LD (32AAh), A

4052h
LD A, 30h
Sekvence sektor erase - krok 6
na n�jakou adresu ve spr�vn�m sektoru zapi� 30
4054h
LD (DE), A

4055h
LD B, 0
chvilku po�kej
4057h
EX (SP), HL

4058h
DJNZ 4057h

405Ah
LD A, (DE)
a pokud sektor nen� vymaz�n �ekej d�l
405Bh
INC A

405Ch
JR NZ, 4057h

405Eh
JR 4007h
v�e znova od za��tku
Programov�n� FLASH mimo MZ800
Vzhledem k tomu, �e obvod 7489 neguje data jsou na v�stupech zapojeny invertory 74LS04, ale v�stup� je osm a invertor� jen �est. Proto se neshoduj� adresy FLASH v MZ800 s adresami v jin�m program�toru - nejsou toti� invertov�ny nejvy��� adresn� bity B17 a B18.
Skute�n� adresa FLASH
Adresa FLASH v MZ800
Str�nkovac� adresa
Mapov�n� BIOSu
00000h
60000h
E0:000h
�
10000h
70000h
F0:000h
�
20000h
40000h
C0:000h
�
30000h
50000h
D0:000h
�
40000h
20000h
A0:000h
�
50000h
30000h
B0:000h
�
60000h
00000h
80:000h
m�d MZ800 str�nky 80h, 81h, 82h a 83h
70000h
10000h
90:000h
m�d MZ700 str�nky 90h, 91h, 92h a 93h


