# Patch 7.4 Simulation Results (BIS)

This repo contains generated results for target dummy rotations. We used 12:30 rotations in a full party, single-target scenario 
to generate expected values for every hit and derive a true expected dps for each comp. This value is not impacted 
by crit-rng (like logs are), so it represents a more precise measurement of the rotation.

Additionally, for each comp we ran 10000 simulations to build up statistical metrics like variance to help visualize 
the lower and upper bounds of each comp. These 10000 simulations are also used to handle jobs with random rotations;
we have solvers for the MNK/BRD/DNC random elements (RDM we treated as static), which should normalize the effects of 
rng on these jobs as well. 

## Contents and Methodology
We experimented with simulating jobs in 8-player comp situations to assess the strength of particular jobs in a target 
dummy situation against their peers. To limit variables, we started with three baseline comps:
* `DRK-GNB-AST-SCH-MNK-NIN-BRD-PCT` (Cards on NIN, PCT)
  * This is intended to be the "meta" raid-buff comp.
* `DRK-GNB-WHM-SCH-MNK-NIN-BRD-PCT`
  * This is a raid buff comp without cards. I use this when the targets for cards might vary to isolate card impact.
* `PLD-WAR-SGE-WHM-SAM-VPR-MCH-BLM`
  * This comp has tests for performance under a low-to-no raid buff situation.

From these comps, we focus on specific roles by testing all **unique permutations without duplicates**. Each comp is 
labeled with the 1-2 jobs it is specifically testing. All permutations are simulated against each other and plotted in 
the same graph for easy side-by-side comparison.

| Comp                                            | Link                                                                                                                | Comment                                                                  |
|:------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------|
| `(Variable Tanks)-AST-SCH-MNK-NIN-BRD-PCT`      | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/TankBuffCompCompare)      |                                                                          |
| `(Variable Tanks)-SGE-WHM-SAM-VPR-MCH-BLM`      | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/TankNoBuffCompCompare)    |                                                                          |
| `DRK-GNB-(Variable Healers)-MNK-NIN-BRD-PCT`    | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/HealerBuffCompCompare)    |                                                                          |
| `PLD-WAR-(Variable Healers)-SAM-VPR-MCH-BLM`    | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/HealerNoBuffCompCompare)  |                                                                          |
| `DRK-GNB-WHM-SCH-(Variable Melees)-BRD-PCT`     | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/MeleeBuffCompCompare)     | Mostly laziness on not using AST here, I didn't want to configure cards. |
| `PLD-WAR-SGE-WHM-(Variable Melees)-MCH-BLM`     | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/MeleeNoBuffCompCompare)   |                                                                          |
| `DRK-GNB-AST-SCH-SAM-NIN-(Variable Ranged)-PCT` | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedBuffCompCompare)    |                                                                          |
| `PLD-WAR-SGE-WHM-SAM-VPR-(Variable Ranged)-BLM` | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedNoBuffCompCompare)  |                                                                          |
| `DRK-GNB-WHM-SCH-MNK-NIN-BRD-(Variable Caster)` | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterBuffCompCompare)    |                                                                          |
| `DRK-GNB-AST-SCH-MNK-NIN-BRD-(Variable Caster)` | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterAstBuffCompCompare) | Ranged cards were used on the caster.                                    |
| `PLD-WAR-SGE-WHM-SAM-VPR-MCH-(Variable Caster)` | [results](https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterNoBuffCompCompare)  |                                                                          |

### Rotations
All rotations are hand-sheeted `12:30` rotations, using pots in the opener, 6 minute, and 12 minute windows.
All rotations attempted to align with a ~6s buff opener. As mentioned before, MNK/BRD/DNC have some random elements
in the simulation:
* MNK: Chakra is randomly simulated based on game values. TFC's are used whenever possible with no holding.
* BRD:
  * The sim uses Heartbreak Shot and Pitch Perfect in open weaves based on randomized procs. 
    * Raging windows are used to determine when to hold HB shot: The sim will start holding them if it would not lose a usage before the next window.
  * Burst Shot and Refulgent Arrow are used interchangeably based on procs.
  * Apex Arrow has its damage simulated based on simulated gauge procs. Blast Arrow is skipped for a Burst/Refulgent if requirements are not met.
    * In practice, a player would work around this. We allow the sim to look at the sum of gauge generated instead of hard resets to avoid having to move Apex from intended usage times.
  * Dot refreshes are not moved: if this leads to an overcap for Refulgent Arrow, the sim makes an exception and allows 2 usages back to back.
    * This is to simplify how a real player would refresh early to avoid this situation.
  * Army's Paeon GCD effects are ignored: we sheeted the gcd increase manually and consistently.
  * All other abilities are not moved or overwritten by the solver.
