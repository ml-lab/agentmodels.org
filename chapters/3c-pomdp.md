---
layout: chapter
title: "Environments with hidden state: POMDPs"
description: Mathematical formalism for POMDPs, Bandit and Restaurant Choice examples. 
---


 
## Introduction: Learning about the world from observation

The previous chapters included MDPs where the transition function is *stochastic*. As a consequence of stochasticity, the agent is *uncertain* about the result of taking an action in a given state. For instance, the agent in the Hiking Problem is uncertain whether choosing the move "right" will result in going right or falling down the hill. This kind of uncertainty cannot be altered or reduced by observation. Transitions occur according to a fixed probability distribution with no parameters the agent can learn from experience. In terms of an agent's uncertainty, an MDP is like a fair lottery, where observing the winning ticket one week does not reduce the agent's uncertainty over the outcome the following week[^mdp].

[^mdp]: This doesn't mean MDPs always represent real-world problems with the structure of lotteries or casino games. The MDP could represent a situation where events are deterministic but the transition function is either unknown or is too complex to include in the model. 

In contrast, we often face problems where our uncertainty can be *reduced* by observation. In choosing between restaurants, we rarely have complete knowledge of restaurants in the neighborhood. We are uncertain about opening hours, the chance of getting a table, the quality of restaurants, the exact distances between locations, and so on. This uncertainty can be reduced by observation: we walk to a restaurant and see whether it's open. In other examples, the environment is stochastic but the agent can gain knowledge of the *distribution* on outcomes. For example, in Multi-arm Bandit problems, the agent learns about the distribution over rewards given by each of the arms.

To represent decision problems where the agent's uncertainty is altered by observations, we use Partially Observable Markov Decision Processes (POMDPs). We first introduce the formalism for POMDPs and then show how to extend our agent model for MDPs to an agent model that solves POMDPs. 


## POMDP Agent Model

### Informal overview

The agent facing a POMDP has initial uncertainty about features of the environment. These features are either external to the agent (e.g. whether a restaurant is open) or directly involve the agent (e.g. the agent's position on a grid). The features influence the environment's transitions or the agent's utilities and so are relevant to the agent's choices.

In a POMDP, the agent does not directly observe transitions or utilities. Instead they learn about the unknown features of the environment indirectly via *observations*. When the agent visits a state, they receive an observation that depends on the state and their previous action (according to a fixed *observation function*). Observations can inform the agent of local, transient facts (e.g. the agent's current grid position) and of persistent features of the environment (e.g. whether a wall exists in a particular location).

The environment in a POMDP has same structure as an MDP, with the addition of an observation function. In an MDP, the agent always "knows" the current state and chooses an action given that knowledge. In a POMDP, the agent has a probability distribution over the current state. At every timestep, they update this distribution on the current observation and then choose an action (based on their updated distribution over states). Their chosen action causes a state transition exactly as in an MDP. 

For a concrete example, consider the Restaurant Choice Problem. Suppose Bob doesn't know whether the Noodle Shop is open. Previously, the agent's state consisted of Bob's *location* on the grid as well as the remaining time. In the POMDP case, the state also represents whether or not the Noodle Shop is open. This fact about the state determines whether Bob can enter the Noodle Shop. When Bob is next to the Noodle Shop, he gets to observe (via the observation function) whether or not it's open.



### Formal model

<a id="notation"></a>
We first define the class of decision probems (POMDPs) and then define an agent model for optimally solving these problems. Our definitions are based on reft:kaelbling1998planning.

A Partially Observable Markov Decision Process (POMDP) is a tuple $$ \left\langle S,A(s),T(s,a),U(s,a),\Omega,O \right\rangle$$, where:

- $$S$$ (state space), $$A$$ (action space), $$T$$ (transition function), $$U$$ (utility or reward function) form an MDP as defined in [chapter 3.1](/chapters/3a-mdp.html), with $$U$$ assumed to be deterministic[^utility]. 

- $$\Omega$$ is the finite space of observations the agent can receive.

- $$O$$ is a function  $$ O\colon S \times A \to \Delta \Omega $$. This is the *observation function*, which maps an action $$a$$ and the state $$s'$$ resulting from taking $$a$$ to an observation $$o \in \Omega$$ drawn from $$O(s',a)$$.

[^utility]: In the RL literature, the utility or reward function is often allowed to be *stochastic*. Our agent models assume that the agent's utility function is deterministic. To represent environments with stochastic "rewards", we treat the reward as a stochastic part of the environment (i.e. the world state). So in a Bandit problem, instead of the agent receiving a (stochastic) reward $$R$$, they transition to a state to which they assign a fixed utility $$R$$. (Why do we avoid stochastic utilities? One focus of this tutorial is inferring an agent's preferences. The preferences are fixed over time and non-stochastic. We want to identify the agent's utility function with their preferences). 

So at each timestep, the agent transitions from state $$s$$ to state $$s' \sim T(s,a)$$ (where $$s$$ and $$s'$$ are generally unknown to the agent) having performed action $$a$$. On entering $$s'$$ the agent receives an observation $$o \sim O(s',a)$$ and a utility $$U(s,a)$$. 

