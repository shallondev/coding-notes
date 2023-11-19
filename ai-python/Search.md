# Search

## Search Problem

Terminology 
- agent: entity that perceives its environment and acts upon that environment.
- state: configuration of a agent and its environment.
    - inital state: state where agent begins.
- actions: choices that can be made in a state.
    - ACTIONS(s): returns a set of actions that can be executed in state s.
- transition model: a description of what state results from performing any applicable action in any state.
    - RESULTS(s, a) returns the state resulting from performing action a in state s.
- state space: the set of all states reachable from the initial state by any sequence of actions.