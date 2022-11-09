# RE: How to (not) clean JSON file in C#

### RE:
It is not a surprise that I received again at the first interview stage, just for another company, the same *puzzle* with JSON cleaning. This time I had a clue about how to start and I could provide a solution to it in a reasonable time. I recommend reading first this [article](https://dev.hodina.net/the-questionable-value-and-the-real-frustration-of-interview-coding-tests). 

### Puzzle statement

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667998391989/NTtYK51g9.png align="left")

### The solution

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1667997346837/AHfX0mrFh.png align="left")

### Final notes
Take it as a starting point and not as a recommended approach. It is far from a perfect solution, but it works at least for the examples provided. 
Creating a text parser can be a very complex task and requires a lot of test cases to cover and test. This is the main reason why I didn't like the puzzle: the statement was quite generic and did not have enough test cases so I had to make a lot of assumptions. *Garbage in, garbage out, haha.*

I hope can help some folks.

*If you had to solve the same puzzle for an interview, please let me know about your approach and opinions on it. Cheers!*