To characterize the behavior of an expected-utility maximizing agent, we need to formalize the belief-updating process. Let $$b$$, the current belief function, be a probability distribution over the agent's current state. Then the agent's succesor belief function $$b'$$ over their next state is the result of a Bayesian update on the observation $$o \sim O(s',a)$$ where $$a$$ is the agent's action in $$s$$. That is:

<a id="belief"></a>**Belief-update formula:**

$$
b'(s') \propto O(s',a,o)\sum_{s \in S}{T(s,a,s')b(s)}
$$

Intuitively, the probability that $$s'$$ is the new state depends on the marginal probability of transitioning to $$s'$$ (given $$b$$) and the probability of the observation $$o$$ occurring in $$s'$$. The relation between the variables in a POMDP is summarized in Figure 1 (below).

<img src="/assets/img/pomdp_graph.png" alt="diagram" style="width: 400px;"/>

>**Figure 1:** The dependency structure between variables in a POMDP. The ordering of events is as follows:

>(1). The agent chooses an action $$a$$ based on belief distribution $$b$$ over their current state (which is actually $$s$$).

>(2). The agent gets utility $$u = U(s,a)$$ when leaving state $$s$$ having taken $$a$$.

>(3). The agent transitions to state $$s' \sim T(s,a)$$, where it gets observation $$o \sim O(s',a)$$ and updates its belief to $$b'$$ by updating $$b$$ on the observation $$o$$.

In our previous agent model for MDPs, we defined the expected utility of an action $$a$$ in a state $$s$$ recursively in terms of the expected utility of the resulting pair of state $$s'$$ and action $$a'$$. This same recursive characterization of expected utility still holds. The important difference is that the agent's action $$a'$$ in $$s'$$ depends on their updated belief $$b'(s')$$. Hence the expected utility of $$a$$ in $$s$$ depends on the agent's belief $$b$$ over the state $$s$$. We call the following the **POMDP Expected Utility of State Recursion**. This recursion defines the function $$EU_{b}$$, which is analogous to the *value function*, $$V_{b}$$, in reft:kaelbling1998planning.

<a id="pomdp_eu_state"></a>**POMDP Expected Utility of State Recursion:**

$$
EU_{b}[s,a] = U(s,a) + \mathbb{E}_{s',o,a'}(EU_{b'}[s',a'_{b'}])
$$

where:

- we have $$s' \sim T(s,a)$$ and $$o \sim O(s',a)$$

- $$b'$$ is the updated belief function $$b$$ on observation $$o$$, as defined <a href="#belief">above</a>

- $$a'_{b'}$$ is the softmax action the agent takes given belief $$b'$$

The agent cannot use this definition to directly compute the best action, since the agent doesn't know the state. Instead the agent takes an expectation over their belief distribution, picking the action $$a$$ that maximizes the following:

$$
EU[b,a] = \mathbb{E}_{s \sim b}(EU_{b}[s,a])
$$

We can also represent the expected utility of action $$a$$ given belief $$b$$ in terms of a recursion on the successor belief state. We call this the **Expected Utility of Belief Recursion**, which is closely related to the Bellman Equations for POMDPs: <a id="pomdp_eu_belief"></a>

