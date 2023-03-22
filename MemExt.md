MemExt
Obsah
    • Úvod 
    • Realizace zapojení 
        ? Popis zapojení 
        ? Instalace modulu 
    • Pouívání modulu MemExt 
        ? Pøíklady nastavení stránkování 
        ? Inicializace stránkování 
        ? Postup pøi programování FLASH 
        ? Programování FLASH mimo MZ800 
        ? Obsah pamìti FLASH 

Úvod
Modul MemExt umoní pouívat v MZ800 a 1 MB operaèní pamìti. Obsahuje stránkovací systém, kterı pøevádí 16-ti bitovou adresu mikroprocesoru Z80 na 20-ti bitovou adresu pamìti. 
      Slovníèek pojmù:
    • Logická pamì - to je pamì kterou známe jako fyzickou RAM z MZ800 pøed instalací MemExt. Má velikost 64kB a mùe mít adresu 0-FFFFh. 
    • Fyzická pamì - to je celková pamì. Její velikost je 1MB a mùe mít adresu 0-FFFFFh. Je rozdìlená na dvì èásti 512 kB RAM a 512 kB FLASH. 
    • Stránkovací pamì - je prostøedníkem mezi logickou a fyzickou pamìtí. Její velikost je pouhıch 16 bajtù. 
    • Stránkování - je hlavní èinností modulu MemExt. Je to pøevod 16-ti bitové adresy na 20-ti bitovou. Ve skuteènosti ovšem nejvyšší 4 adresní bity (A12-A15) adresují jednu z 16-ti bunìk stránkovací pamìti a její 8-mi bitovı vıstup pak doplní 12 niších adresních bitù (A0-A11) na 20-ti bitovou adresu. 
    • Stránkovací adresa - má tento tvar "BB:AAA" kde BB /0-FFh/ je obsah stránkovací pamìti, "AAA" /0-FFFh/ jsou spodní nestránkované bity adresy. 
    • Stránka - má velikost 4 kB. Logická pamì má 16 stránek, fyzická pamì pak 256 z toho 128 v RAM a 128 ve FLASH. 
    • Mapování - je mapování pamìtí VRAM a PROM pomocí I/O portù 0E0h-0E4h. Mapování je nadøazeno stránkování. Pokud pøimapujeme èást FLASH budu ji oznaèovat jako PROM, protoe do mapované FLASH není monı zápis. Pro programování FLASH je nutné pouít stránkování. MemExt rozšiøuje vlastnosti mapování PROM na monost volit ze dvou 16 kB èásti FLASH pøepínaèem MZ700/MZ800. 
Nastavení stránkovací pamìti se provádí zápisem na I/O port E7h, kde se 8 bitù datové sbìrnice zapisuje do buòky urèené nejvyššími 4-mi bity 16-ti bitové adresní sbìrnice, nastavení tedy musí bıt provedeno instrukcí OUT (C), r kdy vyšší 4 bity registru B urèují adresu ve stránkovací pamìti. Zapojení neumoòuje stránkovací pamì programovì èíst. 

Popis zapojení
    • Adresní dekodér I/O portu E7h - Je tvoøen obvody LS138 a LS10 (3 hradla).
Vıstupem je pin 15 obvodu LS138 a nabıvá log "0" pøi zápisu do I/O portu E7h. 
    • Logika módu stránkování / mapování - Sestává pouze z obvodu LS157.
Vstupem S (pin 1 LS157) je signál /CSROM (pùvodní /CS pro pamì PROM). Mux s vıstupem Q3 (pin 9) funguje jako invertor signálu /CSROM a jeho aktivita zakazuje stránkovací pamì a generuje signál /CS pro pamì FLASH. Pokud je vstup /CSROM aktivní (log "0") platí:
- signál /ME (memory enabled) u obvodù 7489 je v log "1",
- signál /CS u pamìti FLASH je v log "0",
- B12 = A12 (mux s vıstupem Q1 pin 4),
- B13 = A13 (mux s vıstupem Q4 pin 12),
- B14, B15 = log "0" (vıstupy stránkovací pamìti v log "1" a invertor),
- B16 = stav pøepínaèe, mód MZ700 = log "1", mód MZ800 = log "0", (mux s vıstupem Q2 pin 7),
- B17, B18 = log "1" (vıstupy stránkovací pamìti v log "1"). 
    • Stránkovací pamì - Je tvoøen dvìma obvody 7489, pìti invertory obvodu LS04, osmi odpory 1k (obvod 7489 má na vıstupu otevøenı kolektor)
Pamì je neaktivní jedinì v pøípadì aktivity signálu /CSROM (mód mapování). Jinak pamì neustále generuje podle bitù A12 - A15 adresní sbìrnice signály B12 - B19. V pøípadì zápisu do I/O portu E7h je aktivní signál /WE (pin 3) a bity datové sbìtnice se zapíší do buòky stránkovací pamìti vybrané bity A12 - A15 adresní sbìrnice 
    • Logika vıbìru RAM / FLASH - Sestává z obvodu LS02 (4 hradla) a invertoru obvodu LS04.
