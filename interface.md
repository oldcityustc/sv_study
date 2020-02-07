A simple interface declaration is as follows (see Syntax 25-1 for the complete syntax):

```verilog
interface identifier;
...
interface_items
...
endinterface [ : identifier ]
```

An interface can be instantiated hierarchically like a module, with or without ports. 

For example:

```verilog
myinterface #(100) scalar1(), vector[9:0]();
```

In this example, 11 instances of the interface of type myinterface have been instantiated, and the first
parameter within each interface is changed to 100. One myinterface instance is instantiated with the name
scalar1, and an array of 10 myinterface interfaces are instantiated with instance names vector[9] to
vector[0].



### 25.3.2 Interface example using a named bundle

The simplest form of a SystemVerilog interface is a bundled collection of variables or nets. When an
interface is referenced as a port, the variables and nets in it are assumed to have ref and inout access,
respectively. The following interface example shows the basic syntax for defining, instantiating, and
connecting an interface. Usage of the SystemVerilog interface capability can significantly reduce the
amount of code required to model port connections.



```verilog
interface simple_bus; // Define the interface
logic req, gnt;
logic [7:0] addr, data;
logic [1:0] mode;
logic start, rdy;
endinterface: simple_bus

module memMod
    (
        simple_bus a, // Access the simple_bus interface
		input logic clk
    );
logic avail;
// When memMod is instantiated in module top, a.req is the req
// signal in the sb_intf instance of the 'simple_bus' interface
always @(posedge clk) a.gnt <= a.req & avail;
endmodule
module cpuMod(simple_bus b, input logic clk);
...
endmodule

module top;
logic clk = 0;
simple_bus sb_intf(); // Instantiate the interface
memMod mem(sb_intf, clk); // Connect the interface to the module instance
cpuMod cpu(.b(sb_intf), .clk(clk)); // Either by position or by name
endmodule
```



### Below is dangerous!!!

In the preceding example, if the same identifier, sb_intf, had been used to name the simple_bus
interface in the memMod and cpuMod module headers, then implicit port connections also could have been
used to instantiate the memMod and cpuMod modules into the top module, as follows:

```verilog
module memMod (simple_bus sb_intf, input logic clk);
...
endmodule
module cpuMod (simple_bus sb_intf, input logic clk);
...
endmodule
module top;
logic clk = 0;
simple_bus sb_intf();
memMod mem (.*); // implicit port connections
cpuMod cpu (.*); // implicit port connections
endmodule
```

The following interface example shows how to specify a generic interface reference in a module definition:

```verilog
// memMod and cpuMod can use any interface
module memMod (interface a, input logic clk);
...
endmodule
    
module cpuMod(interface b, input logic clk);
...
endmodule
    
interface simple_bus; // Define the interface
logic req, gnt;
logic [7:0] addr, data;
logic [1:0] mode;
logic start, rdy;
endinterface: simple_bus
    
module top;
logic clk = 0;
simple_bus sb_intf(); // Instantiate the interface
// Reference the sb_intf instance of the simple_bus
// interface from the generic interfaces of the
// memMod and cpuMod modules
memMod mem (.a(sb_intf), .clk(clk));
cpuMod cpu (.b(sb_intf), .clk(clk));
endmodule
    
```

## 25.4 Ports in interfaces

```verilog
interface i1 (input a, output b, inout c);
wire d;
endinterface
```

The wires a, b, and c can be individually connected to the interface and thus shared with other interfaces.



The following example shows how to specify an interface with inputs, allowing a wire to be shared between
two instances of the interface:

```verilog
interface simple_bus (input logic clk); // Define the interface
logic req, gnt;
logic [7:0] addr, data;
logic [1:0] mode;
logic start, rdy;
endinterface: simple_bus

module memMod(simple_bus a); // Uses just the interface
logic avail;
always @(posedge a.clk) // the clk signal from the interface
a.gnt <= a.req & avail; // a.req is in the 'simple_bus' interface
endmodule

module cpuMod(simple_bus b);
...
endmodule

module top;
logic clk = 0;
    
simple_bus sb_intf1(clk); // Instantiate the interface
simple_bus sb_intf2(clk); // Instantiate the interface
    
memMod mem1(.a(sb_intf1)); // Reference simple_bus 1 to memory 1
cpuMod cpu1(.b(sb_intf1));
memMod mem2(.a(sb_intf2)); // Reference simple_bus 2 to memory 2
cpuMod cpu2(.b(sb_intf2));
endmodule
```

## 25.5 Modports(useful!!!)

To restrict interface access within a module, there are modport lists with directions declared within the
interface. The keyword modport indicates that the directions are declared as if inside the module.

```verilog
interface i2;
	wire a, b, c, d;
	modport master (input a, b, output c, d);
	modport slave (output a, b, input c, d);
endinterface
```

​	In this example, the modport list name (master or slave) can be specified in the module header, where
the interface name selects an interface and the modport name selects the appropriate directional information for the interface signals accessed in the module header.

```verilog
module m (i2.master i);
...
endmodule

module s (i2.slave i);
...
endmodule

module top;
i2 i();
m u1(.i(i));
s u2(.i(i));
endmodule
```

The modport list name (master or slave) can also be specified in the port connection with the module
instance, where the modport name is hierarchical from the interface instance.

```verilog
module m (i2 i);
...
endmodule

module s (i2 i);
...
endmodule

module top;
i2 i();
m u1(.i(i.master));
s u2(.i(i.slave));
endmodule
```



