#### TOC

* [Casos presentados en octubre 2017](#casos-presentados-en-octubre)
* [Caso 13 - Crédito a CCR por Estorno Ajuste Inicial](#caso-13)
    * [Procedimiento estorno ajuste inicial](#procedimiento-ai)
    * [Script entorno ajuste inicial](#script-ai)
* [Caso 15 - Crédito a CCR por Anulación de pedido facturado](#caso-15)
    * [Procedimiento](#procedimiento)
    * [Script](#script)



#### Casos presentados en octubre

![Imagen de casos presentados](/images/Casos.png)

#### Caso 13
#### Crédito a CCR por Estorno Ajuste Inicial
[Volver](#toc)
##### Descripción AI
Ocurre cuando se **anula una deuda**, ya sea esta con 
factura o sin factura y la deuda tiene ajuste inicial por CCR.   
Cuando se anula la deuda, el importe de los ajustes realizados, debe ir al 
Crédito de la cuenta pagos anticipados contra documentos por cobrar de la
consultora. 

|Cuenta                |Debit |Credit|
|----------------------|------|------|
|Documentos por Cobrar |   X  |      |
|Pagos Anticipados     |      |  X   |

**Nota:**    
Se consideran ajustes iniciales que tienen factura, los ajustes realizados 
a pedidos sin factura, no se toman en cuenta ya que el integrador hace lo mismo para los
ajustes iniciales.

##### Procedimiento AI
[Volver](#toc)

1. Buscar **CodigoTitulo** en la tabla **Deudas.PagamentoContaCorrenteNatBO_v1** con los filtros:

|Tipobaixa  |tipoRegistro|origem                    |databaixa            |
------------|------------|--------------------------|---------------------|
|Crédito CCR|Crédito CCR |Estorno Ajuste Inicial    |Parámetro @DesdeFecha| 

2. **Inner Join**  con la tabla **Deudas.TituloPadraoERPOut_v1** en el campo **CodigoTitulo** para obtener
el codigoPedido y con éste buscar la factura.


3. **Left Join** con la tabla **Facturas.NotaFiscalBoliviaPadraoERPOut_v1**  en el campo CodigoPedido para 
obtener la fecha de la factura si es que ésta existe.

4. Con la query completa, obtener:

    - De la tabla Deudas.PagamentoContaCorrenteNatBO_v1 los valores de:
        - CodigoPessoa
        - Valor (Importe de crédito/débito) a aplicar.

    - De la tabla Deudas.TituloPadraoERPOut_v1 los valores de:
        - CodigoPedido

Los datos de estructura comercial para determinar el segmento de oficina de la cuenta se obtienen de la
tabla Natura.Revendedores.CodigoEstruturaNivel1


##### Script AI
[Volver](#toc)

```SQL

Declare @DesdeFecha Date = '01/11/2017'

Select		CodigoPessoa, CodigoPedido, Fecha, Glosa, CodigoCuenta, Debit, Credit
From
(Select		P.CodigoPessoa, 
			T.CodigoPedido, 
			Fecha = Cast(T.Dataoperacao as DATE),
			Glosa = 'Credito CCR por Estorno Ajuste Inicial',
			CodigoCuenta = '1130201' + Case When R.CodigoEstruturaNivel1 = 2 Then 'SCZ' 
				When R.CodigoEstruturaNivel1 = 3 Then 'CBA'
				When R.CodigoEstruturaNivel1 = 4 Then 'LPZ'End,
			--R.DescricaoEstruturaNivel1,
			Debit = Valor,
			Credit = 0,
			T.TimesTamp
From		Intermedia.Deudas.PagamentoContaCorrenteNatBO_v1 P
Inner Join	Intermedia.Deudas.TituloPadraoERPOut_v1 T
On			P.CodigoTitulo = T.CodigoTitulo
Left Join   Intermedia.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
On			T.CodigoPedido = F.CodigoPedido
And			F.Situacao = 1
Left Join	Intermedia.Natura.Revendedores R
ON			R.CodigoPessoa = P.CodigoPessoa
Where		databaixa >= @DesdeFecha
And			TipoRegistro = 'Crédito CCR' 
And			TipoBaixa='Crédito CCR' 
And			Origem='Estorno Ajuste Inicial'
And			T.operacao = 1 --- 1803
And			F.Data <= T.DataOperacao 


union all

Select		P.CodigoPessoa, 
			T.CodigoPedido, 
			Fecha = Cast(T.Dataoperacao as DATE),
			Glosa = 'Credito CCR por Estorno Ajuste Inicial',
			CodigoCuenta = '2120501' + Case When R.CodigoEstruturaNivel1 = 2 Then 'SCZ' 
				When R.CodigoEstruturaNivel1 = 3 Then 'CBA'
				When R.CodigoEstruturaNivel1 = 4 Then 'LPZ'End,
			--R.DescricaoEstruturaNivel1,
			Debit = 0,
			Credit = Valor,
			T.TimesTamp
From		Intermedia.Deudas.PagamentoContaCorrenteNatBO_v1 P
Inner Join	Intermedia.Deudas.TituloPadraoERPOut_v1 T
On			P.CodigoTitulo = T.CodigoTitulo
Left Join   Intermedia.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
On			T.CodigoPedido = F.CodigoPedido
And			F.Situacao = 1
Left Join	Intermedia.Natura.Revendedores R
ON			R.CodigoPessoa = P.CodigoPessoa
Where		databaixa >=@DesdeFecha
And			TipoRegistro = 'Crédito CCR' 
And			TipoBaixa='Crédito CCR' 
And			Origem='Estorno Ajuste Inicial'
And			T.operacao = 1 
And			F.Data <= T.DataOperacao)X 

Order by	codigoPessoa, Timestamp, CodigoCuenta
```

#### Caso 15
#### Crédito a CCR por Anulación de pedido facturado
[Volver](#toc)
##### Descripción
Ocurre cuando se **anula una deuda**, ya sea esta con
factura o sin factura y la deuda tiene pagos parciales o totales
provenientes de la cobranza de alguna entidad o por ajuste inicial de CCR.   
Cuando se anula la deuda, el importe de los pagos o ajustes realizados, debe ir al
Crédito de la cuenta pagos anticipados contra documentos por cobrar de la
consultora. 

|Cuenta                |Debit |Credit|
|----------------------|------|------|
|Documentos por Cobrar |   X  |      |
|Pagos Anticipados     |      |  X   |

**Nota:**  
Se consideran los pagos realizados a pedidos que tienen factura, los pagos realizados 
a pedidos sin factura, no se toman en cuenta ya que el integrador hace lo mismo para los
ajustes iniciales y para los pagos parciales o totales los toma como Crédito a CCR por 
Pagos a Pedidos No Facturados.

##### Procedimiento
[Volver](#toc)

1. Buscar **CodigoTitulo** en la tabla **Deudas.PagamentoContaCorrenteNatBO_v1** con los filtros:   

|Tipobaixa  |tipoRegistro|origem                    |databaixa            |
------------|------------|--------------------------|---------------------|
|Crédito CCR|Crédito CCR |Anulación Pedido Facturado|Parámetro @DesdeFecha| 


2. **Inner Join**  con la tabla **Deudas.TituloPadraoERPOut_v1** en el campo **CodigoTitulo** para obtener 
el codigoPedido y con éste buscar la factura.


3. **Left Join** con la tabla **Facturas.NotaFiscalBoliviaPadraoERPOut_v1**  en el campo CodigoPedido para 
obtener la fecha de la factura si es que ésta existe.

4. Con la query completa, obtener: 

    - De la tabla Deudas.PagamentoContaCorrenteNatBO_v1 los valores de:
        - CodigoPessoa
        - Valor (Importe de crédito/débito) a aplicar.

    - De la tabla Deudas.TituloPadraoERPOut_v1 los valores de:
        - CodigoPedido

Los datos de estructura comercial para determinar el segmento de oficina de la cuenta se obtienen de la
tabla Natura.Revendedores.CodigoEstruturaNivel1 . 

Todas las transaciones se realizan dentro del plazo de vencimiento de la deuda, por lo tanto todos van a
la cuenta Documentos Por Cobrar **Vigente**



##### Script
[Volver](#toc)
```SQL

Declare @DesdeFecha Date = '01/11/2017'

Select		P.CodigoPessoa, 
			T.CodigoPedido, 
			Fecha = Cast(T.Dataoperacao as DATE),
			Glosa = 'Credito CCR por Anulación de pedido',
			CodigoCuenta = '1130201' + 
            Case When R.CodigoEstruturaNivel1 = 2 Then 'SCZ' 
				 When R.CodigoEstruturaNivel1 = 3 Then 'CBA'
				 When R.CodigoEstruturaNivel1 = 4 Then 'LPZ'End,
			Debit = Valor,
			Credit = 0
From		Intermedia.Deudas.PagamentoContaCorrenteNatBO_v1 P
Inner Join	Intermedia.Deudas.TituloPadraoERPOut_v1 T
On			P.CodigoTitulo = T.CodigoTitulo
Left Join   Intermedia.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
On			T.CodigoPedido = F.CodigoPedido
And			F.Situacao = 1
Left Join	Intermedia.Natura.Revendedores R
ON			R.CodigoPessoa = P.CodigoPessoa
Where		databaixa >= @DesdeFecha
And			TipoRegistro = 'Crédito CCR' 
And			TipoBaixa='Crédito CCR' 
And			Origem='Anulación Pedido Facturado'
And			T.operacao = 1 
And			F.Data is not null
And			Cast(T.Dataoperacao as date) >= @DesdeFecha


union all

Select		P.CodigoPessoa, 
			T.CodigoPedido, 
			Fecha = Cast(T.Dataoperacao as DATE),
			Glosa = 'Credito CCR por Anulación de pedido',
			CodigoCuenta = '2120501' + 
            Case When R.CodigoEstruturaNivel1 = 2 Then 'SCZ' 
				 When R.CodigoEstruturaNivel1 = 3 Then 'CBA'
				 When R.CodigoEstruturaNivel1 = 4 Then 'LPZ'End,
			Debit = 0,
			Credit = Valor 
From		Intermedia.Deudas.PagamentoContaCorrenteNatBO_v1 P
Inner Join	Intermedia.Deudas.TituloPadraoERPOut_v1 T
On			P.CodigoTitulo = T.CodigoTitulo
And			T.operacao = 1 
Left Join   Intermedia.Facturas.NotaFiscalBoliviaPadraoERPOut_v1 F
On			T.CodigoPedido = F.CodigoPedido
And			F.Situacao = 1
Left Join	Intermedia.Natura.Revendedores R
ON			R.CodigoPessoa = P.CodigoPessoa
Where		databaixa >=@DesdeFecha 
And			TipoRegistro = 'Crédito CCR' 
And			TipoBaixa='Crédito CCR' 
And			Origem='Anulación Pedido Facturado'
And			F.Data is not null
And			Cast(T.Dataoperacao as date) >= @DesdeFecha
Order by	CodigoPedido, CodigoCuenta
```
[Volver](#toc)
