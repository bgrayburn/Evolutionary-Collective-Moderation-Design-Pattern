## Intent
Enable collective determination and evolution of natural language moderation policies with enforcement carried out by a moderation bot.

## Motivation
Consider a digital space (channel) consisting of users, moderation policies, and messages. In many such digital spaces, channel policies are set by the Moderators, who are individuals given that status either by Administrators or other Moderators. Moderators are then responsible for interpretation and enforcement of those policies. This can lead to moderator's subjective interpretation of policies influeing the community's identity and path, decreasing agency for the community as a whole. 

A digial space that aims to maintain community agency in regards to self-definition of policies must meet the following criteria:
- Policies are determined by the community as a whole
- Policy's enforcement must be consistent
- Policy's enforcement must be independent of who is enforcing that policy.


Democractic processes (ie. voting) are know methods of achieving the first criteria and are the method adopted by this design pattern. This could be majority voting, rank choice voting, quadratic voting, or any other voting system that can be clearly understood by community members. For the purpose of this design pattern, examples will use majority voting for simplicity, but that is not meant to be a defining attribute of this pattern and should be determined outside of this pattern and serve as context under which this pattern is implemented.

Achieving independent interpretation and enforcement of policies requires that the entity doing the interpretation and enforcement of policies is not a member of that community.

Achieving consistent interpretation and enforcement requires a lack of variable internal state of the deciding entity. This is impossible to achieve if the entity is a human. Therefor, let us consider the entity interpreting and enforcing policies to be algorithmic. At the time of this writing, a strong option for this might be an artifical intelligence bot that can receive messages, look up or otherwise utilize current policies, and return a decision about any resulting actions to be taken (ex. redact the message, kick/ban a user, etc.). 

From here on let the moderating entity be called the Moderation Bot (`ModBot`).

Of note, no interpretation of an ambiguous policy will be objective. Even using algorithmic decision making will lead to the use of additional information influencing results. Whether this information takes form as implementation details of the algorithm, information absorbed through a training process, or through another means, the algorithm must resolve ambiguities in order to complete the action of making a decision.

While subjectivity can not be avoided in the presence of ambiguous policies if a decision is to be reached, often ambiguities are only discovered over time as communities face new challenges that are unspecifically addressed by current policies. Additionally, it may be determined that upon witnessing the result of implementing certain policies, those policies had unintended consequences leading to decisions going against the will of the community such as overly restrictive policies that limit the ability of the group to self-express.

In order to adapt to community learnings about the gap between current policies and their intended state, members need the ability to propose replacing a current policy with an updated version. This update can be voted on, at which point the moderation bot can deactivate the old policy and activate the new policy.

## Applicability
Use the Evolution Collective Moderation pattern when:
- a digital space with clear membership because of the need for a clear notion of who is a member
- a durable community where members have "skin in the game" (*more technical description here*)

## Structure
```mermaid
erDiagram
  accTitle: Evolution Collective Moderation Entity Relationship Diagram
MESSAGE {
  boolean approved
}
POLICY {
  boolean active
  int votes_for
  int votes_against
}
CHANNEL }o--o{ USER : contains
CHANNEL ||--o{ MESSAGE : contains
MODBOT ||--|| POLICY : enforces
MODBOT ||--|{ MESSAGE : approves
MODBOT ||--o{ POLICY : activate
USER ||--o{ POLICY : propose
USER }o--o{ POLICY : vote
USER ||--|{ MESSAGE : create

CHANNEL ||--|| MODBOT : contains
```

## Participants
- **Channel**
    - represents a shared digital space (such as a chat room) which is the context for `Messages` to be posted and read by `Users`. 
- **Policy**
    -  defines a natural language criteria for allowed messages and resulting actions if that criteria is violated. In addition this object stores the votes for and against as voted on by users, as well as whether the policy is currently active, with the values themselves likely updated by the Moderation Bot (although this is an implementation detail that must be determined in context).
- **User**
    - represents a person interacting with a `Channel`
- **Moderation Bot** (ModBot)
    - A service that can interact with a `Channel` to determine and enforce policies 
- **Message**
    - A text string that is created by a `users` and validated by a `ModBot`
    - A message can also be an instruction to the `ModBot` to propose, replace, or deactivate a policy

