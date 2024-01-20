*This post is a distillation of Fixing the Good Regulator Theorem by John Wentworth.*

Imagine you’re a crane operator on a construction site. You need to work with people on the ground to move heavy stuff around and not hurt anybody in the process. Obviously the crane driver needs outside information to act: the view out the window, messages coming over the radio. You also need to actually use this information to do different things depending on your environment. I can confidently say that somewhere in the crane driver's mind, there's a little mental model of the construction site containing only the information needed for the job. But how can I say this so confidently? Why can't some crane drivers ignore external information, or just act on information without constructing a mental model? Well, you would be FIRED if you didn't even look out the window. And as for the second question, we will see that under some assumptions about the interconnectedness and bandwidth of the brain, there must be a mental model.

This intuition should apply to any AI system: it needs to have some internal model of the environment in order to do anything coherently. Ideally, I'd love to say that any AI’s world model must be a causal model - but that is above my pay grade for now. The Good Regulator Theorem is a baby step in this direction. The theorem states that a good regulator of a system must be a model of that system.

In this post I will explain 3 forms of the Good Regulator Theorem as it currently stands. Along the way, we will build up the result to handle incomplete information using probabilities, and we will make the theorem mechanistic using some assumptions about minds.

# The Original Good Regulator Theorem


This theorem is by Conant and Ashby (link)

Keep in mind the image of a crane operator working at a construction site. This theorem attempts to formalise this setting as such:

*   Random variable S which represents the state of the system at a moment in time. Imagine freezing the construction side, and writing down the positions of all workers, the positions and velocities of all building materials, the current temperature, etc...
*   Random variable R which represents the action taken by the regulator at a particular moment in time. The value of R is drawn from a set of all possible actions the crane operator can take: for example, we could have $R = (\theta, r, h)$ which represents the angle, radius, and height we want to move the crane arm to. 
*   Random variable Z which represents the outcome after applying the regulator’s action R to the system state S. For example, it could be a percentage completion score of the project after moving the crane. Or it could be the resulting state a second after the crane driver takes their action. For this theorem’s setup, we assume Z is a deterministic function of S and R. That is, once we fix S and R, there is no random noise which can affect the outcome Z. (I know this is unrealistic! But we will fix this soon).

Our goal is to choose the probability distribution over the agent's action R conditional on the system state S. We call this a policy, $\pi$.

Think of these values using this computational graph. S is randomly sampled, R is sampled conditional on S, then Z is computed.

