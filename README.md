# MemExt
Rozsireni pameti Sharp MZ-800 na 512kB RAM a 512 kB FLASH
Obsah
    • Úvod 
    • Realizace zapojení 
        ◦ Popis zapojení 
        ◦ Instalace modulu 
    • Používání modulu MemExt 
        ◦ Příklady nastavení stránkování 
        ◦ Inicializace stránkování 
        ◦ Postup při programování FLASH 
        ◦ Programování FLASH mimo MZ800 
        ◦ Obsah paměti FLASH 

Úvod
Modul MemExt umožní používat v MZ800 až 1 MB operační paměti. Obsahuje stránkovací systém, který převádí 16-ti bitovou adresu mikroprocesoru Z80 na 20-ti bitovou adresu paměti. 
      Slovníček pojmů:
    • Logická paměť - to je paměť kterou známe jako fyzickou RAM z MZ800 před instalací MemExt. Má velikost 64kB a může mít adresu 0-FFFFh. 
    • Fyzická paměť - to je celková paměť. Její velikost je 1MB a může mít adresu 0-FFFFFh. Je rozdělená na dvě části 512 kB RAM a 512 kB FLASH. 
    • Stránkovací paměť - je prostředníkem mezi logickou a fyzickou pamětí. Její velikost je pouhých 16 bajtů. 
    • Stránkování - je hlavní činností modulu MemExt. Je to převod 16-ti bitové adresy na 20-ti bitovou. Ve skutečnosti ovšem nejvyšší 4 adresní bity (A12-A15) adresují jednu z 16-ti buněk stránkovací paměti a její 8-mi bitový výstup pak doplní 12 nižších adresních bitů (A0-A11) na 20-ti bitovou adresu. 
    • Stránkovací adresa - má tento tvar "BB:AAA" kde BB /0-FFh/ je obsah stránkovací paměti, "AAA" /0-FFFh/ jsou spodní nestránkované bity adresy. 
    • Stránka - má velikost 4 kB. Logická paměť má 16 stránek, fyzická paměť pak 256 z toho 128 v RAM a 128 ve FLASH. 
    • Mapování - je mapování pamětí VRAM a PROM pomocí I/O portů 0E0h-0E4h. Mapování je nadřazeno stránkování. Pokud přimapujeme část FLASH budu ji označovat jako PROM, protože do mapované FLASH není možný zápis. Pro programování FLASH je nutné použít stránkování. MemExt rozšiřuje vlastnosti mapování PROM na možnost volit ze dvou 16 kB části FLASH přepínačem MZ700/MZ800. 
Nastavení stránkovací paměti se provádí zápisem na I/O port E7h, kde se 8 bitů datové sběrnice zapisuje do buňky určené nejvyššími 4-mi bity 16-ti bitové adresní sběrnice, nastavení tedy musí být provedeno instrukcí OUT (C), r kdy vyšší 4 bity registru B určují adresu ve stránkovací paměti. Zapojení neumožňuje stránkovací paměť programově číst. 

Popis zapojení
    • Adresní dekodér I/O portu E7h - Je tvořen obvody LS138 a LS10 (3 hradla).
Výstupem je pin 15 obvodu LS138 a nabývá log "0" při zápisu do I/O portu E7h. 
    • Logika módu stránkování / mapování - Sestává pouze z obvodu LS157.
Vstupem S (pin 1 LS157) je signál /CSROM (původní /CS pro paměť PROM). Mux s výstupem Q3 (pin 9) funguje jako invertor signálu /CSROM a jeho aktivita zakazuje stránkovací paměť a generuje signál /CS pro paměť FLASH. Pokud je vstup /CSROM aktivní (log "0") platí:
- signál /ME (memory enabled) u obvodů 7489 je v log "1",
- signál /CS u paměti FLASH je v log "0",
- B12 = A12 (mux s výstupem Q1 pin 4),
- B13 = A13 (mux s výstupem Q4 pin 12),
- B14, B15 = log "0" (výstupy stránkovací paměti v log "1" a invertor),
- B16 = stav přepínače, mód MZ700 = log "1", mód MZ800 = log "0", (mux s výstupem Q2 pin 7),
- B17, B18 = log "1" (výstupy stránkovací paměti v log "1"). 
    • Stránkovací paměť - Je tvořen dvěma obvody 7489, pěti invertory obvodu LS04, osmi odpory 1k (obvod 7489 má na výstupu otevřený kolektor)
