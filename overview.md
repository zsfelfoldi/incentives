## A decentralized node incentivization model for P2P networks

Author: Zsolt Felföldi (zsfelfoldi@ethereum.org)

This document gives a quick overview of a node incentivization model that is currently being implemented in the Go Ethereum light client but is also intended to serve as a PoC for incentivizing future Ethereum network protocols. The goal of this model is to create an honest and efficient market of services, realized without central coordination, with resource allocation and load balancing based purely on p2p policies and market signals.

## Basic terms and assumptions

- nodes are identified by their cryptographic signer keys and therefore have pseudonym identities (a single physical entity can have multiple identites)
- a communication channel between two nodes is established by a third node introducing one to the other
- requests are sent through a channel with the expectation of receiving a reply
	- nodes capable of answering certain types of requests are called servers
	- nodes interested in sending certain types of requests are called clients
	- a node can play both client and server roles
	- a client can decide whether a reply to one of its own requests is valid or not
- connection is a client/server relationship that exists for limited time periods when both parties are willing to keep it active
	- server promises to answer certain request types at a specified limited rate with low latency and high probability
	- client obeys request rate limitations and determines the subjective quality of service based on the distribution of response times compared to other available servers
- we assume an advertisement medium where servers can announce their capabilities and clients can find them (discv5 node discovery DHT is intended to do this job)
	- request types are organized hierarchically to make it easier to specify supported subsets
	- we assume that advertisement has an inherent difficulty or can artificially be made difficult/expensive enough to ensure that a high enough percentage of advertisements is useful

## Goals and challenges

### Subjective quality of service

Clients should be able to find the servers that offer the best compromise for them in terms of service quality vs price and establish reliable connections with them (by "quality" we mean consistently correct and quick responses). The most obvious challenge here is that service quality is subjective and cannot be directly proven to a third party so promises are not enforceable. "Caveat emptor" applies maximally, it is always up to the clients to use a strategy that incentivizes honest server behavior and never rewards cheating attempts more than honest operation. Communication between client and server should make it clear what the expected/promised "honest" server behavior is.

### Building trust

When choosing a server to pay for, the client needs to predict the price-to-value ratio received after the payment. Ideally, this should be based on past quality statistics and the pricing information announced by the server. Server selection based on actual prices ensures efficient resource allocation. This requires a peer-to-peer trust building process though because price information published by the server is just an unenforceable promise. The only reliable information a client has is already received service and already sent payments. A server can always disappear right after receiving the next payment and the client has to consider this possibility. This means that trust building between client and server has to start with the server offering a limited amount of free service. The client strategy should apply an initial "paranoid" approach to new servers which can then gradually be transformed into a more efficient "trusted" mode of operation based on the price feedback received from the server.

### Load balancing and connection policies

Another important challenge is that on a really short time scale the price information is practically meaningless even in the most honest relationship. Practice has shown that individual request costs and response times are really noisy and need some averaging time to become meaningful. These factors affect the final price-to-value ratio and the client can not assume that longer term statistics collected during lower price levels will apply at higher prices. In general, clients should avoid servers with extremely fluctuating prices.

It would also be very inefficient to start negotiating prices right before sending an individual request. If we want quick replies then load balancing (deciding where to send the next request) needs some pre-negotiated rate limiting policies for connections. Price negotiation should ideally apply to building and maintaining longer term connections with the desired rate limiting parameters. Servers limit the frequency of transient overloads (where response times rise significantly over the usual range) by dynamically adjusting the limit of the total capacity of accepted connections. Connection capacity (guaranteed request rate) is therefore a scarce resource in the system. Servers can charge both for guaranteed capacity and actually served requests.

Note that this capacity-based model does not mean that a client can not just quickly connect to send a single request (along with a small sum over a payment channel) but this should be handled as an edge case where the achievable price-to-value ratio can be significantly worse.

## Service tokens and request costs

In order to reduce price fluctuations and improve serving capacity allocation, servers are allowed to sell a limited amount of their service in advance. Service tokens are issued by individual servers and request costs are nominated in these tokens. Request costs (when nominated in service tokens) should depend only on the actual physical difficulty of serving those requests and be independent from actual market conditions. The token sale price (nominated in real currencies) can of course change according to market supply and demand.

### Token stability

One of the implicit promises attached to tokens is "token stability" which means that the average costs and serving times of similar requests do not change and the purchasing power of tokens stays stable. Token-nominated request costs should usually depend on the usage of the most important physical bottlenecks in the serving system (if there are multiple important bottlenecks then issuing multiple types of service tokens is also possible). Individual request serving difficulty can fluctuate heavily though. Practice has shown that database-heavy operations are often orders of magnitude quicker than the worst case estimates thanks to caching. Therefore we allow servers to determine individual request costs after serving them but expect them to announce a price table with worst case estimates and never charge more than that.