![](https://39669.cdn.cke-cs.com/rQvD3VnunXZu34m86e5f/images/36f3c83a66aacc55474d318c88ee0ad3a662c7e9fbdf4313.png)

In this setting we can prove the following theorem:

### **Theorem (Good Regulator, Original)**

> Let $\pi^*$ be a policy, where $\pi^*(r \ | \ s)$ gives the probability of selecting some regulator output $r$ given the system is in state $s$. Suppose that $\pi^*$ minimises the entropy of $Z$, and doesn't have any unnecessary complexity. Then, $\pi^*$ is a deterministic policy. That is, for any system state $s$, $\pi^*$ returns a corresponding $r$ with probability 1.

I’ll give a proof soon. But first, let’s interpret this result. It’s saying that the regulator’s output must be completely determined by the system S. Or: the only relevant information is the information in the system - nothing more, nothing less.

Here, an optimal regulator is one which makes the output $Z$ very regular (i.e. low in entropy).

### **Proof**

First we assume the following things about $\pi^*$:

*   *Optimal:* For all policies $\pi, \ H_{\pi^*}(Z) \leq H_{\pi}(Z)$.
*   *Simple:* We assume that $\pi^*$ has no unnecessary complexity. Fix a system state $s$. Let $r, r'$ be regulator outputs where $r \neq r'$. Suppose $f(r, s) = f(r', s)$. ($f$ is just a deterministic function giving the outcome $Z$). Then $r$ has probability 0 or $r'$ has probability 0. 

We want to show that, for each system state $s$, there exists some regulator output $r$ with probability 1. First we will show that for each $s$, there is only one corresponding outcome $Z$ under the policy $\pi^*$. Then we will show that for that outcome $z$, there is only one $r$ with nonzero probability.

**Lemma:** Fix a system state $s$. Let $r, r'$ be regulator outputs with nonzero probability. Then $f(r, s) = f(r', s)$. ($f$ is just a deterministic function which gives the outcome $Z$)

**Proof of Lemma:** First we introduce some notation. Let $\pi^*(r | s) = \alpha$, and $\pi^* (r' | s) = \beta$. We know $0 < \alpha < 1$, and likewise for $\beta$.

Suppose for contradiction that $f(r, s) \neq f(r', s)$. Then define a new policy $\bar{\pi}$ where:

$$\begin{aligned}   
    \bar{\pi}(r \ | \ s) &= \alpha + \beta \\ 
    \bar{\pi}( r' \ | \  s) &= 0
\end{aligned}$$

and all other probabilities are exactly the same as $\pi^*$. Then we show that $\bar{\pi}$ can achieve a lower entropy than $\pi^*$.

That is, we show that $H_{\pi^*}(Z) - H_{\bar{\pi}}(Z) > 0$:

My earlier working is in writersblock2.txt

Now let's analyse the first term. We know that $\frac{\alpha + \beta}{\alpha} > 1$. So, $\log \left(\frac{\alpha + \beta}{\alpha}\right) > 0$. Since $\alpha > 0$, we know that the whole term is greater than 0.

An analagous argument holds for the other term. So, we get $H_{\pi^*}(Z) - H_{\bar{\pi}}(Z) > 0$. This is a contradiction because $\pi^*$ was supposed to be entropy-minimising. So, $f(r, s) = f(r', s)$    $\square$

With this lemma nailed down, we can complete the proof. By the lemma, we know that each system state $s$ corresponds to only one outcome $Z$. Then by the simplicity assumption, we know that only one regulator output $r$ can correspond to the outcome $Z$ (for a fixed system state $s$). So, given some system state $s$, an optimal and simple regulator policy must only give one regulator output $r$ corresponding to $s$.

# Good Regulators with Incomplete Information

This theorem is by John Wentworth (link)

One issue with Theorem #1 is that it doesn't cope with uncertainty. If we are looking to apply the theorem in the real world, we need to make sure it holds when the agent is uncertain of their knowledge.

So in this section, we will go part of the way towards describing how the agent's mind works internally. We will attempt to show that agents in uncertain environments must use probabilities to be successful. 

Here's the setup: The crane driver looks out the window and observes the sate of the construction site ($X$). Then they wait 5 seconds without looking outside. In that time, people and objects have moved slightly to a new configuration ($S$). Then, using only their knowledge from 5 seconds ago, the crane driver chooses what to do ($R$). Here's a picture:

![](https://39669.cdn.cke-cs.com/rQvD3VnunXZu34m86e5f/images/e0fd376241cc08413b74e596a070de888d7f461dbbe00e6f.png)

More formally: First, we sample the “incomplete information” $X$. Then we sample $S$ from some( conditional distribution given $X$. We want to find a policy $\pi^*$, where $\pi^*(r\ |\ x)$ represents the probability of selecting regulator output $R$ conditional on the incomplete information $X$. 

One "design choice" John made was to redefine optimal to mean “maximising expected utility”. This is more familiar here on LessWrong, but it is a simpler criterion than minimising entropy. This proof (and the third proof) may not hold for entropy-minimising regulators.

One note about [mechanism vs behaviour](https://www.lesswrong.com/posts/gvK5QWRLk3H8iqcNy/gears-vs-behavior): this theorem does not say that the optimal policy must internally reconstruct the conditional distribution $g(S|X)$. When I say "internally reconstruct", I mean it in the sense of mechanistic interpretability. As in, imagine that some algorithm is running to compute the action $R$ conditional on $X$. Then this version of the Good Regulator Theorem only tells us that the algorithm *could* be constructing a model. The theorem tells us that for any such algorithm, it is *possible* to produce another algorithm which has the same input-output behaviour, but does actually recreate the conditional distribution $g(S|X)$. 

(If that paragraph felt confusing to you, stick around to the 3rd version of the theorem. We will discuss mechanism vs behaviour more there.)

Setting the limitations aside, in this setting the good regulator theorem becomes: 

> **Theorem (Good Regulator, Incomplete Information):** Let $\pi^*$ be a policy, where $\pi^*(r\ |\ x)$ gives the probability of selecting regulator output $r$ given incomplete information $x$. 
>
> Also let $g: X \to \text{Dist}(S)$ be a function which gives conditional probability distributions on $S$. (Here, the notation $\text{Dist} (S)$ refers to the set of all probability distributions over the random variable $S$). That is,
>$$g(x) = P (S \ | \  X=x)$$
>We assume the following:
> * *Optimal:* $\pi^*$ optimises expected utility function, for some utility function $u: \mathcal Z \to \mathbb R$
> * *Simple*: For all $x_1, x_2$, if $g(x_1) = g(x_2)$ then $\pi^*(R | X=x_1) = \pi^*(R | X=x_2)$. Intuitively, if two pieces of incomplete information lead to the same distribution over $S$ (at least as far as we know), then we should act the same.
> 
> Then the theorem states that $\exists \ \varphi: \text{im} (g) \to \text{Dist} (R)$ such that $\varphi ( g (x) ) = \pi^*(R \ | \ X=x)$ for all possible incomplete information $x$. (Here, $\text{im} (g)$ refers to the image of all $x$ values under the function $g$). In other words, for any fixed $x$, the distribution over actions given by $\pi^*$ must correspond uniquely to the conditional distribution over $S$

Intuitively, this theorem means that whatever value your policy $\pi^*$ is computing, there is an equivalent way to compute it which involves first creating a “model” of the system $S$ (in this case a conditional distribution function $g$), then throwing away the incomplete information and deciding what to do solely using that model.

A quick note on conditional distributions: Some people might find my use of conditional distributions in this post confusing. There are multiple interpretations of a conditional distribution - it can be a one-variable function which, for some fixed $x$, takes as input any system state $s$ and returns the probability of $s$ conditional on $x$. Alternatively, it can be seen as a two-variable function which returns $\mathbb P (s | x)$ for any given $s, x$ combination. Both these interpretations are valid. More details are available in probability textbooks presumably, or I'm happy to discuss this in the comments. Now for the proof!

NOTE: I Think I just came up with an original thought! I think we can cheese the result from Part 1 to prove this theorem for minimum entropy as well! Just create a new diagram which matches part 1 by using the posterior distributions.... need some of the details to work out. But let's see.

Another note.... I think that choosing a utility function is an annoying feature of this theorem. For any regulator R, we can choose a utility  function u such that R is optimal.

Or, under a completely uniform utility function, all agents are optimal as they cannot be any better. Clearly if the utility function is 1 for all possible outputs, then the agent does not need to model the environment.

I think the amount of the world the agent needs to model depends on their utility function. and it depends on f which maps R, S to Z.

OOHHHH it looks like my proof here differs from John's because it is just wrong. John is not really saying anything different from the origianl proof; in fact it is very similar! He just says that the proof is more intutivie in an expected utility framework. 

Maybe if I feel like it I can rewrite this section as a new proof just cheesing the old result to squeeze out a bit more info on uncertain environments.

Also just in general I feel like my writing is too verbose at times.

**Proof:** I claim that $\varphi$ can be explicitly written as

$$\varphi(g(x)) = \underset{\pi \in \text{Dist}(R)}{\text{argmax}} \left[ \sum_s \sum_r u(f(r, s)) \cdot \pi(R=r) \cdot P (S=s \ | \ X=x) \right]$$

with some consistent tie-breaker for cases when the output of argmax is not unique. (Note that $f$ is just some deterministic function which gives the outcome $Z$)

Intuitively, this function just returns the best action f > 0or a given distribution over $x$ values. Now we need to prove that this map is well-defined, and results in an optimal and simple policy.

First note that the map $g$ is well-defined, because the operation of conditioning is well-defined for any two random variables.

Since $\pi^*$ is optimal, we know that $E_{\pi^*}(U(Z)) \geq E{\pi^*}(U(Z))$. So we know that, averaged over all $x$, $\pi^*$ is optimal. But in the explicit formula given for $\varphi$, we optimise $E(U)$ for only a single $x$ value. So we need the following lemma:

**Lemma:** Let $\pi$ be a policy, and let $x$ be some incomplete information. Then

$$\begin{aligned}  & \sum_s \sum_r u(f(r, s)) \cdot \pi(R=r \ | \ X=x) \cdot P (S=s \ | \ X=x)  \\\leq & \sum_s \sum_r u(f(r, s)) \cdot \pi^*(R=r \ | \ X=x) \cdot P (S=s \ | \ X=x)\end{aligned}$$

Essentially, $\pi^*$ must get the highest expected utility conditional on each possible x.

**Proof of Lemma:** I won't spell out the full proof. But briefly, we can prove this by contradiction by assuming that $\pi^*$ is not optimal for some $x$. Then we can create a new policy which performs strictly better than $\pi^*$ just by taking the optimal action given $x$, and keeping all other actions the same. This violates the assumption that $\pi^*$ is optimal. Another way of thinking about this is that for any $x_1 \neq x_2$, the di > 0stributions $(s, r) \ | \ x_1$ are independent of the distributions over $(s, r)\ |\ x_2$.     $\square$

Now, back to the candidate for $\varphi$:

$$\varphi(g(x)) = \underset{\pi \in \text{Dist}(R)}{\text{argmax}} \left[  \sum_s \sum_r u(f(r, s)) \cdot \pi(R=r) \cdot P (S=s \ | \ X=x)  \right]$$

Using this lemma, we know that the output of $\varphi$ is an optimal policy. But wait! As defined, $\varphi$ is not actually a well-defined function. For a given distribution over system states, there may be many different optimal policies. We need to have one input corresponding to only one output.

Thankfully, this requirement lines up exactly with the notion of "simplicity" originally used by Conant and Ashby. If $\pi^*$ does not have any unnecessary complexity, then any two $x$ values which result in the same distribution over system states should also correspond to the same policy distribution. In other words, $\pi^*$ should not give two different outputs in the same system state.

This condition is exactly the same as imposing some consistent tie-breaker on the function $\varphi$ given earlier. For example, $\varphi$ could have a look-up table for cases where the argmax is not unique.

So we have proven that there exists some deterministic function $\varphi$ which relies only on the output of $g$, and results in an optimal and simple policy.

# A More Mechanistic Good Regulator Theorem

If we impose more assumed structure on the regulator, we can demonstrate that the regulator must internally reconstruct the distribution over system states.

Imagine you're a crane driver again. First, you observe the location of the construction workers and materials ($X$). You use this information to construct a model ($M(X)$). Then, you look away from the windows to see your controls - these tell you the current position of the crane and its fuel level ($Y$). Using your model and the details about the crane, you need to decide the best action to take. See the below picture:

![](https://39669.cdn.cke-cs.com/rQvD3VnunXZu34m86e5f/images/2d1232e273cf038b226712e14c5d3577d935734ecdb5adaa.png)

Let's be more formal. The setup here is based on two random variables $X$ and $Y$ which represent the incomplete information about the environment. The model is first computed from $X$ using some deterministic function $M$. The regulator policy depends only on $Y$ and $M(X)$. Then the outcome $Z$ is computed using some deterministic function of $S$, $Y$, and $R$.

> **Theorem (Good Regulator, Mechanistic):** Suppose a regulator is a tuple $(M^* , \pi_R^*)$. Here, $M^*: \mathcal{X} \to \mathcal{M}$ is a deterministic function which gives a model of the environment. Also, $\pi_R^*$ is a policy over $R$ conditional on $M^*(X)$ and $Y$ \- that is, $\pi_R^*(r \ |\ M^*(x), y )$ is a function in one free variable which gives the probability of choosing any regulator output $r$ given some fixed $M^*(x),\ y$.
> 
> We make the following assumptions:
> 
> *   *Optimal:* For all regulators $(M, \pi_R)$,  
>     $E_{M, \pi_R}(u(Z)) \leq E_{M^*, \pi_R^*}(u(Z))$  
>     That is, the regulator optimises expected utility for some utility function u.
> *   *Simple:* For all optimal regulators $\left(\widetilde{M}, \widetilde{\pi}_R\right)$,  
>     $MI_{\widetilde{M}}\left[X; \widetilde{M}(X)\right] \leq MI_{M^*}\left[X ; M^*(X)\right]$  
>     That is, the model has minimal mutual information with $X$.
> *   *Rich Set of Games:* There are enough possible $Y$ values to distinguish between different distributions over system states. That is,  
>     $g(x_1) \neq g(x_2) \implies \exists \ y\  \text{s.t. }   \pi^*_R(R \ |\ M^*(x_1), y) \neq \pi^*_R(R \ |\ M^*(x_2), y)$ 
> 
> The result is that there exists an isomorphism $\varphi$ between $M(X)$ and the distribution over system states conditional on $X$. By isomorphism, I mean an invertible map which preserves the corresponding $X$ value.

**Set Theory Lemma:** First we prove an important set theory lemma for this theorem. Suppose we have sets $X, B, C$ and functions $f: X \to A$ and  $g: X \to B$ as in the picture below.

![](https://39669.cdn.cke-cs.com/rQvD3VnunXZu34m86e5f/images/366552e3739414e68212701ee2570f128750041e01a32e97.png)

Suppose that $f(x) = f(x') \implies g(x) = g(x')$. Then the lemma is that there exists some function $h: im(f) \to im(g)$ such that $h(f(x)) = g(x)$

In particular, I claim $h(f(x)) = g(x)$. It suffices to show that this map is well defined, and this immediately follows from the condition that $f(x) = f(x') \implies g(x) = g(x')$. Basically, for any two identical inputs, the outputs are also the same.     $\square$

**Proof of Theorem:** With this lemma out of the way, here are the type signatures for the functions involved in this proof:

*   $f: \mathcal{S} \times \mathcal{Y} \times \mathcal{R} \to \mathcal{Z}$ . This is the deterministic function which determines the outcome $Z$.
*   $u: \mathcal{Z} \to \mathbb R$ . This is the utility function over outcomes.
*   $g: \mathcal{X} \to \text{Dist}(S)$. This is the same as in the previous theorem. $g$ takes as input some incomplete information $x$, and returns the distribution over system states $s$ conditional on $x$.

The set theory lemma will be used to find the isomorphism $\varphi$. Note this corollary of the lemma: there exists an X-preserving bijection if $M^*(x_1) = M^*(x_2) \iff g(x_1) = g(x_2)$. So we just need to prove both directions of this implication.

First, we show that $g(x_1) = g(x_2) \implies M^*(x_1) = M^*(x_2)$.

Let $g(x_1) = g(x_2)$. Now suppose for contradiction that $M^*(x_1) \neq M^*(x_2)$.

Then we define a new function $\widetilde{M}$ which has less mutual information with $X$. Define $\widetilde{M}$ such that all outputs are the same as $M^*$, except:

$$\widetilde{M}(x_1) = \widetilde{M}(x_2)$$

Note that $\widetilde{M}$ is still optimal. To see why, note that $\pi^*_R$ must achieve the same expected utility for $M^*(x_1)$ and $M^*(x_2)$. This is because the system state distributions are the same, so we can achieve the same optimal expected utility in both cases. So when we switch our model function to $\widetilde{M}$, which only outputs one of these values, the result will still be that same optimal expected utility.

Then, the difference in mutual information under the two model functions is strictly positive:  
 

This is a contradiction because $M^*$ was supposed to be mutual-information-minimal among optimal model functions. So, $M^*(x_1) = M^*(x_2)$.

Next, we show that $M^*(x_1) = M^*(x_2) \implies g(x_1) = g(x_2)$.

Recall that we assumed there was a sufficiently rich set of games. The reason for this name is that, conceptually, $Y$ chooses which "game" the regulator needs to prepare to play. So with enough possible games, the regulator needs to keep all the information from $X$ which could be relevant to any future games. The contrapositive of that condition is: 

$$\forall \ y,\ \pi^*_R(R \ | \ M^*(x_1), y) = \pi^*_R(R \ | \ M^*(x_2), y) \implies g(x_1) = g(x_2)$$

In other words, the conditional state distribution is a function of the optimal policy over $R$. Now we need to show that the optimal policy over $R$ is a function of the model.

Note that we assumed $\pi^*_R$ was given at the start of the theorem. We do not need to assume $\pi^*_R$ is unique or simple, merely that it is optimal when combined with $M$. So, given our chosen $\pi^*_R$, of course it will give the same probability distribution if it takes the same inputs. That is,

$$\forall \ y, \ M^*(x_1) = M^*(x_2) \implies \pi^*R(R \ | \ M^*(x_1), y) = \pi^*R(R \ | \ M^*(x_2), y)$$

So the optimal policy over R is a function of the model. That completes the proof.