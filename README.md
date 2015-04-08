# respect matrix
(c) mark p xu neyer
2015
bsd

Define a respect matrix as follows: Entry `M[i,j]` is a real number in `[-1,1]` representing how much person `i` respects person `j`.

The matrix M is initialized as an identity matrix: everyone is assumed to respect themselves and to have no opinion of everyone else. Each person `i` can add whatever entries they want, to entry `i,j` specifying how much they respect person `j`. 1 means total respect , -1 means absolute disrespect.

Let a 'respect path from a to b' be a sequence of indices `p = [i_1 = a, i_2, i_3... i_n = b]` starting with 'a' and ending with 'b'. We can compute the 'implied respect' of that path as follows:

    def implied_respect(M, p):
      # assume people respect themselves 
      total = 1
      last_edge_positive = True
    
      # go through this path one step at a time
      for a,b in by_consecutive_pairs(p): 
    
        if last_edge_positive:
          this_respect = M[a,b]
        else:
          this_respect = M[b,a]
    
        if this_respect == 0:
          return 0
    
        total = total*this_respect
        # invariant:
        # `total` contains the implied respect of p[0] for b
        last_edge_positive = this_respect > 0
    
      return total

This value is the 'implied respect' of path p from person a towards person b

You can see from this that if person a has a negative respect towards person b, they will also have a negative respect towards anyone who respects person b - but not the people person b respects.  Negative respect values cause a 'flip' of the direction of the edge that is inspected for the next coefficient.  If A has negative respect for B, the next thing we look at is how C feels about B. If C respects B, and A has negative respect for B, then A _also_ has negative respect for C. This prevents B from trolling A's friends (by falsely claiming to respect them), and allows for A to have respect for people who also disrespect people A disrepects. After the multiplication is performed, the following invariant holds: '`total` contains the implied respect of p from `p[0]` to `b`':


So, given two people, A and B, there is some number of paths between A and B. The distribution of the implied respects on each of these paths gives us information about how A feels about B, as directly expressed by A (if A has expressed an opinion) as well as implied by all the stated respects of the other members of the matrix.  For example, if the distribution is uniformly positive, it means that A probably will respect B. A doesn't directly disrespect B, or respect anyone who disrespects B, or disrespect anyone who B respects. Depending upon who these people are, a uniformly positive distribution will be exceedingly rare.

We can compute the 'total implied respect' from person a to person b by summing the implied respects across all paths from a to b:

    def total_implied_respect(M, a, b):
      return sum([implied_respect(M,p) for p in one_pair_all_paths(M, a, b)])

This 'total implied respect' would be a good proxy for strangers determing whether they wanted to interact with one another. If the interaction between strangers went negatatively, the new negative edges on the graph could be used by both parties to warn their friends.

## Controversial:

The 'total implied respect' from a to be can be compared with the 'explicit stated respect' of a to b, (that is, the value M[a,b]) in order to compute the 'dissonance' of a's explicit respect for b.

For example, suppose A says A respects B, C, and D at 1.  Meanwhile, B and C both respect D at -1.  The explicit stated respect of A for D is 1, while the implied respect of A for D is 1 -1 -1 = -1 , because the paths ABD and ACD both have total weight -1.  The divergence between A's 'explicit respect' for D, and the 'implied respect' generated by the matrix suggest that A's respect has less internal consistency.

We can compute the 'total dissonance' of person a's respect as follows: 

   def single_edge_dissonance(M, a, b):
     explicit_respect = M[a,b]
     implied_respect = total_implied_respect(M, a, b)
     # use a square so we have positive deltas
     # also sum of squares and other nice properties of distance metrics
     # also a bunch of tiny dissonances don't matter as much as a few big ones
     return (explicit_respect - implied_respect)**2
   
   def total_dissonance(M, a):
     sum_of_squares = sum([single_edge_dissonance(M, a, b) for b in all_others(M, a)])
     return sum_of_squares**0.5
   
# this would make politics much more reasonable. 

The introduction of the 'soundness' metric gives people an incentive to make sure people they respect get along with each other - which means a whole host of parties interestd in resolving conflicts between two people, as opposed to just a man wearing a black dress with a white collar who reads from an old book and then, after hearing two men wearing suits aruging about the meaning of words, bangs a stick. 



