## 26.2 Package declarations

SystemVerilog packages provide an additional mechanism for sharing parameters, data, type, task, function,sequence, property, and checker declarations among multiple SystemVerilog modules, interfaces, programs,and checkers.

The package declaration creates a scope that contains declarations intended to be shared among one or
more compilation units, modules, interfaces, or programs. Items within packages are generally type
definitions, tasks, and functions. Items within packages shall not have hierarchical references to identifiers
except those created within the package or made visible by import of another package. A package shall not
refer to items defined in the compilation unit scope. (See 3.12.1.) It is also possible to populate packages
with parameters, variables, and nets. This may be useful for global items that are not conveniently passed
down through the hierarchy. Variable declaration assignments within the package shall occur before any
initial or always procedures are started, in the same way as variables declared in a compilation unit or
module.
The following is an example of a package:

```verilog
package ComplexPkg;
    
typedef struct {
shortreal i, r;
} Complex;
    
function Complex add(Complex a, b);
add.r = a.r + b.r;
add.i = a.i + b.i;
endfunction
    
function Complex mul(Complex a, b);
mul.r = (a.r * b.r) - (a.i * b.i);
mul.i = (a.r * b.i) + (a.i * b.r);
endfunction
    
endpackage : ComplexPkg
```



## 26.3 Referencing data in packages

**The compilation of a package shall precede the compilation of scopes in which the package is imported.**
One way to use declarations made in a package is to reference them using the package scope resolution
operator ::.

```verilog
ComplexPkg::Complex cout = ComplexPkg::mul(a, b);
```

An alternate method for utilizing package declarations is via the import declaration (see Syntax 26-2).

The import declaration provides direct visibility of identifiers within packages. It allows identifiers
declared within packages to be visible within the current scope without a package name qualifier. Two
forms of the import declaration are provided: explicit import and wildcard import. Explicit import allows
control over precisely which symbols are imported:

```verilog
import ComplexPkg::Complex;
import ComplexPkg::add;
```

In the following example, the import of the enumeration type teeth_t does not import the enumeration
literals ORIGINAL and FALSE. In order to refer to the enumeration literal FALSE from package q, either add
import q::FALSE or use a full package reference as in teeth = q::FALSE;.

```verilog
package p;
typedef enum { FALSE, TRUE } bool_t;
endpackage
package q;
typedef enum { ORIGINAL, FALSE } teeth_t;
endpackage
module top1 ;
import p::*;
import q::teeth_t;
teeth_t myteeth;
initial begin
myteeth = q:: FALSE; // OK:
myteeth = FALSE; // ERROR: Direct reference to FALSE refers to the
end // FALSE enumeration literal imported from p
endmodule
module top2 ;
import p::*;
import q::teeth_t, q::ORIGINAL, q::FALSE;
teeth_t myteeth;
initial begin
myteeth = FALSE; // OK: Direct reference to FALSE refers to the
end // FALSE enumeration literal imported from q
endmodule
```

A wildcard import allows all identifiers declared within a package to be imported provided the identifier is
not otherwise defined in the importing scope: A wildcard import is of the following form:

```
import ComplexPkg::*;
```

The effect of importing an identifier into a scope makes that identifier visible without requiring access using
the scope resolution operator. Importing does not copy the declaration of that identifier into the importing
scope. The imported identifier shall not be visible outside that importing scope by hierarchical reference into that scope or by interface port reference into that scope.

### It is a little complicated for the import searching mechanism, pay attention. 

## 26.4 Using packages in module headers

Package items that are imported as part of a module, interface, or program header are visible throughout the
module, interface, or program, including in parameter and port declarations.
For example:

```verilog
package A;
typedef struct {
bit [ 7:0] opcode;
bit [23:0] addr;
} instruction_t;
endpackage: A

package B;
typedef enum bit {FALSE, TRUE} boolean_t;
endpackage: B

module M import A::instruction_t, B::*;
#(WIDTH = 32)
(input [WIDTH-1:0] data,
input instruction_t a,
output [WIDTH-1:0] result,
output boolean_t OK
);
...
endmodule: M
```

## 26.5 Search order rules

Table 26-1 describes the search order rules for the declarations imported from a package. For the purposes
of the discussion that follows, consider the following package declarations:

```verilog
package p;
typedef enum { FALSE, TRUE } BOOL;
const BOOL c = FALSE;
endpackage
package q;
const int c = 0;
endpackage
```

![image-20200208110548651](C:\Users\huaming\AppData\Roaming\Typora\typora-user-images\image-20200208110548651.png)

![image-20200208110614508](C:\Users\huaming\AppData\Roaming\Typora\typora-user-images\image-20200208110614508.png)

## 26.6 Exporting imported names from packages

## 26.7 The std built-in package

