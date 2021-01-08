# Jef - Vieuw Only

| Applsrv.programs                          | Applsrv.desktop.functions                                    | Desktop.functions             | Common             |
| ----------------------------------------- | ------------------------------------------------------------ | ----------------------------- | ------------------ |
| `Programs` - Programma/Colom Omschrijving | `DataLogic` - Provider Service AS400 en andere services      | `MainPanel` GUI               | `Constante`        |
| `df` - DataFormatter                      | `Function` - Main classe bij het opstarten (DI) Adapters resolver crud events | `Search`, `Update`,  `Insert` | `Attribute`        |
| `rm` - Resolve method                     |                                                              |                               | `TranslationTable` |

```
master detail link ()  parameters van details zetten op master (data) 
(parameters) -> op de detail

order ->record na onze insert -> resync -> keydataset

autorefresh ->(default true) bij het veranderen van parameters 

alwaysrefresh ->(default false) geen parameter (hoeft geen ) -> navigatie events 

bij update en delete zetten we always refresh uit we willen geen onodige gets uitvoeren
```



## 1: Programma Lijst maken

- !!!!!JUISTE NAAM
- extends ProgramDescription
- record opstellen (Type, key_version, dataset)
- key versie mee geven zie rpg
- Dataset is onze formatter

```java
public final class EXORDHDR11 extends ProgramDescription {
 
    private static final int KEY_VERSION = 1;
 
    public EXORDHDR11() {
        super(LIST, EXORDHDR11::keyRecord, EXORDHDRD1::dataSet002);
    }
    private static AS400DataSet keyRecord() {
        return AS400DataSet.builder(KEY_VERSION).with(new WMSCompany()).build();
    }
}
```

## 2: DataSet Opstellen

- Static AS400DataSet 
- builder verwacht onze data version key van rpg + alle atributen aan onze rpg kant

```java
public final class EXORDHDRD1 {
 
    private static final int DATA_VERSION = 1;
 
    private EXORDHDRD1() {}
 
    public static AS400DataSet dataSet002() {
        return AS400DataSet.builder(DATA_VERSION)
                .with(new Order(), new OrderDate(), new DeliveryDate(), new Carrier(), new OrderStatus(),
                        new IsRushOrder())
                .build();
    }
 
}
```

## 3: Atributen opstellen

- static final name opstellen met constructor 
- lengte meegeven zelfde lengte als de rpg attribute
- extends Text, Date, Numeric 

```
public final class AdressType extends Text {
    public static final String NAME = "ADRESS_TYPE";
    public static final int LENGTH = 3;
    public static final String CODE_KEY = "EXADRTYP";
    /**
     * Constructor with the default attribute name.
     */
    public AdressType() {
        this(NAME);
    }
    
    /**
     * Constructor with a specific attribute name
     * 
     * @param name
     *            name of the attribute.
     */
    public AdressType(String name) {
        super(name, LENGTH);
    }

}
```

## 4: DataLogic

- Extend DataContext 
- !!!! Providers Toevoegen 
- Set parameter voor je company
- Gebruik van constenten die al gekend zijn met eventueel prefixen
- nieuwe provider toevoegen aan je DataContext Providers
- @Override start voor je data te refreshen (OR 11)

```java
final class DataLogic extends DataContext {
 
    private Provider orderList;
 
    DataLogic(ApplicationManager applicationManager, String company) throws FunctionException {
        super(applicationManager);
 
        orderList = new AS400Provider(this, LIST_DATATABLENAME, EXORDHDR11.class);
        orderList.setParameter(WMSCompany.NAME, company);
        addProvider(orderList);
    }
 
    @Override
    public void start() throws FunctionException {
        orderList.refresh();
    }
 
}
```

## 5: Function

- Extend GUIFunctionContext 
- @override createDataContext
	- WMSCompany get key toevoegen aan de  new DataLogic object
