### TASK 1
use classicmodels;

## TASK 1.1 - Single Table Queries
# QUERY 1
# Check the amount of Customers in each country in order to get insights into the market shares.
select country, count(customerNumber) as AmountOfCustomers
from customers
group by country
order by AmountOfCustomers desc;

# QUERY 2
# TOP 5 productCodes which are the best selling products in terms of Quantity (Volume)
select productCode, sum(quantityOrdered) as TotalQuantityOrdered
from orderdetails
group by productCode
order by TotalQuantityOrdered desc
limit 5;

# QUERY 3
# TOP 5 productCodes which are the best selling products in terms of Total Sales Revenue
select productCode, sum(priceEach*quantityOrdered) as TotalSalesRevenue
from orderdetails
group by productCode
order by TotalSalesRevenue desc
limit 5;

# QUERY 4
# The total amount (= Total Quantity in Stock) and the total Value (Total Current Assets) 
# of products that the company currently has in stock.
select sum(quantityInStock) as TotalQuantityInStock, sum(quantityInStock*buyPrice) as TotalValueInventory
from products
order by TotalValueInventory desc;

## TASK 1.2 - Queries using
# QUERY 1
# Display the number of days it takes per country to process the order. (Start: order date, End: shipped Date)
select p.productName, o.orderDate, o.shippedDate, datediff(o.shippedDate, o.orderDate) as LeadTimes
from orders o
join orderdetails od
using (orderNumber)
join products p
where o.shippedDate is not Null
order by LeadTimes desc;

# QUERY 2
# Display the Number of Employees located in the offices within each country
select o.country, count(e.employeeNumber) as NumberOfEmployees
from employees e
join offices o
using (officeCode)
group by o.country
order by NumberOfEmployees desc;

# QUERY 3 (Additional Query)
# Display of the Gross Margin for each Product Name in descending order in order to see the potentially most profitable products
select p.productName, sum(od.priceEach)-sum(p.buyPrice) as GrossMargin
from products p
join orderdetails od
using (productCode)
group by productName
order by GrossMargin desc;

# QUERY 4 (Additional Query)
# Top 5 worst performing sales representatives within the company in terms of their sales generated
# This is useful in order to know which sales representatives might need additional training or other support
select e.firstName, e.lastName, SUM(p.amount) as TotalSales
from Employees e
join Customers c on e.employeeNumber = c.salesRepEmployeeNumber
join Payments p on c.customerNumber = p.customerNumber
group by e.employeeNumber
order by totalSales asc
limit 5;

## TASK 1.3 - Subqueries
# QUERY 1
# The number of different products available in each product line (= different types of vehicles)
select pl.productLine, productCount.numProducts
from productlines pl
join (select p.productLine, count(*) as numProducts
      from products p
      group by p.productLine) as productCount
using (productLine);

# QUERY 2
# Check if there are products which have never been ordered
select productName
from products
where productCode not in (
	select distinct productCode
	from orderdetails);

# QUERY 3 (Additional Query)
# List the customers who placed more orders than the average number of orders placed (by customer)
select customerName, orderCount
from (
    select c.customerName, COUNT(o.orderNumber) as orderCount
    from Customers c
    join Orders o
    using (customerNumber)
    group by c.customerName
    ) as customerOrders
where orderCount > (select avg(orderCount)
                    from (
                        select count(o2.orderNumber) as orderCount
                        from Customers c2
                        join Orders o2
                        using (customerNumber)
                        group by c2.customerName
                    ) as avgOrders)
order by orderCount desc;


## TASK 1.4 - Windows Function
# QUERY 1
# The cumulated Sales by each Product Line (= Type of Vehicle)
select pl.productLine,
       sum(sum(od.quantityOrdered * od.priceEach)) over (partition by pl.productLine) as cumulativeSales
from Orders o
join OrderDetails od
using (orderNumber)
join Products p
using (productCode)
join ProductLines pl
using (productLine)
group by pl.productLine
order by cumulativeSales desc;

# QUERY 2
# Identify the top-performing employees in sales by quarter
select e.firstName, e.lastName, sum(p.amount) as quarterlySales,
       year(p.paymentDate) as paymentYear, quarter(p.paymentDate) as paymentQuarter,
       rank() over (partition by year(p.paymentDate), quarter(p.paymentDate)
                    order by sum(p.amount) desc) as salesRank
from Employees e
join Customers c
on e.employeeNumber = c.salesRepEmployeeNumber
join Payments p
using (customerNumber)
group by e.employeeNumber, year(p.paymentDate), quarter(p.paymentDate);