Paměť je neaktivní jedině v případě aktivity signálu /CSROM (mód mapování). Jinak paměť neustále generuje podle bitů A12 - A15 adresní sběrnice signály B12 - B19. V případě zápisu do I/O portu E7h je aktivní signál /WE (pin 3) a bity datové sbětnice se zapíší do buňky stránkovací paměti vybrané bity A12 - A15 adresní sběrnice 
    • Logika výběru RAM / FLASH - Sestává z obvodu LS02 (4 hradla) a invertoru obvodu LS04.
Vstupem je signál /CAS, který aktivoval původní paměťové čipy 4164 - nyní vstupuje na piny 6 a 8 obvodu LS02. Druhým rozhodovacím signálem je nejvyšší adresní výstup stránkovací paměti B19 - ten rozhodne zda bude aktivována paměť RAM nebo FLASH. Paměť FLASH může být také aktivovaná signálem /CSROM. 
    • Paměti FLASH/RAM - Obvody 29F040 a 628512.
Tady snad jedině že 628512 nemá uspořádané piny jako 29F040 tak jsou adresní vodiče v zapojení poněkud přeházené, ale vzhledem k tomu že je to RAM tak to snad ani nevadí. 

Instalace modulu
    • Nejprve je třeba si připravit pomocný datový konektor se sedmi vodiči a připájet je podle obrázku mb.jpg na základní desku. Pořadí vodičů na pomocném konektoru (od zadní části MZ800) je:
A14
/WR
/RD
/IORQ
A15
7/8
/CAS. 
    • Druhým krokem je odstavení stávajících čipů RAM 4164. To je realizováno přerušením signálu CAS. Podle obrázku mb.jpg najděte nad obvodem 8253 čtyři odpory v řadě (na mém obrázku je čtvrtý odpor nahrazen propojkou, kterou jsem tam dal kvůli testování abych nemusel stále pájet). Vypájejte nožičku čtvrtého odporu směřující k obvodu 74LS125 a připojte ji na +5V. 
    • Teď vyndejte z patice paměť PROM a místo ní vložte modul a připojte pomocný datový konektor. 
    • Mužete zkusit co to udělá, když se to zapne. 

Příklady nastavení stránkování
Ovládání stránkovacího mechanizmu je velice jednoduché tak jen pár příkladů: 
      Ať logickou paměť 2000-2FFF tvoří stránka 00 (první stránka RAM) 
    • LD BC, 20E7 - B=adresa stránkovací paměti 2, C=port stránkování 
    • LD A, 00 - A=stránka 
    • OUT (C), A
Ať logickou paměť 9000-9FFF tvoří stránka 7F (poslední stránka RAM) 
    • LD BC, 90E7 - B=adresa stránkovací paměti 9, C=port stránkování 
    • LD A, 7F - A=stránka 
    • OUT (C), A
Ať logickou paměť 4000-4FFF tvoří stránka 80 (první stránka FLASH) 
    • LD BC, 40E7 - B=adresa stránkovací paměti 4, C=port stránkování 
    • LD A, 80 - A=stránka 
    • OUT (C), A
Ať logickou paměť C000-CFFF tvoří stránka FF (poslední stránka FLASH) 
    • LD BC, C0E7 - B=adresa stránkovací paměti C, C=port stránkování 
    • LD A, FF - A=stránka 
    • OUT (C), A
Představme si, že máme odmapované všechny PROM a SHARP je v módu MZ-800 a chceme pomocí stránkování provést ekvivalent příkazu OUT (E4), A. 
    • LD BC, 00E7 - B=adresa stránkovací paměti 0, C=port stránkování 
    • LD A, 80 - A=stránka 
    • OUT (C), A 
    • LD B, E0 - B=adresa stránkovací paměti E 
    • LD A, 82 - A=stránka (stránka 81 je znakový generátor) 
    • OUT (C), A 
    • LD B, F0 - B=adresa stránkovací paměti F 
    • INC A - A=stránka 83 
    • OUT (C), A 

