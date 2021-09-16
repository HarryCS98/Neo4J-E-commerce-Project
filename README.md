# Neo4J Project for e-commerce data

### Contents of project
- Data Set
- Graphical Model
- Database creation
- Database Query's

## Data Set
https://github.com/HarryCS98/Neo4J-E-commerce-Project/blob/main/e-commerce-2021.csv



## Graphical Model

**Explanation:**

My Model for my Database contains three nodes and two relationships. I have a Customer node this contains the country the customer is from and a CustomerID this customerID. By having only Country and CustomerID in the customer node we can eliminate the need to repeat any customer nodes reducing the space and complexity of the database. The Customer node is connected to the Order node by an ordered relationship. This relationship does not contain any properties and is just used to allow us to connect the customer nodes to their multiple Order nodes. Our order nodes contain the invoice date and the invoice number. By only having the invoice date and the invoice number contained within the Order node it means that we can group all order with the same Invoice number into one Order node rather than having order nodes with extra properties which would lead us to having to have multiple order nodes for the same order. There is a small cavate here and that is that there is one order that spans over a two minute period which means that some orders with that InvoiceNo being in the first minute and some in the second this means in this one case there are two nodes created for the same order this can be fixed by removing the time from the InvoiceDate where it then just creates one order node but I weighed up the pros and cons and I thought it was more beneficial to have the time in the database and have one extra node then loose out by not having any time data within the database. The contains relationship connects the order and produce node. This relationship takes advantage of the fact that we can put properties on the relationship rather as well as the nodes this means we can move the Quantity and Unit Price properties onto the relationship between each order and the product that has been ordered this allows us to only have one product node for every product in the database rather than having duplicate product nodes for every order which contain the quantity and unit price. I originally only moved the quantity to the contains relationship but then I found out that there are some instances in the database where the same product could have a different price (this was probably due to a special offer or a price reduction) Once I realised this I moved the Unit Price over to the contain relationship as well. Finally, my product node contains a description of the product and its stockcode this means as I said before that we can reduce the amount of product nodes down to one product per product node reducing the size and complexity of the database.


