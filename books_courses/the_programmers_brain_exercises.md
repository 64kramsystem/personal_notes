# The Programmer's Brain Exercises

- [The Programmer's Brain Exercises](#the-programmers-brain-exercises)
  - [2.4: Iconic memory](#24-iconic-memory)
  - [2.5: Beacons](#25-beacons)
  - [2.6: Code concepts (chunking) reproduction](#26-code-concepts-chunking-reproduction)
  - [3.2: Elaboration](#32-elaboration)
  - [4.1: Cognitive load](#41-cognitive-load)
  - [4.2: Dependency graphs/State tables](#42-dependency-graphsstate-tables)
  - [5.1: Variables framework](#51-variables-framework)
  - [5.2: Deep understanding](#52-deep-understanding)
  - [5.4: Text comprehension](#54-text-comprehension)
  - [5.5: Role of lines](#55-role-of-lines)
  - [5.6: Variable names](#56-variable-names)
  - [5.7: Identifier operations](#57-identifier-operations)
  - [5.8: Summarizing code](#58-summarizing-code)
  - [6.1: Mental models](#61-mental-models)

## 2.4: Iconic memory

Select a piece of code that is somewhat familiar to you. It can be something from your own codebase, or a small and simple piece of code from GitHub. It doesn’t matter all that much what code you select, or the programming language used. Something the size of half a page works best, and if possible, printing it on paper is encouraged.

Look at the code for a few seconds, then remove it from sight and try to answer the following questions:

- What is the structure of the code?
  - Is the code nested deeply or it is flat?
  - Are there any lines that stand out?
- How is whitespace used to structure the code?
  - Are there gaps in the code?
  - Are there large blobs of code?

## 2.5: Beacons

Select an unfamiliar codebase, but do select one in a programming language that you are familiar with. If possible, it would be great to do this exercise on a codebase where you know someone familiar with the details. You can then use that person as a judge of your understanding. In the codebase, select one method or function.

Study the selected code and try to summarize the meaning of the code.

Actively notice beacons that you use: Whenever you have an "aha" moment where you get a bit closer to the functionality of the code, stop and write down what it was that led you to that conclusion. This could be a comment, a variable name, a method name, or an intermediate value - all of those can be beacons.

Reflect: When you have a thorough understanding of the code and a list of beacons, reflect using these questions:

- What beacons have you collected?
- Are these code elements or natural language information?
- What knowledge do they represent?
- Do they represent knowledge about the domain of the code?
- Do they represent knowledge about the functionality of the code?

## 2.6: Code concepts (chunking) reproduction

Select a codebase you are somewhat familiar with—maybe something you work with regularly, but not mainly. It can also be something you personally wrote a while ago. Make sure you have at least some knowledge of the programming language the code is written in. You have to know more or less what the code does, but not know it intimately. You want to be in a situation similar to the chess players; they know the board and the pieces but not the setup. In the codebase, select a method or function, or another coherent piece of code roughly the size of half a page, with a maximum of 50 lines of code.

Study the selected code for a bit, for a maximum of two minutes. After the timer runs out, close or cover the code.

Take a piece of paper, or open a new file in your IDE, and try to recreate the code as best as you can.

When you are sure you have reproduced all the code you possibly can, open the original code and compare. Reflect using these questions:

- Which parts did you produce correctly with ease?
- Are there any parts of the code that you reproduced partly?
- Are there parts of the code that you missed entirely?
- Do you understand why you missed the lines that you did?
- Do the lines of code that you missed contain domain concepts that are unfamiliar to you?

## 3.2: Elaboration

Use this exercise the next time you learn a new programming concept. Answering the following questions will help you elaborate and strengthen the new memory:

- What concepts does this new concept make you think of? Write down all the related concepts.
- Then, for each of the related concepts you can think of, answer these questions:
  - Why does the new concept make me think of this concept that I already know?
  - Does it share syntax?
  - Is it used in a similar context?
  - Is this new concept an alternative to one I already know?
- What other ways do you know to write code to achieve the same goal? Try to create as many variants of this code snippet as you can.
- Do other programming languages also have this concept? Can you write down examples of other languages that support similar operations? How do they differ from the concept at hand?
- Does this concept fit a certain paradigm, domain, library, or framework?

## 4.1: Cognitive load

The next time you read unfamiliar code, try to monitor your own cognitive load. When the code is hard to process and you feel the need to make notes or follow the execution step by step, it is likely you are experiencing a high cognitive load.
When you experience high cognitive load, it is worthwhile to examine which parts of the code are creating the different types of cognitive load. You can use the following table to analyze this.

| lines of code | Intrinsic cognitive load | Extraneous cognitive load |
| ------------- | ------------------------ | ------------------------- |
|               |                          |                           |

## 4.2: Dependency graphs/State tables

Create a dependency graph and a state table for difficult code.

## 5.1: Variables framework

Find some code that you are unfamiliar with and examine the variables taking note of the following for each:

- The name of the variable
- The type of the variable
- The operations in which the variable plays a role
- The role of the variable according to Sajaniemi’s roles of variables framework

Fill out this table for each variable you find in the code.

| Variable name | Type | Operations | Role |
| ------------- | ---- | ---------- | ---- |
|               |      |            |      |

Once you have filled out the table, reflect on your decisions about the role of each variable. How did you determine the role? Which of the other aspects played a part in your decision? Was it influenced by the name of the variable, its operations, comments in the code, or maybe your own experience with the code?

## 5.2: Deep understanding

Find another piece of unfamiliar code in your own codebase.
Now follow these steps to gain a deep understanding of this code:

1. Find a focal point in the code. Since you are not fixing a bug or adding a feature, your entry point into the code will likely be the start of the code - for example, a main() method.
2. Determine the slice of code related to the focal point, either on paper or within the IDE. This might require some refactoring of the code to move the code involved in the slice closer together.
3. Based on your exploration in step 2, write down what you learned about the code. For example, what entities and concepts are present in the code, and how do they related to each other?

## 5.4: Text comprehension

Study a piece of unfamiliar code for a fixed amount of time (say, 5 or 10 minutes, depending on the length). After studying the code for this fixed amount of time, try to answer the following concrete questions about it:

- What was the first element (variable, class, programming concept, and so on) that caught your eye?
- Why is that?
- What was the second thing you noticed?
- Why?
- Are these two things (variables, classes, programming concepts) related?
- What concepts are present in the code? Do you know all of them?
- What syntactic elements are present in the code? Do you know all of them?
- What domain concepts are present in the code? Do you know all of them?

The result of this exercise might motivate you to look up more information about unfamiliar programming or domain concepts in the code. When you encounter an unfamiliar concept, it is best to try to study it before diving into the code again. Learning about a new concept at the same time as reading new code will likely cause excessive cognitive load, making both the concept and the code less effective.

## 5.5: Role of lines

Select a piece of unfamiliar code and give yourself a few minutes to decide on the most important lines in the program. Once you have selected the lines, answer these questions:

- Why did you select those lines?
- What role do those lines have? For example, are they lines that perform initialization, or input/output, or data processing?
- How are these lines connected to the overall goal of the program?

## 5.6: Variable names

Fill out the following table for all of the variable names.

| Name | Domain? | Concept? | Is the name understandable without looking at the code? |
| ---- | ------- | -------- | ------------------------------------------------------- |
|      |         |          |                                                         |

Using the table of variable names, you can answer these questions:

- What is the domain or topic of the code?
- What programming concepts are used?
- What can you learn from these names?
- Which names are related to each other?
- Are there names that are ambiguous when looked at without context?
- What meaning could ambiguous names have in this codebase?

## 5.7: Identifier operations

Select a piece of unfamiliar code and write down the names of all the variables, functions, and classes in the code. Then list all the operations associated with each identifier.

| Identifier name | Operation(s) |
| --------------- | ------------ |
|                 |              |

Once you’ve created this table, read through the code again. Has filling in the table helped you gain a deeper understanding of the roles of the variables and the meaning of the problem as a whole?


## 5.8: Summarizing code

Summarize a piece of code by filling in the following table. Of course, you can add more information to the summary than I’ve suggested in this exercise.

| Origin                                                |     |
| ----------------------------------------------------- | --- |
|                                                       |     |
| Goal of the code: What is the code trying to achieve? |     |
| Most important lines of code                          |     |
| Most relevant domain concepts                         |     |
| Most relevant programming constructs                  |     |
| Decisions made in the creation of the code            |     |

## 6.1: Mental models

Consider a piece of code you used in the last few days. What mental models of that code did you use while programming? Did those mental models concern the computer or code execution or other aspects of programming?
