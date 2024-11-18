# DSD LAB FAT CODES
## 1. Random Sequence Counter

```verilog
module random_sequence_counter(
    input clk,
    input reset,
    output reg [2:0] count
);
    always @(posedge clk or posedge reset) begin
        if (reset)
            count <= 3'b000;
        else begin
            case (count)
                3'b000: count <= 3'b101;
                3'b101: count <= 3'b011;
                3'b011: count <= 3'b110;
                3'b110: count <= 3'b001;
                3'b001: count <= 3'b100;
                3'b100: count <= 3'b010;
                3'b010: count <= 3'b000;
                default: count <= 3'b000;
            endcase
        end
    end
endmodule

```

---
## 2. Ring and Johnson Counter
### 2a. Ring Counter

```verilog
module ring_counter (
    output reg [3:0] Q,
    input clk,
    input rst
);
    always @(posedge clk or posedge rst) begin
        if (rst)
            Q <= 4'b1001;
        else
            Q <= {Q[0], Q[3:1]};
    end
endmodule

module ring_counter_test;
    wire [3:0] Q;
    reg clk;
    reg rst;

    ring_counter RC (
        .Q(Q),
        .clk(clk),
        .rst(rst)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        #10 rst = 0;
        #200 $finish;
    end
endmodule
```
### 2b. Johnson Counter

```verilog
module johnson_counter (
    output reg [3:0] Q,
    input clk,
    input rst
);
    always @(posedge clk or posedge rst) begin
        if (rst)
            Q <= 4'b0;
        else
            Q <= {~Q[0], Q[3:1]};
    end
endmodule

module johnson_counter_test;
    wire [3:0] Q;
    reg clk;
    reg rst;

    johnson_counter JC (
        .Q(Q),
        .clk(clk),
        .rst(rst)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        #10 rst = 0;
        #200 $finish;
    end
endmodule
```

---
## 3. Magnitude comparator

```verilog
module comparator(
  input [3:0] A, 
  input [3:0] B,
  output A_grt_B, 
  output A_less_B, 
  output A_eq_B
);
  assign A_grt_B = (A > B);
  assign A_less_B = (A < B);
  assign A_eq_B = (A == B);
endmodule

module comparator_tb;
  reg [3:0] A, B;
  wire A_grt_B, A_less_B, A_eq_B;

  comparator comp(
    .A(A), 
    .B(B), 
    .A_grt_B(A_grt_B), 
    .A_less_B(A_less_B), 
    .A_eq_B(A_eq_B)
  );

  initial begin
    $monitor("A = %0d, B = %0d -> A_grt_B = %b, A_less_B = %b, A_eq_B = %b", 
              A, B, A_grt_B, A_less_B, A_eq_B);

    repeat(5) begin
      A = $random % 16;
      B = $random % 16;
      #10;
    end
    $finish;
  end
endmodule
```
---
## 4. Array Multiplier

```verilog
module ha(output c, output s, input a, input b);
  assign c = a & b;
  assign s = a ^ b;
endmodule

module fa(output cout, output s, input a, input b, input cin);
  assign cout = (a&b) | (b&cin) | (a&cin);
  assign s = a ^ b ^ cin;
endmodule

module arr_mul(input [3:0] A, input [3:0] B, output [7:0] z);
  wire [3:0] p[3:0];
  wire [10:0] c;
  wire [5:0] s;
  
  genvar i, j;
  generate
    for (i=0; i<4; i=i+1)
    begin
      for (j=0; j<4; j=j+1)
      begin
        assign p[i][j] = A[i] & B[j];
      end
    end
  endgenerate
  
  assign z[0] = p[0][0];

  // row-1
  ha HA1(c[0], z[1], p[0][1], p[1][0]); 
  fa FA1(c[1], s[0], p[0][2], p[1][1], c[0]);
  fa FA2(c[2], s[1], p[0][3], p[1][2], c[1]);
  ha HA2(c[3], s[2], p[1][3], c[2]);

  // row-2
  ha HA3(c[4], z[2], p[2][0], s[0]);
  fa FA3(c[5], s[3], p[2][1], c[4], s[1]); 
  fa FA4(c[6], s[4], p[2][2], c[5], s[2]);
  fa FA5(c[7], s[5], p[2][3], c[3], c[6]);

  // row-3
  ha HA4(c[8], z[3], p[3][0], s[3]);
  fa FA6(c[9], z[4], p[3][1], c[8], s[4]);
  fa FA7(c[10], z[5], p[3][2], c[9], s[5]);
  ha HA5(z[7], z[6], p[3][3], c[10]);
  
endmodule

module tb_arr_mul;
  reg [3:0] A, B;
  wire [7:0] z;
  
  arr_mul uut(
    .A(A),
    .B(B),
    .z(z)
  );
  
  initial begin
    $monitor("Time=%0t A=%b B=%b z=%b (%d)", $time, A, B, z, z);
    
    A = 4'b0010; B = 4'b0011; #10;
    A = 4'b1100; B = 4'b0101; #10;
    A = 4'b1111; B = 4'b1111; #10;
    A = 4'b0000; B = 4'b1111; #10;
    
    $finish;
  end
endmodule
```
---
## 5. Ripple Carry Adder