### 25.5.1 Example of named port bundle

This interface example shows how to use modports to control signal directions as in port declarations. It uses the modport name in the module definition.

```verilog
interface simple_bus (input logic clk); // Define the interface
logic req, gnt;
logic [7:0] addr, data;
logic [1:0] mode;
logic start, rdy;
    
modport slave (input req, addr, mode, start, clk,
output gnt, rdy,
ref data);
    
modport master(input gnt, rdy, clk,
output req, addr, mode, start,
ref data);
    
endinterface: simple_bus

module memMod (simple_bus.slave a); // interface name and modport name
logic avail;
always @(posedge a.clk) // the clk signal from the interface
a.gnt <= a.req & avail; // the gnt and req signal in the interface
endmodule

module cpuMod (simple_bus.master b);
...
endmodule

module top;
logic clk = 0;
simple_bus sb_intf(clk); // Instantiate the interface
initial repeat(10) #10 clk++;
memMod mem(.a(sb_intf)); // Connect the interface to the module instance
cpuMod cpu(.b(sb_intf));
endmodule
```

### 25.5.2 Example of connecting port bundle

This interface example shows how to use modports to restrict interface signal access and control their
direction. It uses the modport name in the module instantiation.

```verilog
interface simple_bus (input logic clk); // Define the interface
logic req, gnt;
logic [7:0] addr, data;
logic [1:0] mode;
logic start, rdy;
modport slave (input req, addr, mode, start, clk,
output gnt, rdy,
ref data);
modport master(input gnt, rdy, clk,
output req, addr, mode, start,
ref data);
endinterface: simple_bus

module memMod(simple_bus a); // Uses just the interface name
logic avail;
always @(posedge a.clk) // the clk signal from the interface
a.gnt <= a.req & avail; // the gnt and req signal in the interface
endmodule

module cpuMod(simple_bus b);
...
endmodule

module top;
logic clk = 0;
simple_bus sb_intf(clk); // Instantiate the interface
initial repeat(10) #10 clk++;
memMod mem(sb_intf.slave); // Connect the modport to the module instance
cpuMod cpu(sb_intf.master);
endmodule
```

### 25.5.3 Example of connecting port bundle to generic interface

This interface example shows how to use modports to control signal directions. It shows the use of the
interface keyword in the module definition. The actual interface and modport are specified in the module
instantiation.

```verilog
interface simple_bus (input logic clk); // Define the interface
logic req, gnt;
logic [7:0] addr, data;
logic [1:0] mode;
logic start, rdy;
modport slave (input req, addr, mode, start, clk,
output gnt, rdy,
ref data);    
modport master(input gnt, rdy, clk,
output req, addr, mode, start,
ref data);
endinterface: simple_bus

module memMod(interface a); // Uses just the interface
logic avail;
always @(posedge a.clk) // the clk signal from the interface
a.gnt <= a.req & avail; // the gnt and req signal in the interface
endmodule
    
module cpuMod(interface b);
...
endmodule
    
module top;
logic clk = 0;
simple_bus sb_intf(clk); // Instantiate the interface
memMod mem(sb_intf.slave); // Connect the modport to the module instance
cpuMod cpu(sb_intf.master);
endmodule
```

## 25.8 Parameterized interfaces

Interface definitions can take advantage of parameters and parameter redefinition in the same manner as
module definitions. The following example shows how to use parameters in interface definitions.

```verilog
interface simple_bus #(AWIDTH = 8, DWIDTH = 8)
(input logic clk); // Define the interface
logic req, gnt;
logic [AWIDTH-1:0] addr;
logic [DWIDTH-1:0] data;
logic [1:0] mode;
logic start, rdy;
    
modport slave( input req, addr, mode, start, clk,
output gnt, rdy,
ref data,
import task slaveRead,
task slaveWrite);
              
// import into module that uses the modport
modport master(input gnt, rdy, clk,
output req, addr, mode, start,
ref data,
import task masterRead(input logic [AWIDTH-1:0] raddr),
task masterWrite(input logic [AWIDTH-1:0] waddr));
               
// import requires the full task prototype
task masterRead(input logic [AWIDTH-1:0] raddr); // masterRead method
    ...
endtask
task slaveRead; // slaveRead method
...
endtask
task masterWrite(input logic [AWIDTH-1:0] waddr);
...
endtask
task slaveWrite;
...
endtask
endinterface: simple_bus
               
module memMod(interface a); // Uses just the interface keyword
logic avail;
always @(posedge a.clk) // the clk signal from the interface
a.gnt <= a.req & avail; //the gnt and req signals in the interface
always @(a.start)
if (a.mode[0] == 1'b0)
a.slaveRead;
else
a.slaveWrite;
endmodule
module cpuMod(interface b);
enum {read, write} instr;
logic [7:0] raddr;
always @(posedge b.clk)
if (instr == read)
b.masterRead(raddr); // call the Interface method
// ...
else
b.masterWrite(raddr);
endmodule
module top;
logic clk = 0;
simple_bus sb_intf(clk); // Instantiate default interface
simple_bus #(.DWIDTH(16)) wide_intf(clk); // Interface with 16-bit data
initial repeat(10) #10 clk++;
memMod mem(sb_intf.slave); // only has access to the slaveRead task
cpuMod cpu(sb_intf.master); // only has access to the masterRead task
memMod memW(wide_intf.slave); // 16-bit wide memory
cpuMod cpuW(wide_intf.master); // 16-bit wide cpu
endmodule
```

## 25.9 Virtual interfaces
