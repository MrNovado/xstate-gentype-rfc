# The problem

XState is __not__ strongly and soundly typed and offers very limited support for type-guarding against internal inconsistency, making scaling/using/navigating/reasoning-about big schemas potentially problematic and painful (machine-implementation is mostly unaware of its internal workings):

- `createMachine' options` config is disconnected from its machine' `schema` and is unable to suggest which `actions`, `guards`, `activities`, etc. a user has to implement to create a working instance.
- `matches` tool is disconnected from its machine' `schema` and is unable to suggest which states paths are available to match against.
- Machine' `schema` is limited to state-taxonomy and does not model events/actions/transitions, etc.
- There are no support for `Map<State, Event>`typings (restricting event variant to a state node).
- There are no support for `Map<Event, Transition>` typings (restricting transition variant to a state node)
- There are no `Map<Event, TransitionTargetSet>` typings to guard against targeting non-existent states.

# The solution (wip)

```typescript
/**
 * Atoms to generate exhaustive sound schema typings
 * (and implementation as well?) for the actual actors.
 *
 * Process:
 * 1. Describe a machine using components.
 * 2. Compile a machine into schema typings.
 * 3. Use the schema to create an actor.
 *
 * ====== not an issue?
 * # What about strict state2Event mapping?
 * # What about strict event2Transition mapping?
 * # What about strict exhaustive 'matches' interface? 
 */

function Actor(props) {
    return null;
}

function StateNode(props) {
    return null;
}

function StateEvent(props) {
    return null;
}

function Transition(props) {
    return null;
}

/**
 * This is basically a schema component;
 *
 * Used for:
 * -> generating impl typings
 */
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

/**
 * Compiling the machine component will generate sound and exhaustive schema typings,
 * as well as the createMachine helper with sound impl typings:
 *
 * type Event = ... | ...;
 * type Context = ...;
 *
 * type Schema = ...;
 * const machineSchema: Schema = ...;
 *
 * type Implementation = { guards: ..., actions: ..., };
 *
 * export default crm
 *      = (impl: Implementation)
 *      => createMachine<Context, Schema, Event>(machineSchema, impl);
 *
 * This way you wont need to worry about missing impl artefacts.
 * Or createMachine non-exhaustive unsound typings.
 *
 * ======
 * # Is it at all beneficial?
 * # Is typing Impl by hand difficult?
 * # What about state2Event/event2Transition mappings?
 * # What about state2Event/event2Transition internal inconsistancies?
 */

 /**
  * Generating exhaustive typings may potentually safe-guard ourselves
  * from internal inconsistencies, like targeting non-existent state:
  * 
  * [
  * <StateNode name="x"> ... transition-target="t" <- typo
  * <StateNode name="y"/>
  * <StateNode name="z"/>
  * ]
  */

  /**
   * Generating exhaustive strict state-matching interface:
   * 
   * {
   *    a: { ab: {}, ac: {}, ad: {} },
   *    b: { bc: {} },
   *    d: { dd: {} }
   * }
   * 
   * type ExhaustivePatterns = [a] | [a, ab] | [a, ac] | [a, ad] | [b] | [b, bc] | ...;
   * export default matches 
   *    = (pattern: ExhaustivePatterns)
   *    => xstate.matches(pattern.join("."));
   */
   ```
