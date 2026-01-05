# Patch 7.4 Simulation Results (BIS)

This repo contains generated results for target dummy rotations. We used 12:30 rotations in a full party scenario 
to generated expected values for every hit and derive a true expected dps for each comp. This value is not impacted 
by crit-rng (like logs are), so they represent a more precise measurement of the rotation.

Additionally, for each comp we ran 10000 simulations to build up statistical metrics like variance to help visualize 
the lower and upper bounds of each comp. These 10000 simulations are also used to handle jobs with random rotations;
we have solvers for the MNK/BRD/DNC random elements (RDM we treated as static), which should normalize the effects of 
rng on these jobs as well. 

## Contents and Methodology
We experimented on simulating jobs in 8-player comp situations to assess the strength of particular jobs in a target 
dummy situation against their peers. To limit the volume of things we would check, we started with three baseline comps:
* `DRK-GNB-AST-SCH-MNK-NIN-BRD-PCT` (Cards on NIN, PCT)
  * This is intended to be the "meta" raid-buff comp.
* `DRK-GNB-WHM-SCH-MNK-NIN-BRD-PCT`
  * This is a raid buff comp without cards.
* `PLD-WAR-SGE-WHM-SAM-VPR-MCH-BLM`
  * This comp has tests for performance under a low-to-no raid buff situation.

From these comps, we focus on specific roles by testing all **unique permutations without duplicates**. Each comp is 
labeled with the 1-2 jobs it is specifically testing. All permutations are simulated against each other and plotted in 
the same graph for easy side-by-side comparison.

* `(Variable Tanks)-AST-SCH-MNK-NIN-BRD-PCT` [results](outputs/TankBuffCompCompare)
* `(Variable Tanks)-SGE-WHM-SAM-VPR-MCH-BLM` [results](outputs/TankNoBuffCompCompare)
* `DRK-GNB-(Variable Healers)-MNK-NIN-BRD-PCT` [results](outputs/HealerBuffCompCompare)
* `PLD-WAR-(Variable Healers)-SAM-VPR-MCH-BLM` [results](outputs/HealerNoBuffCompCompare)
* `DRK-GNB-WHM-SCH-(Variable Melees)-BRD-PCT` [results](outputs/MeleeBuffCompCompare)
  * Mostly laziness on not using AST here, I didn't want to configure cards.
* `PLD-WAR-SGE-WHM-(Variable Melees)-MCH-BLM` [results](outputs/MeleeNoBuffCompCompare)
* `DRK-GNB-AST-SCH-SAM-NIN-(Variable Ranged)-PCT` [results](outputs/RangedBuffCompCompare)
* `PLD-WAR-SGE-WHM-SAM-VPR-(Variable Ranged)-BLM` [results](outputs/RangedNoBuffCompCompare)
* `DRK-GNB-WHM-SCH-MNK-NIN-BRD-(Variable Caster)` [results](outputs/CasterBuffCompCompare)
* `DRK-GNB-AST-SCH-MNK-NIN-BRD-(Variable Caster)` [results](outputs/CasterAstBuffCompCompare)
  * Ranged cards were used on the caster.
* `PLD-WAR-SGE-WHM-SAM-VPR-MCH-(Variable Caster)` [results](outputs/CasterNoBuffCompCompare)

### Rotations
All rotations are hand-sheeted 12 minute and 30s rotations, using Pots in the opener, 6 minute, and 12 minute windows.
All rotations attempted to align with a ~6s buff opener. As mentioned before, MNK/BRD/DNC have some random elements
in the simulation:
* MNK: Chakra is randomly simulated based on game values. TFC's are used whenever possible with no holding.
* BRD
  * The sim uses Heartbreak Shot and Pitch Perfect in open weaves based on randomized procs. 
    * Raging windows are used to determine when to hold HB shot: The sim will start holding them if it would not lose a usage before the next window.
  * Burst Shot and Refulgent Arrow are used interchangeably based on procs.
  * Apex Arrow has it's damage simulated based on simulated gauge procs. Blast Arrow is skipped for a Burst/Refulgent if requirements are not meant.
    * In practice, a player would work around this. We allow the sim to look at the sum of gauge generated instead of hard resets to avoid having to move Apex from intended usage times.
  * Dot refreshes are not moved: if these leads to an overcap for Refulgent Arrow, the sim makes an exception and allows 2 usages back to back.
    * This is to simplify how a real player would refresh early to avoid this situation.
  * Army's Paeon GCD effects are ignored: we sheeted the gcd increase manually and consistently.
  * All other abilities are not moved or overwritten by the solver.
