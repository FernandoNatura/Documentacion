## TOC

* [OINV - salida por ventas MV Tipo 13](#oinv)
* [ORIN Notas de cr&eacute;dito MV Tipo 14](#orin)
* [OPDN Entrada de mercancias por compra - Object Type 20](#opdn)
* [OIGN Entrada por movimientos - Object Type 59](#oign)
* [OIGE Salida de mercancias - Object type 60](#oige)
* [OWTR Traslado de Stock - Object Type 67](#owtr)
* [OIPF Costos de importaci&oacute;n - Object Type 69](#oipf)
* [QUERIES](#queries) 

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
    *  67     Traspaso de inventario Tipo documento IM
  Query   *  69     Landed Costs
    *  162    Inventory Valuation

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
                        Cast(T0."BASE_REF" As Varchar(10)) As DOCNUM,
                        T0."DocLineNum",
                        '' as "PedidoGera",                        
                        T0."Comments",
                        T0."JrnlMemo",                      
                        '' As "Origen",
                        T0."Warehouse"as "Destino",
                        TO_CHAR(T0."DocDate", 'dd/mm/yyyy') As Fecha,
                        T1."ActId" As "CuentaContable",
						T1."AcctName" As "NombreCuentaContable",                                 
                        --T0."CardCode" As CODIGO_PROVEEDOR,
                        --T0."CardName" As NOMBRE_PROVEEDOR,
                        T0."ItemCode",
                        T0."Dscription",                        
                        T0."InQty", 
                        T0."OutQty",
                        --T0."Currency",
                        --T0."CalcPrice" as PRECIO_UNITARIO,
                        Abs(T0."TransValue") As VALOR
        from            OINM T0 
        Inner Join 		OACT T1
        ON				T0."InvntAct" = T1."AcctCode"
        Where			T0."TransType" = 20 ---COMPRAS
        And 			T0."DocDate" Between '2017-03-01' And '2017-03-31'
        Order By		T0."BASE_REF", T0."DocLineNum"

        --- TOTALES COMPRAS  TIPO 20
Select 
                        Count(*)  As Cantidad,
                        sum(Abs(T0."TransValue")) As VALOR
        from            OINM T0
        Where			T0."TransType" = 20 ---COMPRAS
        And 			T0."DocDate" Between '2017-03-01' And '2017-03-31'
```

####     OJDT 
#### -Entrada de Diario (Asiento contable) Object Type 30





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
-- Entradas AL INVENTARIO TIPO 59
Select
                        C."DocNum",
                        --D."DocEntry",
                        D."LineNum" As "DocLineNum",
                        '' As "PedidoGera",
                        C."Comments",
                        C."JrnlMemo",
                        '' As "Origen",
                        A."Warehouse" As "Destino",                                                
                        TO_CHAR(C."DocDate", 'dd/mm/yyyy') As Fecha,                        
                        CC."ActId" As CuentaContable,
                        CC."AcctName" As NombreCuentaContable,
                        A."ItemCode",
                        A."Dscription",
                        A."InQty",
                        A."OutQty",
                        --A."Currency",
                        --A."CalcPrice" As PRECIO_UNITARIO,
                        Abs(A."TransValue") As VALOR
        From            OIGN C  -- Cabecera Entrada de Mercancias
        Inner Join      IGN1  D  -- lineas de la Entrada de mercancias
        ON              C."DocEntry" = D."DocEntry"
        Inner Join      OINM A  --- DIARIO DE ALMACENCES...
        ON              A."CreatedBy" = D."DocEntry" 
        And             A."DocLineNum" = D."LineNum"
        Inner           Join OACT CC -- Cuentas contables
        ON              D."AcctCode" = CC."AcctCode"
        Where           C."DocDate" between '2017-02-01' And '2017-02-28' 
        and             A."TransType"=59 -- Entradas  
        --And             "Warehouse" <> 'AMVEN'
        And             "Warehouse" = 'AMVEN' -- Mall Ventura
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
--- SALIDAS DE MERCANCIAS TIPO 60
Select 
                        Cast(T0."BASE_REF" As Varchar(20)) As "DocNum",                       
                        T0."DocLineNum",
                        T2."U_NPedidoGera" AS PEDIDOGERA,
                        T0."Comments",
                        T0."JrnlMemo",                        
                        T0."Warehouse" As "Origen",
                        '' As "Destino",
                        TO_CHAR(T0."DocDate", 'dd/mm/yyyy') As Fecha,  
                        T1."ActId" AS CUENTA_CONTABLE,
                        T1."AcctName" AS NOMBRE_CUENTA_CONTABLE,
                        T0."ItemCode",
                        T0."Dscription",
                        T0."InQty",
                        T0."OutQty",
                        --T0."CalcPrice" As COSTO_UNITARIO,
                        abs(T0."TransValue") as VALOR 
        from            OINM T0
        Inner Join      OACT T1
        On              T0."CardCode" = T1."AcctCode"
        Inner Join		IGE1 T2
        On				T0."CreatedBy" = T2."DocEntry"
        And				T0."DocLineNum" = T2."LineNum"
        Where           T0."DocDate"
        Between         '2017-03-01' And '2017-03-31'
        And             T0."TransType" = 60 -- SALIDAS
        And             T0."Warehouse" <> 'AMVEN'
		Order by        T0."DocDate", "BASE_REF", "DocLineNum" 

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
Select 
                        Cast((T0."DocNum") As Varchar(10)) As DOCNUM,
                        T1."DocLineNum",
                        '' As "PedidoGera",
                        T0."Comments",
                        T0."JrnlMemo",
                        --Cast(T0."DocEntry" as Varchar(10)) As DOCENTRY, 
                        T2."FromWhsCod" As Origen, 
                        T2."WhsCode" As Destino,
                        TO_CHAR(T0."DocDate", 'dd/mm/yyyy') As Fecha, 
                        '' As "CuentaContable",
                        '' as "NombreCuentaContable", 
                        T1."ItemCode",
                        T1."Dscription",
                        T1."OutQty" As "InQty",
                        T1."OutQty" ,
                        --T1."CalcPrice" As COSTO_UNITARIO,
                        Abs(T1."TransValue") As Valor
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
And                     (T2."FromWhsCod" = 'AMVEN' or T2."WhsCode" = 'AMVEN') -- Mall Ventura
--And                     (T2."FromWhsCod" <> 'AMVEN' And T2."WhsCode" <> 'AMVEN') -- Resto
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

####     OIPF 
#### - Costos de importaci&oacute;n - Object Type 69
[volver](#toc)   
   Costos de importaci&oacute;n
    Transtype (object type):
        IPF 69 - Import file
    *  IPF1    Precios de entrega - L&iacute;neas



#### Otras Tablas
[volver](#toc)

```SQL

/* Chart of Accounts */
SELECT * FROM OACT

/* Item Master */
SELECT * FROM OITM

/* Employee Master (HR) */
SELECT * FROM OHEM

/* Incomming Payments and the line level Information */
SELECT * FROM ORCT

```

####         Libro Ventas 
```SQL
        Select 
                        T0."NumAtCard" As NumeroFactura, 
                        T0."DocTotal" As Total, 
                        T0."DocDate" As Fecha, 
                        T0."U_NIT" As Nit, 
                        T0."U_RAZSOC" As RazonSocial, 
                        T0."U_CODCTRL" As CodigoControl, 
                        T0."U_NROAUTOR"  As NumeroAutorizacion,
                        T0."JrnlMemo" 
        From            OINV T0
        --Inner Join    INV1 T1
        --On            T0."DocEntry" = T1."DocEntry"
        Where           T0."DocDate" Between '2017-02-01' And '2017-02-28'
        Order by        T0."NumAtCard"
```

[volver](#toc) 

*  IM :  Traspasos
*  OA :  Salidas de materiales
*  DI :  Precios de entrega para costos
*  RF :  Facturas del Mall Ventura
*  EP :  Entrada de materiales
*  RC :  Notas de cr&eacute;dito, anulacion de facturas Mall Ventura


## QUERIES
[volver](#toc)

#### Query Asientos de Ventas Belmonte

```SQL
Declare @Fecha Date = '01/11/2016'


Declare @Ges int = year(@Fecha)
Declare @Mes int = month(@Fecha)
Declare @Ofic int = 3 --- LPZ
--Declare @Ofic int = 2 --- CBB
--Declare @Ofic int = 1 --- SCZ

Declare @T Table(Fecha Date, TC decimal(10,2), Venta Decimal(10,2), Intereses Decimal(10,2), IdGlo Int, siCont smallint)
Insert into @T
Exec bSelContVentasDia @Ges, @Mes, @Ofic, 'C'

Declare @R table(Fecha Date, Grupo int, Cuenta Varchar(100), NombreCuenta Varchar(100), Debe Decimal(10,2), Haber Decimal(10,2))

Insert Into @R
Select  
        Cast(G.Fecha as DATE)As Fecha, 
        Grupo = Case When N.codNomenUsr in ('1-1-03-02-001','2-1-02-01-006') then 1 Else (
                Case When N.CodNomenUsr in('4-1-01','4-1-02') Then 2 Else 0 End
        ) End,
        N.CodNomenUsr, 
        N.Descripcion, 
        M.DebeBs, 
        M.HaberBs  
From @T T
        Inner Join bContglo G
        ON G.IDglo = T.IdGlo
        Inner Join bContMov M 
        ON T.IdGlo = M.IDglo
        Inner JOin bNomenclador N
        ON N.CodNomen = M.Codnomen
        Where siCont = 1 And N.IdMask = 102
Order By M.Idglo    

Declare @Asiento Table(Fecha Date, Cuenta Varchar(100), NombreCuenta Varchar(100), Debe Decimal(10,2), Haber Decimal(10,2))

Insert Into @Asiento
Select Fecha, MIN(Cuenta) As Cuenta, MIN(NombreCuenta) As NombreCuenta, SUM(Debe)As Debe, SUM(Haber)As Haber from @R
Where Grupo = 1
Group by Fecha, Grupo

Insert Into @Asiento
Select Fecha, MIN(Cuenta) As Cuenta, MAX(NombreCuenta) As NombreCuenta, SUM(Debe)As Debe, SUM(Haber)As Haber from @R
Where Grupo = 2
Group by Fecha, Grupo

Insert Into @Asiento 
Select Fecha, Cuenta, NombreCuenta, Debe, Haber From @R Where Grupo=0

Select * from @Asiento 
Order By Fecha, cuenta

```
### Query Asientos de Ventas SAP
```SQL
Select 
	TO_CHAR(T0."TaxDate", 'dd/mm/yyyy') As Fecha,
	T1."ActId",
	T1."AcctName", 
	SUM(T0."Debit")as DEBIT,
	SUM(T0."Credit") as CREDIT
	 from JDT1 T0
Inner Join OACT T1
ON T0."Account" = T1."AcctCode" 
Where Cast("TaxDate" as date) between '2016-11-01' And '2016-11-30'
And "TransType" = 30 
And "U_NPedidoGera" is not null and "U_NPedidoGera"<>''
And "Segment_1" = 'LPZ'
--And "Segment_1" = 'CBB'
--And "Segment_1" = 'SCZ'
Group by T0."TaxDate", T1."ActId", T1."AcctName"
Order By cast("TaxDate" as date), "ActId"
```
### Query Cobranzas SAP

```SQL
Select 
	TO_CHAR(T0."TaxDate", 'dd/mm/yyyy') As Fecha,
	T1."ActId",
	T1."AcctName", 
	SUM(T0."Debit")as DEBIT,
	SUM(T0."Credit") as CREDIT
	 from JDT1 T0
Inner Join OACT T1
ON T0."Account" = T1."AcctCode" 
Where Cast("TaxDate" as date) between '2016-11-01' And '2016-11-30'
And "TransType" = 30 
And "U_NPedidoGera" is  null or "U_NPedidoGera"=''
And "Segment_1" = 'LPZ'
--And "Segment_1" = 'CBB'
--And "Segment_1" = 'SCZ'
Group by T0."TaxDate", T1."ActId", T1."AcctName"
Order By cast("TaxDate" as date), "ActId"
```
