```SQL
Select 			AL."TransId",
				AL."Line_ID",
				AL."DebCred",
				AL."LineMemo" As "Origen",
				Cast(AL."U_NUM_FACT" as Integer) as "Numero Factura",
				AL."U_NUMORDEN" As "Numero Autorizacion",
				Cast(AL."U_FECHAFAC" as Date) As "Fecha",
				AL."U_RUC" As "NIT",
				AL."U_CARDNAME" As "Razon Social",
				AL."U_CODALFA" As "cod. Control",
				Cast(AL."U_IMPORTE" As Decimal(10,2)) As "Monto",
				Cast(AL."U_ICE" As Decimal(10,2)) As "ICE",
				Cast(AL."U_EXENTO" As Decimal(10,2)) As "EXCENTO",
				Cast(AL."U_IMPORTE" As Decimal(10,2)) As "Neto",
				Round(AL."U_IMPORTE" * 0.13,2) As "IVA", --as Decimal(10,2)) As "IVA",
				AL."ContraAct" As "Consultora",
				AL."U_FACANU"
from 			OJDT AC
Inner Join 		JDT1 AL
On				AC."TransId" = AL."TransId"
Inner Join 		OACT ACT
ON 				ACT."AcctCode" = AL."Account"
Where 			AL."TaxDate" between '20171201' And '20171231'
And 			AC."U_AsInteres" = 'SI'
--And				AC."TransId" = 35277
And				ACT."Segment_0" = '2130101'
And AC."TransId" not in(Select "StornoToTr" From OJDT X where X."TaxDate" = AC."TaxDate" And X."StornoToTr" <> 0 And X."U_AsInteres"='SI')
And AC."StornoToTr" is null
Order by 		AL."LineMemo", AL."TaxDate",  Cast(AL."U_NUM_FACT" as Integer)
 
```