$$
EU[b,a] = \mathbb{E}_{s \sim b}( U(s,a) + \mathbb{E}_{s',o,a'}(EU[b',a']) )
$$

where $$s'$$, $$o$$, $$a'$$ and $$b'$$ are distributed as in the Expected Utility of State Recursion.

Unfortunately, finding the optimal policy for POMDPs is a difficult task. In fact, even in the special case where the observations are deterministic and the horizon is finite, the task of determining whether the optimal policy has expected utility greater than some constant is PSPACE-complete refp:papadimitriou1987complexity.

### Implementation of the Model
<a id="pomdpCode"></a>

As with the agent model for MDPs, we provide a direct translation of the equations above into an agent model for solving POMDPs. The variables `nextState`, `nextObservation`, `nextBelief`, and `nextAction` correspond to $$s'$$,  $$o$$, $$b'$$ and $$a'$$ respectively, and we use the Expected Utility of Belief Recursion. The following codebox defines the `act` and `expectedUtility` functions. We define the functions `updateBelief`, `transition`, `observe` and `utility` below. 

<!-- pomdp_agent -->
~~~~
var act = function(belief) {
  return Infer({ model() {
    var action = uniformDraw(actions);
    var eu = expectedUtility(belief, action);
    factor(alpha * eu);
    return action;
  }});
};

var expectedUtility = function(belief, action) {
  return expectation(
    Infer({ model() {
      var state = sample(belief);
      var u = utility(state, action);
      if (state.terminateAfterAction) {
        return u;
      } else {
        var nextState = transition(state, action);
        var nextObservation = observe(nextState);
        var nextBelief = updateBelief(belief, nextObservation, action);
        var nextAction = sample(act(nextBelief));
        return u + expectedUtility(nextBelief, nextAction);
      }
    }}));
};

// To simulate the agent, need to transition
// the state, sample an observation, then
// get agent's action (after agent updates belief)

// *startState* is agent's actual startState (unknown to agent)
// *priorBelief* is agent's initial belief function

var simulate = function(startState, priorBelief) {

  var sampleSequence = function(state, priorBelief, action) {
    var observation = observe(state);
    var belief = updateBelief(priorBelief, observation, action);
    var action = sample(act(belief));
    var output = [ [state, action] ];

    if (state.terminateAfterAction){
      return output;
    } else {
      var nextState = transition(state, action);
      return output.concat(sampleSequence(nextState, belief, action));
    }
  };
  return sampleSequence(startState, priorBelief, 'noAction');
};
~~~~

## Applying the POMDP agent model

### Two-arm, deterministic, IRL Bandits

We apply the POMDP agent to a simplified variant of the Multi-arm Bandit Problem. In this variant, pulling an arm produces a *prize* deterministically. The agent begins with uncertainty about the mapping from arms to prizes and learns over time by trying the arms. In our concrete example, there are only two arms. The first arm is known to have the prize "chocolate" and the second arm either has "champagne" or has no prize at all ("nothing"). See Figure 2 (below) for details. We refer to Bandit problems with prizes (e.g. chocolate) as "IRL Bandits" to distinguish them from the standard Bandit problems with numerical rewards. <!-- TODO I don't like this explanation of the term "IRL bandit" - reading the paragraph, you don't get an explanation why "IRL" was chosen or what it has to do with inverse reinforcement learning. - Daniel -->

<img src="/assets/img/3c-irl-bandit.png" alt="diagram" style="width: 500px;"/>

>**Figure 2:** Diagram for deterministic Bandit problem used in the codebox below. The boxes represent possible deterministic mappings from arms to prizes. Each prize has a utility $$u$$. On the right are the agent's initial beliefs about the probability of each mapping. The true mapping (i.e. true *latent state*) has a solid outline.

In our implementation of this problem, the two arms are labeled "0" and "1" respectively. The *action* of pulling `Arm0` is also labeled "0" (and likewise for `Arm1`). After taking action `0`, the agent transitions to a state corresponding to the prize for `Arm0` and the gets to observe this prize. States are Javascript objects that contain a property for counting down the time (as in the MDP case) as well as a `prize` property. States also contain the *latent* mapping from arms to prizes (called `armToPrize`) that determines how an agent transitions on pulling an arm.

