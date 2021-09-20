# Decentralized probatilistic revenue sharing

Groundbreaking initiatives are being pioneered in the hope of providing financial access and justice to humanity. 
Among them, the [Interledger Protocol](https://interledger.org) stands out by providing a global and currency-agnostic
network suitable for micropayments and payment streaming. Unlike previous attempts, this open system allows digital ledgers
of fundamentally different types to interoperate by routing payments and invoices akin to how generic information is routed
through the Internet.

If lowering the necessary friction for creating revenue streams is the very first step towards an inclusive financial system
operating across borders, soon appears the tangible question of how those revenues streams can be managed in a secure and
fair way.

This repository aims to explore and implement a revenue sharing scheme with the following characteristics:

- Collective freedom for members sharing a revenue stream to govern their revenue streams as desired
- Individual freedom for a member to manage owned shares as desired
- Individual freedom for a member to temporarily redirect revenue to chosen wallets, without transferring shares
- Public proof that revenue is shared exactly as expected, promoting fair arrangements
- Decentralized and trustless, guarantee that no member can cheat any other member
- Instantaneous, any change in governance has an immediate effect on how revenue is shared

Proposed stack must also follow the key principles behind the Interledger ecosystem:

- Preserve the privacy of everyone involved
- Preserve the low barrier of entry, any additional revenue sharing fees must be close to insignificant compared to revenue streams
- Preserve the freedom to switch to any other method, no vendor lock-in

To guarantee openness and reusability, proposed stack must also obey those generic principles:

- Use only open-source technologies with permissive licenses
- Remain itself fully open-source, from documentation to implentation
- Achievable under current technical possibilities
- Core features must strive to be portable and documented to allow so
- Core design directly based on elements from the Interledger protocol

The solution outlined in this repository is best suited for the nascent [Web Monetization API](https://webmonetization.org) allowing
browser users to stream payments to wallets identified by URLs. On a very high-level, it builds on the concept of
[probabilistic revenue sharing](https://webmonetization.org/docs/probabilistic-rev-sharing) and proves that an initially simple idea
can be magnified to unforeseen opportunities.


## Current status

A proposal will been submitted to [Grant For The Web](https://www.grantfortheweb.org/) on September 22nd for the
July 2021 round.


## Trustless probabilistic revenue sharing

Wallets are identified by [payments pointers](https://paymentpointers.org), URLs that resolves to HTTP payment endpoints. Given
that services and APIs such as the Web Monetization expect a single payment pointer, there is an obvious need for implementing a robust
revenue sharing solution. Allowing individual to form partnerships is a crucial factor for ensuring the future of Interledger
and Web Monetization, [as discussed in this article](https://yiibu.github.io/web-monetization).

The very basis of this work is [probabilistic revenue sharing](https://webmonetization.org/docs/probabilistic-rev-sharing). Given a list
of payments pointers sharing revenue from a stream, a weighted random selection process guarantees that revenue for each payment pointer
will approach what was agreed upon. If Alice should receive 60% of a revenue stream while Bob should receive 40%, applying a random selection
where Alice has a 60% chance of being selected for any given payment is a straightforward method ensuring that revenue is shared as intended.
This statement becomes quickly accurate as the number of payments grows.

However, **how can the selection process be guaranteed to be fair, beyond any doubt?** Any agreement is as strong or as weak as the trust
in the enforcement of that agreement is. Given the openess and world-wide reach of Interledger, any solution must suit grey regulatory areas
and allow building constructive economic relationship even when members have not yet developed trust. As outlined in
[this article](https://www.fairpayzone.com/2021/04/web-monetization-and-payments-meet.html), lack of trust between parties is commonly
expected at the start of an economic relationship. Promoting a cycle of fair exchanges build the trust capital that is necessary to
strengthen relationships towards mutual benefits.

It follows that any revenue sharing solution in this context should be trustless from a technological point of view so that parties involved
have a sane basis for building trust from a human point of view. To achieve this requirement, this project aims to provide a solution based
on blockchain technology. A series of guidelines exposed in this document aims to ensure that the solution:

- Remains financially inclusive in spite of the speculative and volatile nature of blockchain
- Incur predicatble fees that are provably sensible given the context of Web Monetization
- Ensure high performance since any delay in content delivery tend to reduce monetization
- Never handle any money in any form, merely handles the selection process
- Do not put pressure on using a particular currency
- Does not lock users in
- Promote openness, is not meant to be a mandatory solution
- Do not require users to divulge more private information than payment pointers
- Promote fair revenue sharing governance by design

The [Convex network](https://convex.world) has been chosen as a platform of choice for this project. The following sections provide detail
as to how those guildines will be respected and how the set goals will be achieved.


## Share-based governance

Convex Lisp is a programming language providing a unique [smart contract framework](https://convex.world/cvm). On one hand, this framework
is Turing-complete and any other Turing-complete networks with common smart contract capabilities will be able to replicate the solutions
developed in this repository. On the other hand, the key differenciating factors behind Convex Lisp provide additional guarantees typically not
met in other decentralized technologies. Hence, it is an opportunity to work on a portable solution while exploring new ventures.

The benefits of describing revenue sharing agreements *"on chain"* is twofold. First, the state of the network is securely replicated
world-wide in a trustless manner, meaning no party can tamper with agreements. Second, smart contracts enforce revenue sharing rules
automatically in an indisputable way. Rules, just like the possible evolution of those rules, are clearly defined in code the moment
a revenue sharing scheme is created by members.

Following sections review the scope of smart contracts, what can be currently leveraged, what needs to be built atop in the context of
Web Monetization and this grant. They focus on the concept of shares, rights to be selected as a payment receiver for a given revenue stream.
Shares can be staked on payments pointers by owners to rise their chance of being selected for each and every subsequent payment.

Those smart contracts:

- Will be portable to other networks with similar capabilities
- Will remain open for peer-review and reusability, with the freedom to modify them as desired
- Will set the foundations for building a web UI ; in a matter of clicks, users will have the ability to create and manage revenue sharing
streams without much technical abilities, computational power, or prior blockchain knowledge


## Creating and transferring shares

Shares can be conceptualized as fungible tokens, a common notion found in many decentralized networks. Each share represents the likelihood
of being selected as payment receiver when a payment stream is initiated in the context of a specific revenue stream. In Convex Lisp, a fungible
token can be created in a couple of lines.

Let us suppose that Alice is the original founder. For the sake of simplicity, 100 shares are minted where each share represents a 1% chance
of being selected as payment receiver for any given payment:

```clojure
;; L


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

From now on, Alice owns 60 shares, providing a 60% chance of being selected as payment receiver, while Bob owns the remaining 40 shares, providing
a 40% chance.

Just like any aspect of the Convex stack, Convex Lisp libraries are [open-source](https://github.com/Convex-Dev/convex/tree/master/convex-core/src/main/cvx/convex)
and developed in the interest of the public in a non-competitive way. Even on the Convex network, users are free to implement other schemes. However,
the asset framework provided by the Convex Foundation aims to provide interoperability between diffent kind of digital assets on the network so that
users keep the freedom to transfer and trade them as desired.


## Staking shares on payments pointers

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
However, it can be conceptualized as a mapping of owner accounts to mappings of payment pointers to allocated shares:

```clojure
;; Where Alice owns account #20 and Bob account #50.
;;
(def stakes
     {#20 {"$wallet.com/alice" 60}
      #50 {"$wallet.com/bob"   25
           "$wcallet.com/foo"  15}})
```

Both requirements will be implemented in the scope of the fungible token implementation underlying shares.


## Governance "à la carte"

While creating shares is straightforward, new questions appear, such as:

- Should shares be always transferrable?
- Should there be a member selection process first, allowing transfers only to current members?
- If so, should there be a member eviction mechanism?
- Should shares be transferrable only between members of a group or with outsiders as well?
- How can additional shares be minted, if required?

Not only are there no unique answers to such questions, group requirements are expected to evolve over time. Part of those answers resides
in the implementation of shares. Current implementation of fungible tokens can be readily augmented to put arbitrary restrictions on transfers
since such capabilities are part of the broader asset framework.

More complex governance issues, such as membership restrictions, are commonly discussed and prototyped in the context of DAOs (Decentralized
Autonomous Organizations, groups governed on a blockchain network). However, DAOs are usually much broader in scope and much more abstract regarding the aspects
they are set to manage. In contrast, this project is well-scoped to the context of revenue sharing and governance issues specific to that matter.
Hence, the many resources already existing in the space of DAOs, such as voting mechanisms, can be readily selected and improved within the specific
requirements of the project.


## License

Copyright © 2021 Adam Helinski, the Convex Foundation, and contributors

Licensed under the Apache License, Version 2.0
