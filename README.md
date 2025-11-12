# AdventureWorksDW2019

SELECT * FROM DimProduct d;
SELECT * FROM DimSalesTerritory ds;
SELECT * FROM FactInternetSales f;
SELECT * FROM DimCustomer dc;

-- Câu 1: Từ bảng DimProduct, DimSalesTeritory và FactInternetSales, hãy truy vấn ra các thông tin sau của các đơn hàng được đặt trong năm 2013 và 2014 
SELECT f.SalesOrderNumber, f.SalesOrderLineNumber, d.ProductKey,d.EnglishProductName, ds.SalesTerritoryCountry, f.SalesAmount, f.OrderQuantity
FROM DimProduct d
JOIN FactInternetSales f ON d.ProductKey = f.ProductKey
JOIN DimSalesTerritory ds ON f.SalesTerritoryKey = ds.SalesTerritoryKey
WHERE YEAR(f.OrderDate) = (2013 & 2014);

-- Câu 2: Từ bảng DimProduct, DimSalesTerritory và FactInternetSales, tính tổng doanh thu (đặt tên là InternetTotalSales) 
--và số đơn hàng (đặt tên là NumberofOrders) của từng sản phẩm theo mỗi quốc gia từ bảng DimSalesTerritory. Kết quả trả về gồm có các thông tin sau: Xem lai cau nay
SELECT ds.SalesTerritoryCountry, f.ProductKey, d.EnglishProductName, f.SalesOrderNumber,SUM(f.SalesAmount) AS InternetTotalSales, COUNT(f.OrderQuantity) AS NumberofOrders
FROM DimProduct d
JOIN FactInternetSales f ON d.ProductKey = f.ProductKey
JOIN DimSalesTerritory ds ON f.SalesTerritoryKey = ds.SalesTerritoryKey
GROUP BY ds.SalesTerritoryCountry, f.ProductKey, d.EnglishProductName, f.SalesOrderNumber, f.SalesAmount, f.OrderQuantity;

SELECT SalesTerritoryCountry
, InternetSales.ProductKey
, SUM(SalesAmount) as InternetTotalSales
, COUNT(InternetSales.ProductKey) as NumberOfOrders
FROM FactInternetSales AS InternetSales
LEFT JOIN DimProduct AS Product on Product.ProductKey = InternetSales.ProductKey
LEFT JOIN DimSalesTerritory AS Territory on Territory.SalesTerritoryKey = InternetSales.SalesTerritoryKey
GROUP BY SalesTerritoryCountry
, InternetSales.ProductKey
ORDER BY InternetSales.ProductKey, SalesTerritoryCountry;

-- Câu 3: Từ bảng DimProduct, DimSalesTerritory và FactInternetSales, hãy tính toán % tỷ trọng doanh thu của từng sản phẩm (đặt tên là PercentofTotaInCountry) 
-- trong Tổng doanh thu của mỗi quốc gia. Kết quả trả về gồm có các thông tin sau: #935 dong
SELECT ds.SalesTerritoryCountry, d.ProductKey, d.EnglishProductName, f.SalesAmount AS InternetTotalSales, (((f.SalesAmount - f.TotalProductCost)/f.SalesAmount)*100) AS PercentofTotalInCountry
FROM DimProduct d
JOIN FactInternetSales f ON f.ProductKey = d.ProductKey
JOIN DimSalesTerritory ds ON ds.SalesTerritoryKey = f.SalesTerritoryKey
GROUP BY ds.SalesTerritoryCountry, d.ProductKey, d.EnglishProductName, f.SalesAmount, (((f.SalesAmount - f.TotalProductCost)/f.SalesAmount)*100);

WITH TotalCountry AS
(
SELECT  SalesTerritoryCountry
, SUM(SalesAmount) as TotalByCoutry
FROM FactInternetSales AS InternetSales
LEFT JOIN DimSalesTerritory AS Territory on Territory.SalesTerritoryKey = InternetSales.SalesTerritoryKey
GROUP BY SalesTerritoryCountry
)

SELECT  Territory.SalesTerritoryCountry
, InternetSales.ProductKey
, SUM(SalesAmount) as InternetTotalSales
, COUNT(InternetSales.ProductKey) as NumberOfOrders
, Format(SUM(SalesAmount)/TotalByCoutry ,'P') as PercentofTotaInCountry 
FROM FactInternetSales AS InternetSales
LEFT JOIN DimProduct AS Product on Product.ProductKey = InternetSales.ProductKey
LEFT JOIN DimSalesTerritory AS Territory on Territory.SalesTerritoryKey = InternetSales.SalesTerritoryKey
LEFT JOIN TotalCountry ON TotalCountry. SalesTerritoryCountry = Territory.SalesTerritoryCountry
GROUP BY  Territory.SalesTerritoryCountry
, InternetSales.ProductKey
, TotalByCoutry
ORDER BY   Territory.SalesTerritoryCountry;


