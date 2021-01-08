# Jef -  Create Update Edit Delete

| Applsrv.programs                          | Applsrv.desktop.functions                                    | Desktop.functions             | Common             |
| ----------------------------------------- | ------------------------------------------------------------ | ----------------------------- | ------------------ |
| `Programs` - Programma/Colom Omschrijving | `DataLogic` - Provider Service AS400 en andere services      | `MainPanel` GUI               | `Constante`        |
| `df` - DataFormatter                      | `Function` - Main classe bij het opstarten (DI) Adapters resolver crud events | `Search`, `Update`,  `Insert` | `Attribute`        |
| `rm` - Resolve method                     |                                                              |                               | `TranslationTable` |

## 

## 1: Update programma

- !!!!!JUISTE NAAM
- extends ProgramDescription
- record opstellen (Type, key_version, dataset)
- key versie mee geven zie rpg
- Dataset is onze formatter

```java
public class EXORDHDR01 extends ProgramDescription {
 
    private static final int KEY_VERSION = 1;
 
    public EXORDHDR01() {
        // keyRecord, AS400DataSet dataRecord type
        super(RESOLVER, EXORDHDR01::keyRecord, EXORDHDRR1::record);
        // Auto-generated constructor stub
    }
 
    private static AS400DataSet keyRecord() {
        return AS400DataSet.builder(KEY_VERSION).with(new WMSCompany(), new Order()).build(); 
    }
}
```

## 2: DataSet Opstellen van Resolver

- Static AS400DataSet 
- builder verwacht onze data version key van rpg + alle atributen aan onze rpg kant

```java
public final class EXORDHDRR1 {
 
    private static final int RECORD_VERSION = 1;
 
    private EXORDHDRR1() {
        // Utility class
    }
 
    public static AS400DataSet record() {
        return builder(RECORD_VERSION).with(new Order(), new OrderDate(), new DeliveryDate(),
                new Carrier().makeRequired(false), new OrderStatus(), new IsRushOrder()).build();
    }
 
}
```

## 3: Rysync programma

- !!!!!JUISTE NAAM
- extends ProgramDescription
- record opstellen (Type, key_version, dataset)
- key versie mee geven zie rpg
- Dataset is onze formatter

```
public class EXORDHDR21 extends ProgramDescription {
    private static final int KEY_VERSION = 1;
    public EXORDHDR21() {
        super(RESYNC, EXORDHDR21::keyRecord, EXORDHDRD1::dataSet002);
    }
    private static AS400DataSet keyRecord() {
        return AS400DataSet.builder(KEY_VERSION)
                .with(new WMSCompany(), new Order())
                .build();
    }
}
```

## 4: Providers toevoegen aan DataLogic

- provider toevoegen
- add master details
- voeg resolver toe
- en voeg functie toe getOrderResyncRow
	om van het update een data record te vragen  na onze insert van de update

```java
// update order
 Provider orderUpdate = new AS400Provider(this, UPDATE_DATATABLENAME, EXORDHDR01.class);
 // voor de index row te vinden
 orderUpdateOrderOrdinal = orderUpdate.getColumnOrdinal(Order.NAME);
 // auto refresh alleen als je wit triggeren en niet automatishe
 // allways refresh -> geen parameter refresht die ook
 addMasterDetailLink(orderList, orderUpdate, false);
 addProvider(orderUpdate);
 addResolver(orderUpdate.createResolver());
 
 // resync
 orderResync = new AS400Provider(this, RESYNCER_DATATABLENAME, EXORDHDR21.class);
 // nodig voor update en delete , bij insert moeten we andere manier doen
 addMasterDetailLink(orderUpdate, orderResync, false);
 addProvider(orderResync);
```

```java
Object[] getOrderResyncRow(Object[] orderUpdateRow) throws FunctionException {
    orderResync.setParameter(Order.NAME, orderUpdateRow[orderUpdateOrderOrdinal]);
    // call order provider
    orderResync.refresh();
    return orderResync.getCurrentRow();
}
```

## 5: Adapters en events toevoegen aan Function

- InsertEventAdapter InsertDFEA
- UpdateEventAdapter UpdateDFEA
- DeleteDFEA
- toevoegen aan one function

```java
    adapters.add(new InsertEventAdapter(orderUpdate, orderResolver, orderList));
    // update
    adapters.add(new UpdateEventAdapter(orderUpdate, orderResolver, orderList));
    // delete
    adapters.add(new DeleteDFEA(orderUpdate, orderResolver, orderList, deriveUIName(UPDATE_PANELNAME)));
 
 
 
private final class InsertEventAdapter extends InsertDFEA {
 
    private InsertEventAdapter(Provider provider, Resolver resolver, Provider masterProvider) {
        super(provider, resolver, masterProvider, UPDATE_PANELNAME);
    }
 
    // van de update een datarecord vragen en voegt die toe aan die provider
    // na onze insert via de update
    // record toevoegen in onze resync provider -> parameter
    @Override
    public Object[] getMasterRow(Object[] providerRow) throws FunctionException {
        return getDataContext().getOrderResyncRow(providerRow);
    }
}
 
private final class UpdateEventAdapter extends UpdateDFEA {
 
    private UpdateEventAdapter(Provider provider, Resolver resolver, Provider masterProvider) {
        super(provider, resolver, masterProvider, UPDATE_PANELNAME);
    }
 
    @Override
    public Object[] getMasterRow(Object[] providerRow) throws FunctionException {
        return getDataContext().getOrderResyncRow(providerRow);
    }
}
```

## 6: Update class 

- Update extends StandardDialogPanel
- constructor met DialogType
- appendLabelAndComponent

```
@SuppressWarnings("serial")
public final class Update extends StandardDialogPanel {
    public Update(DialogType type) {
        super(type);
        Boolean readOnly = type == DELETE;
        JefMigBuilder builder =
                new JefMigBuilder(MigLayoutFactory.create(MigLayoutFactory.Type.ONE_DATA_COLUMN), UPDATE_DATATABLENAME);
        builder.appendLabeledComponent(type == INSERT ? JEFTEXT : JEFLABEL, Order.NAME);
        builder.appendLabeledComponent(readOnly ? JEFLABEL : JEFDATE, OrderDate.NAME);
        builder.appendLabeledComponent(readOnly ? JEFLABEL : JEFDATE, DeliveryDate.NAME);
        builder.appendLabeledComponent(readOnly ? JEFLABEL : JEFTEXT, Carrier.NAME);
        builder.appendLabeledComponent(readOnly ? JEFLABEL : JEFCOMBOBOX, OrderStatus.NAME);
        builder.appendLabeledComponent(JEFCHECKBOX, IsRushOrder.NAME).setEnabled(!readOnly);
        add(builder.getJefPanel());
    }
}
```

