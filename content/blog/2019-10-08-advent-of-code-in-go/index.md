---
title: "Creating an Advent of Code Application Using Go Plugins"
author: ""
type: ""
date: 2019-10-08T17:53:49-04:00
subtitle: ""
image: ""
tags: 
    - go
    - advent of code
    - bash
    - puzzle
summary: "I've always enjoyed solving puzzles, and love coding puzzles. This post walks through how
I have set up my Advent of Code project using Go Plugins for different days' solutions. No spoilers!"
---

The best way to learn programming is to write code. The advice is almost always "build something,"
but it can be hard to figure out what a good "something" would be, especially for beginners. In my
mind, that's where coding puzzles come in. Some of my favorites are
[Advent of Code](https://adventofcode.com), [Project Euler](https://www.projecteuler.net) (for the
more mathematically inclined) and [Rosalind](http://rosalind.info/problems/locations) (focused on
bioinformatics). I've spent a fair bit of time working through Advent of Code puzzles in a few
different languages -- R, Python, Julia, and Go so far. I can't say that I've completed all 25
challenges for a year, nor that I've done all four of those languages for every puzzle that I've
completed. But I've gotten a tremendous amount of enjoyment and satisfaction from doing the puzzles
that I have worked on, and I've learned a ton.

In this post, I'm going to talk about how I have structured my Go solutions (later posts will get
into the puzzle solutions themselves in detail). I really wanted to embed my solutions in a single
application and have everything runnable through a common command. I found the Go
[Plugin package](https://golang.org/pkg/plugin) to be just about the best solution for doing this,
though not without its challenges. Most things I read about the plugin package said that it wasn't
mature enough, or that it's too clunky to work well. While it definitely is a little clunky, I think
it does alright for this job.

The main challenge that I was facing was how to dynamically include different Go source files for
execution from a main program. I wanted to specify at runtime which package to import and run, which
is not possible as far as I know. I didn't want to run the solution code, write the answers to disk,
and then have the main program just read in those answers. The other solution that I had come across
was to create a map with dates pointing to solution functions. I definitely thought about how I
might do that, but decided that manually importing every solution file and filling in the map with
the functions from each file was just too much of a pain, and not a very elegant solution.

## Go Plugins

So what exactly is a Go plugin? From the package docs:

> A plugin is a Go main package with exported functions and variables that have been built with:

```
go build -buildmode=plugin
```

Importantly, even though a Go plugin is a `main` package, *the `main()` function is not run*. This
is nice because it can allow you to export functionality from a `main` package that you may have
written for some other purpose. But it is a pain because Go will not compile a `main` package
without a `main()` function, so you still need one even if you're creating a package specifically to
be a plugin, and you know you won't actually use `main()`. The other irritating part is that you
have to provide type information to the caller program about the entities from the plugin that you
want to use. I was able to make this work by having every plugin implement a function with the
signature `func Solve(fname string)`, which takes the path to the problem input as its argument,
runs the solution functions, and prints the answers. That way, I can have the same code in my main
program regardless of the intermediate steps involved in any given day's solution.

## Program Structure

*You can find the full source code on [GitHub](https://github.com/stephenfeagin/Go-AdventOfCode)*.

So given this brief overview of Go plugins, how does it work in practice? The project has this
overall file structure:

```
.
|__ AOC.go
|__ puzzles/
|  |__ 2018/
|  |  |__ 01/
|  |  |  |__ input.txt
|  |  |  |__ main.go
```

`AOC.go` is the main Advent of Code program file. It includes the code to load plugins and execute
the solution function from those plugins. The `puzzles/` directory contains sub-directories for each
day's puzzle, where `input.txt` is the puzzle input (generated for each unique user of
adventofcode.com) and `main.go` contains the source code for the solutions.

As mentioned above, all of the `main.go` solution files contain a function with the signature
`func Solve(fname string)`. Beyond that, they also generally contain a
`func readInput(fname string)` as well as functions for the different parts of the puzzle, taking in
the data structure(s) produced by `readInput()`. They also all contain a blank `func main(){}` so
that they can be compiled as `main` packages, as required by the `plugin` package. For any puzzle
solution, we have to compile it as a plugin to provide a `.so` file with a predictable name that the
`AOC.go` file can locate and access. To do this, we run:

```bash
$ cd puzzles/$YEAR/$DAY
$ go build -buildmode=plugin -o $YEAR$DAY.so
```

`AOC.go` is the most interesting file as far as the plugin system is concerned. After reading in the
year and day for the desired puzzle, it has to attempt to locate a solution directory and `.so`
(shared object) file for that date:

```go
import (
    "fmt"
    "os"
    "path/filepath"
    "plugin"
)

dir := filepath.Join("puzzles", year, day)
// Check for solution directory
if s, err := os.Stat(dir); os.IsNotExist(err) || !s.IsDir() {
    fmt.Println("No solution available for", year, day)
    os.Exit(1)
}
// Check for .so file
pluginPath := filepath.Join(dir, year+day+".so")
p, err := plugin.Open(pluginPath)
if err != nil {
    fmt.Printf("No such file %s.so\n", year+day)
    os.Exit(1)
}
```

Assuming that our program survives through that point (i.e. we have the correct subdirectory inside
the `puzzles/` directory and that we have compiled it with `-buildmode=plugin` to create a `.so`
file), we then need to find the `Solve()` function and run it. We need to know the name of the
symbol that we're looking up, as well as its type/signature. 

```go
symbol, err := p.Lookup("Solve")
if err != nil {
    fmt.Println(err.Error())
    os.Exit(1)
}

solve, ok := symbol.(func(string))
if !ok {
    fmt.Println("Plugin has no 'Solve' function")
    os.Exit(1)
}
```

All that's left is to provide the path to the puzzle's input file and run the `Solve()` function.

```go
inputFile := filepath.Join(dir, "input.txt")
solve(inputFile)
```

I have set things up so that the `Solve()` functions print out the answers, rather than returning
them as a string. So once we call `solve()` in the parent program, we're all done! Running from the
command line is as easy as

```bash
$ go build -o AOC
$ ./AOC 2018 1
```

## Helper Scripts

It would definitely get tedious to create new solution directories by hand, write the solution code,
then compile it with the `-buildmode=plugin` flag by hand. Especially when it turns out that the
solution needs to be fixed or refactored, and then needs to be compiled yet again. So I created a
couple of different helper scripts to make these tasks a little easier.

### `new_puzzle.sh`

When I want to start working on a new solution, I like having a template for the files ready to go.
In the project root, I have a directory called `template` containing `input.txt.templ`, which is
blank (the input is taken from the AOC website), and `main.go.templ` which contains a skeleton file
with `func Solve(fname string)`, `func ReadInput(fname string)`, `func Part1()`, and `func Part2()`,
as well as the empty `func main(){}`. `new_puzzle.sh` copies the basic templates into a new
directory for the year and day that I want to solve. After doing some input validation (which I
won't get into here), the main part of the script creates the directory, creates a basic
`README.md`, and copies over a couple of templates for the source code and the input file.

```bash
YEAR=$1
DAY=$2

DIR="puzzles/$YEAR/$DAY"
mkdir -p $DIR
cp template/input.txt.templ $DIR/input.txt
cp template/main.go.templ $DIR/main.go

README="# [$YEAR Day $DAY:](https://adventofcode.com/$YEAR/day/$DAY)\n\n"
echo -e $README > $DIR/README.md
```

### `build_plugin.sh`

It's also nice to be able to build a plugin from the project root, without worrying about changing
directories, making sure the `.so` file is in the right place, and typing everything out by hand.
Once again skipping over input validation in this post, the script is fairly straightforward:

```bash
YEAR=$1
DAY=$2
DIR="puzzles/$YEAR/$DAY"

if [[ -d $DIR ]]; then
    cd $DIR
    go build -buildmode=plugin -o $YEAR$DAY.so
else
    echo "$DIR does not exist"
    exit 1
fi
```

### `build_all_plugins.sh`

Finally, it's sometimes helpful to be able to compile every plugin you've got, instead of just one
at a time. This is especially useful after cloning the git repository, since the `.so` files are
ignored. This was a great opportunity for me to learn more about control flow in bash. Again, it's
not a very complex script, but I find it incredibly useful.

```bash
cd puzzles

# every subdir in puzzles/ is a year
for YEAR in *; do
    # cd into the year's dir
    cd $YEAR
    echo "$YEAR"

    # every subdir in puzzles/$YEAR/ is a day
    for DAY in *; do
        # cd into day's dir
        cd $DAY
        echo -e "\t$DAY"

        # if there's a main.go file, build it
        if [[ -f main.go ]]; then
            go build -buildmode=plugin -o $YEAR$DAY.so
        fi

        # cd back into the year dir
        cd ..
    done
done
```

## Conclusion

Go's plugin system does have a lot of restrictions -- it's only available on MacOS and Linux, it
requires the plugin and the caller program to be compiled with the exact same version of Go and of
any dependencies that are be imported in both, etc. But if you are working solo or if the plugins
and the calling program will be built on the same machine at roughly the same time, it's not a
bad solution. My Advent of Code use case is obviously not the most complex or technically demanding
application, so I can't speak to the usefulness of Go plugins in other contexts. But I've enjoyed
working with them and have learned a lot from the experience. If it looks like the plugin package
might be useful for you, I'd strongly encourage you to give it a try!

## Resources

- [Advent of Code](https://adventofcode.com)
- [GitHub Repository](https://github.com/stephenfeagin/Go-AdventOfCode) for my Advent of Code project
- Go [Plugin package documentation](https://golang.org/pkg/plugin/)
- Vladimir Viven, ["Writing Modular Go Programs with Plugins"](https://medium.com/learning-the-go-programming-language/writing-modular-go-programs-with-plugins-ec46381ee1a9)