* DNC
  * The sim replaces almost all gcds and ogcds following the generally recommended priority system.
  * Dance timings, Standard refreshes, and Flourish/Devilment usages are never moved.

MNK and DNC also scan other rotations in the comp to simulate the chakra/esprit generation from the party.

We acknowledge there might be user error in the hand-sheeted rotations and rng-solvers. We invite people to call out 
blatant errors in rotations if the mistake is easily verifiable. If you have a disagreement with the specific optimizations
in the sheet, please provide **completed, 12:30 rotations via xiv-in-the-shell export with gearsets** for us 
to review and resimulate. If the outcome is significantly different (100s of dps), we would post revised results.

### Important Simulations
There are many generated outputs and plenty of them might not be very insightful, so I'll outline the most important ones.
* `rotations`: These are generated damage outputs from the actual rotations and are the base data for the visualizations here.
  * These are included specifically to help validation of this work over time.
  * This is also where you will notice the simulated probabilities on random events for MNK/BRD/DNC.
* `comp_dps`: This tracks simulated percentiles for all the comps in terms of DPS @ `12:27.500`
* `comp_dps_over_time`: Tracks expected dps for every damage application, which can be used to see specific spikes in damage by comp.

### Files and Outputs
* [configs](configs) contains the original csvs and 8-player comp configurations for generating these outputs.
* [outputs](outputs) contains simulated results for all of these comps in `csv` and `html` formats.
  * https://htmlpreview.github.io/? can be used to view `html` files without any other tools by appending the full url.
  * Example: https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterAstBuffCompCompare/comp_dps_over_time/comp_dps_over_time.html


## Outcomes

TODO

## Disclaimers

### Mistakes

To reiterate:
> We acknowledge there might be user error in the hand-sheeted rotations and rng-solvers. We invite people to call out 
blatant errors in rotations if the mistake is easily verifiable. If you have a disagreement with the specific optimizations
in the sheet, please provide **completed, 12:30 rotations via xiv-in-the-shell export with gearsets** for us 
to review and resimulate. If the outcome is significantly different (100s of dps), we would post revised results.

Additionally, we accept there might be bugs in the underlying sim code and will be on constant lookout for that. 
We do verify our expected damage against gearset-annotated logs to make sure we accurately simulate the correct ranges.
We also verified against the generated rotations in xivgear.app for line-by-line expected value 
(we consistently were within 1-2 damage per line). All that said, I personally am not an expert on every job and there
are certainly... decisions... you might find in some sheets. I also coded the solvers so there's definitely the possibility
of mistakes.

### You should not use the results of these tools to discriminate against any jobs in a public setting
These results come from an idealistic scenario and are biased towards jobs that play into a raid-buff meta. 
Some of the jobs that are under-performing in these simulations favor better in more real-world situations since they
are less dependent on their team and can have more consistent output. The goal of this assessment is to provide a 
new perspective on job performance that is not feasible to see via submitted logs.

## Credits
* Ama for providing an open-source simulation tool in python, which was used extensively for this project
  * [Pip](https://pypi.org/project/ama-xiv-combat-sim/#description)
  * [xivraider](https://xivraider.com/)
* Shanzhe and the supporting team for [xivintheshell](https://xivintheshell.com/) for a public rotation tool.
* [xivgear.app](https://xivgear.app/) for their gearset support and damage validation.
* [Caro](https://www.youtube.com/@CaroKannffxiv) and Parryprog for writing the vast majority of the rotations and reviewing work.
  * They will be attempting speed-kills in 7.4, please support them!
* Sleepocat for being the original testers for many of these simulations and visualizations.


### P.S
If you are interested in speeds or spreadsheeted theorycraft, please reach out to me on discord (apollo.van.waddleburg). 
We are slowly rolling out many of the tools we used to generate this report to interested parties and would love 
to develop a community around damage simulation and speed kills.