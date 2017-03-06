# CMPLXSYS530_ASS4

# Model Proposal for _[Project Name]_

Zachary Hajian-Forooshani

* Course ID: CMPLXSYS 530,
* Course Title: Computer Modeling of Complex Systems
* Term: Winter, 2017



&nbsp; 

### Goal 
*****
The goal is to model the interactions of an ant and its parasite in space. I am interested in the spatial patterns the ants create. 
&nbsp;  
### Justification
****
ABM is useful for this approach becuase the organisms I want to model have relatively simple behavior that I expect will lead to emergent phenomenon. 

&nbsp; 
### Main Micro-level Processes and Macro-level Dynamics of Interest
****

For the ants I will be looking at the rate in which ants are creating new nests and the cost of creating them. For the parasites I will be looking at the cost of their reproduction and the rate at which they kill ant nests.

&nbsp; 


## Model Outline
****
&nbsp; 
### 1) Environment

I will make the world infinte becuase I dont want to have that big effect from the edges of the lattice.
It will be 2D.
I plan to have a few different versions of the model. One with an environment that is all homogenous and another where the environment is more heterogenous. In the heterogenous model I will have certain cells that parasites will not be able to exists for prolonged peroids of time. For now I have the patches as owning if they are a nesting site for ants. 


&nbsp; 

### 2) Agents

# Ants
I currently have the ants set up as owning a population. This population grows with ticks but can be lowered by the presence of parasites. When the population is zero then the nest will die and disappear.
Ant nets will bud in the moore neighborhood given a probability and it will cost the budding nest some amount of its population.

# Parasites (phorids)
I have the parasites as owning population. The population increases when they are in the presence of ants but decreases as they are searching for ants.
Parasites will bud once they reach a certain population level and it will cost  a certain about to reproduce.
Parasites move on the lattice randomly until they find an ant. 


# Ants
```
  ask ants [
    grow-nest-pop
    bud-nest
    nest-death
    stochastic-death
  ]



to stochastic-death
  if random-float 100 < prob-nest-death [die set nesting-site? false]
end
to grow-nest-pop ;; ant procedure
   set population-nest population-nest + 1
end

to bud-nest
 ; if population-nest > 100 [
   if random-float 100 < prob-of-budding [
     ask one-of neighbors [ if not any? ants-here [sprout-ants 1 set nesting-site? true] ]
        set population-nest (population-nest / bud-cost-ant) ;; lose half of population in budding process
      ]
;  ]
end

;to bud-nest ;; got this one online dont really understand it
;  if population-nest > 100 [ if random-float 100 < 5 [
;    let parent self
;      ask neighbors with [ not any? ants-here] [
;        let destination self
;        ask parent [ hatch 1 [ move-to destination] ]
;        ]
;      ]
;    ]
;end

to nest-death ;; ant procedrue
  if population-nest < 0 [die set nesting-site? false]
end
```

# Parasites
```
  ask phorids [
   if nesting-site? = false [ wiggle fd 1] ; if not at an ant site wiggle phorid!
   if nesting-site? = true [ attack-ants ] ; if phorids on ant patch then attack ants
   leave-ant-nest
   phorids-age
   phorid-babies
    ]
    
                            ;;;;;; Phorid Procedures ;;;;;

to phorid-babies
  if population-phorid > 100 [
  set population-phorid (population-phorid / repo-cost-phorid)
  hatch 1 [ rt random-float 360 fd 1]
]
end
to wiggle  ;; phorid procedure
  if remainder ticks 10 = 0 [
  rt random 40
  lt random 40
  if not can-move? 1 [ rt 180 ]
  ]
end

to leave-ant-nest ;; phorid procedure
  if not any? ants-here [fd 1 wiggle]
end

to attack-ants ;; phorid procedure
  let azteca one-of ants-here
  if azteca != nobody
    [ask azteca [set population-nest population-nest - phorids-reduce-azteca ]
      set population-phorid population-phorid + phorids-gain-from-azteca]
end

to phorids-age ;; phorid procedure
  if remainder (ticks + 1) 10 = 0 [ set population-phorid population-phorid - 5 ]
  if population-phorid < 0 [ die ]
end
```



&nbsp; 

### 3) Action and Interaction 
 
**_Interaction Topology_**

Ants bud in their moore neighborhood and parasites fly around randomly until they encounter ants.
 
**_Action Sequence_**

_What does an agent, cell, etc. do on a given turn? Provide a step-by-step description of what happens on a given turn for each part of your model_


Sequences are given below
```
  ask ants [
    grow-nest-pop
    bud-nest
    nest-death
    stochastic-death
  ]


  ask phorids [
   if nesting-site? = false [ wiggle fd 1] ; if not at an ant site wiggle phorid!
   if nesting-site? = true [ attack-ants ] ; if phorids on ant patch then attack ants
   leave-ant-nest
   phorids-age
   phorid-babies
    ]
```    

&nbsp; 
### 4) Model Parameters and Initialization

_Describe and list any global parameters you will be applying in your model._

_Describe how your model will be initialized_

_Provide a high level, step-by-step description of your schedule during each "tick" of the model_

The model is set up by randomly seeding ant nesting sites (I color the patches to keep track of what is an ant patch). I then randomly set a number of phorids on the lattice.

To initialize the model I have the probability of ant nets being born on lattice and the number of inital parasites. 

```
to setup
  clear-all

    ask patches [
      ifelse random 100 < nests [
      set pcolor yellow
      set nesting-site? TRUE
      sprout-ants 1 [set color red]
       ]

      [ set pcolor green
        set nesting-site? FALSE]

      set cluster nobody
    ]

  set-default-shape phorids "airplane"
      create-phorids phorid-population [
      set color black
      setxy random-xcor random-ycor
    ]

reset-ticks
  end

```
Here is the go procedure for the model (gives description of what happens in a tick)

```to go

  ask phorids [
   if nesting-site? = false [ wiggle fd 1] ; if not at an ant site wiggle phorid!
   if nesting-site? = true [ attack-ants ] ; if phorids on ant patch then attack ants
   leave-ant-nest
   phorids-age
   phorid-babies
    ]

  ask ants [
    grow-nest-pop
    bud-nest
    nest-death
    stochastic-death
  ]

    ask patches [
    ifelse nesting-site? = true [set pcolor yellow] [set pcolor green]
    if not any? ants-here [set nesting-site? false] ;; if no ants not nesting site
    ]
tick
  end
  ```



&nbsp; 

### 5) Assessment and Outcome Measures

_What quantitative metrics and/or qualitative features will you use to assess your model outcomes?_

Right now I am looking at the frequency distribution of cluster sizes of ant nests. My clusters are made of any nests that are connected to others by their moore neighborhoods.

&nbsp; 

### 6) Parameter Sweep

_What parameters are you most interested in sweeping through? What value ranges do you expect to look at for your analysis?_


I will be sweeping through the cost to budding for ants and the cost of reproduction of parasites. I will also look at how much the parasites gain from ants. The ranges will be between 0 and 1 becuase they will be proportions.