## Collaborations
### Policy Selection
```mermaid
sequenceDiagram
  actor USER1
  actor USER2
  actor USER3
  USER1->>MODBOT: /propose Policy J
  activate MODBOT
  MODBOT->>POLICY_J: create policy
  MODBOT->>CHANNEL: Call for votes on Policy J
  USER1->>MODBOT: /vote for Policy J
  USER2->>MODBOT: /vote against Policy J
  USER3->>MODBOT: /vote for Policy J
  MODBOT->>POLICY_J: activate
  MODBOT->>CHANNEL: Announce Policy J is active
  deactivate MODBOT
```

### Policy Application

Assuming an active `Policy` that forbids "offensive" `Messages` and requires that both the `Message` be redacted and the `User` removed from the `Channel`:
```mermaid
sequenceDiagram
  actor USER
  USER->>CHANNEL: This is an offensive message that violates policy
  CHANNEL->>MODBOT: receives message
  activate MODBOT
  MODBOT->>POLICY: check policy
  POLICY->>MODBOT: 
  MODBOT->>CHANNEL: redacts message
  MODBOT->>USER: removes from CHANNEL
  deactivate MODBOT
```

## Consequences
Some of the benefits and liabilities of the Visitor pattern are:
1. *No reliance on a person (user) for policy enforcement.* Relying on a person (user) to enforce policy creates several problems that are avoided here. Specifically avoided are the influece of an individual `User`'s (the mod's) personal biases and self-interest both when determining policies and when choosing how to apply them.
3. *Collective determination of accepted policies.* All users that are members of a channel have the ability to contribute to determining the acceptance of proposed policies as well as to propose policies themselves.
4. *Limited to ability of ModBot to interpret and enforce policies.* The `ModBot` must understand what action is being called for within a policy and have the ability to execute that action. Without further guardrails, it is possible to write a policy that can not be acted upon.
5. *Ambiguous policies lead to influence of algorithmic bias.* Because policies may be ambiguous, they can introduce an oppurtunity for algorithmic bias in the `ModBot` to determine the outcome of a policy decision. If `Users` in a `Channel` become aware of this happening, they are able to vote to remove the ambiguity from the policy, but doing so takes aligned community action.

## Implementation
*Once a design for a reference implementation is established, it will be described at a high-level here*

## Sample Code
*Once a design for a reference implementation is established, it will be described in detail here*

## Known Uses
To date, there are no known examples of this pattern being applied. The intent of this pattern document is to motivate it's implementation.  Hopefully this section can be updated over time as this pattern is explored further in practical applications.

## Related Patterns
- AI First pass, Human Final Moderation (*need better name for this*)
- *What would you call the pattern(s) associated with PolicyKit*
- *What would you call the pattern used in current chat spaces with a human moderator?*
### Subpatterns:
- AI Moderation
- Democratic Policy Selection

## *Variations* (this is not in the og template for design patterns, not sure if I want to leave this in, these might indicate other Related Patterns)
- control whether a message is added to the history based on policies
- different types of voting systems (is there a way to make this pattern independent of voting systems? maybe using something like PolicyKit policies?)
- ability to ask the bot directly if a comment is ok (pre-check)
- ability to wait for a message to be flagged by the community before it is reviewed by the bot

## *Todos*
- [ ] make sure all diagrams have meaningful accesibility titles and descriptions

## *Outstanding Questions*
- Can this whole pattern be split apart into multiple useful patterns? Such as 1 for voting mechanism and 1 for enforcement? 
- should `Policies` be created by a `user` with a status of whether or not they are approved?
- Is there a need for more clarity around the "evolutionary" part of this pattern? (ex. How policies are updated over time)

 <p xmlns:cc="http://creativecommons.org/ns#" xmlns:dct="http://purl.org/dc/terms/"><a property="dct:title" rel="cc:attributionURL" href="https://github.com/bgrayburn/Evolutionary-Collective-Moderation-Design-Pattern">Evolutionary Collective Moderation Design Pattern</a> by <a rel="cc:attributionURL dct:creator" property="cc:attributionName" href="https://brianrayburn.tech">Brian Rayburn</a> is licensed under <a href="https://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1" target="_blank" rel="license noopener noreferrer" style="display:inline-block;">CC BY-SA 4.0<img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1" alt=""><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1" alt=""><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/sa.svg?ref=chooser-v1" alt=""></a></p> 