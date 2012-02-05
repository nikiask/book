Until now, we have been using the toplevel to evaluate programs.  As
your programs get larger, you'll want to save them in files so that
they can be re-used and shared. There are other advantages to doing
so, including separate compilation, _i.e._, the ability to compile
different portions of your program separately, ...

 to partition a program into multiple files
that can be written and compiled separately, making it easier to
construct and maintain the program. Perhaps the most important reason
to use files is that they serve as \emph{abstraction boundaries} that
divide a program into conceptual parts. We will see more about
abstraction during the next few chapters as we cover the OCaml module
system, but for now let's begin with an example of a complete program
implemented in a single file.

\labelsection{unique-example}{Single-file programs}

For this example, let's build a simple program that removes duplicate
lines in an input file. That is, the program should read its input a
line at a time, printing the line only if it hasn't seen it before.

\index{file suffixes!.ml (compilation unit)}
One of the simplest implementations is to use a list to keep track of
which lines have been read. The program can be implemented as a single
recursive function that 1) reads a line of input, 2) compares it with
lines that have been previously read, and 3) outputs the line if it
has not been read. The entire program is implemented in the single
file \hbox{\lstinline/unique.ml/}, shown in Figure~\reffigure{unique1}
with an example run.

In this case, we can compile the entire program in a single step with
the command
%
\hbox{\lstinline/ocamlc -o unique unique.ml/},
%
where \hbox{\lstinline/ocamlc/} is the OCaml
compiler, \hbox{\lstinline/unique.ml/} is the program file, and
the \hbox{\lstinline/-o/} option is used to specify the program
executable \hbox{\lstinline/unique/}.

\begin{figure}
\begin{center}
\begin{tabular}{l}
File: unique.ml\\
\hline
\begin{ocamllisting}
let rec unique already_read =
   output_string stdout "> ";
   flush stdout;
   let line = input_line stdin in
      if not (List.mem line already_read) then begin
         output_string stdout line;
         output_char stdout '\n';
         unique (line :: already_read)
      end else
         unique already_read;;

(* "Main program" *)
try unique [] with
   End_of_file ->
      ();;
\end{ocamllisting}
\\
\\
Example run\\
\hline
\begin{ocamllisting}
% ocamlc -o unique unique.ml
% ./unique
> Great Expectations
Great Expectations
> Vanity Fair
Vanity Fair
> Great Expectations
> Paradise Lost
Paradise Lost
\end{ocamllisting}
\end{tabular}
\end{center}
\caption{A program to print only unique lines.}
\labelfigure{unique1}
\end{figure}

\labelsubsection{main}{Where is the main function?}

\index{compilation units!main function@\textit{main} function} Unlike C programs, OCaml program do
not have a ``\texttt{main}'' function. When an OCaml program is evaluated, all the statements in the
implementation files are evaluated in order.  In general, implementation files can contain arbitrary
expressions, not just function definitions. For this example, the main program is the
essentially \lstinline$try$ expression in the \hbox{\lstinline/unique.ml/} file, which gets
evaluated when the \hbox{\lstinline/unique.cmo/} file is evaluated.  We say ``essentially'' because
the main function is really the entire program, which is evaluated starting from the beginning when
the program is executed.

\labelsubsection{compilers}{OCaml compilers}

\index{ocamlc}
\index{ocamlopt}
The INRIA OCaml implementation
provides two compilers---the \hbox{\lstinline/ocamlc/} byte-code
compiler, and the \hbox{\lstinline/ocamlopt/} native-code
compiler.  Programs compiled with \hbox{\lstinline/ocamlc/} are
interpreted, while programs compiled with \hbox{\lstinline/ocamlopt/}
are compiled to native machine code to be run on a specific operating
system and machine architecture.  While the two compilers produce
programs that behave identically functionally, there are some
differences.

\begin{itemize}

\item 

Compile time is shorter with the \hbox{\lstinline/ocamlc/}
compiler. Compiled byte-code is portable to any operating system and
architecture supported by OCaml, without the need to recompile. Some
tasks, like debugging, work only with byte-code executables.

\item

