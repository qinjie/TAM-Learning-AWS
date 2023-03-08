# Athena Query - Cloudfront Usage by Region

1. Go to customer's BubbleWand Athena Console. Make sure customers cost explorer report generation is active.

2. Run following query in the Amazon Athena > Query editor.

   ```sql
   SELECT bill_billing_period_start_date as month, 
   -- line_item_resource_id AS distribution, 
   (case when line_item_usage_type like '%DataTransfer%' then product_from_location else product_location end) as ProductLocation, 
   (SUM (case when line_item_usage_type like '%DataTransfer%' then cast(line_item_usage_amount AS decimal(20,2)) ELSE 0 END ))/1024 as "DataTransfer (TB)", 
   SUM (case when line_item_usage_type like '%DataTransfer%' then cast(line_item_unblended_cost AS decimal(8,2)) ELSE 0 END ) as "DataTransfer ($)", 
   SUM (case when line_item_usage_type like '%Requests%' then cast(line_item_usage_amount AS decimal(20,2)) ELSE 0 END ) as Requests, 
   SUM (case when line_item_usage_type like '%Requests%' then cast(line_item_unblended_cost AS decimal(8,2)) ELSE 0 END ) as "Requests ($)", 
   (SUM (case when line_item_usage_type like '%DataTransfer%' then cast(line_item_unblended_cost AS decimal(8,2)) ELSE 0 END ) + SUM (case when line_item_usage_type like '%Requests%' then cast(line_item_unblended_cost AS decimal(8,2)) ELSE 0 END ) ) as "Total Cost ($)" 
   FROM customer_all 
   WHERE product_product_name = 'Amazon CloudFront' AND bill_billing_period_start_date between TIMESTAMP '2022-08-01' and TIMESTAMP '2023-02-28' GROUP BY 1, 2 ORDER by month desc;
   ```

3. Download the result which is a CSV file.

4. Go to Google Spreadsheet https://docs.google.com/spreadsheets/d/1prIjfRjoj7iWOvVGX9nkGu_ERvN0DUFGCbbPvG2JXtQ/edit#gid=0

5. Transfer data from CSV file into **Data** tab in the spreadsheet.

6. Examine other tabs in the spreadsheet.

   * For Pivot table, select any cell in it. In the Pivot table editor > Filters, click on "Select all" then uncheck "(Blanks)" and 0.
   * For Chart, double click on it. In the Chart editor > Setup, make sure the data range is correct.