Inicializace stránkování
Protože HW inicializace stránkování by byla příliš složitá je nutná úprava BIOSu. Následující program nastaví stránkovací paměť takto: logická paměť 0-FFFh bude mít přimapovanou stránku 00, logická paměť 1000-1FFFh bude mít přimapovanou stránku 01, ..., logická paměť F000-FFFFh bude mít přimapovanou stránku 0F.
0000h
JP 748h
C3 48 07
úprava počátečního skoku
0748h
LD BC, 0E7h
01 E7 00
B=0 adresa stránkovací paměti, C=IO port stránkování
074Bh
LD E, Bh
58
E=0 hodnota stránkovaní
074Ch
OUT (C), E
ED 59
ulož hodnotu pro stránkovaní
074Eh
INC E
1C
zvýšení hodnoty pro stránkovací paměť
074Fh
LD A, B
78
zvýšení adresy stránkovací paměti
0750h
ADD A, 10h
C6 10

0752h
LD B, A
47

0753h
JR NZ, 074Ch
20 F7
smyčka poběží celkem 16x
0755h
JP E800h
C3 00 E8
pokračovat na IPL
Postup při programování FLASH
Zadání:
1. mám v logické paměti na adrese 2000-2FFF něco co chci dostat do FLASH do stránky A3
2. stránka A3 ve FLASH je buď prázdná, a nebo mě nezajímá co se stane s obsahem stránek A0, A1, A2, A4 - AF
3. mohu si přistránkovat FLASH na adresy 3000-3FFF (program a zásobník je v logické paměti někde jinde)
Přistránkování FLASH
4000h
LD BC, 30E7h
B=30 (logická paměť 3000-3FFF), C=port stránkování
4003h
LD A, 0A3h
A=A3 (stránka FLASH A3)
4005h
OUT (C), A
nastav stránkování
Hlavní smyčka
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
Čte bajt z FLASH
4011h
CP (HL)
a pokud to není stejné s tím co tam máme dát
skočíme na programování
4012h
JR NZ,401Ch

4014h
INC HL
další adresy
4015h
INC DE

4016h
LD A, H
test jestli to už není vše
4017h
CP 30h

4019h
JR NZ, 4010h

401Bh
RET

Programování bajtu
401Ch
AND (HL)
kontrola zda bajt může být naprogramován:
nelze programovat log "1" na log "0"
pokud nelze musí se sektor paměti FLASH nejprve vymazat
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
na správnou adresu zapiš správný bajt
402Fh
LD (DE), A

4030h
LD C, A
bajt dej i do C
4031h
LD A, (DE)
čti bajt z FLASH 
4032h
CP C
pokud jsou tam správná data
konec programování bajtu 
4033h
JR Z, 400Dh

4035h
BIT 5, A
pokud je konec časového limitu
konec programování bajtu 
4037h
JR Z, 400Dh

4039h
JR 4031h
další průchod čtením a testováním 
Mazání sektoru FLASH (!!! pozor maže celý sektor FLASH to je 16 stránek !!!)
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
na nějakou adresu ve správném sektoru zapiš 30
4054h
LD (DE), A

4055h
LD B, 0
chvilku počkej
4057h
EX (SP), HL

4058h
DJNZ 4057h

405Ah
LD A, (DE)
a pokud sektor není vymazán čekej dál
405Bh
INC A

405Ch
JR NZ, 4057h

405Eh
JR 4007h
vše znova od začátku
Programování FLASH mimo MZ800
Vzhledem k tomu, že obvod 7489 neguje data jsou na výstupech zapojeny invertory 74LS04, ale výstupů je osm a invertorů jen šest. Proto se neshodují adresy FLASH v MZ800 s adresami v jiném programátoru - nejsou totiž invertovány nejvyšší adresní bity B17 a B18.
Skutečná adresa FLASH
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

