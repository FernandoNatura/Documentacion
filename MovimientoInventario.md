## TOC

* [OIGN Entrada por movimientos - Object Type 59](#oign)
* [OIGE Salida de mercancias - Object type 60](#oige)
* [OWTR Traslado de Stock - Object Type 67](#owtr)
* [OPDN Entrada de mercancias por compra - Object Type 20](#opdn)
* [OINV - salida por ventas MV Tipo 13](#oinv)
* [ORIN Notas de cr&eacute;dito MV Tipo 14](#orin)

##  TABLAS 
####     OINM 
#### - Object Type 58
 Whse Journal - Diario de almacen (stock movement table)
    Transtype (object type):
      INM 58 - Inventory match

Table field: Transtype
       Values: 
    *  13     Salidas x venta MV Tipo documento RF 
    *  14     A/R Credit Memo
    *  20     Goods Receipt PO
    *  59     Goods Receipt 
    *  60     Goods Issue 
    *  67     Traspaso de inventario Tipo documento IM Query   
    *  69     Landed Costs
    *  162    Inventory Valuation


####     OIGN 
#### - Entrada por movimientos - Object Type 59
[volver](#toc)  
(oInvetoryGenEntry) Entrada de mercanc&iacute;as
     A goods receipt in the Warehouse Management system (WMS) 
     is the physical inbound movement of goods or materials into the warehouse. 
     It is a goods movement that is used to post goods received from external 
     vendors or from in-plant production. All goods receipts result in an increase 
     of stock in the warehouse.

   *  Transtype (object type):IGN 59 - Inventory General Entry
   *  Tabla de filas: IGN1 Entrada de mercanc&iacute;as
   *  Tipo de documentos: EM


```SQL
/** Entradas TIPO 59 CD  **/
Select 
						
						C."DocType",
                        C."DocNum",
                        --D."DocEntry",
                        D."LineNum" As "DocLineNum",
                        D."U_NPedidoGera" As "PedidoGera",
                        C."Comments",
                        C."JrnlMemo",
                        '' As "Origen",
                        A."Warehouse" As "Destino",                                                
                        AM."WhsName" As "Nombre",                                                
                        TO_CHAR(C."DocDate", 'dd/mm/yyyy') As "Fecha",                        
                        CC."ActId" As "CuentaContable",
                        CC."AcctName" As "NombreCuentaContable",
                        A."ItemCode",
                        A."Dscription",
                        D."unitMsr" AS "UM",
                        Cast(A."InQty" as Int) as "InQty",
                        Cast(A."OutQty" as Int) as "OutQty",
                        --D."LineTotal" as "Valor"
                        Cast(Abs(A."TransValue")as Decimal(10,2)) As "Valor",
                        U."U_NAME" As "Usuario"
        From            OIGN C  -- Cabecera Entrada de Mercancias
        Inner Join      IGN1  D  -- lineas de la Entrada de mercancias
        ON              C."DocEntry" = D."DocEntry"
        Inner Join      OINM A  --- DIARIO DE ALMACENCES...
        ON              A."CreatedBy" = D."DocEntry" 
        And             A."DocLineNum" = D."LineNum"
        Inner           Join OACT CC -- Cuentas contables
        ON              D."AcctCode" = CC."AcctCode"
        Inner Join      OWHS AM  --- Tabla de almacenes
        ON              AM."WhsCode" = A."Warehouse"
        Inner Join      OUSR U  --- Tabla de usuarios
        ON              U."USERID" = C."UserSign"
        Where           C."DocDate" between '2018-06-01' And '2018-06-30' 
        And             A."TransType" = 59
        And             "Warehouse" <> 'AMVEN'
        Order By		C."DocNum", D."LineNum"

-- TOTALES ENTRADAS TIPO 59
Select
						Count(*) As Cantidad,
          				Sum(Abs(A."TransValue")) As VALOR
        From            OINM A         
        Where           A."DocDate" between '2017-03-01' And '2017-03-31' 
        and             A."TransType"=59 -- Entradas  
        And             "Warehouse" <> 'AMVEN' -- Mall Ventura

```

####     OIGE 
#### - Salida de mercancias - Object type 60  
[volver](#toc)    
( oInvetoryGenExit ) Salida de mercancias 
    A goods issuefrom Extended Warehouse Management (EWM) is a physical departure 
    of products from your warehouse. 
    With a goods issue posting, you reduce the stock in the warehouse. 
    You can use a goods issue to indicate goods deliveries to your customers.

    Transtype (object type):
      IGE **60** - Inventory General Exit
    * IGE1    Salida de mercancias - Lineas

```SQL
/****  Salidas TIPO 60 CD  ****/
Select
                        'S' As "DocType",
                        Cast(T0."BASE_REF" As Varchar(20)) As "DocNum",
                        T0."DocLineNum",
                        T2."U_NPedidoGera" AS PEDIDOGERA,
                        T0."Comments",
                        T0."JrnlMemo",
                        T0."Warehouse" As "Origen",
                        AM."WhsName" As "Nombre",                                                
                        '' As "Destino",
                        TO_CHAR(T0."DocDate", 'dd/mm/yyyy') As Fecha,
                        T1."ActId" AS "Cuenta_Contable",
                        T1."AcctName" AS "NombreCuentaContable",
                        T0."ItemCode",
                        T0."Dscription",
                        'Unidad' As "UM",
                        Cast(T0."InQty" as Int) as "InQty",
                        Cast(T0."OutQty" as Int) as "OutQty",
                        --T0."CalcPrice" As COSTO_UNITARIO,
                        Cast(Abs(T0."TransValue")as Decimal(10,2)) As "Valor",
                        U."U_NAME" As "Usuario"
        from            NATURA.OINM T0 ---  Diario de almacen
        Inner Join      NATURA.OACT T1  --- Cuenta contable
        On              T0."CardCode" = T1."AcctCode"
        Inner Join              NATURA.IGE1 T2 -- Salida de mercancias (Lineas)
        On                              T0."CreatedBy" = T2."DocEntry"
        And                             T0."DocLineNum" = T2."LineNum"
        Inner Join      NATURA.OWHS AM  --- Tabla de almacenes
        ON              AM."WhsCode" = T0."Warehouse"
        Inner Join      NATURA.OUSR U  --- Tabla de usuarios
        ON              U."USERID" = T0."UserSign"
        Where           T0."DocDate"
        Between         '2018-06-01' And '2018-06-30'
        And             T0."TransType" = 60 -- SALIDAS
        And             T0."Warehouse" <> 'AMVEN'
                Order by        T0."DocDate", "BASE_REF", "DocLineNum"



--- SALIDAS DE MERCANCIAS TIPO 60
--- SUMA DEL VALOR DE LAS SALIDAS POR MES
 Select                 Count(*) as Cantidad, 
                        Sum(abs(T0."TransValue")) as VALOR 
        from            OINM T0        
        Where           T0."DocDate"
        Between         '2017-03-01' And '2017-03-31'
        And             T0."TransType" = 60 -- SALIDAS
        And             T0."Warehouse" <> 'AMVEN'

		
```


####     OWTR 
#### - Traslado de Stock - Object Type 67
[volver](#toc)   
  Traslado de stocks
    You use this function to transfer inventory from one warehouse to another.

    Transtype (object type):
        WTR **67** - Warehouses Transfers
    *  WTR1    Traslado de stocks - Filas

```SQL
--- TRASPASOS DE MERCANCIA TIPO 67

/*********  Traspasos TIPO 67 CD   ***********/
Select 
                        'T' As "DocType",
                        Cast((T0."DocNum") As Varchar(10)) As "DocNum",
                        T1."DocLineNum",
                        '' As "PedidoGera",
                        T0."Comments",
                        T0."JrnlMemo",
                        --Cast(T0."DocEntry" as Varchar(10)) As DOCENTRY, 
                        T2."FromWhsCod" As "Origen", 
                        AM1."WhsName" As "Nombre",                                                
                        T2."WhsCode" As "Destino",
                        AM2."WhsName" As "Nombre",                                                
                        TO_CHAR(T0."DocDate", 'dd/mm/yyyy') As Fecha, 
                        '' As "CuentaContable",
                        '' as "NombreCuentaContable", 
                        T1."ItemCode",
                        T1."Dscription",
                        'Unidad' AS "UM",
                        Cast(T1."OutQty" as Int) As "InQty",
                        Cast(T1."OutQty" as Int) AS "OutQty",
                        --T1."CalcPrice" As COSTO_UNITARIO,
                        Abs(T1."TransValue") As Valor,
                        U."U_NAME" AS "Usuario"
from                    OWTR T0 
Inner Join              WTR1 T2
ON                      T0."DocEntry" = T2."DocEntry" 
Inner Join              OINM T1
ON                      T0."DocNum" = T1."BASE_REF" 
And                     T1."InQty" = 0 
And                     T1."CreatedBy" = T2."DocEntry" 
And                     T1."DocLineNum" = T2."LineNum"   
Inner Join              OWHS AM1  --- Tabla de almacenes origen
ON                      AM1."WhsCode" = T2."FromWhsCod"
Inner Join              OWHS AM2  --- Tabla de almacenes Destino
ON                      AM2."WhsCode" = T2."WhsCode" --- 
Inner Join      		OUSR U  --- Tabla de usuarios
ON              		U."USERID" = T0."UserSign"
Where                   T1."TransType" = 67 -- Traspasos
And 			        T0."DocDate" between '2018-06-01' And '2018-06-30'
--And                     (T2."FromWhsCod" = 'AMVEN' or T2."WhsCode" = 'AMVEN') -- Mall Ventura
And                     (T2."FromWhsCod" <> 'AMVEN' And T2."WhsCode" <> 'AMVEN') -- Resto
Order By		        T0."DocNum", T1."DocLineNum"

--- TOTALES TRASPASOS TIPO 67
Select 
                        Count(*) As Cantidad,
                        Sum(Abs(T1."TransValue")) As Valor
from                    OWTR T0 
Inner Join              WTR1 T2
ON                      T0."DocEntry" = T2."DocEntry" 
Inner Join              OINM T1
ON                      T0."DocNum" = T1."BASE_REF" 
And                     T1."InQty" = 0 
And                     T1."CreatedBy" = T2."DocEntry" 
And                     T1."DocLineNum" = T2."LineNum"   
Where                   T1."TransType" = 67 -- Traspasos
And 			        T0."DocDate" between '2017-03-01' And '2017-03-31'
--And                     (T2."FromWhsCod" = 'AMVEN' or T2."WhsCode" = 'AMVEN') -- Mall Ventura
And                     (T2."FromWhsCod" <> 'AMVEN' And T2."WhsCode" <> 'AMVEN') -- Resto




```

####     OPDN 
#### - Entrada de mercancias por compra - Object Type 20
[volver](#toc)  
(oPurchaseDeliveryNotes)  Entrada de mercanc&iacute;as por compras locales 
    o importaciones.
    * Transtype (object type): PDN 20 - Purchase Delivery Notes
    * Tabla de Filas: PDN1  Pedido de entrada de mercanc&iacute;as - Filas
    * Tipo de documento: EP

![Imagen de Compras en el SAP B1](images/Compras.PNG)

```SQL
--- ENTRADA POR COMPRAS TIPO 20
Select 
						'I' As "DocType",
                        Cast(T0."BASE_REF" As Varchar(10)) As DOCNUM,
                        T0."DocLineNum",
                        '' as "PedidoGera",                        
                        T0."Comments",
                        T0."JrnlMemo",                      
                        '' As "Origen",
                        T0."Warehouse"as "Destino",
                        AM."WhsName" As "Nombre",
                        TO_CHAR(T0."DocDate", 'dd/mm/yyyy') As Fecha,
                        T1."ActId" As "CuentaContable",
						T1."AcctName" As "NombreCuentaContable",                                 
                        --T0."CardCode" As CODIGO_PROVEEDOR,
                        --T0."CardName" As NOMBRE_PROVEEDOR,
                        T0."ItemCode",
                        T0."Dscription",                        
                        Cast(T0."InQty" AS Int) as "InQty", 
                        Cast(T0."OutQty" AS Int) As "OutQty",
                        --T0."Currency",
                        --T0."CalcPrice" as PRECIO_UNITARIO,
                        Abs(T0."TransValue") As VALOR
        from            OINM T0 
        Inner Join 		OACT T1
        ON				T0."InvntAct" = T1."AcctCode"
Inner Join              OWHS AM  --- Tabla de almacenes origen
ON                      AM."WhsCode" = T0."Warehouse"
        Where			T0."TransType" = 20 ---COMPRAS
        And 			T0."DocDate" Between '2018-06-01' And '2018-06-30'
        Order By		T0."BASE_REF", T0."DocLineNum"

        --- TOTALES COMPRAS  TIPO 20
Select 
                        Count(*)  As Cantidad,
                        sum(Abs(T0."TransValue")) As VALOR
        from            OINM T0
        Where			T0."TransType" = 20 ---COMPRAS
        And 			T0."DocDate" Between '2017-03-01' And '2017-03-31'
```


####     OINV 
#### - salida por ventas MV Tipo 13
[volver](#toc)  
  *  (oInvoice) (ventas) Facturas deudores
  *  Transtype (object type): INV 13 Invoices
  *  INV1 Facturas deudores - lineas
  *  Tipo de documento : RF

![Imagen de Ventas](images/Compras.PNG)

```SQL
--- SALIDAS POR VENTA MALL VENTURA TIPO 13
 Select
                        
                        C."U_ESTADOFC" As ESTADO,
                        C."U_NROAUTOR" As NumeroAutorizacion,
                        C."NumAtCard" As NumeroFactura, 
                        C."U_CODCTRL",     
                        Cast(C."DocNum" as Varchar(10)) AS "DocNum",    
                        "LineNum",
                        '' As "PedidoGera",
                        '' As "Comments",
                        C."JrnlMemo",  
                         A."Warehouse" As ORIGEN,
                         '' As Destino,     
                        TO_CHAR(C."DocDate", 'dd/mm/yyyy') As Fecha,
                        CC."ActId" As CuentaContable,
                        CC."AcctName" As NombreCuentaContable,                       
                        A."ItemCode",
                        A."Dscription",
                        A."InQty",
                        A."OutQty",
                        --A."Currency",
                        --A."CalcPrice" as COSTO_UNITARIO,
                        Abs(A."TransValue") As TransValue  
        From            OINV C  -- cabecera de la venta (LV MALL)
        Inner Join      INV1  D  -- lineas de la venta
        ON              C."DocEntry" = D."DocEntry"
        Inner Join      OINM A
        ON              A."CreatedBy" = D."DocEntry" And A."DocLineNum" = D."LineNum"
        Inner Join      OACT CC -- Cuentas contables
        ON              D."AcctCode" = CC."AcctCode"
        Where           C."DocDate" between '2017-03-01' And '2017-03-31' 
        and             A."TransType"=13 -- Facturas de venta del mall ventura 
        Order By        "U_NROAUTOR", Cast("NumAtCard" as int), C."DocDate"

--- TOTALES VENTAS MALL VENTURA TIPO 13

Select
                        Count(*) As Cantidad,
                        Sum(Abs(A."TransValue")) As TransValue  
        From            OINM A
        Where           A."DocDate" between '2017-03-01' And '2017-03-31' 
        and             A."TransType"=13 -- Facturas de venta del mall ventura 

```

####     ORIN 
#### - Notas de cr&eacute;dito MV Tipo 14  
[volver](#toc)   
(oCreditNote) Nota de cr&eacute;dito de clientes (Anulacion de facturas)
      Transtype (object type):
        RIN 14 - Revert Invoices
    * RIN1 Anulacion de facturas Linea

```SQL
--- NOTAS DE CREDITO MALL VENTURA TIPO 14
Select
                Cast(C."DocNum" as Varchar(20)) As DOCNUM,
                C."U_NROAUTOR" As NumeroAutorizacion,
                C."NumAtCard" As NumeroFactura, 
                C."U_CODCTRL",
                --D."DocEntry",
                C."Comments",
                C."JrnlMemo",
                TO_CHAR(C."DocDate", 'dd/mm/yyyy') As Fecha,
                CC."ActId" As CuentaContable,
                CC."AcctName" As NombreCuentaContable,
                A."Warehouse",
                A."ItemCode",
                A."Dscription",
                A."InQty",
                A."OutQty",
                A."Currency",
                --A."CalcPrice" As COSTO_UNITARIO,
                Abs(A."TransValue") As Valor  
From            ORIN C  -- cabecera de la nota de credito (LV MALL)
Inner Join      RIN1  D  -- lineas de la nota de credito
ON              C."DocEntry" = D."DocEntry"
Inner Join      OINM A
ON              A."CreatedBy" = D."DocEntry" 
And             A."DocLineNum" = D."LineNum"
Inner Join      OACT CC -- Cuentas contables
ON              D."AcctCode" = CC."AcctCode"
Where           C."DocDate" between '2017-01-01' And '2017-01-31' 
and             A."TransType"=14 -- Notas de credito 
Order By        "U_NROAUTOR", "NumAtCard"

--TOTALES NOTAS DE CREDITO TIPO 14
Select			Count(*) As Cantidad,
                Sum(Abs(A."TransValue")) As Valor
From            OINM A
Where           A."DocDate" between '2017-03-01' And '2017-03-31' 
and             A."TransType"=14 -- Notas de credito 

```



