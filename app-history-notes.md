
# BBS Pseudonym Informative Text

## Cryptographic Pseudonyms: A Short History

The discussion of cryptographic pseudonyms for privacy preservation has a long history, with Chaum's 1985 popular article “Security without identification: transaction systems to make big brother obsolete” addressesing many of the features of such systems such as unlinkability and constraints on their use such as one pseudonym per organization and accountability for pseudonym use. Although Chaum's proposal makes use of different cryptographic primitives than we will use here, one can see similarities in the use of both secret and "public" information being combined to create a cryptographic pseudonym.

Lysyanskaya's 2000 paper also addresses the unlinkable aspects of pseudonyms but also provides protections against dishonest users. In addition they provide practical contructions similar to those used in our draft based on discrete logarithm and sigma protocol based ZKPs. Finally as part of the ABC4Trust project three flavors of pseudonyms were defined:

1. *Verifiable pseudonyms* are pseudonyms derived from an underlying secret key.
2. *Certified pseudonyms* are verifiable pseudonyms derived from a secret key that also underlies an issued credential.
3. *Scope-exclusive pseudonyms* are verifiable pseudonyms that are guaranteed to be unique per scope string and per secret key.

The BBS based pseudonyms in our draft are aimed primarily at providing the functionality of the pseudonym flavors 2. and 3. above.

## Overview: BBS Signature Bound Pseudonyms

The BBS signature scheme is based on a three party model of *signer* (aka issuer), *prover* (aka user or holder), and *verifier*.  A *prover* obtains a BBS signature from a *signer* over a list of BBS *messages* and presents a BBS proof (of signature) along with a selectively disclosed subset of the BBS *messages* to a verifier. Each BBS proof generated is unlinkable to other BBS proofs derived from the same signature and from the BBS signature itself. If the disclosed subset of BBS *messages* are not linkable then the presentations cannot be linked.