* DNC:
  * The sim replaces almost all gcds and ogcds following the generally recommended priority system.
  * Dance timings, Standard refreshes, and Flourish/Devilment usages are never moved.

<div style="display: flex; justify-content: space-between; gap: 10px;">
  <figure style="flex: 1; text-align: center;">
    <img src="/figures/brd-rng.png" style="width: 100%;">
    <figcaption>BRD RNG Solver Output</figcaption></figcaption>
  </figure>
  <figure style="flex: 1; text-align: center;">
    <img src="/figures/dnc-rng.png" style="width: 100%;">
    <figcaption>DNC RNG Solver Output</figcaption>
  </figure>
</div>

MNK and DNC also scan other rotations in the comp to simulate the chakra/esprit generation from the party.

We acknowledge there might be user error in the hand-sheeted rotations and rng-solvers. We invite people to call out 
blatant errors in rotations if the mistake is easily verifiable. If you have a disagreement with the specific optimizations
in the sheet, please provide **completed, 12:30 rotations via xiv-in-the-shell export with gearsets** for us 
to review and resimulate. If the outcome is significantly different (100s of dps), we will post revised results.

### Important Simulations
There are many generated outputs and plenty of them might not be very insightful, so I'll outline the most important ones.
* `rotations`: These are generated damage outputs from the actual rotations and are the base data for the visualizations here.
  * These are included specifically to help validation of this work over time.
  * This is also where you will notice the simulated probabilities of random events for MNK/BRD/DNC.
* `comp_dps`: This tracks simulated percentiles for all the comps in terms of DPS @ `12:27.500`
  * By percentiles, it's the statistics version: `x%` means "`x%` of data is less than or equal to this value."
  * This is computed via two different methods
    * `expected +- standard deviation` (`comp_dps_percentiles`)
    * Sampled data percentiles (`comp_dps_sampled_percentiles`)
  * `comp_dps_sampled_percentiles` is what we will use for analysis. It will deviate from pure expected value due to random simulations, but it makes no assumptions on the distribution of the underlying data.
* `comp_dps_over_time`: Tracks expected full comp dps for every damage application, which can be used to see specific spikes in damage by comp.

### Files and Outputs
* [configs](configs) contains the original csvs and 8-player comp configurations for generating these outputs.
* [outputs](outputs) contains simulated results for all of these comps in `csv` and `html` formats.
  * https://htmlpreview.github.io/? can be used to view `html` files without any other tools by appending the full url.
  * Example: https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterAstBuffCompCompare/comp_dps_over_time/comp_dps_over_time.html


## Outcomes

These are initial reactions based on the `comp_dps` graphs generated by our sim. You can also draw similar conclusions
by looking at the `comp_dps_over_time` version if you only care about expected value.

