# Burning Tyger

> Tyger Tyger, burning bright,  
> In the forests of the night;  
> What immortal hand or eye,  
> Could frame thy fearful symmetry?  
> > By: William Blake  

# Cause
For the lack of a low cost solution to an electronic ballot system which addresses 3 key problems.

1. Voter anonymity.
2. Vote validity (including vote deduplication)
3. Instantaneous results.


# Concept (problem/solution)
Solution involves following subjects.

1. Voters.
2. Ballot organizer.
3. Candidates.

Given opposing ideas, the goal here is to determine the most popular one.

Voter registration is considered out of scope for the purposes of this solution. Suffice to say that voters are registered in a central database where each voter (identified by National ID for example) is associated to the public counterpart of a PKI key pair. Note that the candidates are also registered in the same manner in this database. Voter registration takes place prior to the opening of a given ballot.

This database sits behind a certification authority which publishes the X.509 certificates of all voters and candidates.

Suppose 4 ideas compete for most popular. Each idea is represented by a candidate. Ideally each voter must cast a vote only once to any one of the 4 candidates of their choosing; although this may very well not be the case (duplicates can and will happen).

Each voter holds an electronic device which runs proprietary software for the purpose of casting their vote. From here on out this device is assumed to be a smartphone. It is this smartphone that allows each voter to securely hold the private counter part of their PKI key pair. In effect, each voter's identity is represented by their smartphone.

Given the scenario of 4 candidates, when a voter is ready to cast the vote, the following will happen:  

1. Mobile app asks central system for a new vote code; the central system complies and answers with a brand new vote code and the specific method and sequence of encryption.
2. Voter is presented with an interface to input which candidate he/she chooses to vote for. Voter chooses.
3. Mobile app splits the vote into 2 parts. Part one being voter of the impending vote whereas part two being what candidate the vote is casted to.
4. Mobile app prepends the vote code from step 1 to each of the two parts resulting in the final plain texts, each of which will then be fed into the encryption routine.
5. Guided by the response given from central server in step 1, mobile app encrypts each of the two parts using the explicitly specified public keys. In this case server will specify two public keys for each part since there are 4 candidates.
    - Part One cypher text = encrypt(encrypt(encrypt(Part One Plaintext, PartOnePubKey0), PartOnePubKey1), OrganizerPubKey)
    - Part Two cypher text = encrypt(encrypt(encrypt(Part Two Plaintext, PartTwoPubKey0), PartTwoPubKey1), OrganizerPubKey)
6. Mobile app transmits these two parts to the central system at two different timings. The system stores the two parts of the vote on file.

Given the scenario of 4 candidates, when the ballot organizer is ready to close the ballot and initiate a count, the following will happen:  
1. Central system first handles determining a valid list of vote codes. This is achieved by the following steps.  
    - Decrypts all part 1's of the casted votes using organizer private key.
    - Transmits the complete list of part 1 data to each candidate (only to designated part 1 encryptors); whereby the recipient candidate must decrypt each row using their private key and return the resulting list.
    - Each candidate in this cycle handles the task in the same manner except for the the last one. Difference is, since the plain texts are available for this last decryptor, he/she will remove duplicate voters and return only this deduplicated list of vote codes. Note that the voter information is truncated/dropped before returning the result to the central system.
2. Now that the system has a valid list of vote codes each representing a unique voter, it is ready to carry out the final part of the count task. That is, to get a count of votes for each candidate. This is achieved by carrying out the following steps.
    - Decrypts all part 2's of the casted votes using the organizer private key.
    - Transmits the complete list of part 2 data to each candidate (only to designated part 2 encryptors); whereby the recipient candidate must decrypt each row using their private key and return the resulting list. Note that the valid voter list (deduced in Step 1) is also attached to this transmission.
    - Each candidate in this cycle handles the task in the same manner except for the the last one. Difference is, since the plain texts are available for this last decryptor, he/she will eliminate all votes that is not in the valid vote codes attachment. Once elimination is complete, now a count of votes won by each candidate is carried out and the result replied to the central system.
3. Central system reports the ballot results in a proper format.

NOTE: Refer to Notion notes for an alternate [system spec](https://www.notion.so/Voting-system-based-on-PKI-266a4feaf124482297d87f4961987a91).