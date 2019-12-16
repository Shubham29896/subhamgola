module Quartus_architecture_code(clk,input_A,input_B,output_sum);
  input clk;
  input [63:0]input_A,input_B;
  output [63:0]output_sum;
  reg [63:0]output_sum;
  reg [63:0]indata_a,indata_b;
  reg [53:0]finalsum_mantissa,final_mantissa;
  reg [54:0]result_mantissa;
  reg [10:0]result_exponent,exponent_reg_a,exponent_reg_b,bias_exponent_bit;
  reg [10:0]bias_reg,bias_exponent_0,bias_exponent_1;
  reg [52:0]mantissa_bit0,mantissa_bit1,mantissa_bit_a,mantissa_bit_b;
  wire [52:0]sum_mantissa;
  wire carry;
  reg [10:0]temp1;
  reg temp0,complement_flag_a,complement_flag_b;
  reg [63:0]data_a;
  reg [63:0]data_b;
  reg [52:0]local2;
  reg [63:0]nor_data_a,nor_data_b;
  reg fcarry,sign_flag;
  reg [52:0]fsum_mantissa;
  reg overflow;
  reg underflow;
  //cda_uHA csa(.In1(mantissa_bit0),.In2(mantissa_bit1),.out_sum(sum_mantissa),.finalcarry(carry));
  //carry_select_adder csa(.In1(mantissa_bit0),.In2(mantissa_bit1),.out_sum(sum_mantissa),.finalcarry(carry));
  new_adder na(.In1(mantissa_bit0),.In2(mantissa_bit1),.out_sum(sum_mantissa),.finalcarry(carry));
/*initial
begin
  rst_in=1'b0;
  clk=1'b0;
end
always
begin
#1 clk=~clk;
end*/
always@(posedge clk)
begin
      indata_a<=input_A;
      indata_b<=input_B;
      bias_reg<=11'b01111111111;
      exponent_reg_a<=indata_a[62:52];
      exponent_reg_b<=indata_b[62:52];
      fcarry<=carry;
      fsum_mantissa<=sum_mantissa;
end
always@(posedge clk)
begin
  temp0<=(exponent_reg_a+bias_reg)<(exponent_reg_b+bias_reg);
  if(temp0)
    begin
      data_a<=indata_b;
      data_b<=indata_a;
      bias_exponent_0<=(exponent_reg_b+bias_reg);//data exchange
      bias_exponent_1<=(exponent_reg_a+bias_reg);
      bias_exponent_bit<=(exponent_reg_b+bias_reg);
    end
  else
    begin
      data_a<=indata_a;
      data_b<=indata_b;
      bias_exponent_0<=(exponent_reg_a+bias_reg);
      bias_exponent_1<=(exponent_reg_b+bias_reg);      
      bias_exponent_bit<=(exponent_reg_a+bias_reg);
    end