*Note*: in this section we are being loose with our language, e.g., the the statment "...presentations cannot be linked" should be appropriately qualified, see for example [Lysya2000](#Lysya2000).

BBS pseudonyms extend the BBS signature scheme to "bind" a "cryptographic pseudonym" to a BBS signature retaining all the properties of the BBS signature scheme: (a) a short signature over multiple messages, (b) selective disclosure of a subset of messages from *prover* to *verifier*, (c) unlinkable proofs.

In addition BBS pseudonyms provide for:

1. A essentially unique identifier bound to a signature/proof of signature whose linkability is under the control of the *prover* in conjunction with a *verifier* or group of *verifiers*. Such a pseudonym can be used when a *prover* revisits a *verifier* to allow a *verifier* to recognize the prover when they return or for the *prover* to assert their pseudononous identity when visiting a *verifier*
2. Assurance of per *signer* uniqueness, i.e., the *signer* assures that the pseudonyms that will be guaranteed by the signature have not been used with any other signature issued by the signer (unless a signature is intentionally reissued).
3. The *signer* cannot track the *prover* presentations to *verifiers* based on pseudonym values.
4. *verifiers* in separate "pseudonym groups" cannot track *prover* presentations.

To realize the above feature set we embed a two part pseudonym capability into the BBS signature scheme. The pseudonym's cryptographic value will be computed from a secret part, which we call the *nym_secret* and a part that is public or at least shared between the *prover* and one or more *verifiers*. The public part we call the *context_id*. The pseudonym is calculated from these two pieces using discrete exponentiation. This is similar to the computations in [Lysya200](#Lysya2000) and [ABC2014](#ABC2014). The pseudonym is presented to the *verifier* along with a ZKP that the *prover* knows the *nym_secret* and used it and the *context_id* to compute the pseudonym value. A similar proof mechanism was used in [Lysya2000](#Lysya2000). See chapter 19 of [BS2023](#BS2023) for an exposition on these types of ZKPs.

To bind a pseudonym to a BBS signature we have the *signer* utilzed Blind BBS signatures and essentially sign over a commitment to the *nym_secret*. Hence only a prover that know the *nym_secret* can generate a BBS proof from the signature (and also generate the pseudonym proof).

As in [Lysya200](#Lysya2000) we are concerned with the possibility of a dishonest user and hence require that that the *nym_secret* = *prover_nym* + *signer_nym_entropy* be the sum of two parts where the *prover_nym* is a provers secret and only sent to the *signer* in a blinding and hiding commitment. The *signer_nym_entropy* is "added" in by the *signer* during the signing procedure and sent back to the *prover* along with the signature. Note the order of operations. The *prover* chooses their (random) *prover_nym* and commits to it. They then send the commitment along with a ZKP proof that the *prover_nym* makes this commitment. The *signer* verifies the commitment to the *prover_nym* then generates the *signer_nym_entropy* and "adds" it to the *prover_nym* during the signature process. Note that this can be done since we sign over the commitment and we know the generator for the commitment.

## BBS Pseudonym Example Applications

### Certifiable Pseudonyms

*Notes*: Who gets what: *prover* gets unique *psuedonym* per *prover* chosen *context_id*, this is backed by BBS signature and verified using the signers public key. *prover* makes up the *context_id*. *nym_secret* is guaranteed unique to and by the issuer, though not known to the issuer. The *prover* can present the *context_id* plus the pseudonym to identify themselves along with whatever attribute (message) that they choose to reveal. No one else without the *nym_secret* and signature can produce a proof that they "own" the *pseudonym*. The *prover* can create as many different, unlinkable pseudonyms by coming up with different values for the *context_id*. Note that no one else can prove that they are the "owner" of a produced *pseudonym* since they do not know the *nym_secret* (the signature, and other secrets that may be contained in prover committed messages).

### Scope Exclusive Pseudonyms

*Notes*: In this case the verifier or group of verifiers require the use of a specific *context_id*. This allows the verifier (or group of verifiers) to track visits by the *prover* using this credential/pseudonym. A *verifier* can limit data collection, i.e. data retention minimization, by periodically changing the *context_id* since the pseudonyms produced using different *context_ids* cannot be linked. For example a *context_id* like "mywebsite.com/17Nov2024" that changes daily means the verifier could only track visits daily.

### Scope Exclusive Pseudonyms with Monitoring

*Notes*: This is the case where 3rd party monitoring is required. For example (completely ficticious) suppose the credential certifies that the *prover* is qualified to purchase and store some type of controlled substance, e.g., a clas of chemicals. To avoid price fixing or leakage of secret chemical formulas the *prover* purchases these chemicals under a *verifier* (vendor) specific pseudonym. Which prevents the different vendors from colluding on prices or seeing all the chemicals being purchase by a given prover. However for public safety, hording prevention, etc... verifiers/vendors are required to report all purchase to a 3rd party monitor along with the pseudonym under which the purchases were made (and the *context_id* of the vendor). To allow the 3rd party monitor or link these pseudonyms to a prover, the prover would be required to reveal the *nym_secret* associated with this credential only to the *monitor*. Note that this is why we separate *nym_secrets* from other secrets that might be used to "bind" a credential to a holder...

## A Short Selection of References

1. D. Chaum, “Security without identification: transaction systems to make big brother obsolete,” Commun. ACM, vol. 28, no. 10, pp. 1030–1044, Oct. 1985, doi: 10.1145/4372.4373. <a id="Chaum85"></a>
2. A. Lysyanskaya, R. L. Rivest, A. Sahai, and S. Wolf, “Pseudonym Systems,” in Selected Areas in Cryptography, vol. 1758, H. Heys and C. Adams, Eds., in Lecture Notes in Computer Science, vol. 1758. , Berlin, Heidelberg: Springer Berlin Heidelberg, 2000, pp. 184–199. doi: 10.1007/3-540-46513-8_14. <a id="Lysya2000"></a>
3. P. Bichsel et al., “D2.2 - Architecture for Attribute-based Credential Technologies - Final Version,” Aug. 2014. See https://abc4trust.eu/download/Deliverable_D2.2.pdf. <a id="ABC2014"></a>
4.D. Boneh and V. Shoup, “A Graduate Course in Applied Cryptography”. Version 0.6 See https://toc.cryptobook.us/book.pdf <a id="BS2023"></a>
