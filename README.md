# CMPM121-S04-CodeSmells
Group members: Roman Hage, Jack Farris
Code Smells: Duplicated Code, Data Clumps 















Data Clumps:
Introduction:
A data clump is a group of three or four values that keep showing up together, as fields in several classes or as parameters in many function signatures, without ever being given a type of their own. Fowler puts it this way in Chapter 3 (p. 78): “Data items tend to be like children: They enjoy hanging around together.” A group that always travels together “really ought to find a home together.”

How data clumps arise: 
  Gradual growth
  copy-paste reuse
  thinking in primitives 
  boundary data
  prototyping

Why data clumps are a problem:
  Long parameters lists
  Duplicated logics
  Shotgun surgery 
  Inconsistent naming 
  A missing abstraction

Preventing Data clumps:
  Use the rule of three 
  Model domain concepts early 
  Prefer existing types 
  Watch parameter count in code review 
  Make new types cheap 

Code examples: 




