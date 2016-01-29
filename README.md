# Integrated Information Theory pseudocode
*Authors: Bo Marchman and Graham Findlay*  
*(c) 2016 Giulio Tononi - All right reserved*

This is a high-level overview of the process used to locate the major complex within a system and compute its integrated information. For the sake of clarity, some low-level details which are necessary to ensure that the computation is quick and correct in all conceivable cases have been omitted. More details, including a comprehensive Python implementation, are available on the web at www.integratedinformationtheory.org. 

Although this code makes no assumptions about the input data’s format, in practice all computations start with the system’s transition probability matrix, from which all important parameters can be inferred. We begin by describing how to compute core causes and effects in a system, then conceptual structures, and finally the main complex of a system. 

## Mechanisms and Purviews

A mechanism is any subset of elements within a system that has cause-effect power within the system. Any given mechanism may have cause-effect power over (i.e. constrain the past/future states of) multiple different subsets of system elements. Each of these subsets is a possible purview of the mechanism. A mechanism and a purview together specify a probability distribution, called a repertoire, which describes the specific cause-effect power of the mechanism within the system. If the purview under consideration is a past purview (how the mechanism constrains the past), then this repertoire is called a cause repertoire:

    function CauseRepertoire(<Mechanism> M, <Purview> P):
      return (the probabilities of P’s past states, conditioned on M’s current
              state)
           
The function `EffectRepertoire(<Mechanism> M, <Purview> P)` is defined analogously, with future states conditioned on the current state.  

## Small Phi (𝞿)

Given a mechanism `M`, a purview `P`, we are interested in knowing to what degree `M`’s cause-effect power over `P` can be reduced to the cause-effect power of its parts . By partitioning a cause repertoire into parts and determining the degree to which the original, unpartitioned repertoire can be recovered from those parts, we can assess the reducibility of that mechanism’s cause power. 
More precisely, let `M/P` denote the pairing of mechanism `M` with purview `P`. For every possible partition of `M/P`, compute the Earth Mover’s Distance (`EMD`) between the partitioned and unpartitioned cause repertoires of `M/P` and select the partition yielding the smallest distance as the minimum information partition of `M/P`. This is the partition that makes `M`’s cause repertoire most recoverable from its parts. The distance between repertoires given this partition is `M/P`’s integrated cause information, `SmallPhiCause`:

    function SmallPhiCause(<Mechanism> M, <Purview> P):
        if P is Null:
            return 0
        distances = Array[]
        unpartitioned_repertoire = CauseRepertoire(M, P)
        for every partition Z of M/P:
    	      M1, M2 = Split M according to Z
            P1, P2 = Split P according to Z
            partitioned_repertoire = CauseRepertoire(M1, P1) x
                                     CauseRepertoire(M2, P2)
            distance = EMD(partitioned_repertoire, unpartitioned_repertoire)
            distances.append(distance)
        return min(distances)
   
The function `SmallPhiEffect(<Mechanism> M, <Purview> P)` is defined analogously. 

## Concepts

The core cause of a mechanism `M` is the purview over which `M` has the largest cause power in the candidate set, and thus the cause repertoire of `M/core_cause` best describes `M`’s cause power in the set. To find `M`’s core cause, iterate over all possible past purviews `P` in the candidate set. After evaluating `SmallPhiCause` for each purview `P`, select the purview with the largest `SmallPhiCause` as the core cause of the mechanism `M`:

    function CoreCause(<Mechanism> M, <Candidate Set> C):
        core_cause = Null
        for each purview P in C:
            if SmallPhiCause(M, P) > SmallPhiCause(M, core_cause):
                core_cause = P
        return core_cause

The core effect of `M` is computed analogously. A mechanism with both a core cause and a core effect specifies a concept. 

    function Concept(<Mechanism> M, <Candidate Set> C):
        core_cause = CoreCause(M, C)
        core_effect = CoreEffect(M, C)
        if (core_cause is not Null) and (core_effect is not Null):
            cause_repertoire = CauseRepertoire(M, core_cause)
            effect_repertoire = EffectRepertoire(M, core_effect)
            return [cause_repertoire, effect_repertoire]
        else:
            return Null

The phi value of the concept is the minimum of `SmallPhiCause(M, core_cause)` and `SmallPhiEffect(M, core_effect)`:
 
    function SmallPhi(<Mechanism> M, <Candidate Set> C):
        core_cause = CoreCause(M, C)
        core_effect = CoreEffect(M, C)
        return min(SmallPhiCause(M, core_cause), SmallPhiEffect(M, core_effect))

## Conceptual Structure

To compute the conceptual structure of a candidate set of elements, find all mechanisms in the candidate set that give rise to concepts:

    function ConceptualStructure(<Candidate Set> C):
        conceptual_structure = Array[]
        for each mechanism M in powerset(C):
            if Concept(M) is not Null:
                conceptual_structure.append(Concept(M))
        return conceptual_structure
   
## Big Phi (𝚽)

Integrated conceptual information (`BigPhi`) is a measure of the system’s irreducibility (integration). We assess a system’s irreducibility similarly to the way in which we assessed a mechanism’s irreducibility:  Partition the system in question and measure the extent to which it can be recovered from the resulting parts. More precisely: For every unidirectional partition of the candidate set, compute the effect of that partition on the set’s conceptual structure (because the causal properties of the set may be altered by each partition, the conceptual structure associated with the set may change). Compute the extended Earth Mover’s Distance (`XEMD`) between the conceptual structures of the unpartitioned and partitioned candidate sets. Select the partition yielding the smallest `XEMD` distance as the minimum information partition. The distance between conceptual structures given this partition is the candidate set’s integrated (conceptual) information, `BigPhi`:

    function BigPhi(<Candidate Set> C):
        if C == Null:
            return 0
        distances = Array[]
        for each unidirectional partition Z of C:
            C' = C partitioned according to Z 
            distance = XEMD(ConceptualStructure(C), ConceptualStructure(C'))
            distances.append(distance)
        return min(distances)
   
## Major Complex

To find the major complex in a system `S`, compute `BigPhi` for every candidate set which is a subset of `S`. Select the candidate set with largest `BigPhi` as the main complex of the system:

    function MajorComplex(<System> S):
        major_complex = Null
        for each candidate set C in powerset(S):
            if BigPhi(C) > BigPhi(major_complex):
                major_complex = C
        return major_complex
        
___
### Licensing
Default copyright laws apply, because this work is intended for publication in a journal that may not be open-access. 
