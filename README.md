# Bitcoin lottery contract
A lottery in bitcoin script with no escrow, based on random number generation and modular arithmetic

This is a series of smart contracts that allow multiple parties to participate in a fair bitcoin lottery. 2, 3, 4, or 5 players can play. Each player deposits an agreed-upon amount of money into a bitcoin smart contract and only one randomly selected party gets to withdraw the total amount.

# How can I play?

Click here: https://supertestnet.github.io/bitcoin-lottery-contract/ - play with test bitcoin

Or here: https://supertestnet.github.io/bitcoin-lottery-contract/mainnet.html - play with real bitcoin

# Random number generation

A random number is generated by taking a seed from each player, specifically a number 0 through N-1 (where N is the number of players), summing the seeds submitted by all players, and then doing modular arithmetic on the sum over a finite field from 0 through N-1. The result is a random number from 0 through N-1 that no party could predict in advance without knowing what number every other party picked. This random number is then used as the basis for withdrawal logic: if the random number is 0, player 1 gets to withdraw; if it is 1, player 2 gets to withdraw, etc.

# Random numbers? I thought bitcoin script couldn't generate random numbers

It can. Bitcoin script can do modular arithmetic and take inputs from multiple parties. That's all you need to generate random numbers, so bitcoin script can generate random numbers.

# But modular arithmetic requires loops! Bitcoin script doesn't have loops!

You don't need loops to do modular arithmetic if you can copy/paste a function a bunch of times. I wrote a small function that compares X to Y and does some subtraction if X is greater than Y. Then I repeat that as many times as there are players, and voila, the result is modular arithmetic without loops.

# Wait you said every player has to provide a seed. How do you enforce that?

Through hashed timelocked contracts. Before the game, the players each create a random byte string that is at least 16 bytes long and send the hash of their string to the other players. They then must deposit money into a hashed timelocked contract where they only get the money back if they reveal the bytestring they chose. The length of the bytestring encodes the number they picked: if it is 16 bytes, they picked a 0. If it is 17 bytes, they picked a 1, etc. Also, the amount of money they put into this HTLC is equal to the amount needed to make every other player a winner; if any player doesn't reveal their preimage via the htlc, the sats they put up as collateral go to the other players, thus making everyone a winner except the one person who chose not to reveal their preimage. A good consequence of this design is that each player only has one way to avoid losing: they must reveal the number they picked. If they don't, every other player becomes a winner.

# Someone said this project also involves a coinjoin. What's up with that?

Yeah, one of the things I needed to do was make sure everyone in the game sends their money into the lottery address and the collateral addresses at the same time. The only way I know how to do that is with a coinjoin, so I wrote some coinjoin software to enable that. It's included in the index.html file.

# Wait you made a coinjoin implementation in javascript??

Yep.

# ARE YOU NUTS??!?

Yes.

# Who is the coordinator?

The short answer is: I alphabetize the players' pubkeys and choose the "highest" pubkey as the coordinator. I wish I could say there is no coordinator, but that's not quite true. I did manage to reduce the role of the coordinator compared to some other coinjoin software, but at a significant cost: the coordinator's only roles are to cosign the coinjoin and publish its final state, which is not much of a coordinative role, but the downside is that -- to reduce it this far -- I have every player tell their utxo data to every other player, including the coordinator. In most other coinjoin software, the coordinator is the only one who learns in advance which utxos will be part of a particular coinjoin. Here, everyone involved learns that information, so it's not as privacy preserving as other coinjoin software. Still, the coordinator is just another player, and the coordination is minimal, which means (I hope) the coordinator doesn't need to take a fee for their service. That should make this coinjoin software cheaper than other coinjoin software.

# TODO

- Change all the transactions to use a reasonable fee, not a hard coded 500 sats
- Detect possible cheating attempts in advance by noticing unexpected delays and indicate for users what will happen next (e.g. when they will be guaranteed to get their money back)
- Detect if the funding transaction never confirms (such as due to successful mempool manipulation by one of the players) and if that happens, auto abort on behalf of each player
- Automatically warn users if they obviously entered someone's nostr pubkey incorrectly (e.g. not 64 characters, or not hex encoded, or their own pubkey)
- Add resliency. Users report that if they switch apps on mobile while things are getting signed, the progress bar never moves again. I suspect their device severs their websocket connection and when it reconnets, they don't get the messages they need to get because the messages are ephemeral. Topher suggests not using ephemeral events but rather requesting event deletion after an event is no longer needed. (But relays aren't meant to delete events upon deletion requests, clients are just supposed to hide them, so maybe that's dumb?)