```verilog
module FA (
    output s, cout,
    input a, b, cin
);
    assign s = a ^ b ^ cin;
    assign cout = (a & b) | (b & cin) | (cin & a);
endmodule
)
module rca (
    output [3:0] s,
    output cout,
    input [3:0] a, b,
    input cin
);
    wire [2:0] c;

    FA fa1(s[0], c[0], a[0], b[0], cin);
    FA fa2(s[1], c[1], a[1], b[1], c[0]);
    FA fa3(s[2], c[2], a[2], b[2], c[1]);
    FA fa4(s[3], cout, a[3], b[3], c[2]);
endmodule

module rca_tb;
    reg [3:0] a, b;
    reg cin;
    wire [3:0] s;
    wire cout;

    rca uut (s, cout, a, b, cin);

    initial begin
        cin = 0; a = 4'b0110; b = 4'b1100; #10;
        $display("a = %b, b = %b, cin = %b --> sum = %b, cout = %b", a, b, cin, s, cout);

        a = 4'b1110; b = 4'b1000; #10;
        $display("a = %b, b = %b, cin = %b --> sum = %b, cout = %b", a, b, cin, s, cout);

        a = 4'b0111; b = 4'b1110; #10;
        $display("a = %b, b = %b, cin = %b --> sum = %b, cout = %b", a, b, cin, s, cout);

        a = 4'b0010; b = 4'b1001; #10;
        $display("a = %b, b = %b, cin = %b --> sum = %b, cout = %b", a, b, cin, s, cout);

        $stop;
    end
endmodule
```
---
## 6. MUX

```verilog
module mux(input [3:0] i, input [1:0] s, output out);
  assign out =  (~s[0] & ~s[1] & i[0]) |
                (~s[0] & s[1] & i[1]) |
                (s[0] & ~s[1] & i[2]) |
                (s[0] & s[1] & i[3])
endmodule
```

---
## 7. Full Adder and Subtractor

```verilog
module fa(input a,b,cin output s,cout);
  assign s = a ^ b ^ cin;
  assign cout = (a&b) | (b&cin) | (cin&a);
endmodule

module fs(input x,y,bin output d, bout);
  assign d = x ^ y ^ bin;
  assign bout = (~x&y) | (y&bin) | (bin&~x);
endmodule 
```

---
## 8. Flip Flop Conversions
### 8a. SR FF

```verilog
module srff(
    input r, 
    input s, 
    input clk, 
    input rst, 
    output reg q, 
    output qbar
);
    assign qbar = ~q;

    always @(posedge clk or posedge rst) begin
        if (rst)
            q <= 1'b0;  // Reset state
        else begin
            case ({s, r})
                2'b00: q <= q;       // No change
                2'b01: q <= 1'b0;    // Reset
                2'b10: q <= 1'b1;    // Set
                2'b11: q <= 1'bx;       // Invalid input
            endcase
        end
    end
endmodule

module srff_test;
    reg r, s, clk, rst;
    wire q, qbar;

    srff sr1(
        .r(r),
        .s(s),
        .clk(clk),
        .rst(rst),
        .q(q),
        .qbar(qbar)
    );

    always #5 clk = ~clk;

    initial begin
        r = 0; s = 0; clk = 0; rst = 1; #10;
        rst = 0; #10;
        r = 0; s = 1; #10;
        r = 1; s = 0; #10;
        r = 1; s = 1; #50;
        $stop;
    end
endmodule
```

### 8b. JK FF

