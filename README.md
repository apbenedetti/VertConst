Last update to m-file: April 20th, 2012


### Introduction
This algorithm was created in an electrical engineering graduate course entitled *Computer Methodologies for Digital/RF Design*. The course taught algorithms, specifically used in CAD software for integrated chip design. There is little real-world application of the content taught in the course, but it provided me with the opportunity to work on my coding.

A number of algorithms were demonstrated in the lectures on paper, and the goal of the course project was to implement one of these algorithms in code.

The project I chose was in IC layout compaction, specifically in channel routing. A simple way to explain this would be that in an integrated circuit design there are two chips are separated by a space, or "channel."" Each chip has several terminals that have to connect to either terminals on the other chip, and/or other terminals on itself, or to nothing. The goal of channel routing is to adhere to constraints, either vertical or horizontal, while ensuring the interconnects are present and correct. Interconnects connect 2+ terminals in what is called a net. In this particular design, the interconnects are orthogonal (intersect at right-angles).

Layout compaction problems are represented by 2 rows of terminals denoted by the net in which they belong to; oriented top and bottom. Columns and rows are constructed on different physical layers on the ICs, so we are only interested in determining the order that the rows will occupy the channel.

When only horizontal constraints exist, it is easy to implement as we don't care about how wide the channel gets (vertically). A very simple way to solve is give a net its own row. However, my problem deals exclusively with vertical constraints, which requires keeping the channel as narrow (vertically) as possible. The Vertical Constraint Graph was the chosen as the algorithm to solve this problem.

The language I chose for the project was Matlab, due to its native use of matrices. I envisioned that manipulating matrices would be the simplest way of representing the changing diagram.


###The Algorithm

Given a channel...

	3 2 1 4 1 0 2 4
	1 2 1 3 2 1 0 1

We must identify vertical constraints. The vertical constraints are representative of the rules that must be followed in ordering the interconnects.

Any terminals that are NOT across from the same number, or across from a 0, are a vertical constraint, also called an "edge." If we observe the first column, we see that this is an edge because we have two different non-zeros; 3 & 1. Net 1 and net 3 are governed by this edge; net 3 comes down from the top, and net 1 comes up from the bottom so net 3 must occupy a row above net 1. They can not overlap because they occupy the same physical layer in the integrated circuit.

We can remove the non-edges, as these do not pose a problem for layout compaction and we are left with the edges...

	3 4 1 4
	1 3 2 1

We call the top row the "tails" of the edges and the bottom row the "heads" of the edges.

Comparing edge heads to other edge tails and vice versa, we can determine the order of nets occupying the specific rows of the channel.

An *Adjacency List* was not used in this algorithm, but provides a helpful visual representation. We reorganize the individual tails pointing to their respective heads.

	3->1
	4->3
	1->2
	4->1

We then organize all the heads and tails together.

	4->3->1->2
	 |->-|

This translates to the Vertical Constraint Graph, representing the order in which the nets will occupy the rows of the channel.

	4
	3
	1
	2

Instead, an *Adjacency Matrix* was used for this algorithm. An adjacency matrix is an N-by-N 0-matrix (where N is the number of nets), which represents edges with a 1 in the row of the tail, and column of the head.

Since we have 4 nets, our matrix will appear as such...

	- 0 0 0
	0 - 0 0
	0 0 - 0
	0 0 0 -

We will add in the 3/1 edge...

	- 0 0 0
	0 - 0 0
	1 0 - 0
	0 0 0 -

Adding in the remaining 3 edges yields...

	- 1 0 0
	0 - 0 0
	1 0 - 0
	1 0 1 -

As with the *Adjacency List*, by reorganizing information (this time from a matrix) we produce the Vertical Constraint Graph.

	4
	3
	1
	2

The remainder of the algorithm uses the Vertical Constraint Graph to fill in the channel initially provided terminal configuration.

This is what we would see with the provided example (remember that the columns are on another physical layer so they are not present)...

	3 2 1 4 1 0 2 4
	      444444444
	3333333
	111111111111111
	  22222222222
	1 2 1 3 2 1 0 1

####Cycles

The tricky part of this algorithm is when we have what are called cycles. These occur when a series of heads compared to tails, produce an infinite loop. These can be only two nets going back and forth, three different nets, or every net in the problem. Here is an example of a cycle including all nets...

	4 3 2 1
	2 1 3 4

...which produces four different Vertical Constraint Graphs. One being:

	1
	4
	2
	3
	1

When constructing the Vertical Constraint Graph, the first step is to find both the absolute head and absolute tail and place them in. This provides a starting point, which the other constraints are compared to. In some instances, like the new example I have provided, we would not get an absolute head or an absolute tail because the entire channel has a cycle.

For these cases, the last constraint that was analyzed in the Adjacency Matrix is just placed into the Vertical Constraint Graph as the starting point. The other constraints can then be added from there.

Notably, what needs to happen in these cases is that an addition column has to be added for every cycle so that the separated net can be connected (recall that the vertical connections are on a different physical layer). A possible solution we would see for the provided example would be:

	4 3 2 1 0
	      111
	4444444
	22222
	  333
	  1111111
	2 1 3 4 0