| Name                       | Comment                                                                                                                                       | Link                                                                                                                                                                                       |
|:---------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Buff Comp Dps (Tanks)      | DRK comps are the strongest, gaining at most `1000 DPS` against the worst pair (PLD-WAR).                                                     | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/TankBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)      |
| No Buff Comp Dps (Tanks)   | GNB comps are the strongest, gaining at most `600 DPS` against the worst pair (WAR-DRK).                                                      | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/TankNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| Buff Comp Dps (Healers)    | Buff-using healers seem very strong in this experiment, AST/SCH gaining over `3000 DPS` against WHM/SGE.                                      | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/HealerBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| No Buff Comp Dps (Healers) | AST/SCH remains relatively dominant (`+1500 DPS` against WHM/SGH), but AST's performance seems to fall in line with WHM.                      | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/HealerNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)  |
| Buff Comp Dps (Melees)     | MNK/NIN is a clear winner (`~600 DPS` over NIN/RPR), Outside of this, differences are small with SAM/VPR being the worst.                     | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/MeleeBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)     |
| No Buff Comp Dps (Melees)  | MNK/NIN is a narrow winner (`<100 DPS` over MNK/RPR). RPR seems a lot more powerful in this situation, while SAM/DRG has moved to the bottom. | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/MeleeNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)   |
| Buff Comp Dps (Ranged)     | BRD is the strongest in this experiment, even with using SAM for DNC Partner. MCH is down about `3000 DPS`                                    | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| No Buff Comp Dps (Ranged)  | BRD is still a clear winner, but by a smaller margin (`~1300 DPS`). DNC is very slightly above MCH.                                           | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)  |
| AST Buff Comp Dps (Caster) | PCT seems to win by a good margin (`~600 DPS` over RDM), with RDM slightly edging out BLM.                                                    | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterAstBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html) |
| Buff Comp Dps (Caster)     | PCT is still ahead, but the three real casters are within `300 DPS` of each other.                                                            | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)    |
| No Buff Comp Dps (Caster)  | BLM takes a dominant lead in this scenario (`+1500 DPS` over PCT), with PCT actually moving below RDM.                                        | [Link](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/CasterNoBuffCompCompare/comp_dps/comp_dps_sampled_percentiles.html)  |

### Tanks
Tanks seem very balanced. GNB performs the best without buffs, while DRK takes a decent lead with buffs.

### Healers
SCH and AST are very dominant in buff-stacking comps, gaining over 3000 dps over the WHM/SGE pair. 
Another way to put this is in the most extreme situation, an absolutely insane WHM/SGE performance (99% in an otherwise meta buff comp) is about equal to the *average* AST/SCH run
In cases where AST/SCH are the only buff providers, the difference drops dramatically: it seems like the ranking is `SCH > AST >= WHM > SGE` in that case.

### Melees
MNK/NIN seems to be a strong combo, and this is not surprising if you follow top speed runs. These jobs seem to do really
well when supported by (and supporting) other buff jobs. It's still the best in low-buff situations, but not by much.

Regarding SAM/VPR, they seem to be a bit weak. SAM is pretty contextual regarding when it's going to do really well, but I think
it's fair to say that it often comes up a little short. Raid-buffs are just that good. You can also call out that
I didn't use DNC or AST in this test, which would favor SAM/VPR: I might follow up with more experiments later, 
but I will call out that BRD is generally stronger than DNC so the decision-making to bring DNC to justify SAM/VPR seems dubious.

The maiming jobs seem to be in the middle of the pack. I personally haven't looked so much as to why: I think I was expecting
a bit more out of DRG and a bit less out of RPR. I'm not really familiar with what's going on here since we haven't simmed
those jobs much since 7.0 (**Sleepocat** internally banned DRG after that patch).

### Ranged
BRD slaps. MCH sucks. This should not be surprising to anyone. DNC is in a decent spot, but I think it is edged-out by 
BRD mostly because SAM/VPR just don't pack enough punch to justify losing a raid-buff (more like 2, if you consider BRD has 2 of them). 
I went ahead and included SAM into the comp here to highlight this effect. You can also infer this by comparing the 
[buff-dps](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedBuffCompCompare/buff_dps/buff_dps.html) graph and the
[ndps-over-time](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedBuffCompCompare/ndps_over_time/ndps_over_time.html) graph 
(You can click on jobs in the legend to the right to hide them from the graph): BRD and DNC seem to have similar nDPS on average, so the power difference is purely in the 1k+ dps from buffs.

