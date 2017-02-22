# How to pass a programming interview (ranked in importance)

## 2. Study common interview concepts

> A large percentage of interview questions feature data structures and algorithms. For better or worse, this is the truth. We gather question details from our candidates who interview at YC companies, and algorithm questions make up over 70% of the questions that are asked. You do not need to be an expert, but knowing the following list of algorithms and data structures will help at most companies.

* Hash tables
* Linked lists
* Breadth-first search, depth-first search
* Quicksort, merge sort
* Binary search
* 2D arrays
* Dynamic arrays
* Binary search trees
* Dynamic programming
* Big-O analysis

> Depending on your background, this list may look trivial, or may look totally intimidating. That's exactly the point. These are concepts that are far more common in interviews than they are in production web programming, for instance. If you're self-taught or years out of school and these concepts are not familiar to you, you will do better in interviews if you study them. Even if you do know these things, refreshing your knowledge will help. A startingly high percentage of interview questions reduce to breadth-first search or the use of a hash table to count uniques. You need to be able to write a BFS cold, and you need to understand how a hash table is implemented.

> Learning these things is not as hard as many of the people we talk to fear. Algorithms are usually described in academic language, and this can be off-putting. But at its core, nothing on this list is more complicated than the architecture of a modern web app. If you can build a web app (well), you can learn these things. The resource that I recommend is the book The Algorithm Design Manual by Steven Skiena. Chapters 3 through 5 do a great job of going over this material, in a straightforward way. It does use C and some math syntax, but it explains the material well. Coursera also has several good algorithms courses.

> Studying algorithms and data structures helps not only because the material comes up in interviews, but also because the approach to problems taken in an algorithm course is the same approach that works best in interviews. Studying algorithms will get you an interview mindset.

* Master algorithms and data structures.
* Even if you do know them, refreshing your knowledge will help.
* Especially know about breadth-first search and hash table use & implementation
* Refer to The Algorithm Design Manual, especially chapters 3 through 5. Also refer to COursera's algorithms course.
* The approach to problems taken in an algorithms course is the same approach that works best in interviews. Studying algorithms will get you in an interview mindset.


## 3. Get help from your interviewer

> Interviewers help candidates. They give hints, they respond to ideas, and they generally guide the process. But they don't help all candidates equally. Some programmers are able to extract significant help, without the interviewer holding it against them. Others are judged harshly for any hints they are given. You want to be helped.

> This comes down to process and communication. If the interviewer likes your process and you communicate well with them, they will not mind helping. You can make this more likely by following a careful process. The steps I recommend are:

1. Ask questions.
2. Talk through a brute-force solution.
3. Talk through an optimized solution.
4. Write code.

> After you are asked an interview question, start by clarifying what was asked. THis is the time to be pedantic. Clarify every ambiguity you can think of. Ask about edge cases. Bring up specific examples of input, and make sure you are correct about the expected output. Ask questions even if you're almost sure you know the answers. This is useful because it gives you a chance to come up with edge cases and fully spec the problem (seeing how you handle edge-cases is one of the main things that interviewers look for when evaluating an interview), and also because it gives you a minute to collect your thoughts before you need to start solving the problem.

> Next, you should talk through the simplest brute-force solution to the problem that you can think of. YOu should talk, rather than jump right into coding, because you can move faster when talking, and it's more engaging for the interviewer. If the interviewer is engaged, they will step in and offer pointers. If you retreat into writing code, however, you will miss opportunity.

> Candidates often skip the brute-force step, assuming that the brute-force solution to the problem is too obvious, or wrong. This is a mistake. Make sure that you always give a solution to the problem you've been asked (even if it takes exponential time, or an NSA super computer). When you've described a brute-force solution, ask the interviewer if they would like you to implement it, or come up with a more efficient solution. Normally they will tell you to come up with a more efficient solution.

> The process for the more efficient solution is the same as for the brute force. Again talk, don't write code, and bounce ideas off of the interviewer. Hopefully, the question will be similar to something you've seen, and you'll know the answer. If that is not the case, it's useful to think of what problems you've seen that are most similar, and bring these up with the interviewer. Most interview questions are slightly-obscured applications of classic CS algorithms. The interviewer will often guide you to this algorithm, but only if you begin the process.

> Finally, after both you and your interviewer agree that you have a good solution, you should write your code. Depending on the company, this may be on a computer or a whiteboard. But because you've already come up with the solution, this should be fairly straightforward. For extra points, ask your interviewer if they would like you to write tests.

You should always get help from the interviewer, but the way you ask for help is very important. It is recommended that you...

  1. Ask questions
    * Clarify every ambiguity.
    * Ask about edge cases.
    * Bring up specific examples of input.
    * Make sure you are correct about the expected output.
    * (Interviewers are looking at how you handle edge-cases)
  2. Talk through a brute-force solution.
    * Dont' skip the brute-force solution, even though it is "too obvious", "wrong", or "inefficient".
    * Talk through the brute-force solution instead of coding it. Talking is faster than coding, and it's more engaging for the interviewer.
  3. Talk through an optimized solution.
    * After describing the brute-force solution, ask if the interviewer would like you to implement a more efficient solution (usually yes).
    * The question is most likely a slighty-obscured problem that can be solved by a classic CS algorithm.
    * The interviewer will often guide you to this algorithm, but only if you begin the process.
  4. Write code.
    * After you and the interviewer agree that you have a good solution, you should write your code.
  5. (Optional) Write tests.
    * If you have free time, ask the interviewer if they would like you to write tests.
    

