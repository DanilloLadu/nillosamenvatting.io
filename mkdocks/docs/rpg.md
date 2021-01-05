# RPG informatie

```
.PF     # Physical file		data (kolom omschrijvingen)
.Lf     # Logic file		view benaderen (keys) 
								(bepaalt sortering data die opgehaald wordt
								((her)indexering) (uniek benaderen of niet via UNIQUE)

```

## Commands

* `peek` - View edit voor je logic of physical files (Aleen in develop omgeving).
* `Strsql`  - Sql in AS400. `start sql SELECT * FROM TWSMAJDTA/EXORDHDR`
* `upddta` - Nog een manier om je data te zien
* `wrkjob` - Active Job Log benaderen (F10 Voor job info, F1 detail, F14 job log) (Shift + F).
* `wa` - Alle jobs

## Werk Omgeving 

| TWS                           | MAJ               | DTA                              | /    | EXORHDR max (10 chars)`                  |
| ----------------------------- | ----------------- | -------------------------------- | ---- | ---------------------------------------- |
| `TWS` - Transport Warehousing | `MAJ` - Major     | `DTA` - Logic physic data        |      | **EX**ample **OR**der **H**ea**D**e**R** |
|                               | `MIN`- Minor      | `DANLAD` - Persoonlijke Omgeving |      |                                          |
|                               | `EMG` - Emergency |                                  |      |                                          |
|                               | `TS1` - Test      |                                  |      |                                          |
|                               | `/`               |                                  |      |                                          |

## Service Programma's 

| File                                         | Naam            | Info                                                         |
| -------------------------------------------- | --------------- | ------------------------------------------------------------ |
| `EXORDHDRR1`- Filename + R + Index           | Resolver        | `Update`, `Delete`, `Insert`, `New`, `Get` - Nodig Voor Update programma |
| `EXORDHDRL1` - Filename + P + Logical        | Provider        | Lijst programma's roepen providers aan voor data door te sturen naar de data formatter. `Ascending` `Descending` `Next` `Back` `First` `Last` `Fetch` `Detail` `Validate` |
| `EXORDHDRD1` - Filename + D + Index          | Data Formatter  | Data resetten, opvullen, nakijken en record terugsturen of niet |
| `EXORDHDRU1` - Filename + U + Logical        | File Access     | Een feld opvragen uit een file uit een record die je verder niet nodig hebt  - Count - Record checken (setll) |
| `EXORDHDRG1` - Filename + G + Logical        | Get Read        | Meer dan een veld uit een file die je verder niet nodig hebt - Eerste of laatste record ophalen |
| `EXORDHDRS1` +  Filename  +  S + Index       | Service Program | Als je twee databases nodig hebt waar voor bijkomende logica om lijsten niet onodig te openen in andere service programma's - Duplicate code |
| `EXMORDS001` + GroepName + S + 3 cijfers 001 | Converter       | Als men in een tweede tabel wilt zoeken en record uit de eerste tabel wilt tonen |

## Programma's 

| File                                | Naam   | Info                                                         |
| ----------------------------------- | ------ | ------------------------------------------------------------ |
| `EXORDHDR11`- Filename + 1 + Index  | Lijst  | `Update`, `Delete`, `Insert`, `New`, `Get` - Nodig Voor Update programma |
| `EXORDHDR01` - Filename + 0 + Index | Update | Lijst programma's roepen providers aan voor data door te sturen naar de data formatter. |
| `EXORDHDR21` - Filename + 2 + Index | Resync | Data resetten, opvullen, nakijken en record terugsturen of niet |

## XML

**Genereerd op je Physical file**

| Group                   | Entity    |
| ----------------------- | --------- |
| Hoofd Indeling / Domain | Subdomain |

- Records - Concept van je data structuur
- Attribute - Naamgeving(Pascal Casing) / Logishe Naamgeving `Bij sommige attributen moet je een betere naamgeving geven FLAGHRUSH ->  IsRushOrder. Naam moet niet altijd dezelfde zijn-> Niveau van database (Java attribute staat op een hoger niveau)`
- Usage - Type(Conversie) dat verwacht word aan de andere kant (Data is packed Decimal)(logging)

| Type | Info                    |
| ---- | ----------------------- |
| A    | AlphaNumeric -> String  |
| P    | Packed Decimal-> Double |
| D    | Date                    |
| T    | Time                    |

Logicals Blok  *Service programma gebaseerd op je Logic File voor ieder block*

- KeyList   (Keys)

- Providers Voor het benaderen van de data

- Resolver  Voor Crud acties

  

Programma   *Gebaseerd op je Logical(s) File(s)*

- Lijst
- Update
- Rysync
- Omschrijving

Activeren `Only_new`

Deactiveren `Skip`



```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<qtlsjxml group="Example" entity="OrderHeader" version="1.1.0">
    <filetree>
        <mainfile id="1" name="EXORDHDR" rec="EXORDHDRR">
            <fieldlist>
                <field name="COMPANY" attribute="Company" type="A" len="2" label="Company"/>
                <field name="ORDER" attribute="Order" type="A" len="12" label="Order"/>
                <field name="ORDERDATE" attribute="OrderDate" type="P" len="8" dec="0" usage="*DATE" label="Order Date"/>
                <field name="DELIVDATE" attribute="DeliveryDate" type="P" len="8" dec="0" usage="*DATE" label="Delivery date"/>
                <field name="CARRIER" attribute="Carrier" type="A" len="30" label="Carrier"/>
                <field name="STATUS" attribute="Status" type="A" len="2" label="Status"/>
                <field name="FLAGRUSH" attribute="IsRushOrder" type="A" len="1" usage="*IND" label="Rush Order"/>
                <field name="CRTPGNM" attribute="ProgramCreated" type="A" len="10" usage="*LOG" label="Creation program"/>
                <field name="CRTUSER" attribute="UserCreated" type="A" len="10" usage="*LOG" label="Creation user"/>
                <field name="CRTDATE" attribute="DateCreated" type="P" len="8" dec="0" usage="*LOG,*DATE" label="Creation date"/>
                <field name="CRTTIME" attribute="TimeCreated" type="P" len="6" dec="0" usage="*LOG,*TIME" label="Creation time"/>
                <field name="CHGPGNM" attribute="ProgramChanged" type="A" len="10" usage="*LOG" label="Change program"/>
                <field name="CHGUSER" attribute="UserChanged" type="A" len="10" usage="*LOG" label="Change user"/>
                <field name="CHGDATE" attribute="DateChanged" type="P" len="8" dec="0" usage="*LOG,*DATE" label="Change date"/>
                <field name="CHGTIME" attribute="TimeChanged" type="P" len="6" dec="0" usage="*LOG,*TIME" label="Change time"/>
                <field name="JOBNAM" attribute="JobName" type="A" len="10" usage="*LOG" label="Job name   last event (CRT or CHG)"/>
                <field name="JOBUSR" attribute="JobUser" type="A" len="10" usage="*LOG" label="Job user   last event (CRT or CHG)"/>
                <field name="JOBNBR" attribute="JobNumber" type="P" len="6" dec="0" usage="*LOG" label="Job number last event (CRT or CHG)"/>
                <field name="CALLER1" attribute="Caller1" type="A" len="10" usage="*LOG" label="Caller of CRTPGNM (if CRT) or CHGPGNM (if CHG)"/>
                <field name="CALLER2" attribute="Caller2" type="A" len="10" usage="*LOG" label="Caller of CALLER1"/>
            </fieldlist>
            <formatter source="EXORDHDRD1" go="only_new">
                <text>Filter and format procedures for EXORDHDR</text>
            </formatter>
            <formatter source="EXORDHDRC1" go="only_new">
                <text>Convert from EXORDDTL TO EXORDHDR</text>
            </formatter>
            <index name="EXORDHDRL1">
                <text>COMPANY/ORDER - UNIQUE</text>
                <keylist>
                    <key name="COMPANY"/>
                    <key name="ORDER"/>
                </keylist>
                <fileAccess source="EXORDHDRU1" go="only_new"/>
                <getRead source="EXORDHDRG1" go="only_new"/>
                <provider source="EXORDHDRP1" go="only_new"/>
                <resolver source="EXORDHDRR1" go="only_new">
                    <text>Actions ...</text>
                </resolver>
            </index>
            <text>Example Order Header</text>
        </mainfile>
    </filetree>
    <programs>
        <program source="EXORDHDR01" type="UPDATE" go="only_new">
            <based_on>
                <index name="EXORDHDRL1"/>
            </based_on>
            <text>Example order header action</text>
        </program>
        <program source="EXORDHDR11" type="LIST" go="only_new">
            <based_on>
                <index name="EXORDHDRL1"/>
                <index name="EXORDHDRL2"/>
                <index name="EXORDHDRL3"/>
            </based_on>
            <text>Example order header list  danlad</text>
        </program>
       ...
