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
- Based on elements from the Interledger protocol

The solution outlined in this repository is best suited for the nascent [Web Monetization API](https://webmonetization.org) allowing
browser users to stream payments to accounts identified by URLs. On a very high-level, it builds on the concept of
[probabilistic revenue sharing](https://webmonetization.org/docs/probabilistic-rev-sharing) and proofs that an initially simple idea
can be magnified to provide novel opportunities.


## Current status

A proposal will been submitted to [Grant For The Web](https://www.grantfortheweb.org/) on September 22nd for the
July 2021 round.


## License

Copyright © 2021 Adam Helinski, the Convex Foundation, and contributors

Licensed under the Apache License, Version 2.0