Vstupem je signál /CAS, kterı aktivoval pùvodní pamìové èipy 4164 - nyní vstupuje na piny 6 a 8 obvodu LS02. Druhım rozhodovacím signálem je nejvyšší adresní vıstup stránkovací pamìti B19 - ten rozhodne zda bude aktivována pamì RAM nebo FLASH. Pamì FLASH mùe bıt také aktivovaná signálem /CSROM. 
    • Pamìti FLASH/RAM - Obvody 29F040 a 628512.
Tady snad jedinì e 628512 nemá uspoøádané piny jako 29F040 tak jsou adresní vodièe v zapojení ponìkud pøeházené, ale vzhledem k tomu e je to RAM tak to snad ani nevadí. 

Instalace modulu
    • Nejprve je tøeba si pøipravit pomocnı datovı konektor se sedmi vodièi a pøipájet je podle obrázku mb.jpg na základní desku. Poøadí vodièù na pomocném konektoru (od zadní èásti MZ800) je:
A14
/WR
/RD
/IORQ
A15
7/8
/CAS. 
    • Druhım krokem je odstavení stávajících èipù RAM 4164. To je realizováno pøerušením signálu CAS. Podle obrázku mb.jpg najdìte nad obvodem 8253 ètyøi odpory v øadì (na mém obrázku je ètvrtı odpor nahrazen propojkou, kterou jsem tam dal kvùli testování abych nemusel stále pájet). Vypájejte noièku ètvrtého odporu smìøující k obvodu 74LS125 a pøipojte ji na +5V. 
    • Teï vyndejte z patice pamì PROM a místo ní vlote modul a pøipojte pomocnı datovı konektor. 
    • Muete zkusit co to udìlá, kdy se to zapne. 

Pøíklady nastavení stránkování
Ovládání stránkovacího mechanizmu je velice jednoduché tak jen pár pøíkladù: 
      A logickou pamì 2000-2FFF tvoøí stránka 00 (první stránka RAM) 
    • LD BC, 20E7 - B=adresa stránkovací pamìti 2, C=port stránkování 
    • LD A, 00 - A=stránka 
    • OUT (C), A
A logickou pamì 9000-9FFF tvoøí stránka 7F (poslední stránka RAM) 
    • LD BC, 90E7 - B=adresa stránkovací pamìti 9, C=port stránkování 
    • LD A, 7F - A=stránka 
    • OUT (C), A
A logickou pamì 4000-4FFF tvoøí stránka 80 (první stránka FLASH) 
    • LD BC, 40E7 - B=adresa stránkovací pamìti 4, C=port stránkování 
    • LD A, 80 - A=stránka 
    • OUT (C), A
A logickou pamì C000-CFFF tvoøí stránka FF (poslední stránka FLASH) 
    • LD BC, C0E7 - B=adresa stránkovací pamìti C, C=port stránkování 
    • LD A, FF - A=stránka 
    • OUT (C), A
Pøedstavme si, e máme odmapované všechny PROM a SHARP je v módu MZ-800 a chceme pomocí stránkování provést ekvivalent pøíkazu OUT (E4), A. 
    • LD BC, 00E7 - B=adresa stránkovací pamìti 0, C=port stránkování 
    • LD A, 80 - A=stránka 
    • OUT (C), A 
    • LD B, E0 - B=adresa stránkovací pamìti E 
    • LD A, 82 - A=stránka (stránka 81 je znakovı generátor) 
    • OUT (C), A 
    • LD B, F0 - B=adresa stránkovací pamìti F 
    • INC A - A=stránka 83 
    • OUT (C), A 

Inicializace stránkování
Protoe HW inicializace stránkování by byla pøíliš sloitá je nutná úprava BIOSu. Následující program nastaví stránkovací pamì takto: logická pamì 0-FFFh bude mít pøimapovanou stránku 00, logická pamì 1000-1FFFh bude mít pøimapovanou stránku 01, ..., logická pamì F000-FFFFh bude mít pøimapovanou stránku 0F.
0000h
JP 748h
C3 48 07
úprava poèáteèního skoku
0748h
LD BC, 0E7h
01 E7 00
B=0 adresa stránkovací pamìti, C=IO port stránkování
074Bh
LD E, Bh
58
E=0 hodnota stránkovaní
074Ch
OUT (C), E
ED 59
ulo hodnotu pro stránkovaní
074Eh
INC E
1C
zvıšení hodnoty pro stránkovací pamì
074Fh
LD A, B
78
zvıšení adresy stránkovací pamìti
0750h
ADD A, 10h
C6 10

0752h
LD B, A
47

