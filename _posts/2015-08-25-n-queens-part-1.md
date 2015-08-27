---
layout:     post
title:      Unpacking N-Queens (Part 1)
date:       2015-08-25 20:28:30
summary:    Understanding Ruan Pethiyagoda's N-Queens Implementation
categories: n-queens javascript bitwise
---

I recently came across [this tweet](https://twitter.com/ruanpethiyagoda/status/332992908565295104) by Hack Reactor’s Ruan Pethiyagoda.

N-Queens (check out a description of the problem [here](https://en.wikipedia.org/wiki/N_queens), solved, bitwise, in JavaScript, in so few characters he still had room left-over for a comment shout-out. Impressive! Intimidating! ...indecipherable?

Well, let’s unpack it and find out. With a few judicious line breaks, we get the much-improved function below:

    function N(Q,u,ee,n,s,H,R) {
      s=0;
      Q=u?Q:(1<<Q)-1;
      H=~(u|ee|n)&Q;
      while(H)H^=R=-H&H,s+=N(Q,(u|R)<<1,ee|R,(n|R)>>1);
      return s+=ee==Q
    }

With a few variable renames and mild structural tweaks, we get this far cleaner code:

    function N(done,leftDiagonal,cols,rightDiagonal,numSolutions,poss,bit) {
      numSolutions = 0;
      done = leftDiagonal ? done : (1<<done)-1;
      poss = ~(leftDiagonal|cols|rightDiagonal) & done;
      while (poss) {
        var bit = -poss * poss;
        poss = poss ^ bit;
        numSolutions += N(done,(leftDiagonal|bit)<<1, cols|bit,(rightDiagonal|bit)>>1);
      }
      return numSolutions += (cols == done);
    }

Much better! Now let’s take a look at what it’s doing.

###numSolutions

Overall, the function is trying to count the number of solutions to the N-Queens problem. This variable numSolutions is initialized at 0, incremented whenever we find a solution, and eventually returned. So far so good.

###cols

cols is also pretty straightforward: its bits represent the columns of our matrix. If the bit is one, Ruan has successfully placed a queen in that column. When we first call the function, as he’s placed no queens, cols is undefined.

###done

The next variable, ‘done’, is a bit more tricky. It initially represents the size of the chessboard. So for an 8×8 chessboard, ‘done’ initially stores 8. However, it’s used within the function in a different way. The first time the function is invoked, leftDiagonal (and all other variables following numSolutions) will be undefined. As a result, ‘done’ will be converted from a variable representing the size of the matrix into a variable whose binary representation can be used to represent a completed matrix state. As an example, if we invoke N(8), ‘done’ will be set to 0000000000000000000011111111, or 255. Each ‘1’ in our binary representation of ‘done’ represents a placed queen.

Ruan uses this pattern in a couple of ways. First off, it represents a completed matrix. When all the queens are placed in the matrix, cols will equal done, and he can increment his solution counter. Additionally, it can serve as a truncation tool. He can truncate any variable to only its last N digits (for this problem, the only digits he cares about) simply by using `variable = variable & done`. Any bits prior to the last N will be zeroed out.

This is exactly the pattern Ruan uses in the next line, but this post is long enough already! Check in later for Part 2, which will cover leftDiagonal, rightDiagonal, poss and bit.

**Update**: check out part 2 [here](http://www.natewillard.com/n-queens/javascript/bitwise/2015/08/26/n-queens-part-2/).
