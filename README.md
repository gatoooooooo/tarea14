# tarea 14
##### **creando la base de datos DBventas**
```sql
USE MASTER
GO
```
```sql
SET DATEFORMAT YMD
GO
```
```sql
IF DB_ID('BDVENTAS') IS NOT NULL
    DROP DATABASE BDVENTAS
GO
```
```sql
CREATE DATABASE BDVENTAS
GO
```
##### **uso de  DBventas**
```sql
USE BDVENTAS
GO
```
#### **crear las tablas**
**DISTRITO**
```sql
CREATE TABLE DISTRITO(
    COD_DIS CHAR(5) NOT NULL PRIMARY KEY,
    NOM_DIS VARCHAR(50) NOT NULL
)
GO
```
**VENDEDOR**
```sql
CREATE TABLE VENDEDOR(
    COD_VEN CHAR(3) NOT NULL PRIMARY KEY,
    NOM_VEN VARCHAR(20) NOT NULL,
    APE_VEN VARCHAR(20) NOT NULL,
    SUE_VEN MONEY NOT NULL,
    FTI_VEN DATE NOT NULL,
    TIP_VEN VARCHAR(10) NOT NULL,
    COD_DIS CHAR(5) NOT NULL REFERENCES DISTRITO
)
GO
```
**CLIENTE**
```sql
CREATE TABLE CLIENTE(
    COD_CLI CHAR(5) NOT NULL PRIMARY KEY,
    RSO_CLI CHAR(30) NOT NULL,
    DIR_CLI VARCHAR(100) NOT NULL,
    TLF_CLI CHAR(9) NOT NULL,
    RUC_CLI CHAR(11) NULL,
    COD_DIS CHAR(5) NOT NULL REFERENCES DISTRITO,
    FEC_REG DATE NOT NULL,
    TIP_CLI VARCHAR(10) NOT NULL,
    CON_CLI VARCHAR(30) NOT NULL
)
GO
```
**PROVEEDOR**
```sql
CREATE TABLE PROVEEDOR(
    COD_PRV CHAR(5) NOT NULL PRIMARY KEY,
    RSO_PRV VARCHAR(80) NOT NULL,
    DIR_PRV VARCHAR(100) NOT NULL,
    TEL_PRV CHAR(15) NULL,
    COD_DIS CHAR(5) NOT NULL REFERENCES DISTRITO,
    REP_PRV VARCHAR(80) NOT NULL
)
GO
```
**FACTURA**
```sql
CREATE TABLE FACTURA(
    NUM_FAC VARCHAR(12) NOT NULL PRIMARY KEY,
    FEC_FAC DATE NOT NULL,
    COD_CLI CHAR(5) NOT NULL REFERENCES CLIENTE,
    FEC_CAN DATE NOT NULL,
    EST_FAC VARCHAR(10) NOT NULL,
    COD_VEN CHAR(3) NOT NULL REFERENCES VENDEDOR,
    POR_IGV DECIMAL NOT NULL
)
GO
```
**ORDEN_COMPRA**
```sql
CREATE TABLE ORDEN_COMPRA(
    NUM_OCO CHAR(5) NOT NULL PRIMARY KEY,
    FEC_OCO DATE NOT NULL,
    COD_PRV CHAR(5) NOT NULL REFERENCES PROVEEDOR,
    FAT_OCO DATE NOT NULL,
    EST_OCO CHAR(1) NOT NULL
)
GO
```
**PRODUCTO**
```sql
CREATE TABLE PRODUCTO(
    COD_PRO CHAR(5) NOT NULL PRIMARY KEY,
    DES_PRO VARCHAR(50) NOT NULL,
    PRE_PRO MONEY NOT NULL,
    SAC_PRO INT NOT NULL,
    SMI_PRO INT NOT NULL,
	UNI_PRO VARCHAR(30) NOT NULL, 
    LIN_PRO VARCHAR(30) NOT NULL,
    IMP_PRO VARCHAR(10) NOT NULL
)
GO
```
**DETALLE_FACTURA**
```sql
CREATE TABLE DETALLE_FACTURA(
    NUM_FAC VARCHAR(12) NOT NULL REFERENCES FACTURA,
    COD_PRO CHAR(5) NOT NULL REFERENCES PRODUCTO,
    CAN_VEN INT NOT NULL,
    PRE_VEN MONEY NOT NULL,
    PRIMARY KEY (NUM_FAC, COD_PRO)
)
GO
```
**DETALLE_COMPRA**
```sql
CREATE TABLE DETALLE_COMPRA(
    NUM_OCO CHAR(5) NOT NULL REFERENCES ORDEN_COMPRA,
    COD_PRO CHAR(5) NOT NULL REFERENCES PRODUCTO,
    CAN_DET INT NOT NULL,
    PRIMARY KEY (NUM_OCO, COD_PRO)
)
GO
```
**ABASTECIMIENTO**
```sql
CREATE TABLE ABASTECIMIENTO(
    COD_PRV CHAR(5) NOT NULL REFERENCES PROVEEDOR,
    COD_PRO CHAR(5) NOT NULL REFERENCES PRODUCTO,
    PRE_ABA MONEY NOT NULL,
    PRIMARY KEY (COD_PRV, COD_PRO)
)
GO
```
**CONSULTAS SIMPLES**
1) Consultar todos los distritos
```sql
SELECT * FROM DISTRITO;
```
2) Consultar todos los vendedores
```sql
SELECT * FROM VENDEDOR;
```
3) Consultar los clientes de un distrito específico
```sql
SELECT * FROM CLIENTE
WHERE COD_DIS = 'D01';
```
4) Consultar las facturas de un cliente específico
```sql
SELECT * FROM FACTURA
WHERE NUM_FAC = 'FA001';
```
5) onsultar los productos de un proveedor específico
```sql
SELECT * FROM PRODUCTO
WHERE COD_PRO = 'P001';
```
#### CONSULTAS AVANZADAS
1) Obtener el total de ventas (SUM de los importes) por cada vendedor
```sql
SELECT V.COD_VEN, V.APE_VEN, V.NOM_VEN, SUM(F.POR_IGV) AS TotalVentas
FROM VENDEDOR V
JOIN FACTURA F ON V.COD_VEN = F.COD_VEN
GROUP BY V.COD_VEN, V.APE_VEN, V.NOM_VEN
ORDER BY TotalVentas DESC;
```
2) Consultar los productos que no han sido vendidos en ningún detalle de factura
```sql
SELECT P.COD_PRO, P.DES_PRO
FROM PRODUCTO P
LEFT JOIN DETALLE_FACTURA DF ON P.COD_PRO = DF.COD_PRO
WHERE DF.COD_PRO IS NULL;
```
3) Obtener el cliente con el mayor número de compras (cantidad de facturas)
```sql
SELECT C.COD_CLI, C.CON_CLI, COUNT(F.NUM_FAC) AS NumeroCompras
FROM CLIENTE C
JOIN FACTURA F ON C.COD_CLI = F.COD_CLI
GROUP BY C.COD_CLI, C.CON_CLI
ORDER BY NumeroCompras DESC
```
4) Obtener el total de productos vendidos y el importe total por cada factura
```sql
SELECT F.NUM_FAC, F.FEC_FAC, COUNT(DF.COD_PRO) AS TotalProductos, SUM(DF.CAN_VEN * DF.PRE_VEN) AS ImporteTotal
FROM FACTURA F
JOIN DETALLE_FACTURA DF ON F.NUM_FAC = DF.NUM_FAC
GROUP BY F.NUM_FAC, F.FEC_FAC
ORDER BY ImporteTotal DESC;
```
5) Obtener los nombres y apellidos de los vendedores que han vendido productos de un proveedor específico (por ejemplo, 'P01')
```sql
SELECT DISTINCT V.COD_VEN, V.APE_VEN, V.NOM_VEN
FROM VENDEDOR V
JOIN FACTURA F ON V.COD_VEN = F.COD_VEN
JOIN DETALLE_FACTURA DF ON F.NUM_FAC = DF.NUM_FAC
JOIN PRODUCTO P ON DF.COD_PRO = P.COD_PRO
WHERE P.COD_PRO = 'P001';
```
#### INSERT
1) Insertar un nuevo cliente en la tabla CLIENTE
```sql
INSERT INTO CLIENTE VALUES('C0021','FINSETH','AV. LOS PINOS 150','4644217','48632881','D05',CONVERT(DATE,'04/03/2004', 103),'1','GONZALO MORANTE');
```
2) Insertar un nuevo producto en la tabla PRODUCTO
```sql
INSERT INTO PRODUCTO (COD_PRO, DES_PRO, PRE_PRO, COD_PRO)
VALUES ('P0022', 'Cuaderno 27 pulgadas', 10, 'P30');
```
3)Insertar una nueva factura y sus detalles en las tablas FACTURA y DETALLE_FACTURA
```sql
INSERT INTO FACTURA (NUM_FAC, FEC_FAC, POR_IGV, COD_VEN, COD_CLI)
VALUES ('F005', '2024-06-20', 750.00, 'V03', 'C101');

INSERT INTO DETALLE_FACTURA (NUM_FAC, COD_PRO, CAN_VEN, PRE_VEN)
VALUES 
('F005', 'PR101', 1, 500.00),
('F005', 'PR103', 2, 125.00);
```
#### UPDATE
1) Actualizar la dirección de un cliente en la tabla 'CLIENTE'
```sql
UPDATE CLIENTE
SET DIR_CLI = '123 Nueva Calle, Buenos Aires'
WHERE COD_CLI = 'C101';
```
2) Actualizar el precio de un producto en la tabla 'PRODUCTO'
```sql
UPDATE PRODUCTO
SET PRE_PRO = 275.00
WHERE COD_PRO = 'PR103';
```
3) Actualizar el importe de una factura en la tabla FACTURA
```sql
UPDATE FACTURA
SET POR_IGV = 800.00
WHERE NUM_FAC = 'F002';
```
#### DELETE
1)  Eliminar un cliente de la tabla 'CLIENTE'
```sql
DELETE FROM CLIENTE
WHERE COD_CLI = 'C102';
```
2) Eliminar un producto de la tabla 'PRODUCTO'
```sql
DELETE FROM PRODUCTO
WHERE COD_PRO = 'PR104';
```
3) Eliminar una factura de la tabla 'FACTURA'
```sql
DELETE FROM FACTURA
WHERE NUM_FAC = 'F003';
```
#### UNION Y UNION ALL
#### union
**Consulta para obtener todos los clientes de la ciudad de Madrid**
```sql
SELECT COD_CLI, CON_CLI, DIR_CLI
FROM CLIENTE
WHERE DIR_CLI = 'LOS VIÑEDOS 150'
UNION
SELECT COD_CLI, CON_CLI, DIR_CLI
FROM CLIENTE
WHERE DIR_CLI = 'JR. IQUIQUE 132';
```
#### union all
**Consulta para obtener todos los clientes de la ciudad de Madrid**
```sql
SELECT COD_CLI, CON_CLI, DIR_CLI
FROM CLIENTE
WHERE DIR_CLI = 'LOS VIÑEDOS 150'
UNION ALL
SELECT COD_CLI, CON_CLI, DIR_CLI
FROM CLIENTE
WHERE DIR_CLI = 'JR. IQUIQUE 132';
```
#### PROCEDIMIENTO
```sql
CREATE PROCEDURE ObtenerClientesPorDireccion
    @direccion NVARCHAR(100)
AS
BEGIN
    SELECT COD_CLI, CON_CLI, DIR_CLI
    FROM CLIENTE
    WHERE DIR_CLI = @direccion;
END;
```
#### PROCEDIMIENTO ALMACENADO
```sql
EXEC ObtenerClientesPorDireccion @direccion = 'LOS VIÑEDOS 150';
```
**CONSULTAS TRIGGERS**
1) Trigger para actualizar automáticamente el stock cuando se inserta una orden de compra
```sql
CREATE TRIGGER tr_UpdateStockOnInsertOrdenCompra
ON ORDEN_COMPRA
AFTER INSERT
AS
BEGIN
    DECLARE @CodigoOrden NVARCHAR(10);
    DECLARE @CodigoProducto NVARCHAR(10);
    DECLARE @Cantidad INT;
    
    SELECT @CodigoOrden = i.COD_PRV, 
           @CodigoProducto = i.COD_PRV, 
           @Cantidad = i.NUM_OCO
    FROM INSERTED i;

    UPDATE PRODUCTO
    SET @Cantidad = @Cantidad + @Cantidad
    WHERE COD_PRO = @CodigoProducto;
END;
```
2) Trigger para verificar que no se pueda eliminar un cliente que tiene facturas asociadas:
```sql
CREATE TRIGGER tr_PreventDeleteClienteWithFacturas
ON CLIENTE
INSTEAD OF DELETE
AS
BEGIN    IF EXISTS (
        SELECT 1
        FROM FACTURA f
        INNER JOIN DELETED d ON f.COD_CLI = d.COD_CLI
    )
    BEGIN
        RAISERROR ('No se puede eliminar un cliente que tiene facturas asociadas', 16, 1);
    END
    ELSE
    BEGIN
        DELETE FROM CLIENTE
        WHERE COD_CLI IN (SELECT COD_CLI FROM DELETED);
    END
END;
```
