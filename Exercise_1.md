Here is an example of a csv quiz file that we will be using in our program:

```csv
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
```

Each line has the question as the first field and the answer as the second field. In the first part, our program will read this file, present the questions from it to the end user, and check the correctness of the user's inputs using the answers.

We create a file named *main.go* in the directory *src/github.com/gophercises/quiz/* in our GOPATH.

A reasonable default for the quiz file name is *problems.csv*. But we shall allow the user to enter another name as an option. For this we shall be using the `flag` package. We start our program by assigning a variable the csv file name.


```Go
package main

import "flag"

func main() {
  csvFilename := flag.String("csv", "problems.csv", "a csv file in the format of 'question,answer'")
  flag.Parse()
  _ = csvFilename
}
```

To test our program so far we compile it using,
```bash
$ go build .
```
When we run the program as
```bash
$ ./quiz
```
nothing happens.

However, when we run the program as `./quiz -h` or `./quiz --help` we get
```
Usage of ./quiz:
  -csv string
        a csv file in the format of 'question,answer' (default "problems.csv")
```

Here we see the options the program accepts. The `flag` package is providing this functionality for us. When we simply provide a binary to the user, this is how he can find out how to run the program.

Next we try to read the csv file whose name the user has provided as an option.
```Go
package main

import (
  "flag"
  "fmt"
  "os"
)

func main() {
  csvFilename := flag.String("csv", "problems.csv", "a csv file in the format of 'question,answer'")
  flag.Parse()
  
  file, err := os.Open(*csvFilename)
  if err != nil {
    fmt.Printf("Failed to open the CSV file: %s\n", *csvFilename)
    os.Exit(1)
  }
  _ = file
}
```
Remember that `flag.String` returns a pointer. This is why we have to dereference the pointer in the call to `os.Open`. If there is an  error we print out the name of the file that caused the error and exit the program with a non-zero exit code.

Running the program so far gives,
```bash
$ go build . && ./quiz -csv=abc.csv
Failed to open the CSV file: abc.csv
```

We can clearly see the file name that caused the error. In particular, if the file name had a space in it, the user would have been made aware of the fact that he has to enter the file name within quotes.

```bash
$ go build . && ./quiz -csv=ab cd.csv
Failed to open the CSV file: ab
```

```bash
$ go build . && ./quiz -csv="ab cd.csv"
Failed to open the CSV file: ab cd.csv
```

A small variation of the above would be to create a separate function to print out the error and exit the program:

```Go
package main

import (
  "flag"
  "fmt"
  "os"
)

func main() {
  csvFilename := flag.String("csv", "problems.csv", "a csv file in the format of 'question,answer'")
  flag.Parse()
  
  file, err := os.Open(*csvFilename)
  if err != nil {
    exit(fmt.Sprintf("Failed to open the CSV file: %s\n", *csvFilename))
  }
  _ = file
}

func exit(msg string) {
  fmt.Println(msg)
  os.Exit(1)
}
```

Next, we use the `csv` package to create a csv reader. The `csv` package has a `NewReader` function that takes an `io.Reader` as an argument and returns a csv reader. Since `file` is an `io.Reader` we can readily pass it in.

The [io.Reader](https://golang.org/pkg/io/#Reader) and [io.Writer](https://golang.org/pkg/io/#Writer) are some of the most often used interfaces in Go. That is because they are very simple and have one method each. The `Reader` interface makes it easy to read not just from files but from strings, memory buffers, and byte slices as well. Similarly the `http.ResponseWriter` in the `net/http` package implements the `Writer` interface and this would allow us to use `fmt.Fprintf` to print to it if we wanted.

Since we expect the csv file to be reasonably small we read it all at once. If there are no errors we print out the lines for now.

```Go
package main

import (
  "flag"
  "fmt"
  "os"
  "csv"
)

func main() {
  csvFilename := flag.String("csv", "problems.csv", "a csv file in the format of 'question,answer'")
  flag.Parse()
  
  file, err := os.Open(*csvFilename)
  if err != nil {
    exit(fmt.Sprintf("Failed to open the CSV file: %s\n", *csvFilename))
  }
  r := csv.NewReader(file)
  // Read all the lines at once
  lines, err := r.ReadAll()
  if err != nil {
    exit("Failed to parse the provided CSV file.")
  }
  fmt.Println(lines)
}

func exit(msg string) {
  fmt.Println(msg)
  os.Exit(1)
}
```
```bash
$ go build . && ./quiz -csv=problems.csv
[[5+5 10] [1+1 2] [8+3 11] [1+2 3] [8+6 14] [3+1 4] [1+4 5] [5+1 6] [2+3 5] [3+3 6] [2+4 6] [5+2 7]]
```

We can see that we get a 2D slice printed out where each inner slice has two parts, a question and its answer.

We are now going to make a dedicated type for a problem. In the above, the question and answer for a problem were stored in a slice. But if we make a dedicated type for the problem we can have other ways in which we input the problems to the quiz.

```Go
// ...No change here

func main() {

// ...No change here

}

type problem struct {
  q string
  a string
}

// ...No change here
```
