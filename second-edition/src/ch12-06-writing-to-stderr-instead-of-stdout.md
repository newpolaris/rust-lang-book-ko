## 표준출력 대신 표준에러로 에러메시지 출력하기 

지금까지 우리는 모든 출력을 `println!`을 사용하여 터미널에 출력했습니다. 대부분의 터미널은 두 가지 방식의 출력
을 지원합니다: *표준 출력* (`stdout`)은 일반적인 정보전달용이고 *표준 에러* (`stderr`)는 에러 메시지용
입니다. 이렇게 구분지음으로 인해 사용자는 프로그램의 출력을 직접 파일에 작성하면서도 여전히 에러메시지를 화면에 출력할 
수 있습니다. 

`println!` 함수는 오직 표준출력만 사용할 수 있으므로, 우리는 표준에러에 출력을 위한 다른 것을 알아보겠습니다.

### 에러가 어디에 출력될지 검사 

먼저, `minigrep`의 출력 내용이 어떻게 표준출력에 작성되는지를 후에 우리가 표준에러로 바꾸려는 에러메시지를 염두하며
살펴봅시다. 에러가 발생할 것을 인지한채로 우리는 표준출력 스트림을 파일로 변경하고자 합니다. 표준에러 스트림은 변경하지
않을 것이므로, 표준에러로 보내진 모든 출력내용은 화면에 표시될 겁니다. 

커맨드라인 프로그램들은 에러메시지들이 표준에러로 전달되는 것을 상정하고 있기 때문에 표준출력 스트림을 파일로 변경하더라도
우리는 에러메시지가 출력되는 것을 여전히 볼 수 있습니다. 우리 프로그램은 정상 동작하고 있지 않습니다 : 오류메시지 출력이
파일로 저장되고 있거든요!

이런 동작을 시연하는 방법은 프로그램의 실행시킬때 `>`과 표준출력 스트림을 향하게 할 파일이름을 주면 됩니다. 에러가 발생할
여지가 있는 인자는 주지 않습니다. 

```text
$ cargo run > output.txt
```

`>` 문법은 쉘에게 표준출력의 내용을 화면이 아닌 *output.txt*에 출력하게끔 하는 것입니다. 우리가 기대했던 
에러에시지의 화면 출력은 보지 못했으니 이것은 파일 마지막에 기록됐을 겁니다. 다음인 *output.txt*의 내용입
니다.

```text
Problem parsing arguments: not enough arguments
```

역시, 우리의 에러메시지는 표준출력으로 출력되었네요. 이런 에러메시지가 표준에러로 출력된다면 훨씬 유용하고 우리가 같은
방법으로 표준출력을 변경했을때 오직 성공적 실행에 관련된 데이터만 저장할 수 있게 될 겁니다. 지금 바꿔봅시다.

### 에러를 표준에러로 출력하기 

우리는 항목 12-24의 코드를 출력되는 에러메시지들을 변경하는데 사용하고자 합니다. 이번 장 진입부에서 리팩토링한 결과
모든 에러메시지는 하나의 함수 `main`에서 출력되고 있습니다. 표준라이브러리에 존재하는 표준에러에 출력해주는 매크로 
`eprintln!`를 사용하여 `println!`을 사용하여 에러를 출력하던 두 부분을 `eprintln!`을 사용하도록 변경
해봅시다. 

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);

        process::exit(1);
    }
}
```

<span class="caption">항목 12-24: 표준출력에 에러메시지를 출력하던 것을 `eprintln!`을
사용하여 표준에러로 변경하기 </span>

`println!`을 `eprintln!`으로 변경한 후에, 같은 방식으로 `>`을 사용해 표준출력을 변경하는 것 외에 다른 
인자를 주지 않고 프로그램을 다시 실행시켜 봅시다. 

```text
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

이제 우리는 에러를 화면에서 볼 수 있고, 우리가 커맨드라인 프로그램에서 기대한 대로 *output.txt*는 비어있습니다.

이번에는 에러를 발생시키지 않게 인자와 함께 프로그램을 실행시키면서 표준출력을 파일로 변경해봅시다. 

```text
$ cargo run to poem.txt > output.txt
```

터미널에는 아무것도 출력되지 않고, *output.txt*가 보관하게 됩니다. 

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