## 6. Use a dynamic language, but mention C

> I recommend that you use a dynamic language like Python, Ruby or JavaScript during interviews. Of course, you should use whatever language you know best. But we find that many people try interviewing in C, C++ or Java, under the impression these are "real" programming languages. Several classic books on interviewing recommend that programmers choose Java or C++. At startups at least, we've found that this is bad advice. Candidates do better when using dynamic languages. This is true, I think, because of dynamic languages' compact syntax, flexible typing, and list and hash literals. They are permissive languages. This can be a liability when writing complex systems (a highly debatable point), but it's great when trying to cram binary search onto a whiteboard.

> No matter what language you use, it's helpful to mention work in other languages. An anti-pattern that companies screen against is people who only know one language. If you do only know one language, you have to rely on your strength in that language. But if you've done work or side-proejcts in multiple languages, be sure to bring this up when talking to your interviewers. If you have worked in lower-level languages like C, C++, Go, or Rust, talking about this will particularly help.

> Java, C#, and PHP are a problematic case. We've uncovered bias against these languages in startups. This is not fair, but it is the truth. If you have other options, I recommend against using these languages in interviews with startups.

* Prefer to use a dynamic language instead of a relatively lower-level one.
* Dynamic languages have more compact syntax, flexible typing, and list/hash literals.
* This makes writing solutions easier/more succinct.
* Mention you do know C/C++.


## 4. Talk about trade-offs

> Programming interviews are primarily made up of programming questions, and that is what I have talked about so far. However, you may also encounter system design questions. Companies seem to like these especially for more experienced candidates. In a system design question, the candidate is asked how he or she would design a complex real-world design. Examples include designing Google maps, designing a social network, or designing an API for a bank.

> The first observation is that answering system design questions requires some specific knowledge. Obviously no one actually expects you to design Google maps (that took a lot of people a long time). But they do expect you to have some insigh into aspects of such a design. The good news is that these questions usually focus on web backends, so you can make a lot of progress by reading about this area. An incomplete list of things to understand is:

* HTTP (at the protocol level)
* Databses (indexes, querry planning)
* CDNs
* Caching (LRU cache, memcached, redis)
* Load balancers
* Distributed worker systems

> You need to understand these concepts. But more importantly, you need to understand how they fit together to form real ssytems. The best way to learn this is to read about how other engineers have used the concepts. The blog "High Scalability" is a great resource for this. It publishes detailed write-ups of the back-end architecture at real companies. YOu can read about how every concept on the list above is used in real systems.

> Once you've done this reading, answering system design questions is a matter of process. Start at the highest level, and move downward. At each level, ask your interviewer for specifications (should you suggest a simple starting point, or talk about what a mature system might look like?) and talk about several options (applying the ideas from your reading). Discussing tradeoffs in your design is key. Your interviewer cares less about whether your design is good in itself and more about whether you are able to talk about the trade-offs (positives and negatives) of your decisions. Practice this.

* You will encounter system design questions. The candidate is asked how he/she would esign a complex real-world system.
* These questions usually focus on web backends. An incomplete list of things to understand is:
  * HTTP (protocol)
  * Databases (indexes, query planning)
  * CDNs
  * Caching (LRU, memcached, redis)
  * Load balancers
  * Distributed worker systems
* Understand how do these parts fit together to form real systems.
* The "High Scalability" blog is a great resource for this.
* There is a process to answering a system design question.
  * Start at the highest level, and move downward.
  * At each level, ask your interviewer for specifications (suggest a starting point, or talk about what a mature system would look like).
  * Talk about several options (applying the ideas from your reading).
  * Discuss tradeoffs in your design.
  * (The interviewer cares about whether or not you are capable of talking about trade-offs more than whether or not your design is good)


## 5. Highlight results

> The third type of question you may encounter is the experience question. This is where the interviewer asks you to talk about a programming project that you completed in the past. The mistake that many engineers make on this question is to talk about a technically interesting side-project. Many programmers choose to talk about implementing a neural network classifier, or writing a Twitter grammar bot. These are bad choices because it's very hard for the interviewer to judge their scope. Many candidates exaggerate simple side projects (sometimes that never actually worked), and the interviewer has no way to tell if you are doing this.

> The solution is to choose a project that produced results, and highlight the results. This often involves picking a less technically interesting project, but it's worth it. Think (ahead of time) of the programming you've done that had the largest real-world impact. If you've written an iOS game, and 50k people have downloaded it, the download number makes it a good option. If you've written an admin interface during an intership that was deployed to the entire admin staff, the deployment makes it a good thing to talk about. Selecting a practical project will also communicate to the company that you focus on actual work. Programmer too focused on interesting tech is an anti-pattern that companies screen against (these programmers are sometimes not productive).

