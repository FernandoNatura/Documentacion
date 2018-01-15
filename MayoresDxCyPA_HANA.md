### Mayores Documentos x Cobrar y Pagos anticipados HANA  
Este script obtiene los mayores de las cuentas mencionadas por consultora
hasta la fecha.

Se ejecuta en el servidor 192.168.15.200 en el Hana studio.

```SQL
Select		
		"ShortName",
		A."FormatCode",
		--A."Segment_0",
		A."AcctName",
		AC."TransId",
		AL."Line_ID",
		AL."LineMemo",
		TO_CHAR(AC."TaxDate", 'dd/mm/yyyy') As "TaxDate",
		TO_CHAR(AC."CreateDate", 'dd/mm/yyyy') As "CreateDate",
		"U_NPedidoGera" As "PedidoGera",
		--Cast(AL."Debit" as Decimal(10,2)) As "Debit" ,
		--Cast(AL."Credit" as Decimal(10,2)) as "Credit" ,
		Cast(Coalesce((Case When A."Segment_0" like '113020%' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit_DXC",
	    Cast(Coalesce((Case When A."Segment_0" like '113020%' then Coalesce("Credit",0) End),0) as Decimal(10,2)) as "Credit_DXC", 
		Cast(Coalesce((Case When A."Segment_0" Like '212050%' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit_PA",
		Cast(Coalesce((Case When A."Segment_0" Like '212050%' then Coalesce("Credit",0) End),0) as Decimal(10,2)) As "Credit_PA"
FROM	OJDT AC
Inner Join	JDT1 AL
On		AC."TransId" = AL."TransId"
Inner Join	OACT A
On		AL."Account" = A."AcctCode" 
Where	"ShortName" = 'C52720'
And		AC."TaxDate" >= '20170531'
Order by	AC."TaxDate", AC."TransId", AL."Line_ID"
```

### Mayores de cuentas clasificadas por vencimiento

``` SQL
Select		
		"ShortName",
		A."FormatCode",
		--A."Segment_0",
		--A."AcctName",
		AC."TransId",
		AL."Line_ID",
		AL."LineMemo",
		TO_CHAR(AC."TaxDate", 'dd/mm/yyyy') As "TaxDate",
		--TO_CHAR(AC."CreateDate", 'dd/mm/yyyy') As "CreateDate",
		"U_NPedidoGera" As "PedidoGera",
		--Cast(AL."Debit" as Decimal(10,2)) As "Debit" ,
		--Cast(AL."Credit" as Decimal(10,2)) as "Credit" ,
		Cast(Coalesce((Case When A."Segment_0" = '1130201' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit Vigente",
	    Cast(Coalesce((Case When A."Segment_0" = '1130201' then Coalesce("Credit",0) End),0) as Decimal(10,2)) as "Credit Vigente", 
		Cast(Coalesce((Case When A."Segment_0" = '1130202' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit Vencida",
	    Cast(Coalesce((Case When A."Segment_0" = '1130202' then Coalesce("Credit",0) End),0) as Decimal(10,2)) as "Credit Vencida", 
		Cast(Coalesce((Case When A."Segment_0" = '1130203' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit Incobrable",
	    Cast(Coalesce((Case When A."Segment_0" = '1130203' then Coalesce("Credit",0) End),0) as Decimal(10,2)) as "Credit Incobrable", 
		Cast(Coalesce((Case When A."Segment_0" = '212050%' then Coalesce("Debit",0) End),0) as Decimal(10,2)) As "Debit_PA",
		Cast(Coalesce((Case When A."Segment_0" = '212050%' then Coalesce("Credit",0) End),0) as Decimal(10,2)) As "Credit_PA"
FROM	OJDT AC
Inner Join	JDT1 AL
On		AC."TransId" = AL."TransId"
Inner Join	OACT A
On		AL."Account" = A."AcctCode" 
Where	"ShortName" = 'C24849'
And		AC."TaxDate" <= '20171231'
--And 	A."Segment_0" = '1130201'
Order by	AC."TaxDate", AC."TransId", AL."Line_ID"

```
### Totales Mayores de cuentas clasificadas por vencimiento

``` SQL
Select	
		"ShortName",	
	    Sum(Cast(Coalesce((Case When A."Segment_0" = '1130201' then Coalesce("Debit",0) End),0) as Decimal(10,2)) -
	    Cast(Coalesce((Case When A."Segment_0" = '1130201' then Coalesce("Credit",0) End),0) as Decimal(10,2))) as "Total Vigente", 
		Sum(Cast(Coalesce((Case When A."Segment_0" = '1130202' then Coalesce("Debit",0) End),0) as Decimal(10,2)) -
	    Cast(Coalesce((Case When A."Segment_0" = '1130202' then Coalesce("Credit",0) End),0) as Decimal(10,2))) as "Total Vencida", 
		Sum(Cast(Coalesce((Case When A."Segment_0" = '1130203' then Coalesce("Debit",0) End),0) as Decimal(10,2)) -
	    Cast(Coalesce((Case When A."Segment_0" = '1130203' then Coalesce("Credit",0) End),0) as Decimal(10,2))) as "Total Incobrable", 
		Sum(Cast(Coalesce((Case When A."Segment_0" = '212050%' then Coalesce("Credit",0) End),0) as Decimal(10,2)) -
		Cast(Coalesce((Case When A."Segment_0" = '212050%' then Coalesce("Debit",0) End),0) as Decimal(10,2))) As "Total PA"
FROM	OJDT AC
Inner Join	JDT1 AL
On		AC."TransId" = AL."TransId"
Inner Join	OACT A
On		AL."Account" = A."AcctCode" 
Where	"ShortName" = 'C24849'
And		AC."TaxDate" <= '20171231'
--And 	A."Segment_0" = '1130201'
Group by	"ShortName"


```