- Toevoegen aan de menu (package com.essers.mstl.training.applsrv.desktop.functions.order.management.danlad.function)

```java
public final class Function extends GUIFunctionContext {
 
    public Function(ApplicationManager applicationManager, GUIFunctionContext parent, Parameter[] parameters) {
        super(applicationManager, parent, parameters);
    }
 
    @Override
    public DataContext createDataContext(ApplicationManager applicationManager) throws FunctionException {
        return new DataLogic(applicationManager, (String) WMSCompany.getParameter(this).getKey());
    }
 
}
```

## 6: Constanten aanmaken + translation table 

- aanmaken van constanten voor doorheen je aplicatie 
- gebruik van jef constanten + prefixen
- ook constanten aan maken in je functuions + translation table 

```
public final class TranslationTable extends ResourceBundle {

    private static final String UNTILL = "Untill";
    private static final String VERSION = "Version";
    
    private static final TranslationTable INSTANCE = new TranslationTable();
    
    private static final String[][] CONTENTS = new String[][] { { Order.NAME, "Order" },
            { Batch.NAME, "Batch" },
            { Carrier.NAME, "Carrier" },
            { CustomerCity.NAME, "City" },
            { TotalOrdersToInvoice.NAME, "Orders to Invoice" } };
    
    public static ResourceBundle getInstance() {
        return INSTANCE;
    }
    
    @Override
    public Object[][] getContents() {
        return copy(CONTENTS);
    }
    
    @Override
    public ResourceBundle[] getSuperResourceBundles() {
        return new ResourceBundle[] { com.essers.mstl.system.TranslationTable.getInstance() };
    }

}
```

## 7: MainPanel

- Extend FunctionMainPanels
- Koppelen van de naam Databel uit je provider aan Jef Grid

```java
@SuppressWarnings("serial")
public final class MainPanel extends FunctionMainPanel {
 
    public MainPanel() {
        add(new JefGrid(LIST_DATATABLENAME));
    }
 
}
```

## 8: Toevoegen Menu

- Override createMenuBar 
- JefMenuBar 
- MenuFactory aan de menubar toevoegen met list control grid control en actie control 

```
@Override
    public JefMenuBar createMenuBar() {
        JefMenuBar menuBar = new JefMenuBar();
        // add file menu
        JefMenu menu = MenuFactory.create(MenuFactory.Type.FILE);
        // menu.add(MenuItemFactory.createListControls(false));
        menu.add(MenuItemFactory.createListControls(null, true, false, false));
        menu.addSeparator();
        menu.add(MenuItemFactory.createGridControls(MainPanel.class, orderGrid));
        menu.addSeparator();
        menu.add(MenuItemFactory.create(MenuItemFactory.Type.CLOSE));
        menuBar.add(menu);
 
        // add edit menu
        menu = MenuFactory.create(MenuFactory.Type.EDIT);
        menu.add(MenuItemFactory.createActionControls(new MenuItemFactory.Type[] { MenuItemFactory.Type.INSERT,
                MenuItemFactory.Type.UPDATE,
                MenuItemFactory.Type.DELETE }));
        menuBar.add(menu);
        menu.addSeparator();
        return menuBar;
    }
```

## 9: Search Toevoegen

- U lijst programma voorzien met de zoek atributen 
- Constructor van function setInitialFunctionEvent(new FunctionEvent(SEARCH_EVENTCODE)) toevoegen
- configureFunctionEventAdapters overschrijven + aproviders toevoegen en EventAdaptersFactory toevoegen 
- orderlist in je datalogic set blocked zetten + refresh afzetten + provider toevoegen(EventAdaptersFactory)

```java
EXORDHDR11
    private static AS400DataSet keyRecord() {
        return AS400DataSet.builder(KEY_VERSION).with(new WMSCompany(), new Order()).build();
    }
```

Function