~~~~
// Pull arm0 or arm1
var actions = [0, 1];

// Use latent "armToPrize" mapping in state to
// determine which prize agent gets
var transition = function(state, action){
  var newTimeLeft = state.timeLeft - 1;
  return extend(state, {
    prize: state.armToPrize[action], 
    timeLeft: newTimeLeft,
    terminateAfterAction: newTimeLeft == 1
  });
};

// After pulling an arm, agent observes associated prize
var observe = function(state){
  return state.prize;
};

// Starting state specifies the latent state that agent tries to learn
// (In order that *prize* is defined, we set it to 'start', which
// has zero utilty for the agent). 
var startState = { 
  prize: 'start',
  timeLeft: 3, 
  terminateAfterAction:false,
  armToPrize: { 0: 'chocolate', 1: 'champagne' }
};
~~~~

Having illustrated our implementation of the POMDP agent and the Bandit problem, we put the pieces together and simulate the agent's behavior. The `makeAgent` function is a simplified version of the library function `makeBeliefAgent` used throughout the rest of this tutorial[^makeBelief].

The <a href="#belief">Belief-Update Formula</a> is implemented by `updateBelief`. Instead of hand-coding a Bayesian belief update, we simply use WebPPL's built in inference primitives. This approach means our POMDP agent can do any kind of inference that WebPPL itself can do. For this tutorial, we use the inference function `Enumerate`, which captures exact inference over discrete belief spaces. By changing the inference function, we get a POMDP agent that does approximate inference and simulates their future selves as doing approximate inference. This inference could be over discrete or continuous belief spaces. (WebPPL includes Particle Filters, MCMC, and Hamiltonian Monte Carlo for differentiable models). 

[^makeBelief]: One difference between the functions is that `makeAgent` uses the global variables `transition` and `observation`, instead of having a `world` parameter.

~~~~
///fold: Bandit problem is defined as above

// Pull arm0 or arm1
var actions = [0, 1];

// Use latent "armToPrize" mapping in state to
// determine which prize agent gets
var transition = function(state, action){
  var newTimeLeft = state.timeLeft - 1;
  return extend(state, {
    prize: state.armToPrize[action], 
    timeLeft: newTimeLeft,
    terminateAfterAction: newTimeLeft == 1
  });
};

// After pulling an arm, agent observes associated prize
var observe = function(state){
  return state.prize;
};

// Starting state specifies the latent state that agent tries to learn
// (In order that *prize* is defined, we set it to 'start', which
// has zero utilty for the agent). 
var startState = { 
  prize: 'start',
  timeLeft: 3, 
  terminateAfterAction:false,
  armToPrize: {0:'chocolate', 1:'champagne'}
};
///

// Defining the POMDP agent

// Agent params include utility function and initial belief (*priorBelief*)

var makeAgent = function(params) {
  var utility = params.utility;

  // Implements *Belief-update formula* in text
  var updateBelief = function(belief, observation, action){
    return Infer({ model() {
      var state = sample(belief);
      var predictedNextState = transition(state, action);
      var predictedObservation = observe(predictedNextState);
      condition(_.isEqual(predictedObservation, observation));
      return predictedNextState;
    }});
  };

  var act = dp.cache(
    function(belief) {
      return Infer({ model() {
        var action = uniformDraw(actions);
        var eu = expectedUtility(belief, action);
        factor(1000 * eu);
        return action;
      }});
    });

  var expectedUtility = dp.cache(
    function(belief, action) {
      return expectation(
        Infer({ model() {
          var state = sample(belief);
          var u = utility(state, action);
          if (state.terminateAfterAction) {
            return u;
          } else {
            var nextState = transition(state, action);
            var nextObservation = observe(nextState);
            var nextBelief = updateBelief(belief, nextObservation, action);
            var nextAction = sample(act(nextBelief));
            return u + expectedUtility(nextBelief, nextAction);
          }
        }}));
    });

  return { params, act, expectedUtility, updateBelief };
};

