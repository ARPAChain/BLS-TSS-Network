
# DKG Scenario Test Notes

## DKG General Concept

1. Check inputs and make sure they are good
2. update partial public key in node's member entry of the group.
3. Populate commitResult (groupEpoch, publicKey, disqualifiedNodes)
4. tryAddToExistingCommitCache(groupIndex, commitResult):
5. If consensus not reached, get majority member with identical commits.
        if majority members > g.threshold, slash all disqualified nodes.
        else, members not slashed, but will be slashed in postProcessDKG.

## DKG Scenario Tests

Happy path:
    - testDkgHappyPath: No dq nodes -> group.size = 5

1 Disqualified Node:
    - Everyone reports node1
        - test1Dq4Reporter: 4/5 report -> node1 slashed, successful group, size = 4
        - test1Dq3Reporter: 3/5 report -> node1 slashed, successful group, size = 4
        - test1Dq2Reporter: 2/5 report -> happy path, group.size = 5
        - test1Dq1Reporter: 1/5 report -> happy path, group.size = 5 (test skipped)
    - Mixed Reporting
        - testMixed1Dq4Reporter4Target: 4/5 report: 1 reports 2 reports 3 reports 4 reports 1: 

2 Disqualified Nodes
    - Everyone reports node1 + node 2
      - test2Dq4Reporter: 4/5 report -> both nodes disqualified, group.size = 3 (unrealistic, one node needs to self report)
      - test2Dq3Reporter: 3/5 report -> both nodes disqualified, group.size = 3
      - test2Dq2Reporter: 2/5 report -> same as happy path (test skipped)
      - test2Dq1Reporter: 1/5 report -> same as happy path (test skipped)

Non-disqualified majority members < g.threshold (3): (Is this correct behavior???)
    - 3 Disqualified Nodes:
        - test3Dq3Reporter: 3/5 report node1, node2, node3 disqualified.
        -  unable to reach consensus
        -  Group size = 5, nodes not slashed.

    All Different Disqualified Nodes: 
        - testMixed1Dq5Reporter5Target:  5/5 report: (1 reports 2 reports 3 reports 4 reports 5 reports 1)
        - unable to reach consensus 
        - Group size = 5, nodes not slashed.

## PPDKG Scenario Tests

3 Disqualified Nodes
    -testPPDKG3Dq3Reporter:
        - Call test3Dq3Reporter (3 disqualified nodes, slashing never completed)
        - call postProcessDKG as node1
          - 3 nodes slashed

All Different Disqualified Nodes:
    - testMixed1Dq5Reporter5Target:
        - Call testMixed1Dq5Reporter5Target (5 disqualified nodes, slashing never completed)
        - call postProcessDKG as node1
          - 5 nodes slashed