Compile time is longer with the \hbox{\lstinline/ocamlopt/} compiler,
but program execution is usually faster.  Program executables are not
portable, and \hbox{\lstinline/ocamlopt/} is supported on fewer operating
systems and machine architectures than \hbox{\lstinline/ocamlc/}.

\end{itemize}
%
We generally won't be concerned with the compiler being used, since
the two compilers produce programs that behave identically (apart from
performance).  During rapid development, it may be useful to use the
byte-code compiler because compilation times are shorter. If
performance becomes an issue, it is usually a straightforward process
to begin using the native-code compiler.

\labelsection{multiple-files}{Multiple files and abstraction}

\index{compilation units!interfaces}
\index{abstraction!interfaces}
OCaml uses files as a basic unit for providing data hiding and
encapsulation, two important properties that can be used to strengthen
the guarantees provided by the implementation. We will see more about
data hiding and encapsulation in Chapter~\reflabelchapter{modules},
but for now the important part is that each file can be assigned
a \emph{interface} that declares types for all the accessible parts of
the implementation, and everything not declared is inaccessible
outside the file.

\index{file suffixes!.mli (interface)}
In general, a program will have many files and interfaces. An
implementation file is defined in a file with a \hbox{\lstinline/.ml/}
suffix, called a \emph{compilation unit}. An interface for a
file \emph{filename.ml} is defined in a file
named \emph{filename.mli}. There are four major steps to planning and
building a program.

\begin{enumerate}

\item{} 

Decide how to divide the program into separate files.  Each part will
be implemented in a separate compilation unit.

\item{}

  Implement each of compilation units as a file with a \hbox{\lstinline/.ml/} suffix, and optionally
  define an interface for the compilation unit in a file with the same name, but with a
  \hbox{\lstinline/.mli/} suffix.

\item{}

Compile each file and interface with the OCaml compiler.

\item{}

Link the compiled files to produce an executable program.

\end{enumerate}
%
\index{compilation, separate}
One nice consequence of implementing the parts of a program in
separate files is that each file can be compiled separately. When a
project is modified, only the files that are affected must be
recompiled; there is there is usually no need to recompile the entire
project.

Getting back to the example \hbox{\lstinline/unique.ml/}, the
implementation is already too concrete. We chose to use a list to
represent the set of lines that have been read, but one problem with
using lists is that checking for membership
(with \hbox{\lstinline/List.mem/}) takes time linear in the length of
the list, which means that the time to process a file is quadratic in
the number of lines in the file.  There are clearly better data
structures than lists for the set of lines that have been read.

As a first step, let's partition the program into two files. The first
file \hbox{\lstinline/set.ml/} is to provide a generic implementation
of sets, and the file \hbox{\lstinline/unique.ml/} provides
the \hbox{\lstinline/unique/} function as before. For now, we'll keep
the list representation in hopes of improving it later---for now we
just want to factor the project.

\begin{figure}
\begin{center}
\begin{tabular}[t]{l}
File: set.ml\\
\hline
\begin{ocamllisting}
let empty = []
let add x l = x :: l
let mem x l = List.mem x l
\end{ocamllisting}
\\
\\
File: unique.ml\\
\hline
\begin{ocamllisting}
let rec unique already_read =
    output_string stdout "> ";
    flush stdout;
    let line = input_line stdin in
        if not (Set.mem line already_read) then begin
           output_string stdout line;
           output_char stdout '\n';
           unique (Set.add line already_read)
        end else
           unique already_read;;

(* Main program *)
try unique [] with
   End_of_file ->
      ();;
