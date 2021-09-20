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
strengthen the relationship towards mutual benefits.

It follows that any revenue sharing solution in this context should be trustless from a technological point of view so that parties involved
have a sane basis for building trust from a human point of view. To achieve this requirement, this project aims to provide a solution based
on blockchain technology. A series of guidelines and technical implementation exposed in this document aims to ensure that the solution:

- Remains financially inclusive in spite of the speculative and volatile nature of blockchain
- Incur fees that are sensible given the context of Web Monetization
- Ensure high performance since any delay in content delivery tend to reduce monetization
- Promote openness, preserving the freedom to leave the platform
- Do not require users to divulge more private information than payment pointers
- Promote fair revenue sharing governance by design


## License

Copyright © 2021 Adam Helinski, the Convex Foundation, and contributors

Licensed under the Apache License, Version 2.0
