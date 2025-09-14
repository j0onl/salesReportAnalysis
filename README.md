<div>
<div>
<h1>Data Analysis of Profit and Sales - Power Query & Power BI<h1>
</div>
<div>
<h2>1. Introduction </h2>
<p>This project consists of a data analysis of data sales and profitability of a fictitious company (Superstore). 
The project uses Power Query to transform data and Power BI to display and extract the business insights.
The present analysis is based on data available in the dataset and on identified factors such as high discounts, product mix and regional differences. 
However the results may be influenced by other factors not captured in the data (marketing costs, external logistics, or internal company policies for example)." </p>
</div>

<div>
<h2>2. Dataset</h2>

<p>The source of data is easily found on Kaggle (https://www.kaggle.com/datasets/vivek468/superstore-dataset-final).
The original dataset contains the following columns: </p>
<table>
<tr> 
<th>Values</th>
<th> Description</th>
</tr>
<tr>
<td>Row ID</td> 
<td>Unique ID for each row.</td>
</tr>
<tr>
<td>Order ID </td>
<td>Unique Order ID for each Customer.</td>
</tr>
<tr>
<td>Order Date</td>
<td>Order Date of the product.</td>
</tr>
<tr>
<td>Ship Date</td>
<td>Shipping Date of the Product.</td>
</tr>
<tr>
<td>Ship Mode</td>
<td>Shipping Mode specified by the Customer.</td>
</tr>
<tr>
<td>Customer ID</td> 
<td>Unique ID to identify each Customer.</td>
</tr>
<tr>
<td>Customer Name</td> 
<td>Name of the Customer.</td>
</tr>
<tr>
<td>Segment</td> 
<td>The segment where the Customer belongs.</td>
</tr>
<tr>
<td>Country</td> 
<td>Country of residence of the Customer.</td>
</tr>
<tr>
<td>City</td>
<td>City of residence of of the Customer.</td>
</tr>
<tr>
<td>State</td> 
<td>State of residence of the Customer.</td>
</tr>
<tr>
<td>Postal Code</td>
<td>Postal Code of every Customer.</td>
</tr>
<tr>
<td>Region</td> 
<td>Region where the Customer belong.</td>
</tr>
<tr>
<td>Product ID</td> 
<td>Unique ID of the Product.</td>
</tr>
<tr>
<td>Category</td>
<td>Category of the product ordered.</td>
</tr>
<tr>
<td>Sub-Category </td>
<td>Sub-Category of the product ordered.</td>
</tr>
<tr>
<td>Product Name</td>
<td>Name of the Product</td>
</tr>
<tr>
<td>Sales</td>
<td>Sales of the Product.</td>
</tr>
<tr>
<td>Quantity</td> 
<td>Quantity of the Product.</td>
</tr>
<tr>
<td>Discount </td> 
<td>Discount provided.</td>
</tr>
<tr>
<td>Profit </td> 
<td>Profit/Loss incurred."</td>
</tr>
</table>
</div>

<div>
<h2>3. Transformed Data (Power Query - M language)</h2>
<p>The transformation of data was done with M language. I applied the following steps to the dataset:</p>

<pre> 
let
    Origem = Csv.Document(File.Contents("C:\Users\johnl\Desktop\datasets\sales_report_powerQuery\Sample - Superstore.csv"),[Delimiter=",", Columns=21, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Origem, [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"Row ID", Int64.Type}, {"Order ID", type text}, {"Order Date", type text}, {"Ship Date", type text}, {"Ship Mode", type text}, {"Customer ID", type text}, {"Customer Name", type text}, {"Segment", type text}, {"Country", type text}, {"City", type text}, {"State", type text}, {"Postal Code", Int64.Type}, {"Region", type text}, {"Product ID", type text}, {"Category", type text}, {"Sub-Category", type text}, {"Product Name", type text}, {"Sales", type text}, {"Quantity", Int64.Type}, {"Discount", type text}, {"Profit", type text}}),
    //Remove columns  - remove colunas desnecessárias
    #"RemoveCol" = Table.RemoveColumns(#"Tipo Alterado",{"Order ID","Customer ID","Customer Name","Postal Code", "Product ID", "Product Name"}),
    //Transform data of date type US to date type PT
    #"TransformDateType" = Table.TransformColumns(#"RemoveCol",{{"Order Date", each Date.FromText(_, "en-US"), type date},{"Ship Date", each Date.FromText(_, "en-US"), type date}}),
    //Transform data of number to date
    #"TransformColTypeToNumber" = Table.TransformColumnTypes(#"TransformDateType",{{"Sales", type number},{"Profit", type number}}, "en-US"),
    //TRansform discount in number
    #"TransformDiscountTypeToNumber" = Table.TransformColumnTypes(#"TransformColTypeToNumber",{{"Discount", type number}}, "en-US"),
    //Add Column value of Discount
    #"AddColumnDiscountValue" = Table.AddColumn(#"TransformDiscountTypeToNumber","Discount_value", each [Sales] * [Discount], type number),
    //Net Sales
    #"AddColumnLiquidSales" = Table.AddColumn(#"AddColumnDiscountValue", "Net_Sales", each [Sales] - [Discount_value], type number),
    //Loss if 1 (negative) or if 0 (positive)
    #"Loss Profit" = Table.AddColumn(#"AddColumnLiquidSales", "Loss", each if [Profit] < 0 then 1 else 0, Int64.Type),
    //Delivery days
    #"AddColumnDeliveryDays" = Table.AddColumn(#"Loss Profit","Delivery Days",each Duration.Days([Ship Date] - [Order Date]),Int64.Type),
    //Order Year
    #"OrderYear"= Table.AddColumn(#"AddColumnDeliveryDays", "Order Year", each Date.Year([Order Date]), Int64.Type),
    //Order Month
    #"OrderMonth" = Table.AddColumn(#"OrderYear", "Order Month", each Date.Month([Order Date]), Int64.Type),
    //Order Month Name
    #"OrderMonthName"= Table.AddColumn(#"OrderMonth", "Order Month Name", each Date.MonthName([Order Date]), type text)
   
in
    #"OrderMonthName"
</prev>


<p>
The transformation of data starts with the removal of unnecessary columns: Order ID, Customer ID, Customer Name,Postal Code, Product ID and Product Name.
In what concerns the product name, the dataset was very extensive and there were many different names. 
The columns Customer Name, Postal Code and columns with ID were removed for privacy reasons.
The next step was to trasnform data into the correct type of data, highlight the dates and convert them into format date ("dd/mm/yyyy").
After transforming data type some columns were added:Discount Value, Net Sales and Losses.
The last collums to be added were these: delivery days (number), Order Year (int), Order Month(int) and Order Month name (text).
</p>

</div>

<div>
<h2>4. KPI's Criados</h2>
<p>In this step on Power BI I created some measures: Total Sales, Total Profit, Profit Margin, Total Discount, % loss and Average Delivery Days...
The KPI's that I consider relevant are:</p>
<p>Avg Delivery Days = AVERAGE('Sample - Superstore'[Delivery Days])</p>
<p>Loss % by Segment = DIVIDE(SUM('Sample - Superstore'[Loss]),COUNTROWS('Sample - Superstore'),0)</p>
<p>Max Delivery Days = MAX('Sample - Superstore'[Delivery Days])</p>
<p>Min Delivery Days = MIN('Sample - Superstore'[Delivery Days])</p>
<p>Profit Margin = DIVIDE([Total Profit], [Total Sales], 0)</p>
<p>Sales % by SubCategory = DIVIDE(SUM('Sample - Superstore'[Sales]),CALCULATE(SUM('Sample - Superstore'[Sales]), ALL('Sample - Superstore'[Sub-Category])),0)</p>
<p>Total Discount = SUM('Sample - Superstore'[Discount_value])</p>
<p>Total Profit = SUM('Sample - Superstore'[Profit])</p>
<p>Total Sales = SUM('Sample - Superstore'[Sales])</p>
</div>

<div>
<h2>5. Graphs (Power BI)</h2>
	
	<h3>1) Graph 1 - compare Sales and Profit by years and months</h3>
		<p>The sales had a growth over the years with peaks, descents and ascents. The growth over the years was irregular. This can be motivated by seasonal seasons.
