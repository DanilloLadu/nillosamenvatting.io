# RPG Voorbeelden

## DataTypes

```
DCL-C       Define a named constant
DCL-DS      Define a data structure
END-DS      End a data structure

{DCL-PARM}  Define a parameter
DCL-PI      Define a procedure interface (dcl-proc end-proc)
END-PI      End a procedure interface
DCL-PR      Define a prototype
END-PR      End a prototype
DCL-S       Define a standalone field
{DCL-SUBF}  Define a subfield
```

## Declaraties

```
INZ		ERROR HANDELING Fouten voorkomen
QUALIFIED	Voor prefex met data structuur zonder hoef je geen prefix te gebruiken

BINDEC(digits {: decimal-positions})
CHAR(length)
DATE{(format{separator})}
FLOAT(bytes)
GRAPH(length)
IND     indicator(*on, *off)
INT(digits)
OBJECT{(*JAVA:class-name)}
PACKED(digits {: decimal-positions})        gebruik dit boven digits , ditgits aleen als tijdelijke variabele (counter ...)
POINTER{(*PROC)}
TIME{(format{separator})}
TIMESTAMP{(fractional-seconds)}
UCS2(length)
UNS(digits)
VARCHAR(length {:2 | 4})
VARGRAPH(length {:2 | 4})
VARUCS2(length {:2 | 4})
ZONED(digits {: decimal-positions})
```

## DataFormatter oproepen met extra variable 

```
 // Choose (and init) "Data Formatter" procedure to be used
    if (KeysDS.KeyChoice = SEARCH_BY_ORDERDATE);
      RC = Example_OrderHeader_EXORDHDRD1_D002
             ( context : DataVersion : DataSize/Count :  KeysDS.OrderDateTo );
    else;
      RC = Example_OrderHeader_EXORDHDRD1_D002
             ( context : DataVersion : DataSize/Count );
    endif;
    if ( RC <> RC_ok );
      return;
    endif;
  endif;
```

```
 dcl-ds Format002 qualified based(formatPointer);
    OrderDateTo packed(8);
  end-ds;
```

parameter uit Lange call in datastructuur steken

```
// Store the requested settings in the "FormatData" part of the "context"
    if ( %parms = %parmnum(i_OrderDateTo) );
      Format002.OrderDateTo = CXDateTime_Date_To_Num(i_OrderDateTo);
    else;
      Format002.OrderDateTo = *hival;
    endif;
```

```
 // Check Date
    if ( EXORDHDRRec.ORDERDATE > Format002.OrderDateTo );
      return RC_eof;
    endif;
```

## Zoeken beginnen met

          RC = Example_OrderHeader_EXORDHDRP1_Ascending
                 ( context : Count
                 : 2 // frozen keys
                 : 0 // summary keys
                 : KeysDS.Company
                 : %trim(KeysDS.Order) + x'FF'
                 );
        endif;
## Zoeken op Dates

      RC = Example_OrderHeader_EXORDHDRP3_Ascending
             ( context : Count
             : 1 // frozen keys
             : 0 // summary keys
             : KeysDS.Company
             : CXDateTime_Date_To_Num(KeysDS.OrderDate)
             );
## Meerdere zoektermen 

Keychoice

```
  dcl-ds KeysDS qualified based(keysPointer);
    KeyChoice packed(3);
    Company char(2);
    Order char(12);
    Carrier char(30);
    OrderDate date;
    OrderDateTo date;
  end-ds;
```

```
  select;
    when ( KeysDS.KeyChoice = SEARCH_BY_ORDER );
      exsr callEXORDHDRP1;
    when ( KeysDS.KeyChoice = SEARCH_BY_CONTENT );
      exsr callEXORDHDRP2;
    when ( KeysDS.KeyChoice = SEARCH_BY_ORDERDATE );
      exsr callEXORDHDRP3;
  endsl;
```

## Converter C1

Als je uit een record date moet halen voor een andere fetch uit te voeren op een andere database

```
  dcl-ds Format001 qualified based(formatPointer);
    i_ContextRELATIE like(tCX);
  end-ds;
```