var simulate = function(startState, agent) {
  var act = agent.act;
  var updateBelief = agent.updateBelief;
  var priorBelief = agent.params.priorBelief;

  var sampleSequence = function(state, priorBelief, action) {
    var observation = observe(state);
    var belief = ((action === 'noAction') ? priorBelief : 
                  updateBelief(priorBelief, observation, action));
    var action = sample(act(belief));
    var output = [[state, action]];

    if (state.terminateAfterAction){
      return output;
    } else {
      var nextState = transition(state, action);
      return output.concat(sampleSequence(nextState, belief, action));
    }
  };
  // Start with agent's prior and a special "null" action
  return sampleSequence(startState, priorBelief, 'noAction');
};



//-----------
// Construct the agent

var prizeToUtility = {
  chocolate: 1, 
  nothing: 0, 
  champagne: 1.5, 
  start: 0
};

var utility = function(state, action) {
  return prizeToUtility[state.prize];
};


// Define true startState (including true *armToPrize*) and
// alternate possibility for startState (see Figure 2)

var numberTrials = 1;
var startState = { 
  prize: 'start',
  timeLeft: numberTrials + 1, 
  terminateAfterAction: false,
  armToPrize: { 0: 'chocolate', 1: 'champagne' }
};

var alternateStartState = extend(startState, {
  armToPrize: { 0: 'chocolate', 1: 'nothing' }
});

// Agent's prior
var priorBelief = Categorical({ 
  ps: [.5, .5], 
  vs: [startState, alternateStartState]
});


var params = { utility: utility, priorBelief: priorBelief };
var agent = makeAgent(params);
var trajectory = simulate(startState, agent);

print('Number of trials: ' + numberTrials);
print('Arms pulled: ' +  map(second, trajectory));
~~~~

You can change the agent's behavior by varying `numberTrials`, `armToPrize` in `startState` or the agent's prior. Note that the agent's final arm pull is random because the agent only gets utility when *leaving* a state.


### Bandits with stochastic observations

The Bandit problem above is especially simple because pulling an arm *deterministically* results in a prize (which the agent directly observes). So there is a fixed, finite number of beliefs about the `armToPrize` mapping that the agent can have and it depends on the number of arms but not on the number of trials.

The next codebox considers the stochastic version of Bandits. Each arm has a distribution on outcomes and the agent is uncertain about the distribution. The arms yield numerical prizes rather than prizes like chocolate -- so this is a standard Bandit problem from ML and Control Theory refp:kaelbling1996reinforcement. For this standard Bandit problem, the number of possible beliefs for the agent grows with the number of trials. <a id="complexity"></a> <!--TODO_daniel be more precise here and ideally cite the relevant page of the papers].-->

We consider an especially simple Bandit problem, where the agent already knows the reward for `Arm0` and only considers two possible distributions on reward for `Arm1`. This is depicted in Figure 3.

<img src="/assets/img/3c-stochastic-bandit.png" alt="diagram" style="width: 600px;"/>

>**Figure 3:** Structure of Bandit problem where `Arm1` is stochastic. 
<br>

For the following codebox, we use library functions for the environment (`makeBanditPOMDP`) and for simulating the agent (`simulatePOMDP`):

~~~~
///fold: displayTrajectory

// Takes a trajectory containing states and actions and returns one containing
// locs and actions, getting rid of 'start' and the final meaningless action.
var displayTrajectory = function(trajectory) {
  var getPrizeAction = function(stateAction) {
    var state = stateAction[0];
    var action = stateAction[1];
    return [state.manifestState.loc, action];
  };

  var prizesActions = map(getPrizeAction, trajectory);
  var flatPrizesActions = _.flatten(prizesActions);
  var actionsPrizes = flatPrizesActions.slice(1, flatPrizesActions.length - 1);

  var printOut = function(n) {
    print('\n Arm: ' + actionsPrizes[2*n] + ' -- Prize: '
          + actionsPrizes[2*n + 1]);
  };
  return map(printOut, _.range((actionsPrizes.length)*0.5));
};
///


// 1. Construct Bandit POMDP

// Possible distributions on rewards for Arm1
var vs = [0, 1];
var probably1Dist = Categorical({ vs, ps: [0.2, 0.8] });
var probably0Dist = Categorical({ vs, ps: [0.8, 0.2] });