```verilog
module jkff(
    input j, 
    input k, 
    input clk, 
    input rst, 
    output reg q, 
    output qbar
);
    assign qbar = ~q;

    always @(posedge clk or posedge rst) begin
        if (rst)
            q <= 1'b0;
        else begin
            case ({j, k})
                2'b00: q <= q;       // No change
                2'b01: q <= 1'b0;    // Reset
                2'b10: q <= 1'b1;    // Set
                2'b11: q <= ~q;      // Toggle
            endcase
        end
    end
endmodule

module jkff_test;
    reg j, k, clk, rst;
    wire q, qbar;

    jkff jk (
        .j(j), 
        .k(k), 
        .clk(clk), 
        .rst(rst), 
        .q(q), 
        .qbar(qbar)
    );

    always #5 clk = ~clk;

    initial begin
        j = 0; k = 0;
        clk = 0; rst = 1;
        #10 rst = 0;
        #10;
        j = 1; k = 0; #10;
        j = 0; k = 1; #10;
        j = 1; k = 1; #50;
        $stop;
    end
endmodule
```

### 8c. D FF

```verilog
module dff(
    input d, 
    input clk, 
    input rst, 
    output reg q, 
    output qbar
);
    assign qbar = ~q;

    always @(posedge clk or posedge rst) begin
        if (rst)
            q <= 1'b0;
        else
            q <= d;
    end
endmodule

module dff_test;
    reg d, clk, rst;
    wire q, qbar;

    dff d1 (
        .d(d), 
        .clk(clk), 
        .rst(rst), 
        .q(q), 
        .qbar(qbar)
    );

    always #5 clk = ~clk;

    initial begin
        d = 1; clk = 0; rst = 1; #10;
        rst = 0; #10;
        d = 0; #10;
        d = 1; #50;
        $stop;
    end
endmodule
```

### 8d. T FF

```verilog
module tff(
    input t, 
    input clk, 
    input rst, 
    output reg q, 
    output qbar
);
    assign qbar = ~q;

    always @(posedge clk or posedge rst) begin
        if (rst)
            q <= 1'b0;
        else if (t)
            q <= ~q;
    end
endmodule

module tff_test;
    reg t, clk, rst;
    wire q, qbar;

    tff t1 (
        .t(t), 
        .clk(clk), 
        .rst(rst), 
        .q(q), 
        .qbar(qbar)
    );

    always #5 clk = ~clk;

    initial begin
        t = 1; clk = 0; rst = 1; #10;
        rst = 0; #10;
        t = 0; #10;
        t = 1; #50;
        $stop;
    end
endmodule
```
---
## 9. Ripple Up and Ripple Down Counters
### 9a. Ripple Up Counter

```verilog
module tff(
    output reg Q,
    output Qbar,
    input T,
    input clk,
    input rst
);
    assign Qbar = ~Q;

    always @(posedge clk or posedge rst) begin
        if (rst)
            Q <= 1'b0;
        else if (T)
            Q <= ~Q;
    end
endmodule

module ripple_up(
    output [3:0] Q,
    input clk,
    input rst
);
    wire [3:0] Qbar;

    tff t0(Q[0], Qbar[0], 1'b1, clk, rst);
    tff t1(Q[1], Qbar[1], 1'b1, Q[0], rst);
    tff t2(Q[2], Qbar[2], 1'b1, Q[1], rst);
    tff t3(Q[3], Qbar[3], 1'b1, Q[2], rst);
endmodule

module ripple_up_test;
    wire [3:0] Q;
    reg clk;
    reg rst;

    ripple_up RU(
        .Q(Q),
        .clk(clk),
        .rst(rst)
    );

    always #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        #10 rst = 0;
        #200 $finish;
    end
endmodule
```
### 9b. Ripple Down Counter

```verilog
module tff(
    output reg Q,
    output Qbar,
    input T,
    input clk,
    input rst
);
    assign Qbar = ~Q;

    always @(posedge clk or posedge rst) begin
        if (rst)
            Q <= 1'b0;
        else if (T)
            Q <= ~Q;
    end
endmodule

module ripple_down(output [3:0] Q, input clk, input rst);
  wire [3:0] Qbar;
  
  tff t0(Q[0], Qbar[0], 1'b1, clk, rst);
  tff t1(Q[1], Qbar[1], 1'b1, Qbar[0], rst);
  tff t2(Q[2], Qbar[2], 1'b1, Qbar[1], rst);
  tff t3(Q[3], Qbar[3], 1'b1, Qbar[2], rst);
endmodule

module ripple_down_test;
  wire [3:0] Q;
  reg clk;
  reg rst;

  ripple_down RD(.Q(Q), .clk(clk), .rst(rst));
  
  always #5 clk = ~clk;
  
  initial begin
    clk = 0;
    rst = 1;
    #10 rst = 0;
    #200 $finish;
  end
endmodule
```
---
## 10. Encoder and Decoder