```
dcl-proc example_relation_EXRELATC1_convertEXORDREL export;
    dcl-pi *n like(tRC);
      io_Context like(tCX);
      i_ContextRELATIE const like(tCX);
    end-pi;

    // -- Procedure logic --
    
    // Init formatter within the "context" of the caller
    if ( Init_Context_Format( io_Context : 0 : 0
                            : %size(Format001) : %paddr(convertEXORDREL) ) <> RC_ok );
      // The "DataFormat" selection failed. Refer to the job log for more details.
      return SendMsg('CXE0029');
    endif;
    
    // Activate "context" settings
    FormatPointer = Get_Context_FormatData( io_Context );
    
    // Store the requested settings in the "FormatData" part of the "context"
    Format001.i_ContextRELATIE = i_ContextRELATIE;
    
    // Done
    return RC_ok;

  end-proc;
```

```
// -- Internal procedures --

  // This is the EXORDRELRC formatter
  dcl-proc convertEXORDREL;
    dcl-pi *n like(tRC);
      io_Context like(tCX);
      EXORDRELRec const likeds(tEXORDRELRec) options(*nopass);
      i_RemainingCount uns(5) const options(*nopass);
      o_Count uns(5) options(*nopass);
    end-pi;

    // -- The result set to be returned --
    dcl-c MAX_COUNT const(1);
    
    // -- Workfields --
    dcl-s returnCode like(tRC);
    
    // -- Procedure logic --
    
    // Validate incoming context
    if ( Get_Context_FormatProc(io_Context) <> %paddr(convertEXORDREL) );
      // The given "context" does not belong to the called procedure
      // Please contact the IT department.
      return SendMsg('CXE0010');
    endif;
    
    // Init "context" for caller
    if ( %parms = %parmnum(io_Context) );
      Set_Context_FormatDataID( io_Context : 0 : 0 );
      return RC_none;
    endif;
    
    // Check remaining count
    if ( %parms = %parmnum(o_Count) and i_RemainingCount < MAX_COUNT );
      o_Count = 0;
      return RC_more;
    endif;
    
    // Activate "context" settings
    formatPointer = Get_Context_FormatData( io_Context );
    
    // Fetch relation reccord
    returnCode = example_relations_EXRELATP1_Fetch
                   ( Format001.i_ContextRELATIE : EXORDRELRec.COMPANY : EXORDRELRec.RELATIONID );
    
    if ( returnCode = RC_none );
      returnCode = RC_skip;
    endif;
    
    return returnCode;

  end-proc;
```

Contect inializeren 

```
  dcl-s context like(tCX);
  dcl-s contextOrderRelation like(tCX);
```

Dataformatter vervangen met de converter

    // Choose (and init) "Data Formatter" procedure to be used
    RC = example_relations_EXRELATD1_D002( context : DataVersion : DataSize/Count );
    if ( RC <> RC_ok );
      return;
    endif;
    
    if ( KeysDS.KeyChoice = SEARCH_BY_ORDER );
      RC = Init_Context( contextOrderRelation : %paddr(AllocProc) );
      if ( RC <> RC_ok );
        return;
      endif;
    
      RC = example_relation_EXRELATC1_convertEXORDREL( contextOrderRelation : context );
      if ( RC <> RC_ok );
        return;
      endif;
    
    endif;
Call naar de provider van de andere database structuur die dan fetch naar de db van de hoofd db

```
  // Actual "business logic" call(s)
  if ( KeysDS.KeyChoice = SEARCH_BY_RELATIONID );
    exsr callEXRELATP1;
  elseif ( KeysDS.KeyChoice = SEARCH_BY_ORDER );
    exsr callEXORDRELP1;
    .......
  endif;
```

```
  // -- Internal subroutines --

  // EXRELATP1 procedure calls
  begsr callEXORDRELP1;

    select;
      when ( OpCode = OC_list_pos );
        RC = example_orderRelations_EXORDRELP1_Ascending
               ( contextOrderRelation : Count
               : 2 // frozen keys
               : 0 // summary keys
               : KeysDS.Company
               : KeysDS.Order
               );
    
      when ( OpCode = OC_list_next );
        RC = example_orderRelations_EXORDRELP1_Next( contextOrderRelation : Count );
      when ( OpCode = OC_list_back );
        RC = example_orderRelations_EXORDRELP1_Back( contextOrderRelation : Count );
      when ( OpCode = OC_list_first );
        RC = example_orderRelations_EXORDRELP1_First( contextOrderRelation : Count );
      when ( OpCode = OC_list_last );
        RC = example_orderRelations_EXORDRELP1_Last( contextOrderRelation : Count );
      other;
        // Unsupported value found in parameter OpCode (&1).
        CXE0008.OpCode = OpCode;
        RC = SendMsg('CXE0008':CXE0008);
    endsl;

  endsr;
```