```java
public Function(ApplicationManager applicationManager, GUIFunctionContext parent, Parameter[] parameters) {
        super(applicationManager, parent, parameters);
        // search event kan meedere event verwachten
        setInitialFunctionEvent(new FunctionEvent(SEARCH_EVENTCODE));
    }
        @Override
    protected void configureFunctionEventAdapters(Collection<eventadapter> adapters) {
        super.configureFunctionEventAdapters(adapters);
        Provider orderList = getDataContext().getProvider(LIST_DATATABLENAME);
        Provider orderParameters = getDataContext().getProvider(PARAMETER_DATATABLENAME);
        adapters.addAll(EventAdaptersFactory.createList(orderParameters, deriveUIName(SEARCH_PANELNAME), orderList));
    }
```

DataLogic

```java
        // verpligting van zoek ceteria
        orderList.setBlocked(true);
        addProvider(new ParameterProvider(this, PARAMETER_DATATABLENAME, orderList));
        @Override
        public void start() throws FunctionException {
            // Deze is niet nodig omdat we de orderlist blokeren
            // orderList.refresh();
        }  
```

## 10 : Criteria panel / toolbar / panel toevoegen aan de GUI

- Datalogic ColumnValuesProvider toevoegen voor onze RefreshPicklistFEA 	
- ReorderFEA en FilterFEA op de orderlist in onze function
- zoek adappter toevoegen configureFunctionEventAdapters overschrijven plus de andere addapters zelf toevoegen (driveUIName) 
- main panel addLocalOrderingColumns, addLocalMultiFilteringColumn, addLocalSingleFilteringColumn op onze get table van onze order 
- createria panel toevoegen aan de toolbar 

DataLogic

```java
        addProvider(new ColumnValuesProvider(this, CARRIER_FILTER_DATATABLENAME, orderList, Carrier.NAME));
        addProvider(new ColumnValuesProvider(this, IS_RUSH_ORDER_FILTER_DATATABLENAME, orderList, IsRushOrder.NAME));
```

Function

```java
        // reorder event
        adapters.add(new ReorderFEA(orderList, null, null));
        // event add filter
        adapters.add(new FilterFEA(orderList, null, null));
        // new refresh pickup event
        // we moeten providers toevoegen in onze data logic
        adapters.add(new RefreshPicklistFEA());
```

MainPanel

```java
       // sorteren op header van grid van je Attribute
        orderGrid.getTable()
        .getTableHeader()
        .addLocalOrderingColumns(new String[] { OrderDate.NAME, DeliveryDate.NAME });
        // filteren met Multy Filter
        orderGrid.getTable()
        .getTableHeader()
        .addLocalMultiFilteringColumn(LIST_DATATABLENAME, Carrier.NAME,
                new PickListDescription(CARRIER_FILTER_DATATABLENAME, Carrier.NAME, Carrier.NAME));
        // filteren met Single Filter
        orderGrid.getTable()
        .getTableHeader()
        .addLocalSingleFilteringColumn(LIST_DATATABLENAME, IsRushOrder.NAME,
                new PickListDescription(IS_RUSH_ORDER_FILTER_DATATABLENAME, IsRushOrder.NAME, IsRushOrder.NAME),
                true, ASTERISK, NULL);
         // toolbar toevoegen met criteria toevoegen
        add(JefToolBar.joinToolBarWithCriteria(createOrderListToolBar(), createCriteriaPanel()), NORTH);
         
    private static CriteriaPanel createCriteriaPanel() {
        CriteriaPanel panel = new CriteriaPanel();
        // zoeken altijd via zoekscherm
        // Jefmigbuilder voor zowel zoeken als label tonen op je DATATABELNAAM
        JefMigBuilder builder =
                new JefMigBuilder(MigLayoutFactory.create(MigLayoutFactory.Type.WITHOUT_INSETS),
                        PARAMETER_DATATABLENAME, panel);
        builder.appendLabelAndComponent(DataComponentFactory.Type.JEFLABEL, Order.NAME);
 
        return panel;
    }
```