If we notice, the line of sales in months like June to December, it depends on the year, usually falls. On other hand, in January starts to grow again.
However, the profit has been increasing little, but increasing. With some losses in some months. 
The company its selling a lot with a low profit. In other words, the company sells a lot but do not transform the sales in high profit. 
This can be related with high discounts applied or high costs.</p>

	<h3>2) Graph 2 - Compare Sales, Profit and discount value by Sub-Category</h3>
	<p>As we can see, phones and chairs lead in sales volume. On the other hand, tables, bookcases, and supplies show losses.
It is also clear that discounts are affecting the profitability of some subcategories, such as binders and machines, where profits are too low compared to the sales volume.
Some products, despite having high sales, are not profitable—for example, tables. </p>

	<h3>3) Graph 3 - Most profitable products</h3>
	<p>Most profitable sub category: Copiers, Phones, Accessories and Paper, that contribute to profit. 
	Neutral sub category: Binders, chairs, storage and appliances, that contribute positevely but with low margins.
	Losses sub category: Tables, bookcases and supplies.
	Tables needs an urgent review by the company, and bookcases and supplies too. The company must give priority to Copiers and Phones. </p>
	
	<h3>4) Graph 4 - Profit By Region</h3>
	<p>The business its strognly supported by west and east, this may be due, to the fact that there are more company customers in these areas. </p>
	
	<h3>5) Graph 5 - Impact of the discount on profit</h3>
	<p>The chart confirms that high discounts are directly linked to losses. The best results occur in sales with low or no discounts.</p>
	
	<h3>6) Graph 6 - Sum of Profit by category</h3>
	<p>The chart shows that Technology and Office Supplies contribute most to the total profit. However, it is the high-margin subcategories, such as Accessories and Art, that sustain the company’s profitability.
Furniture, on the other hand, poses a risk, as it generates limited profit and includes loss-making products.</p>
	
	<h3>7) Graph 7 - Average Of Delivery Days by year and Region</h3>
	<p>This chart confirms that delivery times are consistent across all regions.</p>
	
</div>	


<div>
<h2>7. Conclusions</h2>
<p>The analysis shows that, despite steady sales growth, the company’s profitability is affected by unstable margins and recurring losses in some subcategories, 
particularly Tables and Bookcases. 
We can see that high discounts are strongly associated with losses, while sales with low or no discounts are more profitable.
It is recommended to limit aggressive discounts, strengthen the focus on profitable subcategories, and review the commercial and logistics strategy in the Furniture category.


<h5>Endnote: It should be noted that this analysis is based solely on the available dataset! 
It seems that the dataset does not capture several external factors that could impact the company’s profitability. 
Although the analysis has been conducted with care to avoid errors, the primary purpose of this work is to gain experience with the technologies used.</h5>
</p>
</div>
</div>