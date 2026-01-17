# Flare CTF #5 writeup
## this is my writeup for the fifth challenge of the Flare CTF

To start, I visited the given link http://ly3ph3sccqwjh6invtiwo6vzk7nmxzcl6kuqxyzd5xulkshpvydn6yqd[.]onion and observed a marketplace-looking site, with an item for sale "FlareCTF Flag LEAKED!" which is where I started looking first.  

The first thing I noticed was that to "buy" something, you have to log in, and when trying to log in, it gives a set of "demo" credentials:

```
Invalid credentials. Try our demo account! Username: buyer Password: silkroad
```

The next thing I did was go into "My orders" in the demo account to see if there would be anything to do with the flag. There wasn't, but looking at the details of an order revealed some important facts.  

The first thing was that order details would contain the following information:

```
Item, Vendor, Buyer, Amount, Status, Seller Notes
```

The second thing was that each order details had a similar link with just one paramater changing. For example, the two order details on the demo account were found at the following addresses:

```
http://ly3ph3sccqwjh6invtiwo6vzk7nmxzcl6kuqxyzd5xulkshpvydn6yqd[.]onion/orders.php?id=157
http://ly3ph3sccqwjh6invtiwo6vzk7nmxzcl6kuqxyzd5xulkshpvydn6yqd[.]onion/orders.php?id=166
```

Changing the number revealed something important: The demo account could indirectly view orders' details made by other users by changing this value, a case of IDOR (Insecure direct object reference).  

For example, I tried the valuer "110" and it returned the following order details:  

```
Item	🔥 PRIVATE LOG CHANNEL ACCESS 2026 😈
Vendor	LivingDangerously
Buyer	shadow_ops
Amount	0.15 BTC
Status	Processing
Seller Notes 	
Checking logs.
```

Notice that the order was made by "shadow_ops", not "buyer", which means this was an order made by a user that wasn't the demo user.  
This means that all that's left to do was find an order id for "FlareCTF Flag LEAKED!" and look for anything telling in the order details.  

To do this, I used caido web proxy with tor by setting localhost:9050 as an upstream SOCKS proxy and found the request to http://ly3ph3sccqwjh6invtiwo6vzk7nmxzcl6kuqxyzd5xulkshpvydn6yqd[.]onion/orders.php?id=n and used the automate tool to send requests for numberical values 0-300 for n and ordering them in order of longest to shortest response length.  
By doing this, I was able to find the order details for an order for "FlareCTF Flag LEAKED!", of which the details were:  

```
Order Details: #25
Item	FlareCTF Flag LEAKED!
Vendor	LivingDangerously
Buyer	shadow_collector
Amount	0.15 BTC
Status	Delivered
Seller Notes 	
I cant believe you actually paid for this, script kiddie xD. Your flag/discount is: flare{0rd3r_4l0ng_th3_51lk_r04d_99}
```
Flag: flare{0rd3r_4l0ng_th3_51lk_r04d_99}
