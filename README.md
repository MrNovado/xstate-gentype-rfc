# The problem

XState is __not__ strongly and soundly typed and offers very limited support for type-guarding against internal inconsistency, making scaling/using/navigating/reasoning-about big schemas potentially problematic and painful (machine-implementation is mostly unaware of its internal workings):

- `createMachine' options` config is disconnected from its machine' `schema` and is unable to suggest which `actions`, `guards`, `activities`, etc. a user has to implement to create a working instance.
- `matches` tool is disconnected from its machine' `schema` and is unable to suggest which states paths are available to match against.
- Machine' `schema` is limited to state-taxonomy and does not model events/actions/transitions, etc.
- There are no `Map<State, Event>`typings restricting event variant to a state node.
- There are no `Map<Event, Transition>` typings restricting transition variant to an event.
- There are no `Map<Event, TransitionTargetSet>` typings to guard against targeting non-existent states.
- Implementation' artifacts (`actions`, `conditions`, etc) are unaware of the state/event-context in which they might be executed forcing a user to cover for unnecessary cases (excessive boilerplate).

# The solution

2-step machine-generation:

- Describe a machine using components.
- Compile this description into a complete and exhaustive schema typings with create-/match-/etc-helpers.

```
Process:
1. Describe a machine using components.
2. Compile a machine into schema typings.
3. Use the schema to create an actor.
```

```typescript
function Machine(props) {
    return (
        <Actor
            context={
                {
                    /* context obj -- types? */
                }
            }
            initial="init"
            states={[
                <StateNode
                    name="init"
                    on={[
                        <StateEvent name="START">
                            <Transition
                                target="pending"
                                cond="start-cond"
                                actions="start-action"
                            />
                        </StateEvent>,
                    ]}
                />,
                <StateNode name="pending"></StateNode>,
                <StateNode name="resolved"></StateNode>,
                <StateNode name="rejected500"></StateNode>,
                <StateNode name="rejectedUnknown"></StateNode>,
            ]}
            on={[
                <StateEvent name="STOP">
                    <Transition target="init" />
                </StateEvent>,
            ]}
        />
    );
}
```

```
Compiling the machine component will generate sound and exhaustive schema typings,
as well as the createMachine helper with sound impl typings:

type Event = ... | ...;
type Context = ...;

type Schema = ...;
const machineSchema: Schema = ...;

type Implementation = { guards: ..., actions: ..., };

export default crm
     = (impl: Implementation)
     => createMachine<Context, Schema, Event>(machineSchema, impl);

This way you wont need to worry about missing impl artefacts.
Or createMachine non-exhaustive unsound typings.
```

```
Generating exhaustive typings may potentually safe-guard ourselves
from internal inconsistencies, like targeting non-existent state:

[
<StateNode name="x"> ... transition-target="t" <- typo
<StateNode name="y"/>
<StateNode name="z"/>
]

```

```
Generating exhaustive strict state-matching interface:
 
{
   a: { ab: {}, ac: {}, ad: {} },
   b: { bc: {} },
   d: { dd: {} }
}
 
type ExhaustivePatterns = [a] | [a, ab] | [a, ac] | [a, ad] | [b] | [b, bc] | ...;
export default matches 
   = (pattern: ExhaustivePatterns)
   => xstate.matches(pattern.join("."));

```
