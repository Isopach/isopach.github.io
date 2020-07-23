---
layout: post
title: Lichess Puzzle Solver 
published: true
categories: [Programming]
excerpt: In the study menu, there's this option called 'puzzles', where the player is presented a chess puzzle and will gain/lose rating points depending on whether it is correctly solved.
---

In the study menu, there's this option called 'puzzles', where the player is presented a chess puzzle and will gain/lose rating points depending on whether it is correctly solved.

---

# Lichess Puzzle Solver

This is a programming post. 

I recently got back into chess after an online friend challenged me to a game, and rediscovered this amazing site called [Lichess](https://lichess.org/). In the study menu, there's this option called 'puzzles', where the player is presented a chess puzzle and will gain/lose rating points depending on whether it is correctly solved.

As I got hooked, I wanted to find a way to play it at work without being so obvious with the big chessboard on display. So I inspected the page's source and discovered a JavaScript variable called `lichess_puzzle`, which holds the notation data as well as the answers!



<details>
  <summary>This is the data from one of the puzzles</summary>

<pre style="white-space: pre-wrap; background: hsl(255);"><code style="background: hsl(0,0,0); ">
lichess.puzzle = {"data":{"game":{"id":"RNvKedKc","perf":{"icon":"#","name":"Rapid"},"rated":true,"players":[{"userId":"khaled_ho","name":"Khaled_ho (2144)","color":"white"},{"userId":"kino512","name":"kino512 (1985)","color":"black"}],"treeParts":[{"ply":0,"fen":"rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1"},{"ply":1,"fen":"rnbqkbnr/pppppppp/8/8/4P3/8/PPPP1PPP/RNBQKBNR b KQkq - 0 1","id":"/?","uci":"e2e4","san":"e4","opening":{"eco":"B00","name":"King\u0027s Pawn"}},{"ply":2,"fen":"rnbqkbnr/pppp1ppp/8/4p3/4P3/8/PPPP1PPP/RNBQKBNR w KQkq - 0 2","id":"WG","uci":"e7e5","san":"e5","opening":{"eco":"C20","name":"King\u0027s Pawn Game"}},{"ply":3,"fen":"rnbqkbnr/pppp1ppp/8/4p3/4P3/5N2/PPPP1PPP/RNBQKB1R b KQkq - 1 2","id":")8","uci":"g1f3","san":"Nf3","opening":{"eco":"C40","name":"King\u0027s Knight Opening"}},{"ply":4,"fen":"rnbqkbnr/ppp2ppp/3p4/4p3/4P3/5N2/PPPP1PPP/RNBQKB1R w KQkq - 0 3","id":"VN","uci":"d7d6","san":"d6","opening":{"eco":"C41","name":"Philidor Defense"}},{"ply":5,"fen":"rnbqkbnr/ppp2ppp/3p4/4p3/3PP3/5N2/PPP2PPP/RNBQKB1R b KQkq - 0 3","id":".\u003e","uci":"d2d4","san":"d4","opening":{"eco":"C41","name":"Philidor Defense"}},{"ply":6,"fen":"rnbqkbnr/ppp2ppp/3p4/8/3pP3/5N2/PPP2PPP/RNBQKB1R w KQkq - 0 4","id":"G\u003e","uci":"e5d4","san":"exd4","opening":{"eco":"C41","name":"Philidor Defense: Exchange Variation"}},{"ply":7,"fen":"rnbqkbnr/ppp2ppp/3p4/8/3NP3/8/PPP2PPP/RNBQKB1R b KQkq - 0 4","id":"8\u003e","uci":"f3d4","san":"Nxd4","opening":{"eco":"C41","name":"Philidor Defense: Exchange Variation"}},{"ply":8,"fen":"rnbqkb1r/ppp2ppp/3p1n2/8/3NP3/8/PPP2PPP/RNBQKB1R w KQkq - 1 5","id":"aP","uci":"g8f6","san":"Nf6","opening":{"eco":"C41","name":"Philidor Defense: Exchange Variation"}},{"ply":9,"fen":"rnbqkb1r/ppp2ppp/3p1n2/8/3NP3/2N5/PPP2PPP/R1BQKB1R b KQkq - 2 5","id":"$5","uci":"b1c3","san":"Nc3"},{"ply":10,"fen":"rnbqk2r/ppp1bppp/3p1n2/8/3NP3/2N5/PPP2PPP/R1BQKB1R w KQkq - 3 6","id":"\u0060W","uci":"f8e7","san":"Be7"},{"ply":11,"fen":"rnbqk2r/ppp1bppp/3p1n2/8/3NP3/2NB4/PPP2PPP/R1BQK2R b KQkq - 4 6","id":"(6","uci":"f1d3","san":"Bd3"},{"ply":12,"fen":"rnbq1rk1/ppp1bppp/3p1n2/8/3NP3/2NB4/PPP2PPP/R1BQK2R w KQ - 5 7","id":"_b","uci":"e8h8","san":"O-O"},{"ply":13,"fen":"rnbq1rk1/ppp1bppp/3p1n2/8/3NP3/2NB4/PPP2PPP/R1BQ1RK1 b - - 6 7","id":"\u0027*","uci":"e1h1","san":"O-O"},{"ply":14,"fen":"r1bq1rk1/pppnbppp/3p1n2/8/3NP3/2NB4/PPP2PPP/R1BQ1RK1 w - - 7 8","id":"\u005cV","uci":"b8d7","san":"Nbd7"},{"ply":15,"fen":"r1bq1rk1/pppnbppp/3p1n2/8/3NP3/2NB3P/PPP2PP1/R1BQ1RK1 b - - 0 8","id":"2:","uci":"h2h3","san":"h3"},{"ply":16,"fen":"r1bq1rk1/ppp1bppp/3p1n2/2n5/3NP3/2NB3P/PPP2PP1/R1BQ1RK1 w - - 1 9","id":"VE","uci":"d7c5","san":"Nc5"},{"ply":17,"fen":"r1bq1rk1/ppp1bppp/3p1n2/2n3B1/3NP3/2NB3P/PPP2PP1/R2Q1RK1 b - - 2 9","id":"%I","uci":"c1g5","san":"Bg5"},{"ply":18,"fen":"r1bq1rk1/1pp1bppp/p2p1n2/2n3B1/3NP3/2NB3P/PPP2PP1/R2Q1RK1 w - - 0 10","id":"SK","uci":"a7a6","san":"a6"},{"ply":19,"fen":"r1bq1rk1/1pp1bppp/p2p1n2/2n3B1/3NPP2/2NB3P/PPP3P1/R2Q1RK1 b - - 0 10","id":"0@","uci":"f2f4","san":"f4"},{"ply":20,"fen":"r1bq1rk1/2p1bppp/p2p1n2/1pn3B1/3NPP2/2NB3P/PPP3P1/R2Q1RK1 w - - 0 11","id":"TD","uci":"b7b5","san":"b5"},{"ply":21,"fen":"r1bq1rk1/2p1bppp/p2p1n2/1pn3B1/3NPP2/P1NB3P/1PP3P1/R2Q1RK1 b - - 0 11","id":"+3","uci":"a2a3","san":"a3"},{"ply":22,"fen":"r2q1rk1/1bp1bppp/p2p1n2/1pn3B1/3NPP2/P1NB3P/1PP3P1/R2Q1RK1 w - - 1 12","id":"]T","uci":"c8b7","san":"Bb7"},{"ply":23,"fen":"r2q1rk1/1bp1bppp/p2p1n2/1pn3B1/1P1NPP2/P1NB3P/2P3P1/R2Q1RK1 b - - 0 12","id":",\u003c","uci":"b2b4","san":"b4"},{"ply":24,"fen":"r2q1rk1/1bp1bppp/p2p1n2/1p4B1/1P1NnP2/P1NB3P/2P3P1/R2Q1RK1 w - - 0 13","id":"E?","uci":"c5e4","san":"Ncxe4"},{"ply":25,"fen":"r2q1rk1/1bp1bppp/p2p1n2/1p4B1/1P1NNP2/P2B3P/2P3P1/R2Q1RK1 b - - 0 13","id":"5?","uci":"c3e4","san":"Nxe4"},{"ply":26,"fen":"r2q1rk1/1bp1bppp/p2p4/1p4B1/1P1NnP2/P2B3P/2P3P1/R2Q1RK1 w - - 0 14","id":"P?","uci":"f6e4","san":"Nxe4"},{"ply":27,"fen":"r2q1rk1/1bp1Bppp/p2p4/1p6/1P1NnP2/P2B3P/2P3P1/R2Q1RK1 b - - 0 14","id":"IW","uci":"g5e7","san":"Bxe7"},{"ply":28,"fen":"r4rk1/1bp1qppp/p2p4/1p6/1P1NnP2/P2B3P/2P3P1/R2Q1RK1 w - - 0 15","id":"^W","uci":"d8e7","san":"Qxe7"},{"ply":29,"fen":"r4rk1/1bp1qppp/p2p4/1p6/1P1NnP2/P2B3P/2P3P1/R2QR1K1 b - - 1 15","id":"(\u0027","uci":"f1e1","san":"Re1"},{"ply":30,"fen":"r4rk1/1bp1q1pp/p2p4/1p3p2/1P1NnP2/P2B3P/2P3P1/R2QR1K1 w - - 0 16","id":"XH","uci":"f7f5","san":"f5"},{"ply":31,"fen":"r4rk1/1bp1q1pp/p2p4/1p3p2/1PPNnP2/P2B3P/6P1/R2QR1K1 b - - 0 16","id":"-=","uci":"c2c4","san":"c4"},{"ply":32,"fen":"r4rk1/1b2q1pp/p2p4/1pp2p2/1PPNnP2/P2B3P/6P1/R2QR1K1 w - - 0 17","id":"UE","uci":"c7c5","san":"c5"},{"ply":33,"fen":"r4rk1/1b2q1pp/p2p4/1pP2p2/2PNnP2/P2B3P/6P1/R2QR1K1 b - - 0 17","id":"\u003cE","uci":"b4c5","san":"bxc5"},{"ply":34,"fen":"r4rk1/1b2q1pp/p7/1pp2p2/2PNnP2/P2B3P/6P1/R2QR1K1 w - - 0 18","id":"NE","uci":"d6c5","san":"dxc5"},{"ply":35,"fen":"r4rk1/1b2q1pp/p7/1pp2p2/2P1nP2/P2B1N1P/6P1/R2QR1K1 b - - 1 18","id":"\u003e8","uci":"d4f3","san":"Nf3"},{"ply":36,"fen":"3r1rk1/1b2q1pp/p7/1pp2p2/2P1nP2/P2B1N1P/6P1/R2QR1K1 w - - 2 19","id":"[^","uci":"a8d8","san":"Rad8"},{"ply":37,"fen":"3r1rk1/1b2q1pp/p7/1Pp2p2/4nP2/P2B1N1P/6P1/R2QR1K1 b - - 0 19","id":"=D","uci":"c4b5","san":"cxb5"},{"ply":38,"fen":"3r1rk1/1b2q1pp/8/1pp2p2/4nP2/P2B1N1P/6P1/R2QR1K1 w - - 0 20","id":"KD","uci":"a6b5","san":"axb5"},{"ply":39,"fen":"3r1rk1/1b2q1pp/8/1pp2p2/4nP2/P2B1N1P/4Q1P1/R3R1K1 b - - 1 20","id":"\u0026/","uci":"d1e2","san":"Qe2"},{"ply":40,"fen":"3r1rk1/1b2q1pp/8/1p3p2/2p1nP2/P2B1N1P/4Q1P1/R3R1K1 w - - 0 21","id":"E=","uci":"c5c4","san":"c4"},{"ply":41,"fen":"3r1rk1/1b2q1pp/8/1p3p2/2p1BP2/P4N1P/4Q1P1/R3R1K1 b - - 0 21","id":"6?","uci":"d3e4","san":"Bxe4"},{"ply":42,"fen":"3r1rk1/4q1pp/8/1p3p2/2p1bP2/P4N1P/4Q1P1/R3R1K1 w - - 0 22","id":"T?","uci":"b7e4","san":"Bxe4"},{"ply":43,"fen":"3r1rk1/4q1pp/8/1p3pN1/2p1bP2/P6P/4Q1P1/R3R1K1 b - - 1 22","id":"8I","uci":"f3g5","san":"Ng5"},{"ply":44,"fen":"3r1rk1/6pp/8/1pq2pN1/2p1bP2/P6P/4Q1P1/R3R1K1 w - - 2 23","id":"WE","uci":"e7c5","san":"Qc5+","check":true},{"ply":45,"fen":"3r1rk1/6pp/8/1pq2pN1/2p1bP2/P6P/4Q1PK/R3R3 b - - 3 23","id":")2","uci":"g1h2","san":"Kh2"},{"ply":46,"fen":"4rrk1/6pp/8/1pq2pN1/2p1bP2/P6P/4Q1PK/R3R3 w - - 4 24","id":"^_","uci":"d8e8","san":"Rde8"}],"clock":"10+0"},"puzzle":{"id":62083,"rating":1797,"attempts":148718,"fen":"3r1rk1/6pp/8/1pq2pN1/2p1bP2/P6P/4Q1PK/R3R3 b - - 3 23","color":"white","initialPly":46,"gameId":"RNvKedKc","lines":{"e2h5":{"h7h6":{"g5e4":{"c5b6":"win"}}}},"vote":2631,"branch":{"ply":47,"fen":"4rrk1/6pp/8/1pq2pNQ/2p1bP2/P6P/6PK/R3R3 b - - 1 24","id":"/J","uci":"e2h5","san":"Qh5","children":[{"ply":48,"fen":"4rrk1/6p1/7p/1pq2pNQ/2p1bP2/P6P/6PK/R3R3 w - - 0 25","id":"ZR","uci":"h7h6","san":"h6","children":[{"ply":49,"fen":"4rrk1/6p1/7p/1pq2p1Q/2p1NP2/P6P/6PK/R3R3 b - - 0 25","id":"I?","uci":"g5e4","san":"Nxe4","children":[]}]}]},"enabled":true},"mode":"play","user":{"rating":2097,"recent":[[62053,9,2010],[62055,-19,2019],[62056,-11,2000],[62057,6,1989],[62061,13,1995],[62062,5,2008],[62063,11,2013],[62064,12,2024],[62065,13,2036],[62066,5,2049],[62073,10,2054],[62074,9,2064],[62077,14,2073],[62080,7,2087],[62083,3,2094]]}},"pref":{"blindfold":0,"coords":2,"rookCastle":0,"animation":{"duration":125.0},"destination":true,"resizeHandle":1,"moveEvent":2,"highlight":true,"is3d":false},"i18n":{"training":"Puzzles","yourPuzzleRatingX":"Your puzzle rating: %s","goodMove":"Good move","butYouCanDoBetter":"But you can do better.","bestMove":"Best move!","keepGoing":"Keep going\u2026","puzzleFailed":"Puzzle failed","butYouCanKeepTrying":"But you can keep trying.","yourTurn":"Your turn","findTheBestMoveForBlack":"Find the best move for black.","findTheBestMoveForWhite":"Find the best move for white.","viewTheSolution":"View the solution","success":"Success","fromGameLink":"From game %s","boardEditor":"Board editor","continueFromHere":"Continue from here","playWithTheMachine":"Play with the computer","playWithAFriend":"Play with a friend","wasThisPuzzleAnyGood":"Was this puzzle any good?","pleaseVotePuzzle":"Help lichess improve by voting, using the up or down arrow:","thankYou":"Thank you!","puzzleId":"Puzzle %s","ratingX":"Rating: %s","playedXTimes:one":"Played %s time","playedXTimes":"Played %s times","continueTraining":"Continue training","retryThisPuzzle":"Retry this puzzle","toTrackYourProgress":"To track your progress:","signUp":"Register","trainingSignupExplanation":"Lichess will provide puzzles that match your ability, making for better training sessions.","thisPuzzleIsCorrect":"This puzzle is correct and interesting","thisPuzzleIsWrong":"This puzzle is wrong or boring","puzzles":"Puzzles","analysis":"Analysis board","rated":"Rated","casual":"Casual","depthX":"Depth %s","usingServerAnalysis":"Using server analysis","loadingEngine":"Loading engine ...","cloudAnalysis":"Cloud analysis","goDeeper":"Go deeper","showThreat":"Show threat","gameOver":"Game Over","inLocalBrowser":"in local browser","toggleLocalEvaluation":"Toggle local evaluation"}}
</code></pre>
</details>

Looks like simple json right? You're right! 

This data will always be returned on `line 7` of the code, along with another script. All we have to do is to write a parser that returns the answers in a way easy for our eyes.

## The Code

```python
import requests, json, sys, re

url = sys.argv[1]
r = requests.get(url)
for line_number, line in enumerate(r.text.splitlines()):
    if line_number == 7:
        # Removes unnecessary parts of data
        puzzle = line[17:-24]
    elif line_number > 7:
        break

result = re.search("(.*)</", puzzle) # Truncates data after </script>
puzzlej = result.group(1)
d = json.loads(puzzlej)
moves = d["data"]["puzzle"]["lines"]
print(moves)
```

When run with a puzzle url in the argument, it will return something like this:

`{'e2h5': {'h7h6': {'g5e4': {'c5b6': 'win'}}}}`

Just take every other key starting from the first and you'll get 100% correct answers in no time!

Maybe the ~~devs~~ dev shouldn't have put the answers in a script on the page, huh? 