```

## Lijst

*1

**Flow**`Keys controleren -> context inializeren (eerste call (refresh java)) -> call naar Formater (lange) -> bind juiste formatter in context -> buffer inializeren (iedere call) -> provider benaderen met de call + context meegeven` `<- RC code terug sturen`

**TODO** 

- Keys Controleren

- Data Structure 

  

**Declaraties**

| Declaraties               | Info                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Keys                      | meegegeven keys                                              |
| Keychoice                 | constanten voor bepalen van juiste provider(logic            |
| Context                   | context voor je provider die gestuurd wordt en terug komt- Init_Context -> eerste call (11) -> rc ok (100) |
| Data Formatter            | Example_OrderHeader_EXORDHDRD1_D002 -> binden van de juiste formater |
| Buffer                    | Context_Init_Buffer -> bij iedere call (100)                 |
| inPSSR                    | sub routine voor je programma errors (exsr, begsr)           |
| Init_LastMessageSpace     | laatste gelogde foutmelding                                  |
| keysPointer %addr(keys)   | Adressering van je keys(variable) in je keypointer           |
| recordPointer %addr(Data) | Adressering van je data(variable) in je recordPointer        |



**Vergelijkingen**

| Vergelijkingen | Info                    |
| -------------- | ----------------------- |
| KeySize        | Controleren keysize     |
| KeyVersion     | Controleren key version |
| OpCode         | Operation codes         |



**Calls**

| Calls              | Info                                                        |
| ------------------ | ----------------------------------------------------------- |
| exsr callResolver  | (Invoke Subroutine) call naar begsb                         |
| begsr callResolver | (Beginning of Subroutine) begin definitie van je subroutine |
| KeyChoice          | Voor juister provider                                       |



**Calls Provider**

| Calls Provider | Info                                                         |
| -------------- | ------------------------------------------------------------ |
| Ascending      | Pointer zetten + next                                        |
| Descending     | Pointer setten + back                                        |
| Next           | (load factor)vanaf context keys positioneren next (10 ok )   |
| Back           | (load factor)vanaf context keys positioneren back (10 ok )   |
| First          | positioneren en terug gaan lezen load factor (begin keys zitten in de context) |
| Last           | positioneren van achter en terug gaan lezen naar voor load factor (begin keys zitten in de context) |
| Fetch          | 1 ophalen of error                                           |
| Detail         | 1 ophalen of leeg                                            |
| Validate       | 1 ophalen of melding van meer of error                       |


**Keys**

| Keys        | Info                            |
| ----------- | ------------------------------- |
| Frozen keys | welke exacte keys ge nodig hebt |
| Summer keys | eerste record van de groepering |



## Update

0*

**TODO** 

- Keys Controleren

- Data Set `Prototype`

  

**Declaraties**

| Declaraties               | Info                                                  |
| ------------------------- | ----------------------------------------------------- |
| Keys                      | meegegeven keys                                       |
| Record                    | van je resolver record (pointer)                      |
| Buffer                    | Context_Init_Buffer -> bij iedere call (100)          |
| inPSSR                    | sub routine voor je programma errors (exsr, begsr)    |
| Init_LastMessageSpace     | laatste gelogde foutmelding                           |
| keysPointer %addr(keys)   | Adressering van je keys(variable) in je keypointer    |
| recordPointer %addr(Data) | Adressering van je data(variable) in je recordPointer |



**Vergelijkingen**

| Vergelijkingen | Info                                                         |
| -------------- | ------------------------------------------------------------ |
| KeySize        | Controleren keysize                                          |
| KeyVersion     | Controleren key version                                      |
| DataVersion    | Controleren Data version met je resolver record (input met data object) |
| OpCode         | Operation codes                                              |



**Calls**

| Calls              | Info                                                        |
| ------------------ | ----------------------------------------------------------- |
| exsr callResolver  | (Invoke Subroutine) call naar begsb                         |
| begsr callResolver | (Beginning of Subroutine) begin definitie van je subroutine |



**Calls Provider**

| Calls Provider | Info                              |
| -------------- | --------------------------------- |
| new            | nieuwe default object (prototype) |
| insert         | toevoegen                         |
| get            | ophalen                           |
| update         | updaten                           |
| delete         | wissen                            |

## Resync

2*

**Flow** `keys controleren -> context inializeren (eenmalig per call geen ok check op 11 lijst)) -> call naar Formater (lange) -> bind juiste formatter in context -> buffer inializeren (iedere call) -> provider benaderen met de call fetch + context meegeven <- RC code terug sturen`

**TODO** 

- Keys Controleren

- Data Structure 

  

**Declaraties**

| Declaraties               | Info                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Keys                      | meegegeven keys                                              |
| Keychoice                 | constanten voor bepalen van juiste provider(logic            |
| Context                   | context voor je provider die gestuurd wordt en terug komt- Init_Context -> eerste call (11) -> rc ok (100) |
| Data Formatter            | Example_OrderHeader_EXORDHDRD1_D002 -> binden van de juiste formater |
| Buffer                    | Context_Init_Buffer -> bij iedere call (100)                 |
| inPSSR                    | sub routine voor je programma errors (exsr, begsr)           |
| Init_LastMessageSpace     | laatste gelogde foutmelding                                  |
| keysPointer %addr(keys)   | Adressering van je keys(variable) in je keypointer           |
| recordPointer %addr(Data) | Adressering van je data(variable) in je recordPointer        |



**Vergelijkingen**

| Vergelijkingen | Info                    |
| -------------- | ----------------------- |
| KeySize        | Controleren keysize     |
| KeyVersion     | Controleren key version |
| OpCode         | Operation codes         |



**Calls**

| Calls              | Info                                                        |
| ------------------ | ----------------------------------------------------------- |
| exsr callResolver  | (Invoke Subroutine) call naar begsb                         |
| begsr callResolver | (Beginning of Subroutine) begin definitie van je subroutine |
| KeyChoice          | Voor juister provider                                       |



**Calls Provider**

| Calls Provider | Info               |
| -------------- | ------------------ |
| Fetch          | 1 ophalen of error |

## Dataformater	

D001          Model voor volgende data Data formatters

D002          Records nakijken

   - data reset opvullen met defailt waarde;

   - mappen en controleren

   - eventueel extra controlen voor bepaalde records door te laten

   - meerder of geen records sturen

        

## Provider

*Provider **programmeren we niet in**. Omdat deze ieder moment gegenereerd kan worden.* 
*En als we provider programmeren kunnen we die alleen voor één programma gebruiken.*



| Calls Provider | Info                                                         |
| -------------- | ------------------------------------------------------------ |
| Ascending      | Pointer zetten + next                                        |
| Descending     | Pointer setten + back                                        |
| Next           | (load factor)vanaf context keys positioneren next (10 ok )   |
| Back           | (load factor)vanaf context keys positioneren back (10 ok )   |
| First          | positioneren en terug gaan lezen load factor (begin keys zitten in de context) |
| Last           | positioneren van achter en terug gaan lezen naar voor load factor (begin keys zitten in de context) |
| Fetch          | 1 ophalen of error                                           |
| Detail         | 1 ophalen of leeg                                            |
| Validate       | 1 ophalen of melding van meer of error                       |

| ProtoType | Info                                     |
| --------- | ---------------------------------------- |
| dcl-pr    | Declaraties prototype met (return codes) |
| records   | input records                            |
| Export    | van de calls                             |

## Resolver

Crud Operaties

**TODO**

- Copy member nakijken

- Bepalen van keys en of je u record  meegeeft in je calls 

  

| Crud   | Info                                                |
| ------ | --------------------------------------------------- |
| new    | nieuwe default object (prototype)                   |
| insert | toevoegen                                           |
| get    | welke velden ken je al en moet je niet meer ophalen |
| update | updaten                                             |
| delete | wissen                                              |

| ProtoType | Info                                     |
| --------- | ---------------------------------------- |
| dcl-ds    | Declaratie data structure                |
| dcl-pr    | Declaraties prototype met (return codes) |
| records   | input records                            |
| Export    | van de calls                             |

**Delete**

- procedure interface

- Find and lock record (chain) -> ophalen van je data 

- Record comparing buffer 

- Set pointer 

- Delete record + unlock (error)

- return RC_ok; 

  

**Get**

- procedure interface       

- Find record       

- Save record for comparison in buffer        

- Fill record + (mappen)         

- return RC_ok;

  

**Insert**

- procedure interface

- Check Duplicate record

- Fill record + (mappen)      

-  Add record

- return RC_ok;

  

**Update**

- procedure interface   
- Find and lock record (chain) -> ophalen van je data   
- Fill record + (mappen) 
- Overwrite record content + unlock (error)     
- return RC_ok; 



**New**     

-  procedure interface     

-  Clear data opvullen met default values

- return RC_ok; 

  

## Lock's

***GEEN LOCKS***

- usage -> input output  of geen 

- read(n) -> lezen   
- chain(n) -> zoeken

***LOCKS***

- usage ->  update delete     
- chain    
- read
- delete
- update