\end{ocamllisting}
\\
\\
Example run\\
\hline
\begin{minipage}{4in}
\begin{ocaml}
% ocamlc -c set.ml
% ocamlc -c unique.ml
% ocamlc -o unique set.cmo unique.cmo
% ./unique
> Adam Bede
@
\begin{topoutput}
Adam Bede
\end{topoutput}
@
> A Passage to India
@
\begin{topoutput}
A Passage to India
\end{topoutput}
@
> Adam Bede
> Moby Dick
@
\begin{topoutput}
Moby Dick
\end{topoutput}
@
\end{ocaml}
\end{minipage}
\end{tabular}
\end{center}
\caption{Factoring the program into two separate files.}
\labelfigure{unique2}
\end{figure}
%
\index{.!compilation units} The new project is shown in Figure \reffigure{unique2}. We have split
the set operations into a file called \hbox{\lstinline/set.ml/}, and instead of using the
\hbox{\lstinline/List.mem/} function we now use the \hbox{\lstinline/Set.mem/} function.  The way to
refer to a definition \ensuremath{f} in a file named \emph{filename} is by capitalizing the filename
and using the infix \hbox{\lstinline/./} operator to project the value. The
\hbox{\lstinline/Set.mem/} expression refers to the \hbox{\lstinline/mem/} function in the
\hbox{\lstinline/set.ml/} file. In fact, the \hbox{\lstinline/List.mem/} function is the same
way. The OCaml standard library contains a file \hbox{\lstinline/list.ml/} that defines a function
\hbox{\lstinline/mem/}.

\index{file suffixes!.cmo (byte code)} Compilation now takes several steps. In the first step, the
\hbox{\lstinline/set.ml/} and \hbox{\lstinline/unique.ml/} files are compiled with the
\hbox{\lstinline/-c/} option, which specifies that the compiler should produce an intermediate file
with a \hbox{\lstinline/.cmo/} suffix. These files are then linked to produce an executable with the
command \hbox{\lstinline/ocamlc -o unique set.cmo unique.cmo/}.

\index{compilation units!cyclic dependencies}
The order of compilation and linking here is
significant. The \hbox{\lstinline/unique.ml/} file refers to
the \hbox{\lstinline/set.ml/} file by using
the \hbox{\lstinline/Set.mem/} function. Due to this dependency,
the \hbox{\lstinline/set.ml/} file must be compiled before
the \hbox{\lstinline/unique.ml/} file, and
the \hbox{\lstinline/set.cmo/} file must appear before
the \hbox{\lstinline/unique.cmo/} file during linking. 
Cyclic dependencies are \emph{not allowed}. It is not legal to have a
file \hbox{\lstinline/a.ml/} refer to a value \hbox{\lstinline/B.x/},
and a file \hbox{\lstinline/b.ml/} that refers to a
value \hbox{\lstinline/A.y/}.

\labelsubsection{defining-interfaces}{Defining an interface}

\index{compilation units!interfaces}
One of the reasons for factoring the program was to be able to improve
the implementation of sets. To begin, we should make the type of
sets \emph{abstract}---that is, we should hide the details of how it
is implemented so that we can be sure the rest of the program does not
unintentionally depend on the implementation details. To do this, we
can define an abstract interface for sets, in a
file \hbox{\lstinline/set.mli/}.