## Service Programma S 001

```
 // -- Constant(s)-

  dcl-c INVOICE '003';
  dcl-c CLOSED '4';

// ------------------------------

  // -- Specific prototype(s) --

  /include qcpylesrc,EXMORDS001
  /include qcpylesrc,EXORDHDRU1

// ------------------------------
  // Procedure for getting a count of order that needs to be payed
  dcl-proc  Example_OrderRelations_EXMORDS001_CountNotPayed export;
    dcl-pi *n packed(3);
      i_Company like(EXORDRELL3Key.COMPANY) const;
      i_RelationId like(EXORDRELL3Key.RELATIONID) const;
    end-pi;

     // -- workfield(s) --
    dcl-s count packed(3) inz(0);
    dcl-s status char(2);
    
    // -- Procedure logic --
    
    // Count records that are on invoice and not closed
    EXORDRELL3Key.COMPANY = i_Company;
    EXORDRELL3Key.RELATIONID = i_RelationId;
    EXORDRELL3Key.ADDRESSTYP = INVOICE;
    setll %kds( EXORDRELL3Key : 3 ) EXORDRELL3;
    reade %kds( EXORDRELL3Key : 3 ) EXORDRELL3 EXORDRELL3Rec;
    dow ( not %eof );
      status = Example_OrderDetail_EXORDHDRU1_GetStatus( i_Company : EXORDRELL3Rec.ORDER );
      if ( status <> '' and status <> CLOSED );
        count += 1;
      endif;
    
      reade %kds( EXORDRELL3Key : 3 ) EXORDRELL3 EXORDRELL3Rec;
    enddo;
    
    return count;

  end-proc;
```

## FileAcces U

```
// -------------------------------

  // Procedure for getting a status
  dcl-proc Example_OrderDetail_EXORDHDRU1_GetStatus export;
    dcl-pi *n like(EXORDHDRL1Rec.STATUS);
      i_Company like(EXORDHDRL1Key.COMPANY) const;
      i_Order like(EXORDHDRL1Key.ORDER) const;
    end-pi;

    // -- Procedure logic --
    
    // Reset record and find record
    reset EXORDHDRL1Rec;
    EXORDHDRL1Key.COMPANY = i_Company;
    EXORDHDRL1Key.ORDER = i_Order;
    chain %kds( EXORDHDRL1Key : 2 ) EXORDHDRL1 EXORDHDRL1Rec;
    return EXORDHDRL1Rec.STATUS;

  end-proc;

// -------------------------------
```

```
  // Procedure for getting a unit of measurement
  dcl-proc Example_OrderDetail_EXORDDTLU1_GetUnitOfMeasurement export;
    dcl-pi *n like(EXORDDTLL1Rec.UOM);
      i_Company like(EXORDDTLL1Key.COMPANY) const;
      i_Order like(EXORDDTLL1Key.ORDER) const;
    end-pi;

    // -- Procedure logic --
    
    // Reset record and find record
    reset EXORDDTLL1Rec;
    EXORDDTLL1Key.COMPANY = i_Company;
    EXORDDTLL1Key.ORDER = i_Order;
    chain %kds( EXORDDTLL1Key : 2 ) EXORDDTLL1 EXORDDTLL1Rec;
    return EXORDDTLL1Rec.UOM;

  end-proc;
```