0753h
JR NZ, 074Ch
20 F7
smyèka pobìí celkem 16x
0755h
JP E800h
C3 00 E8
pokraèovat na IPL
Postup pøi programování FLASH
Zadání:
1. mám v logické pamìti na adrese 2000-2FFF nìco co chci dostat do FLASH do stránky A3
2. stránka A3 ve FLASH je buï prázdná, a nebo mì nezajímá co se stane s obsahem stránek A0, A1, A2, A4 - AF
3. mohu si pøistránkovat FLASH na adresy 3000-3FFF (program a zásobník je v logické pamìti nìkde jinde)
Pøistránkování FLASH
4000h
LD BC, 30E7h
B=30 (logická pamì 3000-3FFF), C=port stránkování
4003h
LD A, 0A3h
A=A3 (stránka FLASH A3)
4005h
OUT (C), A
nastav stránkování
Hlavní smyèka
4007h
LD HL, 2000h
HL=data adresa zdroje
400Ah
LD DE, 3000h
DE=data adresa cíle (programování)
400Dh
LD A, 0F0h
RESET OBVODU FLASH - zápis bajtu F0 kamkoli
400Fh
LD (DE), A

4010h
LD A, (DE)
Ète bajt z FLASH
4011h
CP (HL)
a pokud to není stejné s tím co tam máme dát
skoèíme na programování
4012h
JR NZ,401Ch

4014h
INC HL
další adresy
4015h
INC DE

4016h
LD A, H
test jestli to u není vše
4017h
CP 30h

4019h
JR NZ, 4010h

401Bh
RET

Programování bajtu
401Ch
AND (HL)
kontrola zda bajt mùe bıt naprogramován:
nelze programovat log "1" na log "0"
pokud nelze musí se sektor pamìti FLASH nejprve vymazat
401Dh
CP (HL)

401Eh
JR NZ,403Bh

4020h
LD A, 0AAh
Sekvence programování - krok 1
na adresu ve FLASH xx555 zapiš AA
4022h
LD (3555h), A

4025h
CPL
Sekvence programování - krok 2
na adresu ve FLASH xx2AA zapiš 55
4026h
LD (32AAh), A

4029h
LD A, 0A0h
Sekvence programování - krok 3
na adresu ve FLASH xx555 zapiš A0
402Bh
LD (3555h), A

402Eh
LD A, (HL)
Sekvence programování - krok 4
na správnou adresu zapiš správnı bajt
402Fh
LD (DE), A

4030h
LD C, A
bajt dej i do C
4031h
LD A, (DE)
èti bajt z FLASH 
4032h
CP C
pokud jsou tam správná data
konec programování bajtu 
4033h
JR Z, 400Dh

4035h
BIT 5, A
pokud je konec èasového limitu
konec programování bajtu 
4037h
JR Z, 400Dh

4039h
JR 4031h
další prùchod ètením a testováním 
Mazání sektoru FLASH (!!! pozor mae celı sektor FLASH to je 16 stránek !!!)
403Bh
LD A, 0AAh
Sekvence sektor erase - krok 1
na adresu ve FLASH xx555 zapiš AA
403Dh
LD (3555h), A

4040h
CPL
Sekvence sektor erase - krok 2
na adresu ve FLASH xx2AA zapiš 55
4041h
LD (32AAh), A

4044h
LD A, 80h
Sekvence sektor erase - krok 3
na adresu ve FLASH xx555 zapiš 80
4046h
LD (3555h), A

4049h
LD A, 0AAh
Sekvence sektor erase - krok 4
na adresu ve FLASH xx555 zapiš AA
404Bh
LD (3555h), A

404Eh
CPL
Sekvence sektor erase - krok 5
na adresu ve FLASH xx2AA zapiš 55
404Fh
LD (32AAh), A

4052h
LD A, 30h
Sekvence sektor erase - krok 6
na nìjakou adresu ve správném sektoru zapiš 30
4054h
LD (DE), A

4055h
LD B, 0
chvilku poèkej
4057h
EX (SP), HL

4058h
DJNZ 4057h

405Ah
LD A, (DE)
a pokud sektor není vymazán èekej dál
405Bh
INC A

405Ch
JR NZ, 4057h

405Eh
JR 4007h
vše znova od zaèátku
Programování FLASH mimo MZ800
Vzhledem k tomu, e obvod 7489 neguje data jsou na vıstupech zapojeny invertory 74LS04, ale vıstupù je osm a invertorù jen šest. Proto se neshodují adresy FLASH v MZ800 s adresami v jiném programátoru - nejsou toti invertovány nejvyšší adresní bity B17 a B18.
Skuteèná adresa FLASH
Adresa FLASH v MZ800
Stránkovací adresa
Mapování BIOSu
00000h
60000h
E0:000h
 
10000h
70000h
F0:000h
 
20000h
40000h
C0:000h
 
30000h
50000h
D0:000h
 
40000h
20000h
A0:000h
 
50000h
30000h
B0:000h
 
60000h
00000h
80:000h
mód MZ800 stránky 80h, 81h, 82h a 83h
70000h
10000h
90:000h
mód MZ700 stránky 90h, 91h, 92h a 93h


