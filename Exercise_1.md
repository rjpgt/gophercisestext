Here is an example of a csv quiz file that we will be using in our program:

5+5,10
1+1,2
8+3,11
1+2,3
8+6,14
3+1,4
1+4,5
5+1,6
2+3,5
3+3,6
3+3,6
5+2,7

Each line has the question as the first field and the answer as the second field. In the first part, our program will read this file, present the questions from it to the end user, and check the correctness of the user's inputs using the answers.

The default name of the quiz file will be *problems.csv*. But we shall allow the user to enter another name as an option. For this we shall be using the `flags` package. We start our program by assigning a variable the csv file name.
```Go
package main

import "flag"

func main() {
  csvFilename := flag.String("csv", "problems.csv", "a csv file in the format of 'question,answer'")
  flag.Parse()
  _ = csvFilename
}
```