```
 // Procedure for getting a count of orderDetails
  dcl-proc Example_OrderDetail_EXORDDTLU1_CountRecords export;
    dcl-pi *n packed(3);
      i_Company like(EXORDDTLL1Key.COMPANY) const;
      i_Order like(EXORDDTLL1Key.ORDER) const;
    end-pi;

     // -- workfield(s) --
    dcl-s count packed(3) inz(0);
    
    // -- Procedure logic --
    
    // Reset record and find record
    EXORDDTLL1Key.COMPANY = i_Company;
    EXORDDTLL1Key.ORDER = i_Order;
    setll %kds( EXORDDTLL1key : 2 ) EXORDDTLL1;
    reade %kds( EXORDDTLL1Key : 2 ) EXORDDTLL1 EXORDDTLL1Rec;
    dow ( not %eof );
      count += 1;
      reade %kds( EXORDDTLL1Key : 2 ) EXORDDTLL1 EXORDDTLL1Rec;
    enddo;
    
    return count;

  end-proc;
```

## Get Read G

 Gewoon geneneren in xml

```
 // Get first record for given keys
  dcl-proc Example_OrderHeader_EXORDHDRG1_GetRead export;
    dcl-pi *n ind;
      i_Company like(EXORDHDRL1Key.COMPANY) const;
      i_Order like(EXORDHDRL1Key.ORDER) const options(*omit);
      io_Record char(1) options(*varsize);
      i_ForceRead ind const options(*nopass);
    end-pi;

    // -- Procedure logic --
    
    // Lay record format over parameter "io_Record"
    ptr_EXORD = %addr(io_Record);
    
    // Save time when "io_Record" corresponds to the requested keys
    // We already have the record so no need to read again
    if ( not ( %parms = %parmnum(i_ForceRead) and i_ForceRead ) and
         EXORDHDRL1Rec.COMPANY = i_Company and
         ( %addr(i_Order) = *null or EXORDHDRL1Rec.ORDER = i_Order ) );
      return *on;
    endif;
    
    // Read the record directly into "io_Record"
    EXORDHDRL1Key.COMPANY = i_Company;
    select;
      when ( %addr(i_Order) = *null );
        chain %kds( EXORDHDRL1Key : 1 ) EXORDHDRL1 EXORDHDRL1Rec;
      other;
        EXORDHDRL1Key.ORDER = i_Order;
        chain %kds( EXORDHDRL1Key : 2 ) EXORDHDRL1 EXORDHDRL1Rec;
    endsl;
    if ( not %found );
      clear EXORDHDRL1Rec; // This will clear the "io_Record" content
    endif;
    
    // Done
    return %found;

  end-proc;

// -------------------------------------------------------------------------------------------------

  // Get last record for given keys
  dcl-proc Example_OrderHeader_EXORDHDRG1_GetReadpe export;
    dcl-pi *n ind;
      i_Company like(EXORDHDRL1Key.COMPANY) const;
      i_Order like(EXORDHDRL1Key.ORDER) const options(*omit);
      io_Record char(1) options(*varsize);
      i_ForceRead ind const options(*nopass);
    end-pi;

    // -- Procedure logic --
    
    // Lay record format over parameter "io_Record"
    ptr_EXORD = %addr(io_Record);
    
    // Save time when "io_Record" corresponds to the requested keys
    // We already have the record so no need to read again
    if ( not ( %parms = %parmnum(i_ForceRead) and i_ForceRead ) and
         EXORDHDRL1Rec.COMPANY = i_Company and
         ( %addr(i_Order) = *null or EXORDHDRL1Rec.ORDER = i_Order ) );
      return *on;
    endif;
    
    // Read the record directly into "io_Record"
    EXORDHDRL1Key.COMPANY = i_Company;
    select;
      when ( %addr(i_Order) = *null );
        setgt %kds( EXORDHDRL1Key : 1 ) EXORDHDRL1;
        readpe %kds( EXORDHDRL1Key : 1 ) EXORDHDRL1 EXORDHDRL1Rec;
      other;
        EXORDHDRL1Key.ORDER = i_Order;
        setgt %kds( EXORDHDRL1Key : 2 ) EXORDHDRL1;
        readpe %kds( EXORDHDRL1Key : 2 ) EXORDHDRL1 EXORDHDRL1Rec;
    endsl;
    if ( %eof );
      clear EXORDHDRL1Rec; // This will clear the "io_Record" content
    endif;
    
    // Done
    return not %eof;

  end-proc;
```