end
always@(posedge clk)
begin
    if(bias_exponent_0>bias_exponent_1)
      temp1<=bias_exponent_0-bias_exponent_1;
    else
      temp1<=bias_exponent_1-bias_exponent_0;
    if(temp1!=11'b0)
      begin
        mantissa_bit0<={1'b1,data_a[51:0]};
        local2<={1'b1,data_b[51:0]};
        mantissa_bit1<=local2>>temp1;
      if(data_a[63]==1'b1)
        complement_flag_a<=1'b1;
      else
        complement_flag_a<=1'b0;
      mantissa_bit_a <= complement_flag_a ? (~(mantissa_bit0)+53'b1):mantissa_bit0;
		if(data_b[63]==1'b1)
        complement_flag_b<=1'b1;
      else
        complement_flag_b<=1'b0;
		mantissa_bit_b <= complement_flag_b ? (~(mantissa_bit1)+53'b1):mantissa_bit1;
      nor_data_a<={data_a[63],bias_exponent_bit,mantissa_bit_a[51:0]};
      nor_data_b<={data_b[63],bias_exponent_bit,mantissa_bit_b[51:0]};
      end
      else
      begin
        mantissa_bit0<={1'b1,data_a[51:0]};
        mantissa_bit1<={1'b1,data_b[51:0]};
      if(data_a[63]==1'b1)
        complement_flag_a<=1'b1;
      else
        complement_flag_a<=1'b0;
      mantissa_bit_a <= complement_flag_a ? (~(mantissa_bit0)+53'b1):mantissa_bit0;
		if(data_b[63]==1'b1)
        complement_flag_b<=1'b1;
      else
        complement_flag_b<=1'b0;
		mantissa_bit_b <= complement_flag_b ? (~(mantissa_bit1)+53'b1):mantissa_bit1;
      nor_data_a<={data_a[63],bias_exponent_bit,mantissa_bit_a[51:0]};
      nor_data_b<={data_b[63],bias_exponent_bit,mantissa_bit_b[51:0]};
      end
end
always@(posedge clk)
begin
  finalsum_mantissa<={fcarry,fsum_mantissa};
  if(finalsum_mantissa[53]==1'b1)
    begin
      result_mantissa<={1'b0,finalsum_mantissa};
      result_exponent<=bias_exponent_bit+11'b1;
      if(result_mantissa[0]==1'b1 && result_mantissa[1]==1'b1)
        final_mantissa<=result_mantissa[54:1]+53'b1;
      else
        final_mantissa<=result_mantissa[54:1];
      if(data_a[51:0]<data_b[51:0])
        sign_flag<=1'b1;
      else
        sign_flag<=1'b0;
      if(sign_flag)
        output_sum<={data_b[63],result_exponent,final_mantissa[51:0]};
      else
        output_sum<={data_a[63],result_exponent,final_mantissa[51:0]};
    end
    else
      begin
      if(data_a[51:0]<data_b[51:0])
        output_sum<={data_b[63],bias_exponent_bit,finalsum_mantissa[51:0]};
      else
        output_sum<={data_a[63],bias_exponent_bit,finalsum_mantissa[51:0]};  
      if(output_sum[62:52]>=11'b11111111111)
        overflow<=1'b1;
      else
        overflow<=1'b1;
      if(output_sum[62:52]<=11'b00000000001)
        underflow<=1'b1;
      else
        underflow<=1'b0;
     end
end
endmodule
      

/*module cda_uHA(In1,In2,out_sum,finalcarry);
  input [52:0]In1,In2;
  output [52:0]out_sum;
  output finalcarry;
  wire [52:1]wsha,wsbec,wcbec;
  wire [52:0] wcha;
  wire [52:1]wcm;

half_adder ha0(.A(In1[0]),.B(In2[0]),.Sum(out_sum[0]),.Carry(wcha[0]));
  
half_adder ha1(.A(In1[1]),.B(In2[1]),.Sum(wsha[1]),.Carry(wcha[1]));
not(wsbec[1],wsha[1]);
xor(wcbec[1],wsha[1],wcha[1]);
mux_rsr mxs1(.in0(wsha[1]),.in1(wsbec[1]),.out(out_sum[1]),.sel(wcha[0]));
mux_rsr mxc1(.in0(wcha[1]),.in1(wcbec[1]),.out(wcm[1]),.sel(wcha[0]));


half_adder ha2(.A(In1[2]),.B(In2[2]),.Sum(wsha[2]),.Carry(wcha[2]));
not(wsbec[2],wsha[2]);
xor(wcbec[2],wsha[2],wcha[2]);
mux_rsr mxs2(.in0(wsha[2]),.in1(wsbec[2]),.out(out_sum[2]),.sel(wcm[1]));
mux_rsr mxc2(.in0(wcha[2]),.in1(wcbec[2]),.out(wcm[2]),.sel(wcm[1]));


half_adder ha3(.A(In1[3]),.B(In2[3]),.Sum(wsha[3]),.Carry(wcha[3]));
not(wsbec[3],wsha[3]);
xor(wcbec[3],wsha[3],wcha[3]);
mux_rsr mxs3(.in0(wsha[3]),.in1(wsbec[3]),.out(out_sum[3]),.sel(wcm[2]));
mux_rsr mxc3(.in0(wcha[3]),.in1(wcbec[3]),.out(wcm[3]),.sel(wcm[2]));


half_adder ha4(.A(In1[4]),.B(In2[4]),.Sum(wsha[4]),.Carry(wcha[4]));
not(wsbec[4],wsha[4]);
xor(wcbec[4],wsha[4],wcha[4]);
mux_rsr mxs4(.in0(wsha[4]),.in1(wsbec[4]),.out(out_sum[4]),.sel(wcm[3]));
mux_rsr mxc4(.in0(wcha[4]),.in1(wcbec[4]),.out(wcm[4]),.sel(wcm[3]));


half_adder ha5(.A(In1[5]),.B(In2[5]),.Sum(wsha[5]),.Carry(wcha[5]));
not(wsbec[5],wsha[5]);
xor(wcbec[5],wsha[5],wcha[5]);
mux_rsr mxs5(.in0(wsha[5]),.in1(wsbec[5]),.out(out_sum[5]),.sel(wcm[4]));
mux_rsr mxc5(.in0(wcha[5]),.in1(wcbec[5]),.out(wcm[5]),.sel(wcm[4]));


half_adder ha6(.A(In1[6]),.B(In2[6]),.Sum(wsha[6]),.Carry(wcha[6]));
not(wsbec[6],wsha[6]);
xor(wcbec[6],wsha[6],wcha[6]);
mux_rsr mxs6(.in0(wsha[6]),.in1(wsbec[6]),.out(out_sum[6]),.sel(wcm[5]));
mux_rsr mxc6(.in0(wcha[6]),.in1(wcbec[6]),.out(wcm[6]),.sel(wcm[5]));


half_adder ha7(.A(In1[7]),.B(In2[7]),.Sum(wsha[7]),.Carry(wcha[7]));
not(wsbec[7],wsha[7]);
xor(wcbec[7],wsha[7],wcha[7]);
mux_rsr mxs7(.in0(wsha[7]),.in1(wsbec[7]),.out(out_sum[7]),.sel(wcm[6]));
mux_rsr mxc7(.in0(wcha[7]),.in1(wcbec[7]),.out(wcm[7]),.sel(wcm[6]));


half_adder ha8(.A(In1[8]),.B(In2[8]),.Sum(wsha[8]),.Carry(wcha[8]));
not(wsbec[8],wsha[8]);
xor(wcbec[8],wsha[8],wcha[8]);
mux_rsr mxs8(.in0(wsha[8]),.in1(wsbec[8]),.out(out_sum[8]),.sel(wcm[7]));
mux_rsr mxc8(.in0(wcha[8]),.in1(wcbec[8]),.out(wcm[8]),.sel(wcm[7]));


half_adder ha9(.A(In1[9]),.B(In2[9]),.Sum(wsha[9]),.Carry(wcha[9]));
not(wsbec[9],wsha[9]);
xor(wcbec[9],wsha[9],wcha[9]);
mux_rsr mxs9(.in0(wsha[9]),.in1(wsbec[9]),.out(out_sum[9]),.sel(wcm[8]));
mux_rsr mxc9(.in0(wcha[9]),.in1(wcbec[9]),.out(wcm[9]),.sel(wcm[8]));


half_adder ha10(.A(In1[10]),.B(In2[10]),.Sum(wsha[10]),.Carry(wcha[10]));
not(wsbec[10],wsha[10]);
xor(wcbec[10],wsha[10],wcha[10]);
mux_rsr mxs10(.in0(wsha[10]),.in1(wsbec[10]),.out(out_sum[10]),.sel(wcm[9]));
mux_rsr mxc10(.in0(wcha[10]),.in1(wcbec[10]),.out(wcm[10]),.sel(wcm[9]));


half_adder ha11(.A(In1[11]),.B(In2[11]),.Sum(wsha[11]),.Carry(wcha[11]));
not(wsbec[11],wsha[11]);
xor(wcbec[11],wsha[11],wcha[11]);
mux_rsr mxs11(.in0(wsha[11]),.in1(wsbec[11]),.out(out_sum[11]),.sel(wcm[10]));
mux_rsr mxc11(.in0(wcha[11]),.in1(wcbec[11]),.out(wcm[11]),.sel(wcm[10]));


half_adder ha12(.A(In1[12]),.B(In2[12]),.Sum(wsha[12]),.Carry(wcha[12]));
not(wsbec[12],wsha[12]);
xor(wcbec[12],wsha[12],wcha[12]);
mux_rsr mxs12(.in0(wsha[12]),.in1(wsbec[12]),.out(out_sum[12]),.sel(wcm[11]));
mux_rsr mxc12(.in0(wcha[12]),.in1(wcbec[12]),.out(wcm[12]),.sel(wcm[11]));


half_adder ha13(.A(In1[13]),.B(In2[13]),.Sum(wsha[13]),.Carry(wcha[13]));
not(wsbec[13],wsha[13]);
xor(wcbec[13],wsha[13],wcha[13]);
mux_rsr mxs13(.in0(wsha[13]),.in1(wsbec[13]),.out(out_sum[13]),.sel(wcm[12]));
mux_rsr mxc13(.in0(wcha[13]),.in1(wcbec[13]),.out(wcm[13]),.sel(wcm[12]));


half_adder ha14(.A(In1[14]),.B(In2[14]),.Sum(wsha[14]),.Carry(wcha[14]));
not(wsbec[14],wsha[14]);
xor(wcbec[14],wsha[14],wcha[14]);
mux_rsr mxs14(.in0(wsha[14]),.in1(wsbec[14]),.out(out_sum[14]),.sel(wcm[13]));
mux_rsr mxc14(.in0(wcha[14]),.in1(wcbec[14]),.out(wcm[14]),.sel(wcm[13]));


half_adder ha15(.A(In1[15]),.B(In2[15]),.Sum(wsha[15]),.Carry(wcha[15]));
not(wsbec[15],wsha[15]);
xor(wcbec[15],wsha[15],wcha[15]);
mux_rsr mxs15(.in0(wsha[15]),.in1(wsbec[15]),.out(out_sum[15]),.sel(wcm[14]));
mux_rsr mxc15(.in0(wcha[15]),.in1(wcbec[15]),.out(wcm[15]),.sel(wcm[14]));


half_adder ha16(.A(In1[16]),.B(In2[16]),.Sum(wsha[16]),.Carry(wcha[16]));
not(wsbec[16],wsha[16]);
xor(wcbec[16],wsha[16],wcha[16]);
mux_rsr mxs16(.in0(wsha[16]),.in1(wsbec[16]),.out(out_sum[16]),.sel(wcm[15]));
mux_rsr mxc16(.in0(wcha[16]),.in1(wcbec[16]),.out(wcm[16]),.sel(wcm[15]));


half_adder ha17(.A(In1[17]),.B(In2[17]),.Sum(wsha[17]),.Carry(wcha[17]));
not(wsbec[17],wsha[17]);
xor(wcbec[17],wsha[17],wcha[17]);
mux_rsr mxs17(.in0(wsha[17]),.in1(wsbec[17]),.out(out_sum[17]),.sel(wcm[16]));
mux_rsr mxc17(.in0(wcha[17]),.in1(wcbec[17]),.out(wcm[17]),.sel(wcm[16]));


half_adder ha18(.A(In1[18]),.B(In2[18]),.Sum(wsha[18]),.Carry(wcha[18]));
not(wsbec[18],wsha[18]);
xor(wcbec[18],wsha[18],wcha[18]);
mux_rsr mxs18(.in0(wsha[18]),.in1(wsbec[18]),.out(out_sum[18]),.sel(wcm[17]));
mux_rsr mxc18(.in0(wcha[18]),.in1(wcbec[18]),.out(wcm[18]),.sel(wcm[17]));


half_adder ha19(.A(In1[19]),.B(In2[19]),.Sum(wsha[19]),.Carry(wcha[19]));
not(wsbec[19],wsha[19]);
xor(wcbec[19],wsha[19],wcha[19]);
mux_rsr mxs19(.in0(wsha[19]),.in1(wsbec[19]),.out(out_sum[19]),.sel(wcm[18]));
mux_rsr mxc19(.in0(wcha[19]),.in1(wcbec[19]),.out(wcm[19]),.sel(wcm[18]));


half_adder ha20(.A(In1[20]),.B(In2[20]),.Sum(wsha[20]),.Carry(wcha[20]));
not(wsbec[20],wsha[20]);
xor(wcbec[20],wsha[20],wcha[20]);
mux_rsr mxs20(.in0(wsha[20]),.in1(wsbec[20]),.out(out_sum[20]),.sel(wcm[19]));
mux_rsr mxc20(.in0(wcha[20]),.in1(wcbec[20]),.out(wcm[20]),.sel(wcm[19]));


half_adder ha21(.A(In1[21]),.B(In2[21]),.Sum(wsha[21]),.Carry(wcha[21]));
not(wsbec[21],wsha[21]);
xor(wcbec[21],wsha[21],wcha[21]);
mux_rsr mxs21(.in0(wsha[21]),.in1(wsbec[21]),.out(out_sum[21]),.sel(wcm[20]));
mux_rsr mxc21(.in0(wcha[21]),.in1(wcbec[21]),.out(wcm[21]),.sel(wcm[20]));


half_adder ha22(.A(In1[22]),.B(In2[22]),.Sum(wsha[22]),.Carry(wcha[22]));
not(wsbec[22],wsha[22]);
xor(wcbec[22],wsha[22],wcha[22]);
mux_rsr mxs22(.in0(wsha[22]),.in1(wsbec[22]),.out(out_sum[22]),.sel(wcm[21]));
mux_rsr mxc22(.in0(wcha[22]),.in1(wcbec[22]),.out(wcm[22]),.sel(wcm[21]));


half_adder ha23(.A(In1[23]),.B(In2[23]),.Sum(wsha[23]),.Carry(wcha[23]));
not(wsbec[23],wsha[23]);
xor(wcbec[23],wsha[23],wcha[23]);
mux_rsr mxs23(.in0(wsha[23]),.in1(wsbec[23]),.out(out_sum[23]),.sel(wcm[22]));
mux_rsr mxc23(.in0(wcha[23]),.in1(wcbec[23]),.out(wcm[23]),.sel(wcm[22]));


half_adder ha24(.A(In1[24]),.B(In2[24]),.Sum(wsha[24]),.Carry(wcha[24]));
not(wsbec[24],wsha[24]);
xor(wcbec[24],wsha[24],wcha[24]);
mux_rsr mxs24(.in0(wsha[24]),.in1(wsbec[24]),.out(out_sum[24]),.sel(wcm[23]));
mux_rsr mxc24(.in0(wcha[24]),.in1(wcbec[24]),.out(wcm[24]),.sel(wcm[23]));


half_adder ha25(.A(In1[25]),.B(In2[25]),.Sum(wsha[25]),.Carry(wcha[25]));
not(wsbec[25],wsha[25]);
xor(wcbec[25],wsha[25],wcha[25]);
mux_rsr mxs25(.in0(wsha[25]),.in1(wsbec[25]),.out(out_sum[25]),.sel(wcm[24]));
mux_rsr mxc25(.in0(wcha[25]),.in1(wcbec[25]),.out(wcm[25]),.sel(wcm[24]));


half_adder ha26(.A(In1[26]),.B(In2[26]),.Sum(wsha[26]),.Carry(wcha[26]));
not(wsbec[26],wsha[26]);
xor(wcbec[26],wsha[26],wcha[26]);
mux_rsr mxs26(.in0(wsha[26]),.in1(wsbec[26]),.out(out_sum[26]),.sel(wcm[25]));
mux_rsr mxc26(.in0(wcha[26]),.in1(wcbec[26]),.out(wcm[26]),.sel(wcm[25]));


half_adder ha27(.A(In1[27]),.B(In2[27]),.Sum(wsha[27]),.Carry(wcha[27]));
not(wsbec[27],wsha[27]);
xor(wcbec[27],wsha[27],wcha[27]);
mux_rsr mxs27(.in0(wsha[27]),.in1(wsbec[27]),.out(out_sum[27]),.sel(wcm[26]));
mux_rsr mxc27(.in0(wcha[27]),.in1(wcbec[27]),.out(wcm[27]),.sel(wcm[26]));


half_adder ha28(.A(In1[28]),.B(In2[28]),.Sum(wsha[28]),.Carry(wcha[28]));
not(wsbec[28],wsha[28]);
xor(wcbec[28],wsha[28],wcha[28]);
mux_rsr mxs28(.in0(wsha[28]),.in1(wsbec[28]),.out(out_sum[28]),.sel(wcm[27]));
mux_rsr mxc28(.in0(wcha[28]),.in1(wcbec[28]),.out(wcm[28]),.sel(wcm[27]));


half_adder ha29(.A(In1[29]),.B(In2[29]),.Sum(wsha[29]),.Carry(wcha[29]));
not(wsbec[29],wsha[29]);
xor(wcbec[29],wsha[29],wcha[29]);
mux_rsr mxs29(.in0(wsha[29]),.in1(wsbec[29]),.out(out_sum[29]),.sel(wcm[28]));
mux_rsr mxc29(.in0(wcha[29]),.in1(wcbec[29]),.out(wcm[29]),.sel(wcm[28]));


half_adder ha30(.A(In1[30]),.B(In2[30]),.Sum(wsha[30]),.Carry(wcha[30]));
not(wsbec[30],wsha[30]);
xor(wcbec[30],wsha[30],wcha[30]);
mux_rsr mxs30(.in0(wsha[30]),.in1(wsbec[30]),.out(out_sum[30]),.sel(wcm[29]));
mux_rsr mxc30(.in0(wcha[30]),.in1(wcbec[30]),.out(wcm[30]),.sel(wcm[29]));


half_adder ha31(.A(In1[31]),.B(In2[31]),.Sum(wsha[31]),.Carry(wcha[31]));
not(wsbec[31],wsha[31]);
xor(wcbec[31],wsha[31],wcha[31]);
mux_rsr mxs31(.in0(wsha[31]),.in1(wsbec[31]),.out(out_sum[31]),.sel(wcm[30]));
mux_rsr mxc31(.in0(wcha[31]),.in1(wcbec[31]),.out(wcm[31]),.sel(wcm[30]));


half_adder ha32(.A(In1[32]),.B(In2[32]),.Sum(wsha[32]),.Carry(wcha[32]));
not(wsbec[32],wsha[32]);
xor(wcbec[32],wsha[32],wcha[32]);
mux_rsr mxs32(.in0(wsha[32]),.in1(wsbec[32]),.out(out_sum[32]),.sel(wcm[31]));
mux_rsr mxc32(.in0(wcha[32]),.in1(wcbec[32]),.out(wcm[32]),.sel(wcm[31]));


half_adder ha33(.A(In1[33]),.B(In2[33]),.Sum(wsha[33]),.Carry(wcha[33]));
not(wsbec[33],wsha[33]);
xor(wcbec[33],wsha[33],wcha[33]);
mux_rsr mxs33(.in0(wsha[33]),.in1(wsbec[33]),.out(out_sum[33]),.sel(wcm[32]));
mux_rsr mxc33(.in0(wcha[33]),.in1(wcbec[33]),.out(wcm[33]),.sel(wcm[32]));


half_adder ha34(.A(In1[34]),.B(In2[34]),.Sum(wsha[34]),.Carry(wcha[34]));
not(wsbec[34],wsha[34]);
xor(wcbec[34],wsha[34],wcha[34]);
mux_rsr mxs34(.in0(wsha[34]),.in1(wsbec[34]),.out(out_sum[34]),.sel(wcm[33]));
mux_rsr mxc34(.in0(wcha[34]),.in1(wcbec[34]),.out(wcm[34]),.sel(wcm[33]));


half_adder ha35(.A(In1[35]),.B(In2[35]),.Sum(wsha[35]),.Carry(wcha[35]));
not(wsbec[35],wsha[35]);
xor(wcbec[35],wsha[35],wcha[35]);
mux_rsr mxs35(.in0(wsha[35]),.in1(wsbec[35]),.out(out_sum[35]),.sel(wcm[34]));
mux_rsr mxc35(.in0(wcha[35]),.in1(wcbec[35]),.out(wcm[35]),.sel(wcm[34]));


half_adder ha36(.A(In1[36]),.B(In2[36]),.Sum(wsha[36]),.Carry(wcha[36]));
not(wsbec[36],wsha[36]);
xor(wcbec[36],wsha[36],wcha[36]);
mux_rsr mxs36(.in0(wsha[36]),.in1(wsbec[36]),.out(out_sum[36]),.sel(wcm[35]));
mux_rsr mxc36(.in0(wcha[36]),.in1(wcbec[36]),.out(wcm[36]),.sel(wcm[35]));


half_adder ha37(.A(In1[37]),.B(In2[37]),.Sum(wsha[37]),.Carry(wcha[37]));
not(wsbec[37],wsha[37]);
xor(wcbec[37],wsha[37],wcha[37]);
mux_rsr mxs37(.in0(wsha[37]),.in1(wsbec[37]),.out(out_sum[37]),.sel(wcm[36]));
mux_rsr mxc37(.in0(wcha[37]),.in1(wcbec[37]),.out(wcm[37]),.sel(wcm[36]));


half_adder ha38(.A(In1[38]),.B(In2[38]),.Sum(wsha[38]),.Carry(wcha[38]));
not(wsbec[38],wsha[38]);
xor(wcbec[38],wsha[38],wcha[38]);
mux_rsr mxs38(.in0(wsha[38]),.in1(wsbec[38]),.out(out_sum[38]),.sel(wcm[37]));
mux_rsr mxc38(.in0(wcha[38]),.in1(wcbec[38]),.out(wcm[38]),.sel(wcm[37]));


half_adder ha39(.A(In1[39]),.B(In2[39]),.Sum(wsha[39]),.Carry(wcha[39]));
not(wsbec[39],wsha[39]);
xor(wcbec[39],wsha[39],wcha[39]);
mux_rsr mxs39(.in0(wsha[39]),.in1(wsbec[39]),.out(out_sum[39]),.sel(wcm[38]));
mux_rsr mxc39(.in0(wcha[39]),.in1(wcbec[39]),.out(wcm[39]),.sel(wcm[38]));


half_adder ha40(.A(In1[40]),.B(In2[40]),.Sum(wsha[40]),.Carry(wcha[40]));
not(wsbec[40],wsha[40]);
xor(wcbec[40],wsha[40],wcha[40]);
mux_rsr mxs40(.in0(wsha[40]),.in1(wsbec[40]),.out(out_sum[40]),.sel(wcm[39]));
mux_rsr mxc40(.in0(wcha[40]),.in1(wcbec[40]),.out(wcm[40]),.sel(wcm[39]));


half_adder ha41(.A(In1[41]),.B(In2[41]),.Sum(wsha[41]),.Carry(wcha[41]));
not(wsbec[41],wsha[41]);
xor(wcbec[41],wsha[41],wcha[41]);
mux_rsr mxs41(.in0(wsha[41]),.in1(wsbec[41]),.out(out_sum[41]),.sel(wcm[40]));
mux_rsr mxc41(.in0(wcha[41]),.in1(wcbec[41]),.out(wcm[41]),.sel(wcm[40]));


half_adder ha42(.A(In1[42]),.B(In2[42]),.Sum(wsha[42]),.Carry(wcha[42]));
not(wsbec[42],wsha[42]);
xor(wcbec[42],wsha[42],wcha[42]);
mux_rsr mxs42(.in0(wsha[42]),.in1(wsbec[42]),.out(out_sum[42]),.sel(wcm[41]));
mux_rsr mxc42(.in0(wcha[42]),.in1(wcbec[42]),.out(wcm[42]),.sel(wcm[41]));


half_adder ha43(.A(In1[43]),.B(In2[43]),.Sum(wsha[43]),.Carry(wcha[43]));
not(wsbec[43],wsha[43]);
xor(wcbec[43],wsha[43],wcha[43]);
mux_rsr mxs43(.in0(wsha[43]),.in1(wsbec[43]),.out(out_sum[43]),.sel(wcm[42]));
mux_rsr mxc43(.in0(wcha[43]),.in1(wcbec[43]),.out(wcm[43]),.sel(wcm[42]));


half_adder ha44(.A(In1[44]),.B(In2[44]),.Sum(wsha[44]),.Carry(wcha[44]));
not(wsbec[44],wsha[44]);
xor(wcbec[44],wsha[44],wcha[44]);
mux_rsr mxs44(.in0(wsha[44]),.in1(wsbec[44]),.out(out_sum[44]),.sel(wcm[43]));
mux_rsr mxc44(.in0(wcha[44]),.in1(wcbec[44]),.out(wcm[44]),.sel(wcm[43]));


half_adder ha45(.A(In1[45]),.B(In2[45]),.Sum(wsha[45]),.Carry(wcha[45]));
not(wsbec[45],wsha[45]);
xor(wcbec[45],wsha[45],wcha[45]);
mux_rsr mxs45(.in0(wsha[45]),.in1(wsbec[45]),.out(out_sum[45]),.sel(wcm[44]));
mux_rsr mxc45(.in0(wcha[45]),.in1(wcbec[45]),.out(wcm[45]),.sel(wcm[44]));


half_adder ha46(.A(In1[46]),.B(In2[46]),.Sum(wsha[46]),.Carry(wcha[46]));
not(wsbec[46],wsha[46]);
xor(wcbec[46],wsha[46],wcha[46]);
mux_rsr mxs46(.in0(wsha[46]),.in1(wsbec[46]),.out(out_sum[46]),.sel(wcm[45]));
mux_rsr mxc46(.in0(wcha[46]),.in1(wcbec[46]),.out(wcm[46]),.sel(wcm[45]));


half_adder ha47(.A(In1[47]),.B(In2[47]),.Sum(wsha[47]),.Carry(wcha[47]));
not(wsbec[47],wsha[47]);
xor(wcbec[47],wsha[47],wcha[47]);
mux_rsr mxs47(.in0(wsha[47]),.in1(wsbec[47]),.out(out_sum[47]),.sel(wcm[46]));
mux_rsr mxc47(.in0(wcha[47]),.in1(wcbec[47]),.out(wcm[47]),.sel(wcm[46]));


half_adder ha48(.A(In1[48]),.B(In2[48]),.Sum(wsha[48]),.Carry(wcha[48]));
not(wsbec[48],wsha[48]);
xor(wcbec[48],wsha[48],wcha[48]);
mux_rsr mxs48(.in0(wsha[48]),.in1(wsbec[48]),.out(out_sum[48]),.sel(wcm[47]));
mux_rsr mxc48(.in0(wcha[48]),.in1(wcbec[48]),.out(wcm[48]),.sel(wcm[47]));


half_adder ha49(.A(In1[49]),.B(In2[49]),.Sum(wsha[49]),.Carry(wcha[49]));
not(wsbec[49],wsha[49]);
xor(wcbec[49],wsha[49],wcha[49]);
mux_rsr mxs49(.in0(wsha[49]),.in1(wsbec[49]),.out(out_sum[49]),.sel(wcm[48]));
mux_rsr mxc49(.in0(wcha[49]),.in1(wcbec[49]),.out(wcm[49]),.sel(wcm[48]));


half_adder ha50(.A(In1[50]),.B(In2[50]),.Sum(wsha[50]),.Carry(wcha[50]));
not(wsbec[50],wsha[50]);
xor(wcbec[50],wsha[50],wcha[50]);
mux_rsr mxs50(.in0(wsha[50]),.in1(wsbec[50]),.out(out_sum[50]),.sel(wcm[49]));
mux_rsr mxc50(.in0(wcha[50]),.in1(wcbec[50]),.out(wcm[50]),.sel(wcm[49]));


half_adder ha51(.A(In1[51]),.B(In2[51]),.Sum(wsha[51]),.Carry(wcha[51]));
not(wsbec[51],wsha[51]);
xor(wcbec[51],wsha[51],wcha[51]);
mux_rsr mxs51(.in0(wsha[51]),.in1(wsbec[51]),.out(out_sum[51]),.sel(wcm[50]));
mux_rsr mxc51(.in0(wcha[51]),.in1(wcbec[51]),.out(wcm[51]),.sel(wcm[50]));


half_adder ha52(.A(In1[52]),.B(In2[52]),.Sum(wsha[52]),.Carry(wcha[52]));
not(wsbec[52],wsha[52]);
xor(wcbec[52],wsha[52],wcha[52]);
mux_rsr mxs52(.in0(wsha[52]),.in1(wsbec[52]),.out(out_sum[52]),.sel(wcm[51]));
mux_rsr mxc52(.in0(wcha[52]),.in1(wcbec[52]),.out(wcm[52]),.sel(wcm[51]));
buf(finalcarry,wcm[52]);

endmodule



module half_adder(A,B,Sum,Carry);
  input A,B;
  output Sum,Carry;
  assign Sum=((A^B));
  assign Carry=A&B;
endmodule


module mux_rsr(in0,in1,out,sel);
  input in0,in1;
  input sel;
  output out;
  assign out=sel? in1:in0;
endmodule

module carry_select_adder(In1,In2,out_sum,finalcarry,clk);
  input [52:0]In1,In2;
  output [52:0]out_sum;
  output finalcarry,clk;
  wire [7:0]out_carry0,out_carry1,carry_1;
  wire w1,w2,w3,w4,w5,w6,w7,w8;
  wire [49:0]temp_out0,temp_out1;
  assign clk=1'b1;
  ripple_carry_adder_3b0 rca0(.I1(In1[2:0]),.I2(In2[2:0]),.S(out_sum[2:0]),.C(w1));//0-3adr
  
  
  ripple_carry_adder_3b rca1(.I1(In1[5:3]),.I2(In2[5:3]),.S(temp_out0[2:0]),.C(out_carry0[0])); //1-3adr
  BEC_3b bec1(.b_sum(temp_out0[2:0]),.b_out(temp_out1[2:0]),.c(out_carry1[0]));
  xor(carry_1[0],out_carry1[0],out_carry0[0]);
  mux_carry mc1(.in0(out_carry0[0]),.in1(carry_1[0]),.sel(w1),.out(w2));
  mux_3b m1(.in0(temp_out0[2:0]),.in1(temp_out1[2:0]),.sel(w1),.out(out_sum[5:3]));
  
  
  ripple_carry_adder_4b rca2(.I1(In1[9:6]),.I2(In2[9:6]),.S(temp_out0[6:3]),.C(out_carry0[1])); //2-4adr
  BEC_4b bec2(.b_sum(temp_out0[6:3]),.b_out(temp_out1[6:3]),.c(out_carry1[1]));
  xor(carry_1[1],out_carry1[1],out_carry0[1]);
  mux_carry mc2(.in0(out_carry0[1]),.in1(carry_1[1]),.sel(w2),.out(w3));
  mux_4b m2(.in0(temp_out0[6:3]),.in1(temp_out1[6:3]),.sel(w2),.out(out_sum[9:6]));
  
  
  ripple_carry_adder_5b rca3(.I1(In1[14:10]),.I2(In2[14:10]),.S(temp_out0[11:7]),.C(out_carry0[2])); //3-5adr
  BEC_5b bec3(.b_sum(temp_out0[11:7]),.b_out(temp_out1[11:7]),.c(out_carry1[2]));
  xor(carry_1[2],out_carry1[2],out_carry0[2]);
  mux_carry mc3(.in0(out_carry0[2]),.in1(carry_1[2]),.sel(w3),.out(w4));
  mux_5b m3(.in0(temp_out0[11:7]),.in1(temp_out1[11:7]),.sel(w3),.out(out_sum[14:10]));
  
  
  ripple_carry_adder_5b rca4(.I1(In1[19:15]),.I2(In2[19:15]),.S(temp_out0[16:12]),.C(out_carry0[3])); //4-5adr
  BEC_5b bec4(.b_sum(temp_out0[16:12]),.b_out(temp_out1[16:12]),.c(out_carry1[3]));
  xor(carry_1[3],out_carry1[3],out_carry0[3]);
  mux_carry mc4(.in0(out_carry0[3]),.in1(carry_1[3]),.sel(w4),.out(w5));
  mux_5b m4(.in0(temp_out0[16:12]),.in1(temp_out1[16:12]),.sel(w4),.out(out_sum[19:15]));
  
  
  ripple_carry_adder_7b rca5(.I1(In1[26:20]),.I2(In2[26:20]),.S(temp_out0[23:17]),.C(out_carry0[4])); //5-7adr
  BEC_7b bec5(.b_sum(temp_out0[23:17]),.b_out(temp_out1[23:17]),.c(out_carry1[4]));
  xor(carry_1[4],out_carry1[4],out_carry0[4]);
  mux_carry mc5(.in0(out_carry0[4]),.in1(carry_1[4]),.sel(w5),.out(w6));
  mux_7b m5(.in0(temp_out0[23:17]),.in1(temp_out1[23:17]),.sel(w5),.out(out_sum[26:20]));
  
  
  ripple_carry_adder_7b rca6(.I1(In1[33:27]),.I2(In2[33:27]),.S(temp_out0[30:24]),.C(out_carry0[5])); //6-7adr
  BEC_7b bec6(.b_sum(temp_out0[30:24]),.b_out(temp_out1[30:24]),.c(out_carry1[5]));
  xor(carry_1[5],out_carry1[5],out_carry0[5]);
  mux_carry mc6(.in0(out_carry0[5]),.in1(carry_1[5]),.sel(w6),.out(w7));
  mux_7b m6(.in0(temp_out0[30:24]),.in1(temp_out1[30:24]),.sel(w6),.out(out_sum[33:27]));
  
  
  ripple_carry_adder_8b rca7(.I1(In1[41:34]),.I2(In2[41:34]),.S(temp_out0[38:31]),.C(out_carry0[6])); //7-8adr
  BEC_8b bec7(.b_sum(temp_out0[38:31]),.b_out(temp_out1[38:31]),.c(out_carry1[6]));
  xor(carry_1[6],out_carry1[6],out_carry0[6]);
  mux_carry mc7(.in0(out_carry0[6]),.in1(carry_1[6]),.sel(w7),.out(w8));
  mux_8b m7(.in0(temp_out0[38:31]),.in1(temp_out1[38:31]),.sel(w7),.out(out_sum[41:34]));
  
  
  ripple_carry_adder_10b rca8(.I1(In1[52:42]),.I2(In2[52:42]),.S(temp_out0[49:39]),.C(out_carry0[7])); //8-10adr
  BEC_10b bec8(.b_sum(temp_out0[49:39]),.b_out(temp_out1[49:39]),.c(out_carry1[7]));
  xor(carry_1[7],out_carry1[7],out_carry0[7]);
  mux_carry mc8(.in0(out_carry0[7]),.in1(carry_1[7]),.sel(w8),.out(finalcarry));
  mux_10b m8(.in0(temp_out0[49:39]),.in1(temp_out1[49:39]),.sel(w8),.out(out_sum[52:42]));

endmodule


module BEC_3b(b_sum,b_out,c);
  input [2:0]b_sum;
  output [2:0]b_out;
  output c;
  not(b_out[0],b_sum[0]);//1
  xor(b_out[1],b_sum[0],b_sum[1]);//2
  and(c,b_sum[0],b_sum[1]);
  xor(b_out[2],c,b_sum[2]);//3
endmodule

module BEC_4b(b_sum,b_out,c);
  input [3:0]b_sum;
  output [3:0]b_out;
  output c;
  wire w1;
  not(b_out[0],b_sum[0]);//1
  xor(b_out[1],b_sum[0],b_sum[1]);//2
  and(w1,b_sum[0],b_sum[1]);
  xor(b_out[2],w1,b_sum[2]);//3
  and(c,w1,b_sum[2]);
  xor(b_out[3],c,b_sum[3]);//4
endmodule

module BEC_5b(b_sum,b_out,c);
  input [4:0]b_sum;
  output [4:0]b_out;
  output c;
  wire w1,w2;
  not(b_out[0],b_sum[0]);//1
  xor(b_out[1],b_sum[0],b_sum[1]);//2
  and(w1,b_sum[0],b_sum[1]);
  xor(b_out[2],w1,b_sum[2]);//3
  and(w2,w1,b_sum[2]);
  xor(b_out[3],w2,b_sum[3]);//4
  and(c,w2,b_sum[3]);
  xor(b_out[4],c,b_sum[4]);//5
endmodule

module BEC_7b(b_sum,b_out,c);
  input [6:0]b_sum;
  output [6:0]b_out;
  output c;
  wire w1,w2,w3,w4;
  not(b_out[0],b_sum[0]);//1
  xor(b_out[1],b_sum[0],b_sum[1]);//2
  and(w1,b_sum[0],b_sum[1]);
  xor(b_out[2],w1,b_sum[2]);//3
  and(w2,w1,b_sum[2]);
  xor(b_out[3],w2,b_sum[3]);//4
  and(w3,w2,b_sum[3]);
  xor(b_out[4],w3,b_sum[4]);//5
  and(w4,w3,b_sum[4]);
  xor(b_out[5],w4,b_sum[5]);//6
  and(c,w4,b_sum[5]);
  xor(b_out[6],c,b_sum[6]);//7
endmodule

module BEC_8b(b_sum,b_out,c);
  input [7:0]b_sum;
  output [7:0]b_out;
  wire w1,w2,w3,w4,w5;
  output c;
  not(b_out[0],b_sum[0]);//1
  xor(b_out[1],b_sum[0],b_sum[1]);//2
  and(w1,b_sum[0],b_sum[1]);
  xor(b_out[2],w1,b_sum[2]);//3
  and(w2,w1,b_sum[2]);
  xor(b_out[3],w2,b_sum[3]);//4
  and(w3,w2,b_sum[3]);
  xor(b_out[4],w3,b_sum[4]);//5
  and(w4,w3,b_sum[4]);
  xor(b_out[5],w4,b_sum[5]);//6
  and(w5,w4,b_sum[5]);
  xor(b_out[6],w5,b_sum[6]);//7
  and(c,w5,b_sum[6]);
  xor(b_out[7],c,b_sum[7]);//8
endmodule

module BEC_10b(b_sum,b_out,c);
  input [10:0]b_sum;
  output [10:0]b_out;
  wire w1,w2,w3,w4,w5,w6,w7,w8;
  output c;
  not(b_out[0],b_sum[0]);//1
  xor(b_out[1],b_sum[0],b_sum[1]);//2
  and(w1,b_sum[0],b_sum[1]);
  xor(b_out[2],w1,b_sum[2]);//3
  and(w2,w1,b_sum[2]);
  xor(b_out[3],w2,b_sum[3]);//4
  and(w3,w2,b_sum[3]);
  xor(b_out[4],w3,b_sum[4]);//5
  and(w4,w3,b_sum[4]);
  xor(b_out[5],w4,b_sum[5]);//6
  and(w5,w4,b_sum[5]);
  xor(b_out[6],w5,b_sum[6]);//7
  and(w6,w5,b_sum[6]);
  xor(b_out[7],w6,b_sum[7]);//8
  and(w7,w6,b_sum[7]);
  xor(b_out[8],w7,b_sum[8]);//9
  and(w8,w7,b_sum[8]);
  xor(b_out[9],w8,b_sum[9]);//10
  and(c,w8,b_sum[9]);
  xor(b_out[10],c,b_sum[10]);//10
endmodule

module full_adder(A,B,Cin,Sum,Carry);
  input A,B,Cin;
  output Sum,Carry;
  assign Sum=((A^B)^Cin);
  assign Carry=((A&B)|((A^B)&Cin));
endmodule

module half_adder(A,B,Sum,Carry);
  input A,B;
  output Sum,Carry;
  assign Sum=((A^B));
  assign Carry=A&B;
endmodule

module mux_3b(in0,in1,out,sel);
  input [2:0]in0,in1;
  input sel;
  output [2:0]out;
  assign out=sel? in1:in0;
endmodule

module mux_4b(in0,in1,out,sel);
  input [3:0]in0,in1;
  input sel;
  output [3:0]out;
  assign out=sel? in1:in0;
endmodule

module mux_5b(in0,in1,out,sel);
  input [4:0]in0,in1;//3
  input sel;
  output [4:0]out;
  assign out=sel? in1:in0;
endmodule

module mux_7b(in0,in1,out,sel);
  input [6:0]in0,in1;//3
  input sel;
  output [6:0]out;
  assign out=sel? in1:in0;
endmodule

module mux_8b(in0,in1,out,sel);
  input [7:0]in0,in1;//3
  input sel;
  output [7:0]out;
  assign out=sel? in1:in0;
endmodule

module mux_10b(in0,in1,out,sel);
  input [10:0]in0,in1;//3
  input sel;
  output [10:0]out;
  assign out=sel? in1:in0;
endmodule

module mux_carry(in0,in1,out,sel);
  input in0,in1;
  input sel;
  output out;
  assign out=sel? in1:in0;
endmodule

module ripple_carry_adder_3b(I1,I2,S,C);
  input [2:0]I1,I2;
  output [2:0]S;
  output C;
  wire w1,w2;
  half_adder ha(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa1(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa2(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(C));
  endmodule

  module ripple_carry_adder_3b0(I1,I2,S,C);
  input [2:0]I1,I2;
  output [2:0]S;
  output C;
  wire w1,w2;
  half_adder ha(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa1(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa2(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(C));
  endmodule

  module ripple_carry_adder_4b(I1,I2,S,C);
  input [3:0]I1,I2;
  output [3:0]S;
  output C;
  wire w1,w2,w3;
  half_adder ha(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa2(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa3(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(w3));
  full_adder fa4(.A(I1[3]),.B(I2[3]),.Cin(w3),.Sum(S[3]),.Carry(C));
endmodule

module ripple_carry_adder_5b(I1,I2,S,C);
  input [4:0]I1,I2;
  output [4:0]S;
  output C;
  wire w1,w2,w3,w4;
  half_adder ha(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa2(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa3(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(w3));
  full_adder fa4(.A(I1[3]),.B(I2[3]),.Cin(w3),.Sum(S[3]),.Carry(w4));
  full_adder fa5(.A(I1[4]),.B(I2[4]),.Cin(w4),.Sum(S[4]),.Carry(C));
endmodule

module ripple_carry_adder_7b(I1,I2,S,C);
  input [6:0]I1,I2;
  output [6:0]S;
  output C;
  wire w1,w2,w3,w4,w5,w6;
  half_adder ha(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa2(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa3(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(w3));
  full_adder fa4(.A(I1[3]),.B(I2[3]),.Cin(w3),.Sum(S[3]),.Carry(w4));
  full_adder fa5(.A(I1[4]),.B(I2[4]),.Cin(w4),.Sum(S[4]),.Carry(w5));
  full_adder fa6(.A(I1[5]),.B(I2[5]),.Cin(w5),.Sum(S[5]),.Carry(w6));
  full_adder fa7(.A(I1[6]),.B(I2[6]),.Cin(w6),.Sum(S[6]),.Carry(C));
endmodule

module ripple_carry_adder_8b(I1,I2,S,C);
  input [7:0]I1,I2;
  output [7:0]S;
  output C;
  wire w1,w2,w3,w4,w5,w6,w7;
  half_adder ha(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa2(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa3(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(w3));
  full_adder fa4(.A(I1[3]),.B(I2[3]),.Cin(w3),.Sum(S[3]),.Carry(w4));
  full_adder fa5(.A(I1[4]),.B(I2[4]),.Cin(w4),.Sum(S[4]),.Carry(w5));
  full_adder fa6(.A(I1[5]),.B(I2[5]),.Cin(w5),.Sum(S[5]),.Carry(w6));
  full_adder fa7(.A(I1[6]),.B(I2[6]),.Cin(w6),.Sum(S[6]),.Carry(w7));
  full_adder fa8(.A(I1[7]),.B(I2[7]),.Cin(w7),.Sum(S[7]),.Carry(C));
endmodule

module ripple_carry_adder_10b(I1,I2,S,C);
  input [10:0]I1,I2;
  output [10:0]S;
  output C;
  wire w1,w2,w3,w4,w5,w6,w7,w8,w9,w10;
  half_adder ha1(.A(I1[0]),.B(I2[0]),.Sum(S[0]),.Carry(w1));
  full_adder fa2(.A(I1[1]),.B(I2[1]),.Cin(w1),.Sum(S[1]),.Carry(w2));
  full_adder fa3(.A(I1[2]),.B(I2[2]),.Cin(w2),.Sum(S[2]),.Carry(w3));
  full_adder fa4(.A(I1[3]),.B(I2[3]),.Cin(w3),.Sum(S[3]),.Carry(w4));
  full_adder fa5(.A(I1[4]),.B(I2[4]),.Cin(w4),.Sum(S[4]),.Carry(w5));
  full_adder fa6(.A(I1[5]),.B(I2[5]),.Cin(w5),.Sum(S[5]),.Carry(w6));
  full_adder fa7(.A(I1[6]),.B(I2[6]),.Cin(w6),.Sum(S[6]),.Carry(w7));
  full_adder fa8(.A(I1[7]),.B(I2[7]),.Cin(w7),.Sum(S[7]),.Carry(w8));
  full_adder fa9(.A(I1[8]),.B(I2[8]),.Cin(w8),.Sum(S[8]),.Carry(w9));
  full_adder fa10(.A(I1[9]),.B(I2[9]),.Cin(w9),.Sum(S[9]),.Carry(w10));
  full_adder fa11(.A(I1[10]),.B(I2[10]),.Cin(w10),.Sum(S[10]),.Carry(C));
endmodule
*/


module new_adder(In1,In2,out_sum,finalcarry);
  input [52:0]In1,In2;
  output [52:0]out_sum;
  output finalcarry;
  wire [52:1]wsha,wsbec,wcbec;
  wire [52:0] wcha;
  wire [50:0]wcmc;
  wire wcm;

half_adder ha0(.A(In1[0]),.B(In2[0]),.Sum(out_sum[0]),.Carry(wcha[0]));
  

half_adder ha1(.A(In1[1]),.B(In2[1]),.Sum(wsha[1]),.Carry(wcha[1]));
not(wsbec[1],wsha[1]);
mux_rsr mxs1(.in0(wsha[1]),.in1(wsbec[1]),.out(out_sum[1]),.sel(wcha[0]));
carry_generator mxc1(.Cin_A(In1[1]),.Cin_B(In2[1]),.Cin_C(wcha[0]),.Cout_o(wcm));


half_adder ha2(.A(In1[2]),.B(In2[2]),.Sum(wsha[2]),.Carry(wcha[2]));
not(wsbec[2],wsha[2]);
mux_rsr mxs2(.in0(wsha[2]),.in1(wsbec[2]),.out(out_sum[2]),.sel(wcm));
carry_generator mxc2(.Cin_A(In1[2]),.Cin_B(In2[2]),.Cin_C(wcm),.Cout_o(wcmc[0]));

half_adder ha3(.A(In1[3]),.B(In2[3]),.Sum(wsha[3]),.Carry(wcha[3]));
not(wsbec[3],wsha[3]);
mux_rsr mxs3(.in0(wsha[3]),.in1(wsbec[3]),.out(out_sum[3]),.sel(wcmc[0]));
carry_generator mxc3(.Cin_A(In1[3]),.Cin_B(In2[3]),.Cin_C(wcmc[0]),.Cout_o(wcmc[1]));

half_adder ha4(.A(In1[4]),.B(In2[4]),.Sum(wsha[4]),.Carry(wcha[4]));
not(wsbec[4],wsha[4]);
mux_rsr mxs4(.in0(wsha[4]),.in1(wsbec[4]),.out(out_sum[4]),.sel(wcmc[1]));
carry_generator mxc4(.Cin_A(In1[4]),.Cin_B(In2[4]),.Cin_C(wcmc[1]),.Cout_o(wcmc[2]));


half_adder ha5(.A(In1[5]),.B(In2[5]),.Sum(wsha[5]),.Carry(wcha[5]));
not(wsbec[5],wsha[5]);
mux_rsr mxs5(.in0(wsha[5]),.in1(wsbec[5]),.out(out_sum[5]),.sel(wcmc[2]));
carry_generator mxc5(.Cin_A(In1[5]),.Cin_B(In2[5]),.Cin_C(wcmc[2]),.Cout_o(wcmc[3]));


half_adder ha6(.A(In1[6]),.B(In2[6]),.Sum(wsha[6]),.Carry(wcha[6]));
not(wsbec[6],wsha[6]);
mux_rsr mxs6(.in0(wsha[6]),.in1(wsbec[6]),.out(out_sum[6]),.sel(wcmc[3]));
carry_generator mxc6(.Cin_A(In1[6]),.Cin_B(In2[6]),.Cin_C(wcmc[3]),.Cout_o(wcmc[4]));


half_adder ha7(.A(In1[7]),.B(In2[7]),.Sum(wsha[7]),.Carry(wcha[7]));
not(wsbec[7],wsha[7]);
mux_rsr mxs7(.in0(wsha[7]),.in1(wsbec[7]),.out(out_sum[7]),.sel(wcmc[4]));
carry_generator mxc7(.Cin_A(In1[7]),.Cin_B(In2[7]),.Cin_C(wcmc[4]),.Cout_o(wcmc[5]));


half_adder ha8(.A(In1[8]),.B(In2[8]),.Sum(wsha[8]),.Carry(wcha[8]));
not(wsbec[8],wsha[8]);
mux_rsr mxs8(.in0(wsha[8]),.in1(wsbec[8]),.out(out_sum[8]),.sel(wcmc[5]));
carry_generator mxc8(.Cin_A(In1[8]),.Cin_B(In2[8]),.Cin_C(wcmc[5]),.Cout_o(wcmc[6]));


half_adder ha9(.A(In1[9]),.B(In2[9]),.Sum(wsha[9]),.Carry(wcha[9]));
not(wsbec[9],wsha[9]);
mux_rsr mxs9(.in0(wsha[9]),.in1(wsbec[9]),.out(out_sum[9]),.sel(wcmc[6]));
carry_generator mxc9(.Cin_A(In1[9]),.Cin_B(In2[9]),.Cin_C(wcmc[6]),.Cout_o(wcmc[7]));


half_adder ha10(.A(In1[10]),.B(In2[10]),.Sum(wsha[10]),.Carry(wcha[10]));
not(wsbec[10],wsha[10]);
mux_rsr mxs10(.in0(wsha[10]),.in1(wsbec[10]),.out(out_sum[10]),.sel(wcmc[7]));
carry_generator mxc10(.Cin_A(In1[10]),.Cin_B(In2[10]),.Cin_C(wcmc[7]),.Cout_o(wcmc[8]));


half_adder ha11(.A(In1[11]),.B(In2[11]),.Sum(wsha[11]),.Carry(wcha[11]));
not(wsbec[11],wsha[11]);
mux_rsr mxs11(.in0(wsha[11]),.in1(wsbec[11]),.out(out_sum[11]),.sel(wcmc[8]));
carry_generator mxc11(.Cin_A(In1[11]),.Cin_B(In2[11]),.Cin_C(wcmc[8]),.Cout_o(wcmc[9]));


half_adder ha12(.A(In1[12]),.B(In2[12]),.Sum(wsha[12]),.Carry(wcha[12]));
not(wsbec[12],wsha[12]);
mux_rsr mxs12(.in0(wsha[12]),.in1(wsbec[12]),.out(out_sum[12]),.sel(wcmc[9]));
carry_generator mxc12(.Cin_A(In1[12]),.Cin_B(In2[12]),.Cin_C(wcmc[9]),.Cout_o(wcmc[10]));


half_adder ha13(.A(In1[13]),.B(In2[13]),.Sum(wsha[13]),.Carry(wcha[13]));
not(wsbec[13],wsha[13]);
mux_rsr mxs13(.in0(wsha[13]),.in1(wsbec[13]),.out(out_sum[13]),.sel(wcmc[10]));
carry_generator mxc13(.Cin_A(In1[13]),.Cin_B(In2[13]),.Cin_C(wcmc[10]),.Cout_o(wcmc[11]));


half_adder ha14(.A(In1[14]),.B(In2[14]),.Sum(wsha[14]),.Carry(wcha[14]));
not(wsbec[14],wsha[14]);
mux_rsr mxs14(.in0(wsha[14]),.in1(wsbec[14]),.out(out_sum[14]),.sel(wcmc[11]));
carry_generator mxc14(.Cin_A(In1[14]),.Cin_B(In2[14]),.Cin_C(wcmc[11]),.Cout_o(wcmc[12]));


half_adder ha15(.A(In1[15]),.B(In2[15]),.Sum(wsha[15]),.Carry(wcha[15]));
not(wsbec[15],wsha[15]);
mux_rsr mxs15(.in0(wsha[15]),.in1(wsbec[15]),.out(out_sum[15]),.sel(wcmc[12]));
carry_generator mxc15(.Cin_A(In1[15]),.Cin_B(In2[15]),.Cin_C(wcmc[12]),.Cout_o(wcmc[13]));


half_adder ha16(.A(In1[16]),.B(In2[16]),.Sum(wsha[16]),.Carry(wcha[16]));
not(wsbec[16],wsha[16]);
mux_rsr mxs16(.in0(wsha[16]),.in1(wsbec[16]),.out(out_sum[16]),.sel(wcmc[13]));
carry_generator mxc16(.Cin_A(In1[16]),.Cin_B(In2[16]),.Cin_C(wcmc[13]),.Cout_o(wcmc[14]));


half_adder ha17(.A(In1[17]),.B(In2[17]),.Sum(wsha[17]),.Carry(wcha[17]));
not(wsbec[17],wsha[17]);
mux_rsr mxs17(.in0(wsha[17]),.in1(wsbec[17]),.out(out_sum[17]),.sel(wcmc[14]));
carry_generator mxc17(.Cin_A(In1[17]),.Cin_B(In2[17]),.Cin_C(wcmc[14]),.Cout_o(wcmc[15]));


half_adder ha18(.A(In1[18]),.B(In2[18]),.Sum(wsha[18]),.Carry(wcha[18]));
not(wsbec[18],wsha[18]);
mux_rsr mxs18(.in0(wsha[18]),.in1(wsbec[18]),.out(out_sum[18]),.sel(wcmc[15]));
carry_generator mxc18(.Cin_A(In1[18]),.Cin_B(In2[18]),.Cin_C(wcmc[15]),.Cout_o(wcmc[16]));


half_adder ha19(.A(In1[19]),.B(In2[19]),.Sum(wsha[19]),.Carry(wcha[19]));
not(wsbec[19],wsha[19]);
mux_rsr mxs19(.in0(wsha[19]),.in1(wsbec[19]),.out(out_sum[19]),.sel(wcmc[16]));
carry_generator mxc19(.Cin_A(In1[19]),.Cin_B(In2[19]),.Cin_C(wcmc[16]),.Cout_o(wcmc[17]));


half_adder ha20(.A(In1[20]),.B(In2[20]),.Sum(wsha[20]),.Carry(wcha[20]));
not(wsbec[20],wsha[20]);
mux_rsr mxs20(.in0(wsha[20]),.in1(wsbec[20]),.out(out_sum[20]),.sel(wcmc[17]));
carry_generator mxc20(.Cin_A(In1[20]),.Cin_B(In2[20]),.Cin_C(wcmc[17]),.Cout_o(wcmc[18]));


half_adder ha21(.A(In1[21]),.B(In2[21]),.Sum(wsha[21]),.Carry(wcha[21]));
not(wsbec[21],wsha[21]);
mux_rsr mxs21(.in0(wsha[21]),.in1(wsbec[21]),.out(out_sum[21]),.sel(wcmc[18]));
carry_generator mxc21(.Cin_A(In1[21]),.Cin_B(In2[21]),.Cin_C(wcmc[18]),.Cout_o(wcmc[19]));


half_adder ha22(.A(In1[22]),.B(In2[22]),.Sum(wsha[22]),.Carry(wcha[22]));
not(wsbec[22],wsha[22]);
mux_rsr mxs22(.in0(wsha[22]),.in1(wsbec[22]),.out(out_sum[22]),.sel(wcmc[19]));
carry_generator mxc22(.Cin_A(In1[22]),.Cin_B(In2[22]),.Cin_C(wcmc[19]),.Cout_o(wcmc[20]));


half_adder ha23(.A(In1[23]),.B(In2[23]),.Sum(wsha[23]),.Carry(wcha[23]));
not(wsbec[23],wsha[23]);
mux_rsr mxs23(.in0(wsha[23]),.in1(wsbec[23]),.out(out_sum[23]),.sel(wcmc[20]));
carry_generator mxc23(.Cin_A(In1[23]),.Cin_B(In2[23]),.Cin_C(wcmc[20]),.Cout_o(wcmc[21]));


half_adder ha24(.A(In1[24]),.B(In2[24]),.Sum(wsha[24]),.Carry(wcha[24]));
not(wsbec[24],wsha[24]);
mux_rsr mxs24(.in0(wsha[24]),.in1(wsbec[24]),.out(out_sum[24]),.sel(wcmc[21]));
carry_generator mxc24(.Cin_A(In1[24]),.Cin_B(In2[24]),.Cin_C(wcmc[21]),.Cout_o(wcmc[22]));


half_adder ha25(.A(In1[25]),.B(In2[25]),.Sum(wsha[25]),.Carry(wcha[25]));
not(wsbec[25],wsha[25]);
mux_rsr mxs25(.in0(wsha[25]),.in1(wsbec[25]),.out(out_sum[25]),.sel(wcmc[22]));
carry_generator mxc25(.Cin_A(In1[25]),.Cin_B(In2[25]),.Cin_C(wcmc[22]),.Cout_o(wcmc[23]));


half_adder ha26(.A(In1[26]),.B(In2[26]),.Sum(wsha[26]),.Carry(wcha[26]));
not(wsbec[26],wsha[26]);
mux_rsr mxs26(.in0(wsha[26]),.in1(wsbec[26]),.out(out_sum[26]),.sel(wcmc[23]));
carry_generator mxc26(.Cin_A(In1[26]),.Cin_B(In2[26]),.Cin_C(wcmc[23]),.Cout_o(wcmc[24]));


half_adder ha27(.A(In1[27]),.B(In2[27]),.Sum(wsha[27]),.Carry(wcha[27]));
not(wsbec[27],wsha[27]);
mux_rsr mxs27(.in0(wsha[27]),.in1(wsbec[27]),.out(out_sum[27]),.sel(wcmc[24]));
carry_generator mxc27(.Cin_A(In1[27]),.Cin_B(In2[27]),.Cin_C(wcmc[24]),.Cout_o(wcmc[25]));


half_adder ha28(.A(In1[28]),.B(In2[28]),.Sum(wsha[28]),.Carry(wcha[28]));
not(wsbec[28],wsha[28]);
mux_rsr mxs28(.in0(wsha[28]),.in1(wsbec[28]),.out(out_sum[28]),.sel(wcmc[25]));
carry_generator mxc28(.Cin_A(In1[28]),.Cin_B(In2[28]),.Cin_C(wcmc[25]),.Cout_o(wcmc[26]));


half_adder ha29(.A(In1[29]),.B(In2[29]),.Sum(wsha[29]),.Carry(wcha[29]));
not(wsbec[29],wsha[29]);
mux_rsr mxs29(.in0(wsha[29]),.in1(wsbec[29]),.out(out_sum[29]),.sel(wcmc[26]));
carry_generator mxc29(.Cin_A(In1[29]),.Cin_B(In2[29]),.Cin_C(wcmc[26]),.Cout_o(wcmc[27]));


half_adder ha30(.A(In1[30]),.B(In2[30]),.Sum(wsha[30]),.Carry(wcha[30]));
not(wsbec[30],wsha[30]);
mux_rsr mxs30(.in0(wsha[30]),.in1(wsbec[30]),.out(out_sum[30]),.sel(wcmc[27]));
carry_generator mxc30(.Cin_A(In1[30]),.Cin_B(In2[30]),.Cin_C(wcmc[27]),.Cout_o(wcmc[28]));


half_adder ha31(.A(In1[31]),.B(In2[31]),.Sum(wsha[31]),.Carry(wcha[31]));
not(wsbec[31],wsha[31]);
mux_rsr mxs31(.in0(wsha[31]),.in1(wsbec[31]),.out(out_sum[31]),.sel(wcmc[28]));
carry_generator mxc31(.Cin_A(In1[31]),.Cin_B(In2[31]),.Cin_C(wcmc[28]),.Cout_o(wcmc[29]));


half_adder ha32(.A(In1[32]),.B(In2[32]),.Sum(wsha[32]),.Carry(wcha[32]));
not(wsbec[32],wsha[32]);
mux_rsr mxs32(.in0(wsha[32]),.in1(wsbec[32]),.out(out_sum[32]),.sel(wcmc[29]));
carry_generator mxc32(.Cin_A(In1[32]),.Cin_B(In2[32]),.Cin_C(wcmc[29]),.Cout_o(wcmc[30]));


half_adder ha33(.A(In1[33]),.B(In2[33]),.Sum(wsha[33]),.Carry(wcha[33]));
not(wsbec[33],wsha[33]);
mux_rsr mxs33(.in0(wsha[33]),.in1(wsbec[33]),.out(out_sum[33]),.sel(wcmc[30]));
carry_generator mxc33(.Cin_A(In1[33]),.Cin_B(In2[33]),.Cin_C(wcmc[30]),.Cout_o(wcmc[31]));


half_adder ha34(.A(In1[34]),.B(In2[34]),.Sum(wsha[34]),.Carry(wcha[34]));
not(wsbec[34],wsha[34]);
mux_rsr mxs34(.in0(wsha[34]),.in1(wsbec[34]),.out(out_sum[34]),.sel(wcmc[31]));
carry_generator mxc34(.Cin_A(In1[34]),.Cin_B(In2[34]),.Cin_C(wcmc[31]),.Cout_o(wcmc[32]));


half_adder ha35(.A(In1[35]),.B(In2[35]),.Sum(wsha[35]),.Carry(wcha[35]));
not(wsbec[35],wsha[35]);
mux_rsr mxs35(.in0(wsha[35]),.in1(wsbec[35]),.out(out_sum[35]),.sel(wcmc[32]));
carry_generator mxc35(.Cin_A(In1[35]),.Cin_B(In2[35]),.Cin_C(wcmc[32]),.Cout_o(wcmc[33]));


half_adder ha36(.A(In1[36]),.B(In2[36]),.Sum(wsha[36]),.Carry(wcha[36]));
not(wsbec[36],wsha[36]);
mux_rsr mxs36(.in0(wsha[36]),.in1(wsbec[36]),.out(out_sum[36]),.sel(wcmc[33]));
carry_generator mxc36(.Cin_A(In1[36]),.Cin_B(In2[36]),.Cin_C(wcmc[33]),.Cout_o(wcmc[34]));


half_adder ha37(.A(In1[37]),.B(In2[37]),.Sum(wsha[37]),.Carry(wcha[37]));
not(wsbec[37],wsha[37]);
mux_rsr mxs37(.in0(wsha[37]),.in1(wsbec[37]),.out(out_sum[37]),.sel(wcmc[34]));
carry_generator mxc37(.Cin_A(In1[37]),.Cin_B(In2[37]),.Cin_C(wcmc[34]),.Cout_o(wcmc[35]));


half_adder ha38(.A(In1[38]),.B(In2[38]),.Sum(wsha[38]),.Carry(wcha[38]));
not(wsbec[38],wsha[38]);
mux_rsr mxs38(.in0(wsha[38]),.in1(wsbec[38]),.out(out_sum[38]),.sel(wcmc[35]));
carry_generator mxc38(.Cin_A(In1[38]),.Cin_B(In2[38]),.Cin_C(wcmc[35]),.Cout_o(wcmc[36]));


half_adder ha39(.A(In1[39]),.B(In2[39]),.Sum(wsha[39]),.Carry(wcha[39]));
not(wsbec[39],wsha[39]);
mux_rsr mxs39(.in0(wsha[39]),.in1(wsbec[39]),.out(out_sum[39]),.sel(wcmc[36]));
carry_generator mxc39(.Cin_A(In1[39]),.Cin_B(In2[39]),.Cin_C(wcmc[36]),.Cout_o(wcmc[37]));


half_adder ha40(.A(In1[40]),.B(In2[40]),.Sum(wsha[40]),.Carry(wcha[40]));
not(wsbec[40],wsha[40]);
mux_rsr mxs40(.in0(wsha[40]),.in1(wsbec[40]),.out(out_sum[40]),.sel(wcmc[37]));
carry_generator mxc40(.Cin_A(In1[40]),.Cin_B(In2[40]),.Cin_C(wcmc[37]),.Cout_o(wcmc[38]));


half_adder ha41(.A(In1[41]),.B(In2[41]),.Sum(wsha[41]),.Carry(wcha[41]));
not(wsbec[41],wsha[41]);
mux_rsr mxs41(.in0(wsha[41]),.in1(wsbec[41]),.out(out_sum[41]),.sel(wcmc[38]));
carry_generator mxc41(.Cin_A(In1[41]),.Cin_B(In2[41]),.Cin_C(wcmc[38]),.Cout_o(wcmc[39]));


half_adder ha42(.A(In1[42]),.B(In2[42]),.Sum(wsha[42]),.Carry(wcha[42]));
not(wsbec[42],wsha[42]);
mux_rsr mxs42(.in0(wsha[42]),.in1(wsbec[42]),.out(out_sum[42]),.sel(wcmc[39]));
carry_generator mxc42(.Cin_A(In1[42]),.Cin_B(In2[42]),.Cin_C(wcmc[39]),.Cout_o(wcmc[40]));


half_adder ha43(.A(In1[43]),.B(In2[43]),.Sum(wsha[43]),.Carry(wcha[43]));
not(wsbec[43],wsha[43]);
mux_rsr mxs43(.in0(wsha[43]),.in1(wsbec[43]),.out(out_sum[43]),.sel(wcmc[40]));
carry_generator mxc43(.Cin_A(In1[43]),.Cin_B(In2[43]),.Cin_C(wcmc[40]),.Cout_o(wcmc[41]));


half_adder ha44(.A(In1[44]),.B(In2[44]),.Sum(wsha[44]),.Carry(wcha[44]));
not(wsbec[44],wsha[44]);
mux_rsr mxs44(.in0(wsha[44]),.in1(wsbec[44]),.out(out_sum[44]),.sel(wcmc[41]));
carry_generator mxc44(.Cin_A(In1[44]),.Cin_B(In2[44]),.Cin_C(wcmc[41]),.Cout_o(wcmc[42]));


half_adder ha45(.A(In1[45]),.B(In2[45]),.Sum(wsha[45]),.Carry(wcha[45]));
not(wsbec[45],wsha[45]);
mux_rsr mxs45(.in0(wsha[45]),.in1(wsbec[45]),.out(out_sum[45]),.sel(wcmc[42]));
carry_generator mxc45(.Cin_A(In1[45]),.Cin_B(In2[45]),.Cin_C(wcmc[42]),.Cout_o(wcmc[43]));


half_adder ha46(.A(In1[46]),.B(In2[46]),.Sum(wsha[46]),.Carry(wcha[46]));
not(wsbec[46],wsha[46]);
mux_rsr mxs46(.in0(wsha[46]),.in1(wsbec[46]),.out(out_sum[46]),.sel(wcmc[43]));
carry_generator mxc46(.Cin_A(In1[46]),.Cin_B(In2[46]),.Cin_C(wcmc[43]),.Cout_o(wcmc[44]));


half_adder ha47(.A(In1[47]),.B(In2[47]),.Sum(wsha[47]),.Carry(wcha[47]));
not(wsbec[47],wsha[47]);
mux_rsr mxs47(.in0(wsha[47]),.in1(wsbec[47]),.out(out_sum[47]),.sel(wcmc[44]));
carry_generator mxc47(.Cin_A(In1[47]),.Cin_B(In2[47]),.Cin_C(wcmc[44]),.Cout_o(wcmc[45]));


half_adder ha48(.A(In1[48]),.B(In2[48]),.Sum(wsha[48]),.Carry(wcha[48]));
not(wsbec[48],wsha[48]);
mux_rsr mxs48(.in0(wsha[48]),.in1(wsbec[48]),.out(out_sum[48]),.sel(wcmc[45]));
carry_generator mxc48(.Cin_A(In1[48]),.Cin_B(In2[48]),.Cin_C(wcmc[45]),.Cout_o(wcmc[46]));


half_adder ha49(.A(In1[49]),.B(In2[49]),.Sum(wsha[49]),.Carry(wcha[49]));
not(wsbec[49],wsha[49]);
mux_rsr mxs49(.in0(wsha[49]),.in1(wsbec[49]),.out(out_sum[49]),.sel(wcmc[46]));
carry_generator mxc49(.Cin_A(In1[49]),.Cin_B(In2[49]),.Cin_C(wcmc[46]),.Cout_o(wcmc[47]));


half_adder ha50(.A(In1[50]),.B(In2[50]),.Sum(wsha[50]),.Carry(wcha[50]));
not(wsbec[50],wsha[50]);
mux_rsr mxs50(.in0(wsha[50]),.in1(wsbec[50]),.out(out_sum[50]),.sel(wcmc[47]));
carry_generator mxc50(.Cin_A(In1[50]),.Cin_B(In2[50]),.Cin_C(wcmc[47]),.Cout_o(wcmc[48]));


half_adder ha51(.A(In1[51]),.B(In2[51]),.Sum(wsha[51]),.Carry(wcha[51]));
not(wsbec[51],wsha[51]);
mux_rsr mxs51(.in0(wsha[51]),.in1(wsbec[51]),.out(out_sum[51]),.sel(wcmc[48]));
carry_generator mxc51(.Cin_A(In1[51]),.Cin_B(In2[51]),.Cin_C(wcmc[48]),.Cout_o(wcmc[49]));


half_adder ha52(.A(In1[52]),.B(In2[52]),.Sum(wsha[52]),.Carry(wcha[52]));
not(wsbec[52],wsha[52]);
mux_rsr mxs52(.in0(wsha[52]),.in1(wsbec[52]),.out(out_sum[52]),.sel(wcmc[49]));
carry_generator mxc52(.Cin_A(In1[52]),.Cin_B(In2[52]),.Cin_C(wcmc[49]),.Cout_o(wcmc[50]));
buf(finalcarry,wcmc[50]);

endmodule



module half_adder(A,B,Sum,Carry);
  input A,B;
  output Sum,Carry;
  assign Sum=((A^B));
  assign Carry=A&B;
endmodule


module mux_rsr(in0,in1,out,sel);
  input in0,in1;
  input sel;
  output out;
  assign out=sel? in1:in0;
endmodule

module carry_generator(Cin_A,Cin_B,Cin_C,Cout_o);
  input Cin_A,Cin_B,Cin_C;
  output Cout_o;
  //assign Cout_o=Cin_B&(Cin_A|Cin_C);
  assign Cout_o=(Cin_A&Cin_B)|(Cin_B&Cin_C)|(Cin_A&Cin_C);
  //assign Cout_o=(((Cin_A^Cin_B)&Cin_C)|(Cin_A&Cin_A));
endmodule