var options = {
  numberOfArms: 2,
  armToPrizeDist: {
    0: Delta({ v: 0.7 }), 
    1: probably1Dist  // Note that arm 1 is better in EV
  },  
  numberOfTrials: 11,
  numericalPrizes: true
};

var bandit = makeBanditPOMDP(options);
var startState = bandit.startState;
var world = bandit.world;


// 2. Construct POMDP agent

var priorBelief = Infer({ model() {
  var alternateArmToPrizeDist =  {
    0: Delta({ v: 0.7 }), 
    1: probably0Dist
  };
  var armToPrizeDist = uniformDraw([options.armToPrizeDist,
                                    alternateArmToPrizeDist]);
  return extend(startState, { latentState: armToPrizeDist });
}});

var utility = function(state, action) {
  var prize = state.manifestState.loc;
  return prize === 'start' ? 0 : prize;
};

var params = { 
  priorBelief, 
  utility,
  alpha: 1000  
};

var agent = makePOMDPAgent(params, bandit.world);


// 3. Simulate agent and return state-action pairs

var trajectory = simulatePOMDP(startState, world, agent, 'stateAction');
displayTrajectory(trajectory);
~~~~

Solving Bandit problems optimally quickly becomes intractable without special optimizations. The codebox below shows how runtime scales as a function of the number of trials. (This takes approximately 20 seconds to run.)

<!-- bandit_scaling_number_of_trials -->
~~~~
///fold: Construct world and agent priorBelief as above

var vs = [0, 1];
var probably1Dist = Categorical({ vs, ps: [0.2, 0.8] });
var probably0Dist = Categorical({ vs, ps: [0.8, 0.2] });

var trueArmToPrizeDist = {
  0: Delta({ v: 0.7 }), 
  1: probably1Dist
};

var alternateArmToPrizeDist = extend(trueArmToPrizeDist, { 1: probably0Dist });


var makeBanditWithNumberOfTrials = function(numberOfTrials) {
  return makeBanditPOMDP({
    numberOfTrials,
	numberOfArms: 2,
	armToPrizeDist: trueArmToPrizeDist,
	numericalPrizes: true
  });
};

var getPriorBelief = function(numberOfTrials){
  return Infer({ model() {
    var armToPrizeDist = uniformDraw([trueArmToPrizeDist,
                                      alternateArmToPrizeDist]);
    return makeBanditStartState(numberOfTrials, armToPrizeDist);
  }})
};

var baseParams = { alpha: 1000 };
///

// Simulate agent for a given number of Bandit trials
var getRuntime = function(numberOfTrials) {
  var bandit = makeBanditWithNumberOfTrials(numberOfTrials);
  var world = bandit.world;
  var startState = bandit.startState;
  var priorBelief = getPriorBelief(numberOfTrials)
  var params = extend(baseParams, { priorBelief });
  var agent = makeBanditAgent(params, bandit, 'belief');

  var f = function() {
    return simulatePOMDP(startState, world, agent, 'stateAction');
  };
  
  return timeit(f).runtimeInMilliseconds.toPrecision(3) * 0.001;
};

// Runtime as a function of number of trials
var numberOfTrialsList = _.range(15).slice(2);
var runtimes = map(getRuntime, numberOfTrialsList);
viz.line(numberOfTrialsList, runtimes);
~~~~


Scaling is much worse in the number of arms. The following may take over a minute to run:


<!-- bandit_scaling_number_of_arms -->
~~~~
///fold:

var vs = [0, 1];
var probably1Dist = Categorical({ vs, ps: [0.2, 0.8] });
var probably0Dist = Categorical({ vs, ps: [0.8, 0.2] });

var makeArmToPrizeDist = function(numberOfArms) {
  return map(function(x) { return probably1Dist; }, _.range(numberOfArms));
};

var armToPrizeDistSampler = function(numberOfArms) {
  return map(function(x) { return uniformDraw([probably0Dist,
                                               probably1Dist]); },
             _.range(numberOfArms));
};