An interface should declare types for each of the values that are
publicly accessible in a module, as well as any needed type
declarations or definitions. For our purposes, we need to define a
polymorphic type of sets \hbox{\lstinline/'a set/} abstractly. That
is, in the interface we will declare a type \hbox{\lstinline/'a set/}
without giving a definition, preventing other parts of the program
from knowing, or depending on, the particular representation of sets
we have chosen. The interface also needs to declare types for the
public values \hbox{\lstinline/empty/}, \hbox{\lstinline/add/},
and \hbox{\lstinline/mem/} values, as a declaration with the following
syntax.

\index{val@\lstinline/val/}
\label{keyword:val(signatures)}
\begin{ocaml}
val $\nt{identifier}$ : $\nt{type}$
\end{ocaml}
%
The complete interface is shown in Figure \reffigure{unique3}. The
implementation remains mostly unchanged, except that a specific,
concrete type definition must be given for the type
%
\hbox{\lstinline/'a set/}.

\begin{figure}
\begin{center}
\begin{tabular}{ll}
\begin{tabular}[t]{l}
File: set.mli\\
\hline
\begin{ocaml}
type 'a set
val empty : 'a set
val add : 'a -> 'a set -> 'a set
val mem : 'a -> 'a set -> bool
\end{ocaml}
\\
\\
File: set.ml\\
\hline
\begin{ocaml}
type 'a set = 'a list
let empty = []
let add x l = x :: l
let mem x l = List.mem x l
\end{ocaml}
\end{tabular}
&
\begin{tabular}[t]{l}
Example run (with lists)\\
\hline
\begin{minipage}{2.5in}
\begin{ocaml}
% ocamlc -c set.mli
% ocamlc -c set.ml
% ocamlc -c unique.ml
% ocamlc -o unique set.cmo unique.cmo
% ./unique
> Siddhartha
@
\begin{topoutput}
Siddhartha
\end{topoutput}
@
> Siddhartha
> Siddharta
@
\begin{topoutput}
Siddharta
\end{topoutput}
@
\end{ocaml}
\end{minipage}
\end{tabular}
\end{tabular}
\end{center}
\caption{Adding an interface to the \hbox{\lstinline$Set$} implementation.}
\labelfigure{unique3}
\end{figure}
%
%% \\
%% \\
%% \begin{ocaml}
%% % ocamlc -c set.mli
%% % ocamlc -c set.ml
%% % ocamlc -c unique.ml
%% File "unique.ml", line 8, characters 14-36:
%% This expression has type 'a list but is
%%    here used with type string Set.set
%% \end{ocaml}
%% \begin{tabular}[t]{l}
%% File: unique.ml\\
%% \hline
%% \begin{ocaml}
%% let rec unique alread_read =
%%    output_string stdout "> ";
%%    flush stdout;
%%    let line = input_line stdin in
%%       if not (Set.mem line already_read) then begin
%%          output_string stdout line;
%%          output_char stdout '\n';
%%          (* unique (line :: already_read) *)
%%          unique (Set.add line already_read)
%%       end else
%%          unique already_read;;
%
%% try unique Set.empty with
%%    End_of_file ->
%%       ();;
%% \end{ocaml}
%% \end{tabular}
%% &
%% \begin{tabular}[t]{l}
%% Example run\\
%% \hline
%% \begin{ocaml}
%% % ocamlc -c set.mli
%% % ocamlc -c set.ml
%% % ocamlc -c unique.ml
%% % ocamlc -o unique set.cmo unique.cmo
%% % ./unique
%% > Siddhartha
%% Siddhartha
%% > Siddhartha
%% > Siddharta
%% Siddharta
%% \end{ocaml}
%% \end{tabular}
%% \end{tabular}
%% \end{center}
%% \labelfigure{uniq2}
%% \end{figure}
%
Now, when we compile the program, we first compile the interface
file \hbox{\lstinline/set.mli/}, then the
implementations \hbox{\lstinline/set.ml/}
and \hbox{\lstinline/unique.ml/}.
%
%% But something has changed, the \hbox{\lstinline/uniq.ml/} file
%% no longer compiles! Following the error message, we find that the error is due to the expression
%% \hbox{\lstinline/line :: already_read/}, which uses a \hbox{\lstinline/List/} operation instead of a \hbox{\lstinline/Set/}
%% operation. Since the \hbox{\lstinline/'a set/} type is abstract, it is now an error to treat the set as a list,
%% and the compiler complains appropriately.
%
%% Changing this expression to \hbox{\lstinline/Set.add line already_read/} fixes the error. 
%
Note that, although the \hbox{\lstinline/set.mli/} file must be compiled, it does not
need to be specified during linking
%
\hbox{\lstinline/ocamlc -o unique set.cmo unique.cmo/}.

At this point, the \hbox{\lstinline/set.ml/} implementation is fully
abstract, making it easy to replace the implementation with a better
one (for example, the implementation of sets using red-black trees in
Section~\reflabelsection{balanced-red-black-trees}).

\labelsubsection{transparent-types}{Transparent type definitions}

\index{types!transparent}
In some cases, abstract type definitions are too strict. There are
times when we want a type definition to be \emph{transparent}---that
is, visible outside the file. For example, suppose we wanted to add a
\hbox{\lstinline/choose/} function to the set implementation, where, given a set
$s$, the expression (\hbox{\lstinline/choose $s$/}) returns
some element of the set if the set is non-empty, and nothing
otherwise. One possible way to write this function is to define a
union type \hbox{\lstinline/choice/} that defines the two cases, as shown in
Figure~\reffigure{unique4}.

The type definition for \hbox{\lstinline/choice/} must be transparent (otherwise
there isn't much point in defining the function).  For the type to be
transparent, the interface simply provides the definition.  The
implementation must contain the \emph{same} definition.

\begin{figure}
\begin{center}
\begin{tabular}{l@{\hskip0.25in}l}
\begin{tabular}[t]{l}
Interface file: set.mli\\
\hline
\begin{ocaml}
type 'a set
type 'a choice =
   Element of 'a
 | Empty
val empty : 'a set
val add : 'a -> 'a set -> 'a set
val mem : 'a -> 'a set -> bool
val choose : 'a set -> 'a choice
\end{ocaml}
\end{tabular}
&
\begin{tabular}[t]{l}
Implementation file: set.ml\\
\hline
\begin{ocaml}
type 'a set = 'a list
type 'a choice =
   Element of 'a
 | Empty
let empty = []
let add x l = x :: l
let mem x l = List.mem x l
let choose = function
   x :: _ -> Element x
 | [] -> Empty
\end{ocaml}
\end{tabular}
\end{tabular}
\end{center}
\caption{Extending the \hbox{\lstinline$Set$} implementation.}
\labelfigure{unique4}
\end{figure}

\labelsection{common-errors}{Some common errors}

As you develop programs with several files, you will undoubtably
encounter some errors.

\labelsubsection{interface-errors}{Interface errors}

\index{file suffixes!.cmi (compiled interface)}
When an interface file (with a \hbox{\lstinline/.mli/} suffix) is
compiled successfully with \hbox{\lstinline/ocamlc/}
or \hbox{\lstinline/ocamlopt/}, the compiler produces a compiled
representation of the file, having a \hbox{\lstinline/.cmi/} suffix.
When an implementation is compiled, the compiler compares the
implementation with the interface.  If a definition does not match the
interface, the compiler will print an error and refuse to compile the
file.

\labelsubsubsection{type-mismatch-error}{Type errors}

\index{interfaces!type errors}
For example, suppose we had reversed the order of arguments in the
\hbox{\lstinline/Set.add/} function so that the set argument is first.

\begin{ocaml}
let add s x = x :: s
\end{ocaml}
%
When we compile the file, we get an error. The compiler prints the
types of the mismatched values, and exits with an error code.

\begin{ocaml}
% ocamlc -c set.mli
% ocamlc -c set.ml
@
\begin{toperror}
The implementation set.ml does not match the interface set.cmi:
Values do not match:
  val add : 'a list -> 'a -> 'a list
is not included in
  val add : 'a -> 'a set -> 'a set
\end{toperror}
@
\end{ocaml}
%
The first declaration is the type the compiler inferred for the
definition; the second declaration is from the interface. Note that
the definition's type is not abstract (using
\hbox{\lstinline/'a list/}
instead of \hbox{\lstinline/'a set/}). For this example, we deduce
that the argument ordering doesn't match, and the implementation or the
interface must be changed.

\labelsubsubsection{missing-def-error}{Missing definition errors}

\index{interfaces!missing definitions}
Another common error occurs when a function declared in the interface
is not defined in the implementation. For example, suppose we had
defined an \texttt{insert} function instead of an \texttt{add}
function. In this case, the compiler prints the name of the missing
function, and exits with an error code.

\begin{ocaml}
% ocamlc -c set.ml
@
\begin{toperror}
The implementation set.ml does not match the interface set.cmi:
The field `add' is required but not provided
\end{toperror}
@
\end{ocaml}

\labelsubsubsection{type-def-errors}{Type definition mismatch errors}

\index{interfaces!type mismatches}
\emph{Transparent} type definitions in the interface can also cause an
error if the type definition in the implementation does not match. For
example, in the definition of the \hbox{\lstinline/choice/} type,
suppose we had declared the cases in different orders.

\begin{center}
\begin{tabular}{l@{\hskip0.25in}l}
\begin{tabular}[t]{l}
Interface file: set.mli\\
\hline
\begin{ocaml}
type 'a set
type 'a choice =
   Element of 'a
 | Empty
$\cdots$
\end{ocaml}
\end{tabular}
&
\begin{tabular}[t]{l}
Implementation file: set.ml\\
\hline
\begin{ocaml}
type 'a set = 'a list
type 'a choice =
   Empty
 | Element of 'a
$\cdots$
\end{ocaml}
\end{tabular}
\end{tabular}
\end{center}
%
When we compile the \hbox{\lstinline/set.ml/} file, the compiler produces an error
with the mismatched types.

\begin{ocaml}
% ocamlc -c set.mli
% ocamlc -c set.ml
@
\begin{toperror}
The implementation set.ml does not match the interface set.cmi:
Type declarations do not match:
  type 'a choice = Empty | Element of 'a
is not included in
  type 'a choice = Element of 'a | Empty
\end{toperror}
@
\end{ocaml}
%
\index{interfaces!omitting@omitting the \lstinline$.mli$ file}
The type definitions are required to be \emph{exactly} the same. Some
programmers find this duplication of type definitions to be
annoying. While it is difficult to avoid all duplication of type
definitions, one common solution is to define the transparent types in
a separate \hbox{\lstinline/.ml/} file without an interface, for
example by moving the definition of \hbox{\lstinline/'a choice/} to a
file \hbox{\lstinline/set_types.ml/}. By default, when an interface
file does not exist, the compiler automatically produces an interface
in which all definitions from the implementation are fully visible. As
a result, the type in \hbox{\lstinline/set_types.ml/} needs to be
defined just once.

\labelsubsubsection{compile-errors}{Compile dependency errors}

\index{compilation units!dependency errors}
\index{compiling!inconsistent assumptions}
The compiler will also produce errors if the compile state is
inconsistent. Each time an interface is compiled, all the files that
uses that interface must be recompiled.  For example, suppose we update
the \hbox{\lstinline/set.mli/} file, and recompile it and
the \hbox{\lstinline/unique.ml/} file (but we forget to recompile
the \hbox{\lstinline/set.ml/} file). The compiler produces the
following error.

\begin{ocaml}
% ocamlc -c set.mli
% ocamlc -c unique.ml
% ocamlc -o unique set.cmo unique.cmo
@
\begin{toperror}
Files unique.cmo and set.cmo make inconsistent
assumptions over interface Set
\end{toperror}
@
\end{ocaml}
%
It takes a little work to detect the cause of the error. The compiler
says that the files make inconsistent assumptions for
interface \hbox{\lstinline/Set/}. The interface is defined in the
file \hbox{\lstinline/set.cmi/}, and so this error message states that
at least one of \hbox{\lstinline/set.ml/}
or \hbox{\lstinline/unique.ml/} needs to be recompiled. In general, we
don't know which file is out of date, and the best solution is usually
to recompile them all.

\labelsection{open}{Using \texttt{open} to expose a namespace}

\label{keyword:open}
\index{open!compilation units}
Using the full name \texttt{\emph{Filename}.\emph{identifier}} to refer to
the values in a module can get
tedious. The statement \texttt{open \emph{Filename}} can be used to
``open'' an interface, allowing the use of unqualified names for
types, exceptions, and values. For example,
the \hbox{\lstinline/unique.ml/} module can be somewhat simplified by
using the \hbox{\lstinline/open/} directive for
the \hbox{\lstinline/Set/} module. In the following listing,
the \underline{underlined} variables refer to values from the Set
implementation (the underlines are for illustration only, they don't
exist in the program files).

\begin{center}
\lstset{moreemph={mem,add,empty},emphstyle=\underbar}
\begin{tabular}[t]{l}
File: unique.ml\\
\hline
\begin{ocaml}
open Set
let rec unique already_read =
   output_string stdout "> ";
   flush stdout;
   let line = input_line stdin in
      if not (mem line already_read) then begin
         output_string stdout line;
         output_char stdout '\n';
         unique (add line already_read)
      end else
         unique already_read;;

(* Main program *)
try unique empty with
   End_of_file ->
      ();;
\end{ocaml}
\end{tabular}
\end{center}
%
\index{open!scoping}
Sometimes multiple \texttt{open}ed files will define the same name. In
this case, the \emph{last} file with an \texttt{open} statement will
determine the value of that symbol. Fully qualified names (of the
form \texttt{\emph{Filename}.\emph{identifier}}) may still be used even if
the file has been opened. Fully qualified names can be used to access
values that may have been hidden by an \texttt{open} statement.

\labelsubsection{open-errors}{A note about \texttt{open}}

\index{open!overuse}
Be careful with the use of \texttt{open}. In general, fully qualified
names provide more information, specifying not only the name of the
value, but the name of the module where the value is defined. For
example, the \texttt{Set} and \texttt{List} modules both define
a \texttt{mem} function. In the \texttt{Unique} module we just
defined, it may not be immediately obvious to a programmer that
the \texttt{mem} symbol refers to \hbox{\lstinline/Set.mem/},
not \hbox{\lstinline/List.mem/}.

In general, you should use \hbox{\lstinline/open/} statement
sparingly. Also, as a matter of style, it is better not
to \texttt{open} most of the library modules, like
the \hbox{\lstinline/Array/}, \hbox{\lstinline/List/},
and \hbox{\lstinline/String/} modules, all of which define methods
(like \hbox{\lstinline/create/}) with common names. Also, you should
never \texttt{open}
the \hbox{\lstinline/Unix/}, \hbox{\lstinline/Obj/},
and \hbox{\lstinline/Marshal/} modules! The functions in these modules
are not completely portable, and the fully qualified names can be used
to identify all the places where portability may be a problem (for
instance, the Unix \misspelled{\texttt{grep}} command can be used to find all the
places where \hbox{\lstinline/Unix/} functions are used).

\index{open!vs include@\textit{vs.}~\hbox{\lstinline/#include/}}
The behavior of the \texttt{open} statement is not like
an \hbox{\lstinline/#include/} statement in C. An implementation
file \hbox{\lstinline/mod.ml/} should not include
an \hbox{\lstinline/open Mod/} statement. One common source of errors
is defining a type in a \hbox{\lstinline/.mli/} interface, then
attempting to use \hbox{\lstinline/open/} to ``include'' the
definition in the \hbox{\lstinline/.ml/} implementation. This won't
work---the implementation must include an identical type
definition. This might be considered to be an annoying feature of
OCaml, but it preserves a simple semantics---the implementation must
provide a definition for each declaration in the interface.

\labelsection{debugging}{Debugging a program}

\index{ocamldebug@\lstinline$ocamldebug$}
The \hbox{\lstinline/ocamldebug/} program can be used to debug a
program compiled
with \hbox{\lstinline/ocamlc/}. The \hbox{\lstinline/ocamldebug/}
program is a little like the GNU \hbox{\lstinline/gdb/} program.  It
allows breakpoints to be set; when a breakpoint is reached, control is
returned to the debugger so that program variables can be examined.

To use \hbox{\lstinline/ocamldebug/}, the program must be compiled with the
\hbox{\lstinline/-g/} flag.

\begin{ocaml}
% ocamlc -c -g set.mli
% ocamlc -c -g set.ml
% ocamlc -c -g unique.ml
% ocamlc -o unique -g set.cmo unique.cmo
\end{ocaml}
%
The debugger is invoked using by specifying the program to be debugged
on the \hbox{\lstinline/ocamldebug/} command line.

\begin{ocamldebug}
% ocamldebug ./unique
`
\begin{topoutput}
	Objective Caml Debugger version 3.08.3
\end{topoutput}
`
(ocd) help
`
\begin{toperror}
List of commands: cd complete pwd directory kill help quit shell run reverse
step backstep goto finish next start previous print display source break
delete set show info frame backtrace bt up down last list load_printer
install_printer remove_printer
\end{toperror}
`
\end{ocamldebug}
%
There are several commands that can be used. The basic commands
are \hbox{\lstinline/run/}, \hbox{\lstinline/step/}, \hbox{\lstinline/next/}, \hbox{\lstinline/break/}, \hbox{\lstinline/list/}, \hbox{\lstinline/print/},
and \hbox{\lstinline/goto/}.

\begin{quote}
\begin{itemize}
\item \lstinline]run]: Start or continue execution of the program.
\item \lstinline]break @ module linenum]: Set a breakpoint on line \lstinline/linenum/ in module \lstinline/module/.
\item \lstinline]list]: display the lines around the current execution point.
\item \lstinline]print expr]: Print the value of an expression. The expression must be a variable.
\item \lstinline]goto time]:
%
Execution of the program is measured in time steps, starting from
0. Each time a breakpoint is reached, the debugger prints the current
time. The \hbox{\lstinline/goto/} command may be used to continue
execution to a future time, or to a \emph{previous} timestep.

\item \lstinline]step]: Go forward one time step.
\item \lstinline]next]:
%
If the current value to be executed is a function, evaluate the
function, a return control to the debugger when the function
completes. Otherwise, step forward one time step.
\end{itemize}
\end{quote}
%
For debugging the \hbox{\lstinline/unique/} program, we need to know
the line numbers. Let's set a breakpoint in
the \hbox{\lstinline/unique/} function, which starts in line 1 in
the \hbox{\lstinline/Unique/} module. We'll want to stop at the first
line of the function.

\begin{ocamldebug}
(ocd) break @ Unique 1
`
\begin{topoutput}
Loading program... done.
Breakpoint 1 at 21656 : file unique.ml, line 2, character 4
\end{topoutput}
`
1
(ocd) run
`
\begin{topoutput}
Time : 12 - pc : 21656 - module Unique
Breakpoint : 1
2    <|b|>output_string stdout "> ";
\end{topoutput}
`
(ocd) n
`
\begin{topoutput}
Time : 14 - pc : 21692 - module Unique
2    output_string stdout "> "<|a|>;
\end{topoutput}
`
(ocd) n
`
\begin{topoutput}
> Time : 15 - pc : 21720 - module Unique
3    flush stdout<|a|>;
\end{topoutput}
`
(ocd) n
`
\begin{topoutput}
Robinson Crusoe
Time : 29 - pc : 21752 - module Unique
5       <|b|>if not (Set.mem line already_read) then begin
\end{topoutput}
`
(ocd) p line
`
\begin{topoutput}
line : string = "Robinson Crusoe"
\end{topoutput}
`
\end{ocamldebug}
%
Next, let's set a breakpoint just before calling the \hbox{\lstinline/unique/}
function recursively.

\begin{ocamldebug}
(ocd) list
`
\begin{topoutput}
1 let rec unique already_read =
2    output_string stdout "> ";
3    flush stdout;
4    let line = input_line stdin in
5       <|b|>if not (Set.mem line already_read) then begin
6          output_string stdout line;
7          output_char stdout '\n';
8          unique (Set.add line already_read)
9       end
10       else
11          unique already_read;;
12
13 (* Main program *)
14 try unique Set.empty with
15    End_of_file ->
16       ();;
Position out of range.
\end{topoutput}
`
(ocd) break @ 8
`
\begin{topoutput}
Breakpoint 2 at 21872 : file unique.ml, line 8, character 42
\end{topoutput}
`
(ocd) run
`
\begin{topoutput}
Time : 38 - pc : 21872 - module Unique
Breakpoint : 2
8          unique (Set.add line already_read)<|a|>
\end{topoutput}
`
\end{ocamldebug}
%
\index{ocamldebug!backward execution}
Next, suppose we don't like adding this line of input. We can go back
to time \hbox{\lstinline/15/} (the time just before
the \hbox{\lstinline/input_line/} function is called).

\begin{ocamldebug}
(ocd) goto 15
`
\begin{topoutput}
> Time : 15 - pc : 21720 - module Unique
3    flush stdout<|a|>;
\end{topoutput}
`
(ocd) n
`
\begin{topoutput}
Mrs Dalloway
Time : 29 - pc : 21752 - module Unique
5       <|b|>if not (Set.mem line already_read) then begin
\end{topoutput}
`
\end{ocamldebug}
%
Note that when we go back in time, the program prompts us again for an
input line. This is due to way time travel is implemented
in \hbox{\lstinline/ocamldebug/}.  Periodically, the debugger takes a
checkpoint of the program (using the Unix \hbox{\lstinline/fork()/}
system call).  When reverse time travel is requested, the debugger
restarts the program from the closest checkpoint before the time
requested.  In this case, the checkpoint was taken before the call
to \hbox{\lstinline/input_line/}, and the program resumption requires
another input value.

We can continue from here, examining the remaining functions and
variables. You may wish to explore the other features of the
debugger.  Further documentation can be found in the OCaml reference
manual.