![Graphical model](https://i.gyazo.com/ca03955715eeb212523921c5a8c7ae17.png)



## Database Creation

The first thing we need to do to start the process of creating our database is import the CSV file that contains our data. We do that using the command LOAD CSV WITH HEADERS FROM "file:///e-commerce-2021.csv" AS row this loads our csv from our file that is placed inside our import folder note the HEADERS keyword that is located after the LOAD CSV keyword this lets the Neo4j know that the csv file we are importing has headers for each of it columns. You may also note that we use the AS row at the end of our load command this means we can use the row key word to access that row of data from the csv file when we are creating our nodes and relationships. Next we need to start creating our customer node we do this by using the command MERGE (c:Customer{Country:row.Country,CustomerID:toInteger(row.CustomerID)}). The MERGE keyword lets us know that if there is another Customer node that matches exactly the Customer node we are creating it will just skip over it this is important as the same CustomerID appears multiple times within the data and if we just used the create command we would get a ton a replica nodes that would bung up our database. Next note that Customer label that we use this sets the label of the node and then we follow that up with the curly brackets this is where we can add in our properties for a node or relationship in this case for all the Customer nodes we are going to have a Country property and a CustomerID property and we are first going to set the Country property by using row.Country. As I stated above we can use the row key word to access that row of data from the csv file and then as our csv file has headers we can use the name of the header which is Country to access the specific value. We do the same again for the CustomerID  property apart from this time we take advantage of the toInteger method which allows us to turn a string into an integer (all information from the csv is initially imported as strings and needs to be converted to its appropriate datatype) which is very useful as it means we don’t need to use the method later in our queries where it would increase the complexity of the query’s. In the next part we need to create the Order nodes. We do this by using the command MERGE (o:Order{InvoiceNo:row.InvoiceNo,InvoiceDate: apoc.date.format(apoc.date.parse(row.InvoiceDate, 'ms', 'MM/dd/yyyy HH:mm'), 'ms', 'yyyy-MM-dd HH:mm')}). As with the previous two example we use the MEREGE command instead of the CREATE command, so we don’t have duplicates. In the order node we have two properties these are InvoiceNo and InvoiceDate. We import InvoiceNo as we have our other properties before using the row key word then using the full stop aggregator to add in our header which is this case is InvoiceNo. Now you might be wondering why we haven’t used the toInteger function on this by the sounds of Invoice number you might quite rightly think it contains numbers and therefore should be an interger but when looking through the data we can see that some InvoiceNo contain letters which means we can not use the toInteger method on this property and it needs to be left as a string. The next part of this command seems rather complicated as we need to turn a date and time from a string into a date to do this we need to use the apoc.date.format function but this only takes in actual dates so we then need to use the apoc.date.parse function to read our string which then lets our apoc.date.format function format it into a date we can use later on in our query’s. Something else to note in here is that we include the month before the date when parsing the data string as our data is in the American date format which has the month and then the day. Next we need to create the product nodes again we use the merge command to make sure there are no duplicates. For importing the data in we use the same row and the header name to import for both our properties which are StockCode and Description. Again for stock code you may ask why we don’t use the toInteger method but again some of the stockcodes contain letter which means we have to leave them as strings this is probably due to some products having the same stockcode number but with added letters to help identify different version of the same product i.e. different colours of a product or different patterns. At this point we have created all three of our node types for our database and now we need to create the relationships between them the first relationship we are going to create is the ORDERED relationship this is between the customer nodes and the order nodes this has no special properties so we can just set the label to ORDERED also note here that we use the variables that we assigned to our nodes earlier (c for the customer nodes and o for the order nodes) and we can use these in brackets which signify nodes to create a relationship between both of them also note the arrow head on one side of the ORDERED relationship this tells Neo4j which way the relationship goes. Finally we need to create the CONTAINS relationship this relationship is created the same way as our ORDERED relationship apart from it is between the order and product nodes and that it use create rather than merge this is because there are some orders that contain the same product more than once (see figure 1) so if we use MERGE command it will see that the relationship is a duplicate as there will be multiple CONTAINS relationships going to the same node and in the case of order 537224 the quantity and the unit price are the same as well this means this relationship would not be created as the merge keyword checks for duplicates and therefore the database would not accurately show the data that was given to us. Another difference is that we use the o and p variables rather than the c and o also note that this relationship contains properties so much like with our nodes we add the label and then use the curly brackets to add in our properties. In these curly brackets we can also see a new function the toFloat function this function turns a string into a float. You may ask why we are not using the toInteger function like with the Quantity property and this is because Integer as a datatype does not support decimal points and as the Unitprice Property contains the value of the product in monetary terms it is given to us with decimal points and therefore we need to reflect this in the database hence the use of the to float method and not the toInteger method. 

[![Image from Gyazo](https://i.gyazo.com/01856af1d0efb90f352d243b1f5ce0d7.png)](https://gyazo.com/01856af1d0efb90f352d243b1f5ce0d7)



## Database Query's


## 1.

### Question:

Display all the amount of unique countries the customers are from.

### Method:

To display the amount of unique countries the customers are from we first need to match to all the customer nodes we can do this using the MATCH (c:Customer) and then we need to return the amount of unique countries so we use the RETURN command and then the COUNT function which lets us count the amount of countries but then we need to use the DISTINCT key word else we will count some countries more than once

### Final query:

MATCH (c:Customer) RETURN COUNT(DISTINCT c.Country)

[![Image from Gyazo](https://i.gyazo.com/063b7c8979c46212ca8ebba30bf50764.png)](https://gyazo.com/063b7c8979c46212ca8ebba30bf50764)




## 2.

### Question:

Display the number of different products.

### Method:

To get the number of different products we first need to match to all the product nodes so we use MATCH (p:Product) then we can simply RETURN the amount of unique products COUNT(DISTINCT p.StockCode) note here that we have to use an unique value from our product node as some nodes have the same StockCode but different descriptions so if we counted the nodes we would get an incorrect number (see below).

[
![Image from Gyazo](https://i.gyazo.com/a6e9e2fb7188971648e8f4a0061c96d2.png)](https://gyazo.com/a6e9e2fb7188971648e8f4a0061c96d2)



### Final query:

MATCH (p:Product) RETURN COUNT(DISTINCT p.StockCode)



## 3.

### Question:

Show the most expensive product.

### Method:

To get the most expensive product in the database we first need to match to all the order nodes this is because the price of a product is held within the relationship between an order and a product once we have matched both the order and the product we can get the value of the product from the UnitPrice Property in the CONTAINS relationship we then need to return the UnitPrice and the stock code of the product which we get from the product node and then finally we can order by the UnitPrice values in descending order (DESC) and limit to one which will give us the most expensive product.

### Final query:

MATCH (o:Order)-[c:CONTAINS]->(p:Product) RETURN c.UnitPrice,p.StockCode  ORDER BY c.UnitPrice DESC LIMIT 1

## 4.

### Question:

Which month did the customers spend the most money?

### Method:

First we need to get all the order and product nodes in the database along with all CONTAINS relationships which hold our UnitPrices and our Quantity’s. Which we can then multiply together to get the total cost of an order. When then need to take the month in which that product was bought so we can total up the total cost of that month so we take our invoice date and split it at every hyphen which then gives us a list stored in the month variable we can then return the month list with the number one passed into it which gives us the second number in the list which is the month we then combine this into a variable with the total cost of all order for that much using the sum function and our cost variable which is made up of multiplying the Quantity and Unitprice in each order and then be we order that variable (total amount made up of the month and the sum) in decending order and limit it to the top result which gives us the month where the customers spent the most money.

### Final query:

MATCH (o:Order)-[con:CONTAINS]->(p:Product)

WITH con.Quantity * con.UnitPrice AS cost, split(o.InvoiceDate,'-') AS month

RETURN month[1], sum(cost) AS totalamount

ORDER BY totalamount DESC LIMIT 1


## 5.

### Question:

What’s the most popular product (i.e., most customers purchased it)? note: purchase quantity of a product in a single order doesn’t matter; also, same customer purchasing the same product in any future order doesn’t count.

### Method:

For this question we need to get the most popular product this is defined in the question as how many unique people bought the product so it did not matter about the amount of product that was bought in an order or if the same customer was buying the product many times. So first we have to map of all our nodes and relationships as we will need information from both the customer and the product next need to only be testing unique products and unique Customers so we use the distinct keyword at the beginning of the statement as well is in our count function which counts the customerIDs once we have got all unique products we can simply count how many unique customerIDs have ordered that product then order by the variable amount which holds the count of the unique customerids. Once we have that we can simply put that is decending order and limit to one which gives us the most popular product stock code and the amount.

### Final query:

MATCH(c:Customer)-[ord:ORDERD]->(o:Order)-[con:CONTAINS]->(p:Product)

WITH DISTINCT COUNT(DISTINCT c.CustomerID) AS amount, p.StockCode AS stockcode

RETURN stockcode,amount

ORDER BY amount DESC LIMIT 1




## 6.

### Question:

What did the customers buy together with the most popular product?

### Method:

For this question we first need to find the most popular product which is the same as the last question we can then take the invoice number from all the orders that contain the most popular product and then we can match order that contain that invoice number and then return all of the products that are also contained in those orders.

### Final query:

MATCH(c:Customer)-[ord:ORDERD]->(o:Order)-[con:CONTAINS]->(p:Product)

WITH DISTINCT COUNT(DISTINCT c.CustomerID) AS amount, p.StockCode AS stockcode ORDER BY amount DESC LIMIT 1

MATCH(:Product{StockCode: stockcode})<-[:CONTAINS]-(ord:Order)

WITH ord.InvoiceNo AS invoicenum

MATCH (o:Order{InvoiceNo: invoicenum})-[:CONTAINS]->(p:Product)

RETURN DISTINCT p


## 7.

### Question:

Which customer spent the most money?

### Method:

To get the customer who spent the most money first we need to get all unique customers we then we need to take every order they made and then multiply the quantity and unit price together for each product within each order we can then save this to a variable in our case total put that in descending order and limit to one which gives us the customerid and the total amount the top customer spent.

### Final query:

MATCH (c:Customer)-[:ORDERD]->(o:Order)-[con:CONTAINS]->(p:Product)

RETURN DISTINCT c.CustomerID, sum(con.Quantity * con.UnitPrice) AS total ORDER BY total DESC

LIMIT 1


## 8.

### Question:

Which customers increased spending from the first purchase to the last

### Method:

To get the custromers that increased their spending in each order they placed we first need to collect all the orders that one customer placed then get the amount that each one of these orders cost we can then place them in a list using the collect method and then we can check the order that is one place ahead of the current order in the list is greater than the one we are currently looking at in the list and then return all the customers where this applies. Then finally we need to check to make sure that the customer has more than one order we do this by looking at the list we generated for them and seeing if it is greater than one if it’s not we discard that user.

### Final query:

MATCH (c:Customer)-[:ORDERD]->(o:Order)-[con:CONTAINS]->(p:Product)

WITH DISTINCT c.CustomerID AS Customer, o AS ord, SUM(con.Quantity * con.UnitPrice) AS sumover

ORDER BY ord.InvoiceNo ASC

WITH collect(sumover) AS list, Customer AS customers

WHERE all(i in range(0, size(list) - 2) WHERE list[i] < list[i + 1]) AND size(list) > 1

RETURN customers




