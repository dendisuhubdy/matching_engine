# Exchange Server

The clearing server listens to FIX messages in the incoming queue, use one thread to write the message to the in-memory database,
matches the orders, sends the execution report to the in-memory database.

# Copyright

2019 Bitwyre Technologies LLC