-- Cau 4: Từ bảng FactInternetSales, và DimCustomer, hãy truy vấn ra danh sách top 3 khách hàng có tổng doanh thu tháng 
--(đặt tên là CustomerMonthAmount) cao nhất trong hệ thống theo mỗi tháng.  Kết quả trả về gồm có các thông tin sau
WITH a AS (
	SELECT dc.FirstName + dc.LastName AS Fullname, 
           dc.CustomerKey, 
           YEAR(f.ShipDate) AS YEAR, 
           MONTH(f.ShipDate) AS MONTH, 
           SUM(f.SalesAmount) AS CustomerMonthAmount, 
           ROW_NUMBER() OVER (PARTITION BY YEAR(f.ShipDate), MONTH(f.ShipDate) ORDER BY SUM(f.SalesAmount) DESC) AS RankInMonth
	FROM FactInternetSales f
	JOIN DimCustomer dc ON dc.CustomerKey = f.CustomerKey
	GROUP BY dc.CustomerKey, dc.FirstName + dc.LastName, YEAR(f.ShipDate), MONTH(f.ShipDate)
    )
SELECT YEAR, MONTH, Fullname, CustomerMonthAmount, RankInMonth
FROM a 
WHERE RankInMonth <= 3
ORDER BY YEAR, MONTH;

--Câu 5: (1đ)Từ bảng FactInternetSales,  tính toán tổng doanh thu theo từng tháng (đặt tên là InternetMonthAmount). Kết quả trả về gồm có các thông tin sau:
SELECT YEAR(f.ShipDate) AS Year, MONTH(f.ShipDate) AS Month, SUM(f.SalesAmount) AS InternetMonthAmount
FROM FactInternetSales f
GROUP BY YEAR(f.ShipDate), MONTH(f.ShipDate)
ORDER BY YEAR(f.ShipDate), MONTH(f.ShipDate);

--Câu 6: (1đ)Từ bảng FactInternetSales hãy tính toán % tăng trưởng doanh thu (đặt tên là PercentSalesGrowth) so với cùng kỳnăm trước (ví dụ:Tháng 11 năm 2012 thì so sánh với tháng 11 năm 2011). Kết quả trả   về gồm có các thông tin sau:
WITH MonthlySales AS (
    SELECT
        YEAR(OrderDate) AS OrderYear,
        MONTH(OrderDate) AS OrderMonth,
        SUM(SalesAmount) AS SalesAmount
    FROM FactInternetSales
    GROUP BY YEAR(OrderDate), MONTH(OrderDate)
),
     WithPrevYear AS (
    SELECT
        cur.OrderYear,
        cur.OrderMonth,
        cur.SalesAmount,
        prev.SalesAmount AS PrevYearSales,
        CASE 
            WHEN prev.SalesAmount IS NULL THEN NULL
            ELSE ROUND(100.0 * (cur.SalesAmount - prev.SalesAmount) / prev.SalesAmount, 2)
        END AS PercentSalesGrowth
    FROM MonthlySales cur
    LEFT JOIN MonthlySales prev
        ON cur.OrderMonth = prev.OrderMonth
       AND cur.OrderYear = prev.OrderYear + 1
)
SELECT 
    OrderYear,
    OrderMonth,
    SalesAmount,
    PrevYearSales,
    PercentSalesGrowth
FROM WithPrevYear
ORDER BY OrderYear, OrderMonth;

WITH SalesCal AS
(
SELECT YEAR(OrderDate) as OrderYear
, MONTH(OrderDate) AS OrderMonth
, SUM(SalesAmount) as InternetMonthAmount
FROM FactInternetSales
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
)
SELECT *
, LAG(InternetMonthAmount,12,0) OVER (ORDER BY  OrderYear,OrderMonth) as InternetMonthAmount_LastYear
, CASE
WHEN LAG(InternetMonthAmount,12,0) OVER (ORDER BY  OrderYear,OrderMonth) =0 then 'Not Enough Data'
ELSE FORMAT((InternetMonthAmount-LAG(InternetMonthAmount,12,0) OVER (ORDER BY  OrderYear,OrderMonth))/LAG(InternetMonthAmount,12,0) OVER (ORDER BY  OrderYear,OrderMonth),'P') 
 END as PercentSalesGrowth
FROM SalesCal;
