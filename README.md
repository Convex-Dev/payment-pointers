# Decentralized revenue sharing

Groundbreaking initiatives are being pioneered in the hope of providing financial access and justice to humanity. 
Among them, the [Interledger Protocol](https://interledger.org) stands out by providing a global and currency-agnostic
network suitable for micropayments and payment streaming. Unlike previous attempts, this open system allows digital ledgers
of fundamentally different types to interoperate by routing payments and invoices akin to how generic information is routed
through the Internet.

If lowering the necessary friction for creating revenue streams is the very first step towards an inclusive financial system
operating across borders, soon appears the tangible question of how those revenues streams can be managed in a secure and
fair way.

This repository aims to explore and implement a revenue sharing scheme with the following characteristics:

- Collective freedom for members sharing a revenue stream to govern it as desired
- Individual freedom for a member to manage owned shares as desired
- Public proof that revenue is shared exactly as expected, promoting fair arrangements
- Decentralized and trustless, guarantee that no member can cheat any other member
- Instantaneous, any change in governance has an immediate effect on how revenue is shared

Proposed stack must also follow the key principles behind the Interledger ecosystem:

- Preserve the privacy of everyone involved
- Preserve financial inclusion, cost of sharing revenue must be negligeable
- Preserve the freedom to switch to any other method, no vendor lock-in
- Leverage existing state of the ecosystem

The solution outlined in this repository is best suited for the nascent [Web Monetization API](https://webmonetization.org) allowing
browser users to stream payments to wallets identified by URLs. This project builds on the concept of
[probabilistic revenue sharing](https://webmonetization.org/docs/probabilistic-rev-sharing), providing an in-depth analysis,
and proving that an initially simple idea can be magnified to unforeseen opportunities.

In summary, key deliverables described towards the end of this document consist of:

- A robust revenue sharing framework for Web Monetization and Interledger
- Open-source software implementations for this framework
- A web-based UI allowing anyone to benefit from this framework
- A coherent business model for a fair and self-sustaining ecosystem


## Current status

A proposal will been submitted to [Grant For The Web](https://www.grantfortheweb.org/) on September 22nd for the
July 2021 round.


## Trustless probabilistic revenue sharing

Wallets are identified by [payments pointers](https://paymentpointers.org), URLs that resolves to HTTP payment endpoints. Given
that services and APIs such as the Web Monetization API expect a single payment pointer, there is an obvious need for implementing a robust
revenue sharing solution. Allowing individual to form partnerships is a crucial factor for ensuring the future of Interledger
and Web Monetization, [as discussed in this article](https://yiibu.github.io/web-monetization).

The very basis of this work is [probabilistic revenue sharing](https://webmonetization.org/docs/probabilistic-rev-sharing). Given a list
of payments pointers sharing revenue from  payment streams, a weighted random selection process guarantees that revenue for each payment pointer
will approach what was agreed upon. If Alice should receive 60% of a revenue stream while Bob should receive 40%, applying a random selection
where Alice has a 60% chance of being selected for any given payment will approach that distribution. This statement becomes quickly accurate
as the number of payments grows.

However, **how can the payment pointer selection process be guaranteed to be fair, beyond any doubt?**

Any agreement is as strong or as weak as the trust in the enforcement of that agreement is. Given the openess and world-wide reach of
Interledger, any solution must suit grey regulatory areas and allow building constructive economic relationship even when members have not
yet developed trust. As outlined in [this article](https://www.fairpayzone.com/2021/04/web-monetization-and-payments-meet.html), lack of
trust between parties is commonly expected at the start of an economic relationship. Promoting a cycle of fair exchanges build the trust
capital that is necessary to strengthen relationships towards mutual benefits.

It follows that any revenue sharing solution in this context should be trustless from a technological point of view so that parties involved
have a sane basis for building trust from a human point of view. To achieve this requirement, this project aims to provide a solution based
on blockchain technology. A series of guidelines exposed in this document aims to ensure that the solution:

- Remains financially inclusive in spite of the speculative and volatile nature of blockchain
- Incur predictable fees that are provably sensible in the context of Web Monetization
- Ensure high performance since any delay in content delivery tend to reduce monetization
- Never handle any money in any form, merely handles the payment pointer selection process
- Do not put pressure on using a particular currency
- Promote openness, is not meant to become a monopolistic solution
- Promote fair revenue sharing governance by design

The benefits of describing revenue sharing agreements *"on chain"* is twofold. First, the state of the network is securely replicated
world-wide in a trustless manner, meaning no party can tamper with agreements. Second, agreements are written as smart contracts which
enforce revenue sharing rules automatically in an indisputable way. Rules, just like the possible evolution of those rules, are clearly defined
in code the moment a revenue sharing scheme is created by members.

The [Convex network](https://convex.world) has been chosen as a platform of choice for this project. The following sections provide details
as to how those guildines will be respected and goals achieved. Details are provided regarding smart contracts that:

- Will be portable to other networks with similar capabilities
- Will remain open for peer-review and reusability, with the freedom to modify them as desired
- Will set the foundations for building a web UI ; in a matter of clicks, users will have the ability to create and manage revenue sharing
streams without much technical abilities, computational power, or prior blockchain knowledge

Code excerpts and examples are written in [Convex Lisp](https://convex.world/cvm) which is extensively documented
[at this link](https://convex.world/cvm). However, not understanding the code excerpts should not prevent the reader from understanding
the project.


## Defining revenue streams

This project defines a revenue stream as a flow of money, from payment senders to payment receivers, where:

- Members own a finite supply of shares representing the stream
- A stochastic selection process selects a payment pointer from a known list whenever a payment is streamed
- Share owners stake their shares on payment pointers, as desired, given them weight in the selection process

Owning shares and staking them on a given payment pointer rises the chance of that payment pointer being selected as a payment receiver
for the next payment. This document describes an implementation where:

- A revenue stream is effectively a payment pointer that leads itself to a selection process
- Shares minted for that revenue stream are collaboratively managed by owners using smart contracts

Payment receivers chose the granularity that suit them best. For instance, supposing a news website, revenue streams could be created at
different levels. Hypothetically:

- Global stream for all members, when consumers browse generic pages (eg. "about" page)
- 1 stream per section between members who contribute to each section (eg. sports, finance, ...)
- 1 stream per content creator attached to each article (eg. 85% shares for creator, 15% shares for hosting)


### Creating and transferring shares

Shares can be conceptualized as fungible tokens, a common notion found in many decentralized networks. Each share represents a likelihood
of being selected as payment receiver when a payment stream is initiated in the context of a specific revenue stream. In Convex Lisp, a fungible
token can be created in a couple of lines.

Let us suppose that Alice is the original founder. For the sake of simplicity, 100 shares are minted where each share represents a 1% chance
of being selected as payment receiver for any given payment:

```clojure
(import convex.fungible :as fun)

(def shares
     (deploy (fun/build-token {:initial-holder *address*
                               :supply         100})))
```

In Convex, fungible tokens are components of a much broader asset abstraction, a common interface allowing to easily manage assets of any kind.
Shares are readily transferable to new members. Supposing Bob owns account `#50` and Alice wants to transfer 40 shares:

```clojure
(import convex.asset :as asset)

(asset/transfer #50
                [shares 40])
```

From now on, Alice owns 60 shares, providing her a 60% chance of being selected as payment receiver, while Bob owns the remaining 40 shares,
providing him a 40% chance.

Concretely, the value of `shares` is the number of a newly created automated account. An automated account is not managed by any user, it only
hosts smart contracts which in this case describe how shares are managed. Shares cannot be managed in any other form nor under any other circumstances
but by those described in smart contracts.

Just like any aspect of the Convex stack, Convex Lisp libraries are [open-source](https://github.com/Convex-Dev/convex/tree/master/convex-core/src/main/cvx/convex)
and developed in the interest of the public in a non-competitive way. Even on the Convex network, users are free to implement other schemes. However,
the asset framework provided by the Convex Foundation aims to provide interoperability between diffent kind of digital assets on the network so that
users have the ability to transfer and trade them as desired. This allows share owners to exchange their shares against other kind of assets
if desired.

A more complete example, albeit non-exhaustive, is available at [this link](https://convex.world/examples/fungible-token).


### Staking shares on payments pointers

Any share owner is free to stake shares on any payment pointer. For instance, a content creator benefiting from a monetized platform  would most
likely stake owned shares on a personal wallet. Nothing prevents a share owner from staking shares on several payment pointers at the same time, such
as a personal wallet as well as the wallet of a family member. This simple mechanism allows anyone to direct any percentage of revenue without
transferring shares, as desired and for as long as required.

A materialized view must be maintained to represent how shares are distributed by owners, mappings of payments pointers to accumulated shares:

```clojure
(def payment-pointers
     {"$wallet.com/alice" 60
      "$wallet.com/bob"   25
      "$wallet.com/foo"   15})
```

In parallel, the system must track how each owner stakes shares. Convex provides a more sophisticated method for doing so via its holdings abstraction.
However, it can be broadly conceptualized as a mapping of owner accounts to mappings of payment pointers to allocated shares:

```clojure
;; Where Alice owns account #20 and Bob account #50.
;;
(def stakes
     {#20 {"$wallet.com/alice" 60}
      #50 {"$wallet.com/bob"   25
           "$wcallet.com/foo"  15}})
```

Both requirements will be implemented in the scope of the fungible token implementation underlying shares.


### Stochastic selection process

Given a materialized view tracking global share allocations on payment pointers, as described in the previous section, selecting a payment pointer
randomly is straightforward given that a random number is available.

Blockchains are designed to be fully deterministic since different machines must compute transactions in the exact same way, resulting in the exact
same result. This is why a typical `rand` function, so common in programming languages, is not provided. Solutions usually combine
different informations available on chain, that are considered difficult to predict, hence acting as a sufficient sources of randomness.

On Convex, following parameters can be chosen and mixed, especially under the assumption that the selection process does not require a particularly
strong random value:

- Current timestamp in milliseconds
- Current juice price (computational cost weighted by network congestion)
- Current memory size of network state
- Number of registered peers
- Number of accounts
- Number of scheduled transactions

Since it is effectively payment senders that will ultimately query for payment pointers, inducing the selection process, share owners have no practical
way of biasing the random number generation in a particular direction. Additionally, an off-chain parameter can be provided externally when querying
for the next payment pointer, although this idea requires some careful considerations.

### Governance "à la carte"

While creating and handling shares is straightforward, new questions appear, such as:

- Should shares be always transferrable?
- How could additional shares be minted?
- How could users decide to lock some shares on agreed payment pointers?
- Should there be a member selection process first, allowing transfers only to current members?
- Should shares be transferrable only between members of a group or to outsiders as well?

For instance, it might be desirable for a percentage of shares to be locked on a particular payment pointer, such as the wallet of a charity. Or for
a specified period of time only. Users may want to vote for the creation of additional shares that could be locked in such a way, so that no one would
undergo the risk of transferring personally owned shares anywhere.

Not only are there no unique answers to such questions, group requirements are expected to evolve over time. Part of those answers resides in the
implementation of shares. Current implementation of fungible tokens can be readily augmented to put arbitrary restrictions on minting and transfers since
such capabilities are part of the broader asset framework.

Such governance issues are commonly discussed and prototyped in the context of DAOs (Decentralized Autonomous Organizations, groups governed on a blockchain
network). However, DAOs are usually much broader in scope and much more abstract regarding the aspects they are set to manage. In contrast, this project is
well-scoped to the context of revenue sharing and governance issues specific to that matter. Hence, the many resources already existing in the space of DAOs,
such as voting mechanisms, can be readily selected and improved within the specific requirements of the project.

While envisioning such concerns is essential for imagining a solution that can scale to many use cases, it is just as essential prioritizing common
requirements, straightforward to implement, and gradually build more complex governance features as additive and optional steps. Inclusion cannot be
built on the basis of implementing complex financial instruments. On the contrary, financial instruments must be conceived to promote fair and intuive
behavior by design, with the clear intent of providing resiliency and not burden.


## Recursive payment pointers

Let us define that **direct payment pointers** resolve directly to wallets whereas **indirect payment pointers** resolve to a selection
process. It follows that payment pointer are recursive: a selection process could itself resolve to another indirect payment pointer,
leading to another selection process, and so on.

This property is highly desirable. It provides share owners the ability to setup their own additional selection process without requiring permission
from anyone else. For instance, as a founder, Alice might agree to transfer 50% of total shares to a pool of investors represented by an indirect
payment pointer, in exchange for help in building her monetized platform. Alice would keep the freedom to manage her 50% of shares as desired, investors
would keep their freedom to manage their 50% of shares as desired, without requiring any further involvement.

While elegant and resiliient, this kind of recursive pattern has a practical performance cost. Resolving any additional payment pointer is effectively
an extra HTTP indirection, adding latency, causing exclusive content to be delayed, and ultimately driving monetization down.

This performance problem can be solved by eliminating the need for those HTTP redirections. In the following example, 50 shares are staked on Alice's
payment pointer (most likely her personal wallet) while 50 shares are staked on account `#510`, another automated account used by investors to manage
their 50% of shares between them:

```clojure
(def payment-pointers
     {"$wallet.com/alice" 50
      #510                50})
```

Now, for any new payment, either the selection process resolves to Alice's payment pointer which is returned immediately, either it resolves to that
other account, leading to another selection round completely independant from Alice. Unlike HTTP redirections which require resolving new URLs and calling
an additional services, everything happens in one query without changing environments, which is bound to be much faster.

This significant gain in performance allows imagining arbitrarily complex chains of royalties. It promotes fair revenue sharing where monetized content
can redirect part of revenue to other monetized content it is built on (eg. `news platform members` -> `news article author` -> `photographer's picture`).
Naturally, this is an optimization users may choose to leverage or not, keeping the freedom of choosing any other redirection method.


## Privacy and transparency

Previous sections outlined the fact that a payment receiver, in order to take part in a revenue sharing stream, must disclose 2 kinds of information:

- An account number for receiving and exchanging shares
- How shares are staked on payment pointers of choice by that account number

Payments pointers have been designed to provide a reasonable level of privacy, They are effectively considered public. Accounts on the Convex network
lean towards anonymity since, unlike in other decentralized networks, they are not strictly bound to a key pair. If the owner of an account is supposedly
uncovered, replacing the public key associated with that account is enough to cast doubt upon who really owns that particular account. Naturally, when
Bob provides an account number to Alice in order to receive shares, Alice might rightfully deduce that Bob is the owner of that account. In reality,
this is not necessarily true and if it is, will not necessarily hold true in the future.

Given those statements, given that in any other form of arrangement Bob would have most likely divulge more information to Alice anyways, this project
respects the privacy boundaries promoted by Interledger. However, the fact that share allocations is effectively public deserves more thought.

While under some circumstances it might not be desirable publicly disclosing which accounts own shares and how they are staked, it is arguably desirable
under most circumstances. Indeed, from the point of view of payment receivers, it arguably promotes fair agreements. For instance, it allows a content
creator to ensure that receveived shares, when joining a platform, are fair compared to what other creators have received.

Consumers can quickly and easily verify how their payment streams benefit content creator. If content creators hosting their work on a shared
platform end up receiving an unfair percentage of the revenue stream they helped create, consumers would have the ability to take notice and put pressure
on the platform towards fairer treatment. While browser extensions exist for tracking payment streams, they only offer insights after a period of consumption,
not an instantaneous overview.


## Viability

Following subsections inspect the viability of the project and whether a fair and sustainable business model can stem concepts outlined
previously.


### Performance analysis

Some performance gains can be attributed to avoiding network delays, such as outlined in the section about recursive payment pointers. This section
outlines the generic performance characteristics of the Convex network in the context of Web Monetization. A similar analysis can be applied to
other decentralized networks.

When discussing the performance of any system, a distinction if often made between:

- Read operations which usually bear a low computational cost
- Write operations which usually bear a much higher computational cost

Similarly, on the Convex network and comparables networks, a distinction must be made between:

- Queries which access data and compute a result, akin to read operations
- Transactions which produce a result byt altering network state, akin to write operations

Queries present a particularly high performance profile since they do not require any consensus between network peers. They scale horizontally with
the network: as the network grows, queries tend to become faster, akin to a [Content Delivery Network](https://en.wikipedia.org/wiki/Content_delivery_network).
In contrast, transactions require consensus between peers, adding delays and incurring a series of computationally expensive cryptographic operations.

Such charactericts for such systems are common. However, in the context of Web Monetization, this performance profile is suited. Indeed, transactions
required for creating shares, transferring them, or managing them in any way, will seldom happen. The vast majority of operations, to
an overwhelming extent, will be mere queries when consumers start streaming payments and payment pointers are selected. The fact that the selection process
is effectively stateless is a fundamental proof this project is scalable in spite of relying on blockchain technology. **This allows us to reap the obvious
benefits of a decentralized solution, as described in this document, while approaching the performance characteristics of a centralized solution.**

Furthermore, even in the case of transactions, the Convex technological stack introduces a series of innovations described in greater detail in the
[White Paper](https://raw.githubusercontent.com/Convex-Dev/design/main/papers/convex-whitepaper.pdf). Due to an overall reduced computational cost, current
benchmarks peak at 100,000 transactions per second where a transaction represents an average series of operations. Under such projections, computation induced
by anecdotic transactions becomes negligeable.


### Cost analysis

Naturally, at least to some extent, cost is correlated with the performance profile of the system. This section builds on the previous section
in order to provide an understanding of the cost that will be incurred on the end users, such as content creators and content consumers.

From the point of view of content consumers, queries for selecting payment pointers or reading any other data are almost free of charge. As
outlined in the previous section, queries do not require consensus and present the same kind of performance profile that content delivery
networks do. Hence, they are effectively *"off-chain"* and do not incur any fees *"on-chain"*. Queries are a facility provided by peer
operators at their discretion. For instance, the Convex Foundation runs a peer that serves queries completely free of charge. Furthermore,
the whole technological stack is open-source and the Foundation encourages individuals and organizations to run their own peers however they
like.

However, any solution meant to scale must consider the simple fact that running a peer requires hardware, energy, effort, and knowledge. As an alternative,
the "Self-sustaining ecosystem" section below offers a solution based on Web Monetization itself, allowing peer operators to cover expense and even generate
revenue.

From the point of view of content creators, first, the computational cost of transactions required for setting up and managing revenue sharing can be estimated
upfront. Then, the monetary cost can be estimated under various assumptions using this simple formula:

```
MONETARY_COST = COMPUTATIONAL_COMPLEXITY * TOKEN_PRICE * JUICE_PRICE
```

For instance, if a transaction has a computational cost of 20,000 units, that a billion network tokens cost 10 USD, and each compute unit costs 2
network tokens:

```
MONETARY_COST = 20,000 * 10 / 1e9 * 2 = 0.0004 USD
```
Cost per compute unit varies dynamically to control network congestion and the price of the network token is subject to speculation. Under
such complex relationships, it is vital to demonstrate that any on-chain fees incurred on the Convex network are sensible even under worst
case scenarios projected on those parameters.

Two canonical examples are detailed:

- Shares creation using the current fungible token implementation, one time fees occuring when a revenue stream is created
- Shares transfer, recurring to some extent, albeit not expected to happen often

Computation complexity has been exactly measured using the [Convex Lisp Runner](https://github.com/Convex-Dev/convex.cljc/tree/main/project/run),
without any further optimization:

```
Shares creation = 156,020 compute units
Shares transfer =   3,195 compute units
```

Two scenarios are modeled, mimicking common networks as of today (2021/09/20).

Scenario 1 mimicks the Polkadot network under moderate congestion:

```
Shares creation = 156,020 * 24.78 / 1e9 * 4 ~= 0.015  USD
Shares transfer =   3,195 * 24.78 / 1e9 * 4 ~= 0.0003 USD
```

Scenario 2 mimicks the Ethereum network under particularly high congestion:

```
Shares creation = 156,020 * 3,096.26 / 1e9 * 50 ~= 9.66 USD
Shares transfer =   3.195 * 3.096.26 / 1e9 * 50 ~= 0.2  USD
```

Under foreseen projections, sporadic transaction costs incurred on payment receivers when setting up and managing revenue sharing are negligeable. As a matter
of fact, sharing can be highly granular. For instance, creating a dedicated revenue sharing stream even for a single news article becomes interesting unless the consumer base is
exceptionnally small. As a comparison, [Coil](https://coil.com) currently streams payments at a rate of 0.36 USD per hour.

Even when projecting the absolute worst case scenario, which in reality has no legitimate basis to be predicted, transaction fees remain acceptable. For instance, setting
up a revenue stream for each content creator on a news platform would incur one time fees of approximately 10 USD each time, which is quickly recovered given
uses the indirect payment pointer on all published articles.


### Self-sustaining ecosystem

Up to this point, careful analysis was applied to study if the insights exposed in this document where financially beneficial to users, promoted fair economic
relationships, and offered new ventures. Previous section about cost analysis demonstrated that cost was on average negligeable for payments receivers and hinted
at the idea that payment senders could benefit from this system free of charge. This is because peer operators have the straightforward ability to collect tiny
fees via Web Monetization itself, capturing a negligeable percentage of payments streams that, given an economy of scale,  would provide continuous revenue for
operators.

This is fair and sensible. Indeed, a payment receiver monetizes content via an indirect payment pointer that leads to the well-described probabilistic selection process.
From the point of view of a peer, it is the payment sender who triggers the selection process when accessing monetized content, and this incurs on the peer the tangible
*"off-chain"* cost of running the query that performs the selection. In order to cover expenses and possibly generate additional revenue for the added value it
provides, the peer needs to displace the financial burden received by the sender onto the receiver that benefit from it. In other words, payment
receivers must agree that a tiny but sufficient percentage of their revenue is redirected towards the payment pointer of the peer operator they rely one. This
effectively closes the loop as every party involved make use of Web Monetization in a self-sustaining cycle. Alternatively, although it
would be suitable only for larger organizations, a conglomerate of payment receivers could run its own peer.

How those off-chain fees should be governed remains to be prototyped in a second stage within this grant. Most likely, fees will take into
account the computational cost of the selection process. Given that payment receivers control which peer will handle the selection process,
it is likely that peers will compete in terms of quality of service and price just like connnector nodes from the Interledger network do.


## Technical stack and deliverables

Building on the ideas exposed in this document and on current technology, this project aims to produce open-source software deliverables on 3 different levels.
All aspects will be clearly documented in English, in repositories alongside code. Preferred license for those repositories is the commonly used 
[Apache 2.0 license](https://www.apache.org/licenses/LICENSE-2.0), although any other license may be chosen under grant requirements.

Additionally, we aim to produce high-level content providing a clearer picture on how the technical components described below fit together. We will
release a short series of videos highlighting key concepts and tools, both for technical and non-technical users. We would also like to use any remaining time
to organize a webinar and host a live Q&A session.

All development will be conducted in this public repository which will become a [monorepo](https://en.wikipedia.org/wiki/Monorepo). We will maintain a clear
history of contributions, issues, and projects boards tracking progress over technical components described below. Given the projected timeline and the
characteristics of this project, we will be able to reach out for feedback from the Web Monetization early, after a first iteration stage.


### Web-based UI

The web-based UI will allow anyone to setup and manage revenue streams as described in this document in a matter of clicks, without requiring much technical
knowledge at all. As long as a user has the ability to place a payment pointer in a `<meta>` tag for Web Monetization, revenue sharing can be created and managed
over that indirect payment pointer, as detailed throughout this document. There is no need for any further integration.

First iteration of the UI will allow users to:

- Inspect portfolio of revenue streams, owned shares, and how owned shares are staked on payment pointers 
- Stake owned shares on chosen payments pointers
- Transfer shares
- Create new revenues streams

As soon as this first iteration becomes mature enough, we would like to reach out to the Web Monetization community for initial feedback. Given that no particular
integration is required, the entry barrier for testing this first prototype on the current test network will be very low.

Second iteration will focus on allowing more complex form of governance, optionally. We will focus on allowing users to put desired constraints on
share transfers, as described earlier. A concrete example would be the ability to reliably lock shares on a specific payment pointer, such as a charity
wallet. Or do so within a specifid period of time. Such rules will be selected by merely answering a few simple questions when creating a new revenue stream.

Third iteration will focus on providing a voting mechanism so that such rules can be created at any point in time. A concrete example would be the ability to
vote for the minting of new shares that would then be staked on the payment pointer of a charity wallet. Users would have to ability to create and vote on such
proposals.

In parallel, a simple visualization tool will be developed allowing any user to inspect how shares are staked. For instance, a content consumer will
get immediate insights regarding how fair a content platform is towards a specific content creator.

Lastly, a developer section will point to the repositories of all produced technical components, such as this frontend application, with an overview
of how those components fit together and can be deployed by independently by technical users. This is also essential for ensuring that all aspects
can be easily peer-reviewed.

Computational power required by users will be low. Overall, in terms of capabilities, this will remain a typical web-based UI written in [Re-frame](https://github.com/day8/re-frame)
and [ReactJS](https://reactjs.org). The most intensive part will be signing transactions using [Ed25519](https://en.wikipedia.org/wiki/EdDSA) for which
several Javascript libraries exist, meaning even mobile browser on low-end devices will be capable of performing those digital signatures.

Although the UI will be initially built in English, an [internationalization library](https://github.com/tonsky/tongue) will be used.  Such
internationalization practises do not couple text with the interface that display this text. Effectively, text is maintained separately so
that translations can be prepared and loaded without having to further alter the UI in any way. The UI will also comply to WCAG AA
accessibility standards.


### Smart contracts

Improvement of existing smart contracts (eg. curent fungible token library) and development of new capabititlies (eg. Web Monetization
integration) will be bootstrapped ahead of the web-based UI. After an initial phase of design stabilization, implementation will follow the
same timeline as described in the web-based UI. Smart contracts will be divided in 2 categories, akin to how [CRQS
systems](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation) behave:

- Transactions for governance: creating revenus streams, shares, transfers, ...
- Queries: inspecting governance, payment pointer selection, ...

Contracts will be written in Convex Lisp, an accessible language [thoroughly documented here](https://convex.world/cvm/data-types/numbers). Alongside,
code will be extensively commented in English with the intent of making those contracts portable to other languages. Repository containing those contracts
will provide a rationale and an overview.


### Modified peer

By default, peers of the Convex network speak a fast binary protocol over TCP. While highly efficient, this limits the set of consumers capable of directly
interacting with the network. However, a peer can be embedded in any application which can then provide an API suitable to clients such as browsers.
For instance, [convex.world](https://convex.world) is effectively a backend application embedding a peer while providing a [REST API](https://convex.world/tools/rest-api),
allowing any browser to arbitrarily query and transact over the network.

Similarly, for the purpose of this project, we will produce a backend application embedding a peer and hosting a REST API organized around the smart
contracts described in the previous section. It will act as a link between the browser and the Convex network. It will be available for anyone to run
and we will use grant money to provide it as a service on the test network.

REST API proposed by this modified peer will evolve alongside the development of smart contracts and the web-based UI.

The [convex.world repository](https://github.com/Convex-Dev/convex-web) acts as a concrete example of how such an application is built.

Core utilities for running and/or embedding a peer figures in the [core Convex repository](https://github.com/Convex-Dev/convex) with a Clojure wrapper
accessible in [this repository](https://github.com/Convex-Dev/convex.cljc/tree/main/project/net).


## Previous work

Out of repositories related to Interledger and Web Monetization found on Github, only a few mention payment pointers. Among this very limited set, only 3
attempted to provide utilities for managing them. However, they seem incomplete, unmaintained, and did not seem to attempt going beyond the simple probabilistic
revenue sharing exposed in the [Web Monetization documentation](https://webmonetization.org/docs/probabilistic-rev-sharing).

In contrast, building on the in-depth analysis elaborated by this document, we hope to provide novel insights strenghtening the Interledger ecosystem and,
more generally, the financial resiliency of users.

https://github.com/signalnerve/openmonetizationwallet

https://github.com/Vivid-IOV-Labs/revenue-share-smart-contract

https://github.com/Vivid-IOV-Labs/revenue-share-smart-contract


## License

Copyright © 2021 Adam Helinski, the Convex Foundation, and contributors

Licensed under the Apache License, Version 2.0
