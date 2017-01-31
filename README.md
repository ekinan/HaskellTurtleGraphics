# HaskellTurtleGraphics
This implements Assignment 1 in the Advanced Functional Programming course provided by Chalmers
University (I am doing the one on http://www.cse.chalmers.se/edu/year/2016/course/course/TDA342\_Advanced\_Functional\_Programming/index.html). It is an assignment that's done in pairs with some questions and a report written, but I chose to focus only on the implementation of the required components in the assignment (with some additional features of my own such as the while and ifelse constructs). The assignment also asked to consider a deep or a shallow embedding; I chose a shallow embedding as will be explained later below.

NOTE: This project was compiled and built using GHC version 8.0.2. Also "TProgram" in this project is their "Program" in the assignment. It is built and run using cabal.

The project itself implements a Turtle Graphics DSL, based on the Logo programming language from MIT. A program is a set of instructions that a Turtle must follow - shapes can be generated by the Turtle tracing its movements along the graphics window. See the Logos primer provided on my GitHub repo. for more details (obtained from the same Chalmers course). For my implementation, the primitive motion operations are the following:

	1. Forward x. The Turtle moves forward x units in its current direction.
	2. Right d. The Turtle rotates "right" (clockwise) d degrees. d must be positive.
	3. Left d. The Turtle rotates "left" (counter-clockwise) d degrees. d must be positive.

All of the provided operations are as follows:
	1. idle -- The program that does nothing.

	2. pause -- The program that pauses the Turtle for a single frame.

	3. forward x -- specified above.

	4. right d -- specified above.

	5. left d -- specified above.

	6. penup -- Turns off drawing - the Turtle will not draw on the graphics window when moving forward.

	7. pendown -- Turns drawing on, Turtle will be able to draw its motion on the graphics window.

	8. color -- Changes the Turtle's pen's color. See Pen.hs for the available colors.

	9. die -- Kills the Turtle, rendering it unable to perform any more actions.

	10. limited -- Takes in a program, and runs it either to completion or until some specified number of steps are reached. Note that this does NOT work with parallel Turtle programs - I could have done it, but I liked the idea of specifying a limited Turtle program for a single program only.

	11. >*> -- This operation takes in two programs, p1 and p2, and sequences them together.

	12. <|> -- This operation combines two programs, p1 and p2, into a single parallel program. A parallel program is one where there are two turtles, one executing p1 and the other executing p2.

	13. while -- Takes in a predicate about the Turtle, and executes the passed in program p while that predicate is true.

	14. ifelse -- Takes in a predicate about the Turtle, and executes either p1 if the predicate is true, or p2 if the predicate is false.

	15. backward x -- Moves the Turtle backward by x units.

	16. times -- Takes in an integer n and a program p, and runs p n times.

	17. forever -- Takes in a program and runs it forever, or until the Turtle dies.

	18. runTextual -- Runs the program using a textual interface. In each step, the Turtle's state before the operation is shown, the exact operations executed on that Turtle are displayed, and then the Turtle's state after the operations are shown. Note this does NOT produce readable output for parallel programs; it is highly recommended that this interface be viewed with standard programs.

	19. runGraphical -- Runs the program using a graphical interface provided by the HGL library. In each step, the Turtle's motion is drawn on the window if the pen is enabled. This is the best option to look at when running both a regular or a bunch of parallel programs. Note that the graphics can be crude because HGL uses integer coordinates to access each pixel, while the Turtle uses decimal coordinates. This is especially true if the values for the maximum # of forward steps or the maximum # of degrees that the Turtle can rotate change. The Turtle starts off at a point slightly above the bottom edge of the window at the center.

Some cool example programs can be found in the TProgramExtras.hs module. The one that is currently running (if executed using cabal right after cloning the repo) has four Turtles draw the Triforce from Legend of Zelda.

I wanted to experiment with Type classes and some more advanced Haskell features in this project, so I chose to create an abstract type class representing programs that animate some form of object. The code and explanation for this can be found in the code contained in the TimeStep folder of the project (see code organization below). Using this representation resulted in a shallow embedding - a simple Turtle program (not parallel) is just a function with the following type (w/o all the newtype wrappers):
	Energy -> Turtle -> (NextAction TProgram Energy, Turtle, TSummary)

where NextAction TProgram Energy indicates if the program finished completely, whereby the remaining energy is returned, or if the program partially finished during the time step, whereby the remaining piece of the program is returned. The Turtle part is the final Turtle after the time step, and TSummary summarizes all of the operations that were done on the Turtle in that time step, along with any other events that occurred.

There was an interesting question in the assignment about a Turtle program being a monoid. In my case, it is a monoid with mempty = idle, and mappend = >*>. I define two programs as being equal iff they take the Turtle through the same sequence of step and have it end up at the same final state. Note that under this definition, the monoid laws hold:
			
			idle `mappend` p = p

			p `mappend` idle = p

			(p1 `mappend` p2) `mappend` p3

The first two laws are always true - because the idle program does not do anything at all and because it is instantaneous, the net effect on a Turtle program is to just execute p.

The associativity law is also true with >*> being mappend. Note that with >*> as mappend, we will never run p2 before p1 finishes and p3 before p2 finishes so that the order will always be p1 > p2 > p3, where a > b indicates that program a runs before b. 


The project code is organized as follows (I used the stub cabal package provided by Chalmers for the assginment). All code is in src.

	1. TimeStep
		This holds all of the code relating to the abstract program and parallel program class discussed above, where by program we mean any program that takes some object and does some operations on it. So this type class can be used for programs that are specific to animating rabbits, or animating a lion, or animating even two objects at once (as a pair), etc. - it is defined in as general a way as possible.

	2. Utils
		This contains helpful utility files for the project. The only one I needed was the HGLUtils module, which converts from the Turtle world to the HGL world.

	3. Turtle
		This module contains the code defining the Turtle object used for this project, as well as its pen.

	4. TProgram
		This module contains all of the code implementing the Turtle programming language itself, such as the operations defined above. It also has the code for the textual and graphical interfaces, and for all of the other cool examples that can be drawn with the Turtle programming language.