var getPriorBelief = function(numberOfTrials, numberOfArms) {
  return Infer({ model() {
    var armToPrizeDist = armToPrizeDistSampler(numberOfArms);
    return makeBanditStartState(numberOfTrials, armToPrizeDist);
  }});
};

var baseParams = {alpha: 1000};
///

var getRuntime = function(numberOfArms) {
  var armToPrizeDist = makeArmToPrizeDist(numberOfArms);
  var options = {
    numberOfTrials: 5,
	armToPrizeDist,
	numberOfArms,
	numericalPrizes: true
  };
  var numberOfTrials = options.numberOfTrials;
  var bandit = makeBanditPOMDP(options);
  var world = bandit.world;
  var startState = bandit.startState;
  var priorBelief = getPriorBelief(numberOfTrials, numberOfArms);
  var params = extend(baseParams, { priorBelief });
  var agent = makeBanditAgent(params, bandit, 'belief');

  var f = function() {
    return simulatePOMDP(startState, world, agent, 'stateAction');
  };

  return timeit(f).runtimeInMilliseconds.toPrecision(3) * 0.001;
};

// Runtime as a function of number of arms
var numberOfArmsList = [1, 2, 3];
var runtimes = map(getRuntime, numberOfArmsList);
viz.line(numberOfArmsList, runtimes);
~~~~


### Gridworld with observations

A person looking for a place to eat will not be *fully* informed about all local restaurants. This section extends the Restaurant Choice problem to represent an agent with uncertainty about which restaurants are open. The agent *observes* whether a restaurant is open by moving to one of the grid locations adjacent to the restaurant. If the restaurant is open, the agent can enter and receive utility. 

<!-- Removed to slim down the text
In this POMDP version of Restaurant Choice, a rational agent can exhibit behavior that never occurs in the MDP version:

1. The agent thinks Donut South is closed and Donut North is open, and so goes to the further away Donut North (see next codebox). 

2. The agent goes to Noodle, see that it's closed and so takes the loop round to Veg. This route that doesn't make sense if Noodle is known to be closed (see second codebox). 
-->

The POMDP version of Restaurant Choice is built from the MDP version. States now have the form:

>`{manifestState: { ... }, latentState: { ... }}`

The `manifestState` contains the features of the world that the agent always observes directly (and so always knows). This includes the remaining time and the agent's location in the grid. The `latentState` contains features that are only observable in certain states. In our examples, `latentState` specifies whether each restaurant is open or closed.

The transition function for the POMDP case is essentially the same as in the MDP case. The main difference is that if a restaurant is closed the agent cannot transition to its location. The observation function allows the agent to observe whether a restaurant is open or closed only if the agent is adjacent to the restaurant.

The next two codeboxes use the same POMDP, where all restaurants are open but for Noodle. The first agent prefers the Donut Store and believes (falsely) that Donut South is likely closed. The second agent prefers Noodle and believes (falsely) that Noodle is likely open.

<!-- agent_thinks_donut_south_closed -->
~~~~
///fold:
var getPriorBelief = function(startManifestState, latentStateSampler){
  return Infer({ model() {
    return {
      manifestState: startManifestState, 
      latentState: latentStateSampler()};
  }});
};

var ___ = ' '; 
var DN = { name : 'Donut N' };
var DS = { name : 'Donut S' };
var V = { name : 'Veg' };
var N = { name : 'Noodle' };

var grid = [
  ['#', '#', '#', '#',  V , '#'],
  ['#', '#', '#', ___, ___, ___],  
  ['#', '#', DN , ___, '#', ___],
  ['#', '#', '#', ___, '#', ___],
  ['#', '#', '#', ___, ___, ___],
  ['#', '#', '#', ___, '#',  N ],
  [___, ___, ___, ___, '#', '#'],
  [DS , '#', '#', ___, '#', '#']
];

var pomdp = makeGridWorldPOMDP({
  grid,
  noReverse: true,
  maxTimeAtRestaurant: 2,
  start: [3, 1],
  totalTime: 11
});
///

var utilityTable = {
  'Donut N': 5,
  'Donut S': 5,
  'Veg': 1,
  'Noodle': 1,
  'timeCost': -0.1
};
var utility = function(state, action) {
  var feature = pomdp.feature;
  var name = feature(state.manifestState).name;
  if (name) {
    return utilityTable[name];
  } else {
    return utilityTable.timeCost;
  }
};