* When asked about a previous project you worked on, instead of picking a very technically interesting project, choose a project that produced results, and highlight the results.
* Interviewers are more interested in project impact & scope.


## 1. Be enthusiastic

> Enthusiasm has a huge impact on interview results. About 50% of the Triplebyte candidates who fail interviews at companies fail for non-technical reasons. This is usually described by the company as a “poor culture fit”. Nine times out of ten, however, culture fit just means enthusiasm for what a company does. Companies want candidates who are excited about their mission. This carries as much weight at many companies as technical skill. This makes sense. Excited employees will be happier and work harder.

> The problem is that this can be faked. Some candidates manage to convince every company they talk to that it’s their dream job, while others (who are genuinely excited) fail to convince anyone. We’ve seen this again and again. The solution is for everyone to get better at showing their enthusiasm. This is not permission to lie. But interviewing is like dating. No one wants to be told on a first date that they are one option among many, even though this is usually the case. Similarly, most programmers just want a good job with a good paycheck. But stating this in an interview is a mistake. The best approach is to prepare notes before an interview about what you find exciting about the company, and bring this up with each interviewer when they ask if you have any questions. A good source of ideas is to read the company’s recent blog posts and press releases and note the ones you find exciting.

> This idea seems facile. I imagine you are nodding along as you read this. But (as anyone who has ever interviewed can tell you) a surprisingly small percentage of applicants do this. Carefully preparing notes on why you find a company exciting really will increase your pass rate. You can even reference the notes during the interview. Bringing prepared notes shows preparation.

* Convey enthusiasm for what the company does.
* Prepare notes before the interview about what you find exciting about the company, and bring this up with each interviewer when they ask if you have any questions. Read the company's recent blog posts and press releases and note the ones you find exciting.
* Reference the notes during the interview. Bringing prepared notes shows preparation.


## 7. Practice, practice, practice

> You can get much better at interviewing by practicing answering questions. This is true because interviews are stressful, but stress harms performance. The solution is practice. Interviewing becomes less stressful with exposure. This happens naturally with experience. Even within a single job search, we find that candidates often fail their initial interviews, and then pass more as their confidence builds. If stress is something you struggle with, I recommend that you jumpstart this process by practicing interview stress. Get a list of interview questions (the book _Cracking the Coding Interview_ is one good source) and solve them. Set a 20-minute timer on each question, and race to answer. PRactice writing the answers on a whiteboard (not all companies require this, but it's the worst case, so you should practice it). Pen & paper is a pretty good simulation of a whiteboard. If you have friends who can help you prepare, taking turns interviewing each other is great. Reading a lot of interview questions has the added benefit of providing you ideas to use when in actual interviews. A surprising number of questions are re-used (in full or in part).

> Even experienced (and stress-free) candidates will benefit from this. Interviewing is a fundamentally different skill from working as a programmer, and it can atrophy. But experience programmers often (reasonably) feel that they should not have to prepare for interviews. They study less. This is why junior candidates often actually do better on interview questions than experienced candidates. Companies know this, and, paradoxically, some tell us they set lower bars on the programming questions for experienced candidates.

* Interviews are stressful, and stress harms performance.
* The solution is practice. More exposure means less stress.
* Candidates usually fail their initial interviews, but they pass more afterwards as confidence builds.
* Deal with stress by practicing interview stress.
  * Get a list of interview questions and solve them.
  * Solve them in limited time.
  * Use a whiteboard, or simulate it with pen & paper.
  * If you have friends who can help you prepare, take turns interviewing each other.
  * Reading a lot of interview questions has the added benefit of providing you ideas to use when in actual interviews. A surprising number of questions are re-used.


## 8. Mention credentials

> Credentials bias interviewers. Triplebyte candidates who have worked at a top company or studied at a top school go on to pass interviews at a 30% higher rate than programmers who don't have these credentials (for a given level of performance on our credential-blind screen). I don't like this. It's not meritocratic and it sucks, but if you have these credentials, it's in your interest to make sure that your interviewers know this. You can't trust that they'll read your resume.

* Candidates who mention that they worked at a top company or studied at a top school go on to pass interviews at a significantly higher rate than those who don't have these credentials.


## 9. Line up offers

> If you've ever read fund-raising advice for founders, you'll know that getting the 1st VC to make an investment offer is the hardest part. Once you have one offer, more come pouring in. The same is true of job offers. If you already have an offer, be sure to mention this in interviews. Mentioning other offers in an interview heavily biases the interviewer in your favor.

> This brings up the strategy of making a list of the companies you're interested in, and setting up interviews in reverse order of interest. Doing well earlier in the process will increase your probability of getting an offer from your number one choice. You should do this.

* You are more likely to get an offer if you mention that you already have an offer.
* Make a list of companies you are interested in, and schedule interviews in reverse order of interest.
