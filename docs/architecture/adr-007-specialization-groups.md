# ADR 007: Specialization Groups

## Changelog

- 2019 Jul 31 : Initial Conception

## Context

This idea was first conceived of in order to fulfill the use case of the
creation of a decentralized Computer Emergency Response Team (dCERT), Whose
members would be elected by the Cosmos Community and would fulfill the role of
coordinating the Cosmos community under an emergency situations. This thinking
can be further abstracted into the conception of "blockchain specialization
groups". 

The creation of these groups are the beginning of specialization capabilities
within a wider blockchain community which could be used to enable a certain
level of delegated responsibilities. Examples of specialization which could be
beneficial to a blockchain community include: code auditing, emergency response,
code development etc. This type of community organization paves the way for
individual stakeholders to delegate votes by issue type, if in the future
governance proposals include a field for issue type. 


## Decision

A specialization group can be broadly broken down into
the following functions: 
 - Membership Admittance
   - via governance
   - via special appointment
 - Membership Acceptance
 - Membership Revocation
   - (probably) Without Penalty 
     - member steps down (self-Revocation)
     - replaced by new member from governance
   - (probably) With Penalty 
     - due to breach of soft-agreement (determined through governance)
     - due to breach of hard-agreement (determined by code) 
 - Execution of Duties
   - Special transactions which only execute for members of a specialization group
 - Compensation
   - Group compensation (further distribution decided by the specialization group) 
   - Individual compensation for all constituents of a group from the
     greater community

Election of the members of a specialization group can happen in a wide variety
of ways and be subject to an arbitrary number of associated rules. For its
initial implementation as a part of this ADR we recommend that a general
election abstraction (`Electionator`) is provided as well as a basic
implementation of that abstraction which allows for a continuous election of
members of a specialization group. 

``` golang
// The Electionator abstraction covers the concept space for 
// a wide variety of election kinds.  
type Electionator interface {
    
    // is the election object accepting votes.
    Active() bool 

    // functionality to execute for when a vote is cast in this election, here
    // the vote field is anticipated to be marshalled into a vote type used 
    // by an election. 
    Vote(addr sdk.AccAddress, vote []byte) 

    // here lies all functionality to authenticate and execute changes for
    // when a member accepts being elected
    Accept(sdk.AccAddress) 

    // Register a revoker object
    RegisterRevoker(Revoker)

    // No more revokers may be registered after this function is called
    SealRevokers()

    // register hooks to call when an election actions occur
    RegisterHooks(ElectionatorHooks) 

    // query for the current winner(s) of this election based on arbitrary
    // election ruleset
    QueryWinners() []sdk.AccAddress 

    // query metadata for an address in the election this 
    // could include for example position that an address
    // is being elected for within a group   
    // 
    // this metadata may be directly related to 
    // voting information and/or privileges enabled
    // to members within a group. 
    QueryMetadata(sdk.AccAddress) []byte
}

type ElectionatorHooks interface {
    VoteCast(addr sdk.AccAddress, vote []byte)
    MemberAccepted(addr sdk.AccAddress)
    MemberRevoked(addr sdk.AccAddress, cause []byte)
}

// Revoker defines the function required for an membership revocation rule-set
// used by a specialization group. This could be used to create self revoking,
// and evidence based revoking, etc. Revokers types may be created and
// reused for different election types. 
// 
// When revoking the "cause" bytes may be arbitrarily marshalled into evidence,
// memos, etc.
type Revoker interface {
    RevokeName() string      // identifier for this revoker type 
    RevokeMember(addr sdk.AccAddress, cause []byte) (successful bool)
}
```

Certain level of commonality likely exists between the existing code within
`x/governance` and required functionality of elections. This common
functionality should be abstracted during implementation. Similarly for each 
vote implementation client CLI/REST functionality should be abstracted
to be reused for multiple elections. 

The specialization group abstraction firstly extends the `Electionator`
but also further defines traits of the group. 

``` golang
type SpecializationGroup interface {
    Electionator 
    GetName() string
    GetDescription() string 

    // general soft contract the group is expected
    // to fulfill with the greater community
    GetContract() string

    // messages which can be executed by the members of the group
	Handler(ctx sdk.Context, msg sdk.Msg) sdk.Result

    // logic to be executed at endblock, this may for instance
    // include payment of a stipend to the group members
    // for participation in the security group.   
	EndBlocker(ctx sdk.Context)
}
```

## Status

> Proposed

## Consequences

### Positive

 - increases specialization capabilities of a blockchain

### Negative

 - could be used to increase centralization within a community

### Neutral

## References

  (dCERT ADR)[./adr-008-dCERT-group.md]
 