I will call out that DNC seems to do better in shorter fights since BRD starts out with a weak radiant in the opener.
Peeking at the [comp-dps-over-time](https://htmlpreview.github.io/?https://github.com/apollo-van-waddleburg/patch-74-sim-results/blob/main/outputs/RangedBuffCompCompare/comp_dps_over_time/comp_dps_over_time.html) 
graph, you can see this play out around the 4-5m mark.

MCH is just below everyone else even in no-buff situations (but it's close to DNC!). In a high-buff situations, it's 
likely the single worst job you can bring over BRD or DNC. It could stand to do 2k more dps on average and that would put it into the "conditionally best" tier. I'll also call out that we haven't simmed MCH before this week so ah... maybe
something is off. But no-one is really surprised here... right?

### Caster
The balance between casters is... honestly great (if you exclude SMN). They all seem to push ahead in specific scenarios:
BLM is dominant in low buff comps, PCT in high buff comps, and RDM is cleanly in the middle of the pack (SMN is down-bad).
I want to callout out that RDM is in a place where you can make the call *not* to swap off the job for a no-rez caster 
during prog this tier (you should swap SMN). It gets more interesting if you think about specific opti scenarios 
(like a carry-over run, which SMN is also useless for) where RDM might actually be meta.

I did something special where I compared had an AST buff comp and a no AST buff comp to test the impact of cards 
(but maybe I should have carded the PhysRanged over SMN). PCT shot ahead in that specific scenario as you would 
expect since it's kinda the perfect job for the current meta (I couldn't make SMN work though).

## Disclaimers

### Bias
These simulations bias towards buff-stacking comps. While I do believe the strongest comp will heavily utilize
raid-buffs, the reality for most people in most situations is that people will fail to play around raid-buffs well. 
This failure to perform optimally is not something we try to emulate with these simulations, so it's hard to know exactly
how these results translate to a real world scenario. As such, I wouldn't overthink the results here regarding specific 
underperformers and I encourage you to consider the context of your own situation when evaluating this data.

### Mistakes

To reiterate:
> We acknowledge there might be user error in the hand-sheeted rotations and rng-solvers. We invite people to call out 
blatant errors in rotations if the mistake is easily verifiable. If you have a disagreement with the specific optimizations
in the sheet, please provide **completed, 12:30 rotations via xiv-in-the-shell export with gearsets** for us 
to review and resimulate. If the outcome is significantly different (100s of dps), we will post revised results.

Additionally, we accept there might be bugs in the underlying sim code and will be on constant lookout for them. 
We do verify our expected damage against gearset-annotated logs to make sure we accurately simulate the correct ranges.
We also verified against the generated rotations in xivgear.app for line-by-line expected value 
(we consistently were within 1-2 damage per line). 

All that said, I personally am not an expert on every job and there are certainly... decisions... you might find in some sheets. 
Some of the jobs are very rarely simmed or haven't been simmed at all until this week.
I also coded the solvers so there's definitely the possibility of mistakes.

### Discriminatory Use
**You should not use the results of these tools to discriminate against any jobs in a public setting.**

These results come from an idealistic scenario and are biased towards jobs that play into a raid-buff meta. 
Some of the jobs that are under-performing in these simulations fare better in more real-world situations since they
are less dependent on their team and can have more consistent output. The goal of this assessment is to provide a 
new perspective on job performance that is not feasible to see via submitted logs.

## Credits
* **Ama** for providing an open-source simulation tool in python, which was used extensively for this project
  * [Pip](https://pypi.org/project/ama-xiv-combat-sim/#description)
  * [xivraider](https://xivraider.com/)
* **Shanzhe** and the supporting team for [xivintheshell](https://xivintheshell.com/) for a public rotation tool.
* [xivgear.app](https://xivgear.app/) for their gearset support and damage validation.
* [Caro](https://www.youtube.com/@CaroKannffxiv) and **Parryprog** for writing the vast majority of the rotations and reviewing work.
  * They will be attempting speed-kills in 7.4, please support them!
* **Sleepocat** for being the original testers for many of these simulations and visualizations.
  * For context, I played as a Tank for the team and managed a lot of the simulations, tools, and planning around their runs.

### P.S
If you are interested in speeds or spreadsheet theorycraft, please reach out to me on discord (apollo.van.waddleburg). 
We are slowly rolling out many of the tools we used to generate this report to interested parties and would love 
to develop a community around damage simulation and speed kills.