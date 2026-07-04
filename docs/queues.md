# Queuing

Queuing is process of managing a series of tasks, messages, or events in a specific order. A queue operates on the principle of First In, First Out (FIFO)—meaning the first item added is the first to be processed. Imagine a line of people waiting to be served. Similarly, tasks are put in a queue to be processed one by one.

Queuing helps to decouple components, meaning that one system part generates tasks or messages. Then another part processes them at its own pace, ensuring smooth operations. Queues handle tasks sequentially, preventing overloads in application processing and thus helping manage workloads.

You can implement queuing with Valkey. You can use the following methods:

* [Lists](https://valkey.io/topics/lists/). The tasks are added to one side of the list. When a task is processed, it is removed from the other side of the list. An example of using a list is the queue of orders in an online shop that are processed for shipping.
* [Channels](https://valkey.io/topics/pubsub/). One part of your system creates a channel and puts the messages to it. Another part subscribes to this channel and reads the messages from it. Channels enable the delivery of messages to multiple receivers. For example, a Twitter-like application where multiple users receive real-time updates on a port. Or a weather app that posts weather updates and users, subscribed to it, receive notifications.
* [Streams](https://valkey.io/topics/streams-intro/). Unlike lists where items are wiped out of the list when processed, streams keep the history of events. This enables users to read data from a stream at any moment. A good example of a stream is reviews: a user adds a review and can then view all responses or reactions to it.

This guide will give you an idea of how to use these methods in your app.

## Lists

Let's use the orders in an online shop as an example. Users place orders, which your application adds to a queue. Workers process these orders one by one, removing them from the queue and updating their status in the database. To prevent the app from continuously querying the queue, you can use a blocking queue in Valkey. When the queue is empty, the app is paused until a new order is added.

To ensure that every order is processed successfully, the app first moves the order to a temporary queue from the main queue. After processing the order, it updates its status in the backend database and removes it from the temporary queue.

To implement this workflow, do the following:

1. Create a database `sales` and a table Orders in PG 

    ```sql
    CREATE DATABASE IF NOT EXIST sales;
    ```

2. Create a table `orders` for this database

    ```sql
    \c sales;

    CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    order_details TEXT,
    status VARCHAR(50)
    );
    ```

The application code would look like  this:

```python
import psycopg2
import valkey  # Import the valkey-py library


# Initialize Valkey and PostgreSQL connection
valkey_client = ValkeyClient(host="localhost", port=6379)
main_queue = valkey_client.list("main_queue")
temp_queue = valkey_client.list("temp_queue")

def add_order(order_details):
    """
    Adds a new order to the main queue.
    """
    main_queue.push(order_details)
    print(f"Order added to the queue: {order_details}")

def process_orders():
    """
    Processes orders from the queue and updates their status in the database.
    """
    conn = psycopg2.connect("dbname=sales user=postgres password=yourpassword")
    cur = conn.cursor()

    while True:
        # Blocking call to fetch the next order from the queue
        print("Waiting for orders...")
        order_details = main_queue.pop(blocking=True)

        if order_details:
            # Move the order to the temporary queue
            temp_queue.push(order_details)
            print(f"Processing order: {order_details}")

            # Update order status in PostgreSQL
            cur.execute("UPDATE orders SET status = 'processed' WHERE order_details = %s", (order_details,))
            conn.commit()
            print(f"Order status updated for: {order_details}")

            # Remove the order from the temporary queue
            temp_queue.pop()
        else:
            print("Queue is empty. Waiting for new orders...")

# Example usage
if __name__ == "__main__":
    # Adding orders to the main queue
    add_order("Order 1: Laptop")
    add_order("Order 2: Smartphone")

    # Processing orders from the queue
    process_orders()
```

## Channels

Let's say you're developing a Twitter-like application. In this application, users can create channels and post content. Other users can subscribe to these channels and receive updates whenever new posts are added.

To implement this functionality using Valkey, you can use the following commands:

* `PUBLISH`: This command is used to post updates in a channel.

* `SUBSCRIBE`: This command allows users to subscribe to a channel and receive updates in real time.


The application code would look like this:

```python
import psycopg2
import valkey 

from valkey_client import ValkeyClient

# Connect to Valkey
valkey_client = ValkeyClient(host="localhost", port=6379)

def publish_post(channel, post_content):
    """
    Publishes a new post to the specified channel.
    """
    valkey_client.publish(channel, post_content)
    print(f"Post published in channel '{channel}': {post_content}")

def subscribe_to_channel(channel):
    """
    Subscribes to a channel and listens for new posts.
    """
    pubsub = valkey_client.pubsub()
    pubsub.subscribe(channel)

    print(f"Subscribed to channel '{channel}'. Listening for new posts...")
    for message in pubsub.listen():
        if message['type'] == 'message':
            print(f"New post in channel '{channel}': {message['data']}")

# Example usage
if __name__ == "__main__":
    # Publisher posts content
    publish_post("tech_news", "Percona and Valkey: Ensure your data journey with reliability, scalability, and real-time precision!")

    # Subscriber listens for new posts
    subscribe_to_channel("tech_news")
```

## Streams

Streams extend the capabilities of both lists and channels. Unlike lists, where data is removed once processed, streams preserve the history of events, allowing users to read from them asynchronously at any time. Multiple publishers can add data to streams, and multiple subscribers can read from them simultaneously.

To show how streams work, let's take the scenario of leaving restaurant reviews as the example. A user posts a review about a restaurant, which is stored in a stream named Reviews and assigned a unique ID. Alongside this, a private channel is created for the review, and the author's app subscribes to the channel.

Other users can read the review and leave reactions or responses. For each reaction or a response, the app generates an event and posts it to the author's private channel. The author is notified about the feedback and can read from the stream to learn more or view the history. 

Here's how to implement this scenario:

1. Create a database `restarurant_reviews` in PostgreSQL:
    
    ```sql
    CREATE DATABASE IF NOT EXIST restaurant_reviews;
    ```

2. Create a table `reviews`:

    ```sql
    \c restaurant_reviews;

    CREATE TABLE reviews (
        review_id SERIAL PRIMARY KEY,
        user_id VARCHAR(50),
        review_content TEXT,
        timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
    ```

The code sample to achieve this would look like this:

```python
import psycopg2
import valkey
from valkey_client import ValkeyClient

# Connect to Valkey
valkey_client = ValkeyClient(host="localhost", port=6379)
reviews_stream = "Reviews"

# Connect to PostgreSQL
conn = psycopg2.connect("dbname=restaurant_reviews user=postgres password=yourpassword")
cur = conn.cursor()

def post_review(user_id, review_content):
    """
    Posts a new review and stores it in the Reviews stream and PostgreSQL database.
    """
    # Insert the review into the PostgreSQL database
    cur.execute(
        "INSERT INTO reviews (user_id, review_content) VALUES (%s, %s) RETURNING review_id;",
        (user_id, review_content)
    )
    review_id = cur.fetchone()[0]
    conn.commit()

    # Add the review to the Valkey stream
    valkey_client.stream_add(reviews_stream, {
        "review_id": review_id,
        "user_id": user_id,
        "action": "review",
        "content": review_content
    })

    # Create a private channel for the review author
    private_channel = f"notifications:{user_id}"
    print(f"Review {review_id} posted! Notifications will be sent to {private_channel}.")
    return review_id

def react_or_respond(review_id, responder_id, action, content):
    """
    Adds a reaction or response to the review and sends a notification to the author.
    """
    # Add the reaction/response to the Valkey stream
    valkey_client.stream_add(reviews_stream, {
        "review_id": review_id,
        "responder_id": responder_id,
        "action": action,
        "content": content
    })

    # Notify the review author
    cur.execute("SELECT user_id FROM reviews WHERE review_id = %s;", (review_id,))
    author_id = cur.fetchone()[0]
    if author_id:
        private_channel = f"notifications:{author_id}"
        valkey_client.publish(private_channel, f"New {action} on your review {review_id}: {content}")
        print(f"Notification sent to {private_channel}.")

def view_notifications(user_id):
    """
    Subscribes to the author's notification channel to view real-time updates.
    """
    private_channel = f"notifications:{user_id}"
    pubsub = valkey_client.pubsub()
    pubsub.subscribe(private_channel)
    print(f"Subscribed to {private_channel}. Waiting for notifications...")

    for message in pubsub.listen():
        if message['type'] == 'message':
            print(f"Notification: {message['data']}")

def view_review_history(review_id):
    """
    Retrieves the full history of reactions and responses for a review from the stream.
    """
    events = valkey_client.stream_read(reviews_stream, start="-", end="+")
    history = [event for event in events if event["review_id"] == str(review_id)]
    print(f"History for Review {review_id}:")
    for event in history:
        print(event)

# Example usage
if __name__ == "__main__":
    # Post a review
    review_id = post_review("user_1", "The pasta here is incredible!")

    # React to the review
    react_or_respond(review_id, "user_2", "reaction", "Thumbs up")
    react_or_respond(review_id, "user_3", "response", "I agree! Their pasta is the best.")

    # View notifications for the author
    view_notifications("user_1")

    # View the review's full history
    view_review_history(review_id)
```