It is the responsibility of clients to determine the extent of token stability using long term statistics. If token costs are indeed stable and token accounting is correct then the token price information becomes meaningful for the client. Becoming "trusted" by clients is the interest of the servers because clients with a "paranoid" approach won't trust their price signals, making efficient resource allocation impossible. Untrusted servers have to provide more free service in order to earn trust and later be able to sell most of their capacity at the actual market price. Client trust strategy should ensure that the potential gains achievable by a breach of already gained trust are smaller than the cost of gaining that trust.

### Token spendability

Another important promise is "token spendability" which means that clients can actually buy the usual quality of service with their tokens at the promised stable price levels. Since we dismissed the possibility of controlling demand with prices on a short time scale and decided to use pre-sold tokens of stable value instead, the right way to control short-term demand is by not selling too many tokens in advance. Limiting token issuance in a way that it keeps demand in sync with supply will also ensure that token prices will react to global supply and demand changes on a longer time scale.

Token stability means that there is a more or less constant maximum limit on the rate at which all clients combined can spend tokens. Charging for capacity also sets a minimum limit on total token spending rate if the total capacity limit is reached by paying clients. Token spendability promises that in at least a given percentage of time the capacity limit of the server will not be filled up with token-spending clients so anyone in possession of tokens can connect and spend them with a high enough probability.

Limiting total outstanding token amount also requires limiting token lifetime, otherwise token hoarding could make new token issuance impossible. Buying tokens in advance has a potential cost to clients, but it also has its benefits. One of these is that if they know their needs in the near future then they can buy the dips and don't need to compete for tokens in those periods when many new clients are trying to connect and the available supply is low.

### Client priorities and free service

The control target for token issuance rate is that the total capacity limit is usually not 100% filled with paying clients. Still, there are cases when too many of them want to connect simultaneously and the server has to prioritize. The preference of long-term business relationships over short-term competition should also be realized in the client connection priorities so the priority is based on the token balance of the clients. More exactly, the suggested priority formula is `tokenBalance / requestedCapacity` which means that those clients who want to spend their tokens at the maximum possible rate can do so at a rate proportional to their balance. Putting it another way, every single token that its owner wants to spend at any given moment has a similar chance of being spent.

This also means that the main benefit of buying tokens in advance is that it ensures a stable connection. Buying tokens with a limited lifetime is a commitment to use the service in the near future which is a valuable information for the server and it is rewarded with a commitment to let the client do so. The benefits of this mechanism are symmetrical in the sense that they make the future more predictable for both parties.

Client priority based on token balance also means that it is easy to seamlessly integrate free service into the model. The same condition that ensures token spendability (server capacity limits not always filled by paying clients) also makes it possible to accept clients with zero balance. In order to distribute the available free service capacity to non-paying clients more evenly, Geth LES implementation maintains a virtual negative balance for them which is also expired over time and is associated with network address instead of key ID.

## Client side strategy

### Service value metrics

Clients collect statistical data about response times and amount of different request types served by each server. These statistics can be converted to a scalar "subjective service value" as a function of the subjective valuations of each request type and the response time expectations which are represented by a subjective value modifier vs response time function. These valuations are derived from global market statistics and then applied to the individual statistics of each server in order to gain a scalar total service value for any given time period. Ideally these values should strongly correlate with the service token amounts charged by honest servers.

### Trust building and server selection

When no (or not enough) free service is available, the client needs to choose a server to pay for. This is based on a prediction of the service value received after the payment (the server with the best estimated price-to-value ratio is selected). Gradual trust building for each server can be realized by implementing an ensemble of two predictors using a different approach. Each predictor tries to predict the service value received in case the requested amount is paid, and if it is indeed paid then receives the actual service value as a feedback later (the next time a payment is requested in order to connect).

#### Base predictor

The base predictor realizes the "paranoid approach" to new or unreliable servers. All it does is a simple linear extrapolation of past price-to-value ratio and returns `receivedValue * requestedPayment / (sentPayment+requestedPayment)`. Using this formula the client can slowly converge on using the servers providing better value but ignores the pricing information and is therefore unable to efficiently find the less utilized servers. If some servers are trusted already then this is mostly a disadvantage for the untrusted ones.

#### Trusted predictor

This predictor uses the token accounting and price info from the server to make a prediction. In the simplest case it should look something like `(receivedValue*requestedPayment) / (spentTokens*tokenPrice)`, though different price ratios for individual request types might further complicate the final formula.

#### Ensemble predictor

This predictor makes the final judgement. It evaluates the predictions of the previous two predictors. It allows a certain amount of deviation from the base predictor's value and returns the allowed closest value to the trusted predictor's result. The allowed deviation (which represents trust toward the server) is initially zero and is updated based on the actual received value feedback. It is increased or decreased if a higher/lower deviation allowance would have yielded a more accurate prediction. The deviation limit and its update formula should ensure that a certain level of trust only allows a limited amount of funds to be paid before going to zero in case the server starts to behave unreliably or maliciously.