var latent = {
  'Donut N': true,
  'Donut S': true,
  'Veg': true,
  'Noodle': false
};
var alternativeLatent = extend(latent, {
  'Donut S': false,
  'Noodle': true
});

var startState = {
  manifestState: { 
    loc: [3, 1],
    terminateAfterAction: false,
    timeLeft: 11
  },
  latentState: latent
};

var latentStateSampler = function() {
  return categorical([0.8, 0.2], [alternativeLatent, latent]);
};

var priorBelief = getPriorBelief(startState.manifestState, latentStateSampler);
var agent = makePOMDPAgent({ utility, priorBelief, alpha: 100 }, pomdp);
var trajectory = simulatePOMDP(startState, pomdp, agent, 'states');
var manifestStates = _.map(trajectory, _.property('manifestState'));

viz.gridworld(pomdp.MDPWorld, { trajectory: manifestStates });
~~~~

Here is the agent that prefers Noodle and falsely belives that it is open:

<!-- agent_thinks_noodle_open -->
~~~~
///fold: Same world, prior, start state, and latent state as previous codebox
var getPriorBelief = function(startManifestState, latentStateSampler){
  return Infer({ model() {
    return {
      manifestState: startManifestState, 
      latentState: latentStateSampler()
    };
  }});
};

var ___ = ' '; 
var DN = { name : 'Donut N' };
var DS = { name : 'Donut S' };
var V = { name : 'Veg' };
var N = { name : 'Noodle' };

var grid = [
  ['#', '#', '#', '#',  V , '#'],
  ['#', '#', '#', ___, ___, ___],  
  ['#', '#', DN , ___, '#', ___],
  ['#', '#', '#', ___, '#', ___],
  ['#', '#', '#', ___, ___, ___],
  ['#', '#', '#', ___, '#',  N ],
  [___, ___, ___, ___, '#', '#'],
  [DS , '#', '#', ___, '#', '#']
];

var pomdp = makeGridWorldPOMDP({
  grid,
  noReverse: true,
  maxTimeAtRestaurant: 2,
  start: [3, 1],
  totalTime: 11
});

var latent = {
  'Donut N': true,
  'Donut S': true,
  'Veg': true,
  'Noodle': false
};
var alternativeLatent = extend(latent, {
  'Donut S': false,
  'Noodle': true
});

var startState = {
  manifestState: { 
    loc: [3, 1],
    terminateAfterAction: false,
    timeLeft: 11
  },
  latentState: latent
};

var latentSampler = function() {
  return categorical([0.8, 0.2], [alternativeLatent, latent]);
};

var priorBelief = getPriorBelief(startState.manifestState, latentSampler);
///

var utilityTable = {
  'Donut N': 1,
  'Donut S': 1,
  'Veg': 3,
  'Noodle': 5,
  'timeCost': -0.1
};
var utility = function(state, action) {
  var feature = pomdp.feature;
  var name = feature(state.manifestState).name;
  if (name) {
    return utilityTable[name];
  } else {
    return utilityTable.timeCost;
  }
};
var agent = makePOMDPAgent({ utility, priorBelief, alpha: 100 }, pomdp);
var trajectory = simulatePOMDP(startState, pomdp, agent, 'states');
var manifestStates = _.map(trajectory, _.property('manifestState'));

viz.gridworld(pomdp.MDPWorld, { trajectory: manifestStates });
~~~~

In the next [chapter](/chapters/4-reasoning-about-agents.html), we will start modeling inferences *about* agents.

<!-- TODO
### Possible additions
- Doing belief update online vs belief doing a batch update every time. Latter is good if belief updates are rare and if we are doing approximate inference (otherwise the errors in approximations will compound in some way). Maintaining observations is also good if your ability to do good approximate inference changes over time. (Or least maintaining compressed observations or some kind of compressed summary statistic of the observation -- e.g. .jpg or mp3 form). This is related to UDT vs CDT and possibly to the episodic vs. declarative memory in human psychology. [Add a different *updateBelief* function to illustrate.]
-->


<br>

### Footnotes