이번 시연은 우리가 표준출력에 성공적출력을 표준에러에 에러출력을 의도한 대로 수행하고 있음을 보여줍니다.

## 종합 

이번 장에서는 지금까지 우리가 배웠던 몇 가지 주요 개념을 되짚어보고 Rust 문법에서 범용 I/O 작업수행을 하는 방법을
알아봤습니다. 커맨드라인 인자, 파일, 환경변수, 그리고 `eprintln!`매크로로 에러출력를 사용하여 당신은 이제 
커맨드라인 응용프로그램을 작성할 준비가 됐습니다. 이전 장들의 개념을 활용하여, 당신의 코드는 잘 구조화되고, 적합한 데이터
구조를 사용하여 효율적으로 데이터를 저장하며, 에러를 보기좋게 관리하며, 잘 테스트 할 수 있게 됐습니다.

다음으로, 우리는 함수형 언어의 영향을 받은 Rust의 기능 몇가지를 알아보겠습니다 : 클로저와 반복자. 

<!-- 업데이트된 원본:
## Writing Error Messages to Standard Error Instead of Standard Output

At the moment, we’re writing all of our output to the terminal using the
`println!` function. Most terminals provide two kinds of output: *standard
output* (`stdout`) for general information and *standard error* (`stderr`)
for error messages. This distinction enables users to choose to direct the
successful output of a program to a file but still print error messages to the
screen.

The `println!` function is only capable of printing to standard output, so we
have to use something else to print to standard error.

### Checking Where Errors Are Written

First, let’s observe how the content printed by `minigrep` is currently being
written to standard output, including any error messages we want to write to
standard error instead. We’ll do that by redirecting the standard output stream
to a file while also intentionally causing an error. We won’t redirect the
standard error stream, so any content sent to standard error will continue to
display on the screen.

Command line programs are expected to send error messages to the standard error
stream so we can still see error messages on the screen even if we redirect the
standard output stream to a file. Our program is not currently well-behaved:
we’re about to see that it saves the error message output to a file instead!

The way to demonstrate this behavior is by running the program with `>` and the
filename, *output.txt*, that we want to redirect the standard output stream to.
We won’t pass any arguments, which should cause an error:

```text
$ cargo run > output.txt
```

The `>` syntax tells the shell to write the contents of standard output to
*output.txt* instead of the screen. We didn’t see the error message we were
expecting printed to the screen, so that means it must have ended up in the
file. This is what *output.txt* contains:

```text
Problem parsing arguments: not enough arguments
```

Yup, our error message is being printed to standard output. It’s much more
useful for error messages like this to be printed to standard error so only
data from a successful run ends up in the file. We’ll change that.

### Printing Errors to Standard Error

We’ll use the code in Listing 12-24 to change how error messages are printed.
Because of the refactoring we did earlier in this chapter, all the code that
prints error messages is in one function, `main`. The standard library provides
the `eprintln!` macro that prints to the standard error stream, so let’s change
the two places we were calling `println!` to print errors to use `eprintln!`
instead.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);

        process::exit(1);
    }
}
```

<span class="caption">Listing 12-24: Writing error messages to standard error
instead of standard output using `eprintln!`</span>

After changing `println!` to `eprintln!`, let’s run the program again in the
same way, without any arguments and redirecting standard output with `>`:

```text
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Now we see the error onscreen and *output.txt* contains nothing, which is the
behavior we expect of command line programs.

Let’s run the program again with arguments that don’t cause an error but still
redirect standard output to a file, like so:

```text
$ cargo run to poem.txt > output.txt
```

We won’t see any output to the terminal, and *output.txt* will contain our
results:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

This demonstrates that we’re now using standard output for successful output
and standard error for error output as appropriate.

## Summary

This chapter recapped some of the major concepts you’ve learned so far and
covered how to perform common I/O operations in Rust. By using command line
arguments, files, environment variables, and the `eprintln!` macro for printing
errors, you’re now prepared to write command line applications. By using the
concepts in previous chapters, your code will be well organized, store data
effectively in the appropriate data structures, handle errors nicely, and be
well tested.

Next, we’ll explore some Rust features that were influenced by functional
languages: closures and iterators.
-->
