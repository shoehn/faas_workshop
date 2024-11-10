
### Sample Implementations

In the lecture we have discussed orchestration vs choreography and started to implement a sample mockup. Here is some sample code to start your journey.

#### Inventory Service

```python 
import json
import random

def handle(request):
    # Parse the request to get item details
    request_data = json.loads(request)
    item_id = request_data.get("item_id", "")
    quantity = request_data.get("quantity", 1)

    # Mock check: Randomly determine if the item is in stock
    in_stock = random.choice([True, False])

    # Prepare the response based on stock status
    if in_stock:
        response = {
            "status": "available",
            "item_id": item_id,
            "quantity": quantity,
            "message": f"{quantity} of item {item_id} is in stock."
        }
        # In a choreography model, trigger a stock-confirmed event here
    else:
        response = {
            "status": "out_of_stock",
            "item_id": item_id,
            "quantity": quantity,
            "message": f"Item {item_id} is out of stock."
        }
        # In a choreography model, you could trigger a stock-unavailable event here
    
    # Return response as JSON
    return json.dumps(response)
```

1. **Input**: This function expects a JSON payload with item_id and quantity fields to check if the requested quantity of the item is available.
2. **Mock Stock Check**: It randomly determines if an item is in stock, simulating a simple inventory check.
3. **Response**:
	- If the item is in stock, it responds with "status": "available".
	- If the item is out of stock, it responds with "status": "out_of_stock".
4. **Event-Based Choreography**: For a choreography-based approach, this function could publish an event (e.g., "stock-confirmed" or "stock-unavailable") for other services to listen to.
5. **Orchestration**: In an orchestrated workflow, an **Order Orchestrator** function would call this Inventory Service and proceed to the **Payment Service** only if status is "available".

#### Payment Service

```python 
import json
import random

def handle(request):
    # Parse the request to get payment details
    request_data = json.loads(request)
    order_id = request_data.get("order_id", "")
    amount = request_data.get("amount", 0)

    # Mock payment processing: Randomly determine if the payment succeeds
    payment_successful = random.choice([True, False])

    # Prepare the response based on payment status
    if payment_successful:
        response = {
            "status": "payment_successful",
            "order_id": order_id,
            "amount": amount,
            "message": f"Payment of {amount} for order {order_id} was successful."
        }
        # In a choreography model, trigger a payment-confirmed event here
    else:
        response = {
            "status": "payment_failed",
            "order_id": order_id,
            "amount": amount,
            "message": f"Payment for order {order_id} failed."
        }
        # In a choreography model, you could trigger a payment-failed event here
    
    # Return response as JSON
    return json.dumps(response)
```

1. **Input**: This function expects a JSON payload with order_id and amount fields, simulating a payment request for an order.
2. **Mock Payment Processing**: It randomly determines if the payment is successful or fails, imitating a basic payment processing outcome.
3. **Response**:
	- If payment is successful, it responds with "status": "payment_successful".
	- If payment fails, it responds with "status": "payment_failed".
4. **Event-Based Choreography**: For a choreography-based approach, this function could publish a payment-confirmed or payment-failed event for other services to listen to.
5. **Orchestration**: In an orchestrated workflow, an **Order Orchestrator** function would call this **Payment Service** and proceed to the **Shipping Service** only if status is "payment_successful".

#### Shipping Service

``` python
import json
import random

def handle(request):
    # Parse the request to get shipping details
    request_data = json.loads(request)
    order_id = request_data.get("order_id", "")
    address = request_data.get("address", "")

    # Mock shipping processing: Randomly determine if shipping is successful
    shipping_successful = random.choice([True, False])

    # Prepare the response based on shipping status
    if shipping_successful:
        response = {
            "status": "shipping_scheduled",
            "order_id": order_id,
            "address": address,
            "message": f"Shipping for order {order_id} has been scheduled to {address}."
        }
        # In a choreography model, trigger a shipping-scheduled event here
    else:
        response = {
            "status": "shipping_failed",
            "order_id": order_id,
            "address": address,
            "message": f"Shipping for order {order_id} failed."
        }
        # In a choreography model, you could trigger a shipping-failed event here
    
    # Return response as JSON
    return json.dumps(response)
```

1. **Input**: This function expects a JSON payload with order_id and address fields, simulating a shipping request for an order.
2. **Mock Shipping Processing**: It randomly determines if shipping is scheduled successfully or fails, simulating a basic shipping process.
3. **Response**:
	- If shipping is scheduled successfully, it responds with "status": "shipping_scheduled".
	- If shipping fails, it responds with "status": "shipping_failed".
4. **Event-Based Choreography**: For a choreography-based approach, this function could publish a shipping-scheduled or shipping-failed event for other services to listen to.
5. **Orchestration**: In an orchestrated workflow, an **Order Orchestrator** function would call this **Shipping Service** as the final step if status is "shipping_scheduled".