```verilog
module encoder_4to2(output [1:0] o, input [3:0] i);
  assign o[0] = i[1] | i[3];
  assign o[1] = i[2] | i[3];
endmodule

module decode_2to4(input [1:0] i, output [3:0] o);
  assign o[0] = ~i[1] & ~i[0];
  assign o[1] = ~i[1] & i[0];
  assign o[2] = i[1] & ~i[0];
  assign o[3] = i[1] & i[0];
endmodule

module encoder_8to3(output [2:0] o, input [7:0] i);
  assign o[0] = i[1] | i[3] | i[5] | i[7];
  assign o[1] = i[2] | i[3] | i[6] | i[7];
  assign o[2] = i[4] | i[5] | i[6] | i[7];
endmodule

module decoder_3to8(input [2:0] i, output [7:0] o);
  assign o[0] = ~i[2] & ~i[1] & ~i[0];
  assign o[1] = ~i[2] & ~i[1] &  i[0];
  assign o[2] = ~i[2] &  i[1] & ~i[0];
  assign o[3] = ~i[2] &  i[1] &  i[0];
  assign o[4] =  i[2] & ~i[1] & ~i[0];
  assign o[5] =  i[2] & ~i[1] &  i[0];
  assign o[6] =  i[2] &  i[1] & ~i[0];
  assign o[7] =  i[2] &  i[1] &  i[0];
endmodule
```

---
## Synchronous Counters
### a. Up Counter

```verilog
module up_counter (
    output reg [3:0] Q,
    input clk,
    input rst
);
    always @ (posedge clk or posedge rst) begin
        if (rst) 
            Q <= 4'b0;
        else 
            Q <= Q + 1;
    end
endmodule

module up_counter_test;
    wire [3:0] Q;
    reg clk;
    reg rst;

    up_counter c1 (
        .Q(Q), 
        .clk(clk), 
        .rst(rst)
    );

    always 
        #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        #10 rst = 0;
        #200 $stop;
    end
endmodule

```

### b. Down Counter

```verilog
module down_counter (
    output reg [3:0] Q,
    input clk,
    input rst
);
    always @ (posedge clk or posedge rst) begin
        if (rst) 
            Q <= 4'b1111;
        else 
            Q <= Q - 1;
    end
endmodule

module down_counter_test;
    wire [3:0] Q;
    reg clk;
    reg rst;

    down_counter c1 (
        .Q(Q), 
        .clk(clk), 
        .rst(rst)
    );

    always 
        #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        #10 rst = 0;
        #200 $stop;
    end
endmodule

```

### c. Up-Down Counter

```verilog
module up_down_counter (
    output reg [3:0] Q,
    input clk,
    input rst,
    input up_down
);
    always @ (posedge clk or posedge rst) begin
        if (rst) 
            Q <= 4'b0;
        else if (up_down)
            Q <= Q + 1;
        else
            Q <= Q - 1;
    end
endmodule

module up_down_counter_test;
    wire [3:0] Q;
    reg clk;
    reg rst;
    reg up_down;

    up_down_counter c1 (
        .Q(Q), 
        .clk(clk), 
        .rst(rst),
        .up_down(up_down)
    );

    always 
        #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        up_down = 1;
        #10 rst = 0;
        #50 up_down = 0;
        #50 up_down = 1;
        #200 $stop;
    end
endmodule

```

### d. Mod-N Counter

```verilog
module mod_n_counter (
    output reg [3:0] Q,
    input clk,
    input rst,
    input [3:0] n
);
    always @ (posedge clk or posedge rst) begin
        if (rst) 
            Q <= 4'b0;
        else if (Q == n - 1) 
            Q <= 4'b0;
        else 
            Q <= Q + 1;
    end
endmodule

module mod_n_counter_test;
    wire [3:0] Q;
    reg clk;
    reg rst;
    reg [3:0] n;

    mod_n_counter c1 (
        .Q(Q), 
        .clk(clk), 
        .rst(rst),
        .n(n)
    );

    always 
        #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        n = 4'b0100; // Mod 4 counter
        #10 rst = 0;
        #100 $stop;
    end
endmodule

```
