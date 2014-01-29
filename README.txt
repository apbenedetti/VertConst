Last update: April 20th, 2012

This algorithm was created in an electrical engineering graduate course entitled "Computer Methodologies for Digital/RF Design." The course taught algorithms, specifically used in CAD software for integrated chip design. There is very little application of the content taught in the course, but it provided me with the opportunity to work on my coding skills.

A number of algorithms were demonstrated in the lectures on paper, and the goal of the course project was to implement one of these algorithms in code.

The project I chose was in IC layout compaction in channel routing. A simple way to explain this would be that two chips are separated by a channel. Each chip has several terminals that have to connect to either itself, and/or terminals on the other chip, or to nothing. The goal of the algorithm is adhere to the constraints, either vertical or horizontal, while ensuring the interconnects are present and correct. Interconnects exist in 90deg angles.

Layout compaction problems are represented by 2 rows of terminals; oriented top and bottom. Columns and rows are constructed on different physical layers on the ICs, so we are only interested in determining the order that the rows will occupy the channel.

When only horizontal constraints exist, it is easy to implement as we don't care about how wide the channel gets (vertically). A very simple way to solve is each interconnect can take up its own row. My problem deals with vertical constraints only, which is essentially keeping the channel as narrow (vertically) as possible. The Vertical Constraint Graph was the chosen as the algorithm to solve this problem. It makes use of an adjacency matrix.

The language I chose for the project was Matlab, due to its native use of matrices. I envisioned that manipulating matrices would be the simplest way of representing the changing diagram.



The algorithm...
Given a channel, we must identify vertical constraints or "edges." For example...

3 2 1 4 1 0 2 4
1 2 1 3 2 1 0 1

Any terminals that are NOT across from the same number, or a 0, are an edge. We can remove the non-edges, as these do not pose a problem for layout compaction...
  _ _     _ _
3 2 1 4 1 0 2 4
1 2 1 3 2 1 0 1

And we are left with the edges...

3 4 1 4
1 3 2 1

We call the top row the "tails" of the edges and the bottom row the "heads" of the edges.

Using this information, we can determine which interconnects will occupy the specific rows of the channel. The edges are representative of the rules that must be followed in ordering the interconnects. Observing our first edge (3/1), interconnect 1 will not able to be occupy a row that is higher up that interconnect 3 because interconnect 3 comes down from the top, and interconnect 1 comes up from the bottom.

	An adjancency list was not used in this algorithm, but should provide a visual representation of what I just said above...

	3->1
	4->3
	1->2
	4->1

	4->3->1->2
	 |->-|

	This means the rows in the channel will be in the following order...

	4
	3
	1
	2

As stated earlier, an adjacency matrix was used for this algorithm. An adjacency matrix is an N-by-N 0-matrix (where N is the number of nets), which represents edges with a 1 in the row of the tail, and column of the head.

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

By searching this matrix for a tail that is not a head, and a head that is not a tail, we find our top and bottom channel rows, and the remaining edges are placed inside, yielding...

4
3
1
2

The remainder of the algorithm fills in the graph. This is what we would see with the provided example (remember that the columns are on another physical layer so they are not present)...

3 2 1 4 1 0 2 4
      444444444
3333333
111111111111111
  22222222222
1 2 1 3 2 1 0 1