Channel prespective using our microservice api

Every channel will be provided with a token .
Channel will be using that token and an order_id. If channel receives a failure message it will try with that order_id for retry purpose else for new order it will sending new order_id.

Api structure for client

POST /order_product/ORD12345?token=abc123


Microservice at Mars server

Will be having four tables product_stock, channel, channel_order, ops_list.

Table rows are defined in diagram.

When channel hit the api 
1. Will verify whether token is accurate and then each response object will be pushed to channel vise SQS and each job will be picked up one by one , worker will handle this.

Worker login
1. There will be datasource transaction where we are updating stock and creating order .
2. Check Stock exist if not then send 409 response to channel.
3. Will check if channel order_id exist then will update order_status without updating stock.If channel order_id do not exist then create new order updating stock.
4. All this process will be in datasource transaction , if any step fails then data will be rolled back because of datsource transaction and it will be catch block where we are updating ops_list table.
5. If order status is confirmed then we will be calling ERP update status api. If api response is of failure we will be doing retry attempt. If all attempt failed then we will add it in ops_list with step ERP update.


Order ingestion. How do you handle webhooks and polling, duplicate or out-of-order events, and a channel that is down?
Ans. It will is handle through order_id if it already exist new order will not be created which will stop duplicate order creation.

Inventory consistency. Two channels sell the last unit of a SKU in the same second. Walk through exactly what happens in your design. 
Ans. Suppose API are  called at same time  by multiple channel but the database transaction will be called with async and await so that transaction executes one at a time so it will update the stock for the channel whose api call reaches to ecs first and next channel will be getting 409 as response.

Stock push. How do you update four channels with different rate limits within 60 seconds during a sale? 
Ans. For each channel there will be sepearte Queue which is SQS and job will be pushed to Sqs, each job will be picked be each seperate worker which are running concurrently.

Failure handling. Cover retries, the ERP’s Sunday downtime, Blinkit timeouts, and messages that keep failing. 
ERP list will be displayed from erp table and it will resolved by manually hitting a button and in backend different api will be called up.
(This one i will keep on least priority for v1 deliverables.)

A. Add a second warehouse in Bengaluru. What changes in your design? 
A new table needs to be added names stock which will contain warehouse name and address.There will be mapping of stock and warehouse.

