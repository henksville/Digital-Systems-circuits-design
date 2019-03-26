# Digital-Systems-circuits-design

--------------------------------------LC3 FINITE STATE MACHINE----------------------------------------------

Library ieee;
use ieee.std_logic_1164.all;
use work.clock;

entity Odunze_LC3_FSM IS
port(
	CLK: IN STD_LOGIC;				--clk
	Reset:IN STD_LOGIC;				--reset
	OUT_FROM_IR: IN STD_LOGIC_VECTOR(15 DOWNTO 0);  --instruction
	FROM_NZP: IN STD_LOGIC_VECTOR(2 DOWNTO 0);		--nzp flag
	
-------------------------------------Load enables ----------------------------------	

	LD_IR: OUT STD_LOGIC;				
	LD_MARMUX: out std_logic; 
	LD_REG: out std_logic; 
	LD_PC: OUT STD_LOGIC;				
	LD_CC: OUT STD_LOGIC;
	LD_MAR: OUT STD_LOGIC;				
	LD_MDR: OUT STD_LOGIC;						
	LD_MEM: OUT STD_LOGIC;				
	
-------------------------------------Gates To Bus ----------------------------------

	GATE_PC: OUT STD_LOGIC;			
	GATE_MARMUX: OUT STD_LOGIC;			
	GATE_ALU: OUT STD_LOGIC;			
	GATE_MDR: OUT STD_LOGIC;		

-------------------------------------Muxtiplexer selects & Mem_en----------------------------------
	
	ADDR2_MUX_SEL: OUT STD_LOGIC_VECTOR(1 DOWNTO 0);
	ADDR1_MUX_SEL: OUT STD_LOGIC;			
	SR1_SEL: OUT STD_LOGIC_VECTOR(2 DOWNTO 0);	
	SR2_SEL: OUT STD_LOGIC_VECTOR(2 DOWNTO 0);
	DR_SEL: OUT STD_LOGIC_VECTOR(2 DOWNTO 0);
	SR2MUX_SEL: OUT STD_LOGIC;			
	PCMUX_SEL: OUT STD_LOGIC_VECTOR(1 DOWNTO 0);	
	ALU_SEL: OUT STD_LOGIC_VECTOR(1 DOWNTO 0);
	MEM_EN: OUT STD_LOGIC;
	MEM_RW_EN: OUT STD_LOGIC				
);
 
end Odunze_LC3_FSM;


ARCHITECTURE BEH OF Odunze_LC3_FSM IS
TYPE LC3_STATE IS (S0, S1, S1A, S1B, S2, S2A, S2B, S2C, 
			S3, S4, S4A, S4B,S4C,S4D, S5, S5A, S5B);

signal cpu_state: LC3_STATE;
signal next_state: LC3_STATE;

constant ADD: std_logic_vector(3 downto 0) :=  "0001";
constant AND1: std_logic_vector(3 downto 0) := "0101";
constant LOAD: std_logic_vector(3 downto 0) := "0010";
constant STR: std_logic_vector(3 downto 0) := "0011";
constant NOT1: std_logic_vector(3 downto 0) := "1001";


begin
P1logic: process(Reset, CLK)	
begin
	if (CLK'event and CLK = '1') then 
		if (Reset ='1') then
			cpu_state <= S0;
		else
			cpu_state <= next_state;
		end if;
	end if;
end process;


OUTPUT_FSM: process(OUT_FROM_IR, FROM_NZP, cpu_state, next_state)


variable OPCODE: std_logic_vector(3 downto 0);
variable PC_OFFSET: std_logic_vector(8 downto 0);
variable SR1_IN: std_logic_vector(2 downto 0);
variable SR2_IN: std_logic_vector(2 downto 0);
variable DRIN: std_logic_vector(2 downto 0);
variable IR_5: std_logic;
variable IMMEDIATE: std_logic_vector(4 downto 0);
variable BASEREG: std_logic_vector(2 downto 0);
begin
case cpu_state is
 
--Initialize --
when S0=>

 LD_IR <= '0';
 LD_MARMUX <= '0';
 LD_REG <= '0';
 LD_PC <= '0';
 LD_MAR <= '0';
 LD_MDR <= '0';
 LD_CC <='0';
 LD_MEM <= '0';
 GATE_PC <= '0';
 GATE_MARMUX <= '0';
 GATE_ALU <= '0';
 GATE_MDR <= '0';
 ADDR2_MUX_SEL <= (others => '0');
 ADDR1_MUX_SEL <= '0';
 SR1_SEL <=(others => '0');
 SR2_SEL <=(others => '0');
 DR_SEL <=(others => '0');
 SR2MUX_SEL <= '0';
 PCMUX_SEL <=(others => '0');
 ALU_SEL <=(others => '0');
 MEM_EN <= '0';
MEM_RW_EN <= '0';

 next_state <= S1;

--- Instruction put on the bus -- 
when S1 =>

 LD_PC <= '1';
 LD_MAR <= '0';
 LD_MDR <= '0';
 LD_MARMUX <= '0';
 GATE_PC <= '0';
 PCMUX_SEL <="10";
 next_state <= S1A;



when S1A =>

 LD_PC <= '0';
 LD_MAR <= '1';
 LD_MDR <= '0';
 LD_MARMUX <= '0';
 GATE_PC <= '0';
  PCMUX_SEL <=(others => '0');
 next_state <= S1B;

when S1B =>
 LD_MAR <= '1';
 LD_MDR <= '0';
 LD_MARMUX <= '0';
 GATE_PC <= '0';
  PCMUX_SEL <=(others => '0');
 next_state <= S2;

-- Memory Enabled for Fetch --
when S2 =>
 LD_PC <= '0';
 GATE_PC <= '0';
 MEM_EN <= '1';
MEM_RW_EN <= '0';
 LD_MAR <= '0';
  next_state <= S2A;

when S2A =>

 LD_MDR <= '1';
 GATE_MDR <= '0';
 MEM_EN <= '0';
MEM_RW_EN <= '0';
 next_state <= S2B;

when S2B =>
 LD_IR <= '0';
 LD_PC <= '0';
 LD_MAR <= '0';
 GATE_MDR <= '1';
LD_MDR <= '0';
 next_state <= S3;

when S3 =>
 LD_IR <= '1';
 LD_MAR <= '0';
 LD_MDR <= '0';
 GATE_PC <= '0';
 GATE_MARMUX <= '0';
 GATE_ALU <= '0';
 GATE_MDR <= '0';
 next_state <= S4;


---decode of instruction --
when S4 =>
OPCODE := OUT_FROM_IR(15 downto 12); 
IR_5 := OUT_FROM_IR(5); 
DRIN := OUT_FROM_IR(11 downto 9); 
SR1_IN := OUT_FROM_IR(8 downto 6); 
SR2_IN := OUT_FROM_IR(2 downto 0);
IMMEDIATE := OUT_FROM_IR(4 downto 0); 
BASEREG := OUT_FROM_IR(8 downto 6); 
PC_OFFSET := OUT_FROM_IR(8 downto 0); 

	case OPCODE is
		when ADD =>
			if(IR_5 <= '0') then
				LD_IR <= '0';
				LD_REG <= '1';
				LD_CC <= '1';
				GATE_MDR <= '0';
				GATE_ALU <= '1';

				SR1_SEL <= SR1_IN;
				SR2_SEL <= SR2_IN;
				DR_SEL <= DRIN;
				SR2MUX_SEL <= '0';
				ALU_SEL <=(others => '0');
				next_state <= S1;
			else 
				LD_IR <= '0';
				LD_REG <= '1';
				LD_CC <= '1';
				GATE_MDR <= '0';
				GATE_ALU <= '1';

				SR1_SEL <= SR1_IN;
				DR_SEL <= DRIN;
				SR2MUX_SEL <= '1';
				ALU_SEL <= "00";
				next_state <= S1;
			end if;
		when AND1 =>
				if(IR_5 <= '0') then
				LD_IR <= '0';
				LD_REG <= '1';
				LD_CC <= '1';
				GATE_MDR <= '0';
				GATE_ALU <= '1';

				SR1_SEL <= SR1_IN;
				SR2_SEL <= SR2_IN;
				DR_SEL <= DRIN;
				SR2MUX_SEL <= '1';
				ALU_SEL <= "01";
				next_state <= S1;
			else 
				LD_IR <= '0';
				LD_REG <= '1';
				LD_CC <= '1';
				GATE_MDR <= '0';
				GATE_ALU <= '1';

				SR1_SEL <= SR1_IN;
				DR_SEL <= DRIN;
				SR2MUX_SEL <= '1';
				ALU_SEL <= "01";
				next_state <= S1;
			end if;
		when NOT1 =>
			LD_IR <= '0';
			LD_REG <= '1';
			LD_CC <= '1';
			GATE_MDR <= '0';
			GATE_ALU <= '1';

			SR1_SEL <= SR1_IN;
			DR_SEL <= DRIN;
			SR2MUX_SEL <= '1';
			ALU_SEL <= "10";
			next_state <= S1;
		when LOAD =>
 			LD_IR <= '1';
 			LD_MARMUX <= '0';	
 			LD_REG <= '0';
 			LD_PC <= '0';
 			GATE_PC <= '0';
 			GATE_MARMUX <= '1';
 			GATE_ALU <= '0';
 			GATE_MDR <= '0';
 			ADDR2_MUX_SEL <= "01";
 			ADDR1_MUX_SEL <= '0';
 			SR1_SEL <=(others => '0');
 			SR2_SEL <=(others => '0');
 			DR_SEL <=(others => '0');
 			SR2MUX_SEL <= '0';
 			PCMUX_SEL <= (others => '0');
 			ALU_SEL <=(others => '0');
 			MEM_EN <= '0';
			MEM_RW_EN <= '0';
			next_state <= S4A;
		when STR =>
			ADDR2_MUX_SEL <= "10";
			ADDR1_MUX_SEL <= '0';
			LD_MARMUX <= '1';
			GATE_MARMUX <= '0';
			LD_MAR <= '1';
			next_state <= S5A;
		when others =>
			next_state <= S0;
		end case;
		
		when S4A =>
 			LD_IR <= '0';
 			LD_MARMUX <= '1';	
 			LD_REG <= '0';
 			LD_PC <= '0';
 			LD_MAR <= '1';
 			LD_MDR <= '0';
 			GATE_PC <= '0';
 			GATE_MARMUX <= '1';
 			GATE_ALU <= '0';
 			GATE_MDR <= '0';
 			ADDR2_MUX_SEL <= "10";
 			ADDR1_MUX_SEL <= '1';
 			
 			next_state <= S4B;
		
		when S4B =>

			LD_IR <= '0';
 			LD_MARMUX <= '1';	
 			LD_REG <= '0';
 			LD_PC <= '0';
 			LD_MAR <= '0';
 			LD_MDR <= '1';
 			LD_CC <='0';
 			LD_MEM <= '0';
 			GATE_PC <= '0';
 			GATE_MARMUX <= '0';
 			GATE_ALU <= '0';
 			GATE_MDR <= '0';
 			ADDR2_MUX_SEL <= "10";
 			ADDR1_MUX_SEL <= '1';
 			SR1_SEL <=(others => '0');
 			SR2_SEL <=(others => '0');
 			DR_SEL <=(others => '0');
 			SR2MUX_SEL <= '0';
 			PCMUX_SEL <= (others=>'0');
 			ALU_SEL <=(others => '0');
 			MEM_EN <= '1';
			MEM_RW_EN <= '0';
 			next_state <= S4C;
		when S4C =>
 			LD_IR <= '0';
 			LD_MARMUX <= '1';	
 			LD_REG <= '1';
 			LD_PC <= '0';
 			LD_MAR <= '0';
 			LD_MDR <= '0';
 			LD_CC <='0';
 			LD_MEM <= '0';
 			GATE_PC <= '0';
 			GATE_MARMUX <= '0';
 			GATE_ALU <= '0';
 			GATE_MDR <= '0';
 			ADDR2_MUX_SEL <= "10";
 			ADDR1_MUX_SEL <= '1';
 			MEM_EN <= '1';
			MEM_RW_EN <= '0';
			next_state <=S4D;
		
		when S4D =>
			next_state <=S5;

		when S5 => 

			LD_IR <= '0';
 			LD_MARMUX <= '0';	
 			LD_REG <= '0';
 			LD_PC <= '1';
 			LD_MAR <= '0';
 			LD_MDR <= '0';
 			LD_CC <='0';
 			LD_MEM <= '0';
 			GATE_PC <= '0';
 			GATE_MARMUX <= '0';
 			GATE_ALU <= '0';
 			GATE_MDR <= '1';
 			ADDR2_MUX_SEL <= "10";
 			ADDR1_MUX_SEL <= '1';
 			SR1_SEL <=(others => '0');
 			SR2_SEL <=(others => '0');
 			DR_SEL <= DRIN;
 			SR2MUX_SEL <= '0';
 			PCMUX_SEL <= (others=>'0');
 			ALU_SEL <=(others => '0');
 			MEM_EN <= '0';
			MEM_RW_EN <= '0';
			next_state <=S1;
		when S5A =>
 			MEM_EN <= '1';
			MEM_RW_EN <= '1';
 			LD_MDR <= '0';
 			next_state <= S5B;
		when S5B =>
 			LD_MDR <= '1';
 			SR1_SEL <= SR1_IN;
 			next_state <= S1;
		when others =>
			next_state <= S0;		
		end case;
	end process;


end BEH;




----------------------------------------------------------------------------
----------------------------------------------------------------------------
----------------------------------------------------------------------------
--Odunze Henry
-- LC3 Data Path
----------------------------------------------------------------------------
----------------------------------------------------------------------------
library IEEE;
use IEEE.std_logic_1164.all; 
use IEEE.std_logic_arith.all;
use IEEE.std_logic_unsigned.all;
use IEEE.NUMERIC_STD.all;

 
entity Odunze_FSM_data_path is 
   generic(
	P: integer:= 16;
       	E: integer:= 8;
       	W: integer:= 4
    );
   port(
	CLK: IN STD_LOGIC;				--clk
	Reset:IN STD_LOGIC			--reset
	
);

end Odunze_FSM_data_path;
 
ARCHITECTURE STRUCTURAL OF Odunze_FSM_DATA_PATH IS
-------------------------COMPONENTS--------------------------------------
 
COMPONENT Odunze_gen_reg
 generic(P: integer:= 16);
 
 port( 
    Reset : in std_logic;
    CLK : in std_logic;
    EN: in std_logic;
    OP_A : in std_logic_vector(P-1 downto 0);
    OP_Q : out std_logic_vector(P-1 downto 0)
     );
end COMPONENT;
 
--------------------------------------------------------------------------
----------------------------------------------------------------------------
COMPONENT Odunze_SEXT_generic  
 
generic(
        P : integer:=5;
        M : integer:=16);
    port(
         OP_A: in std_logic_vector (P-1 downto 0);
     OP_Q: out std_logic_vector(M-1 downto 0)
        );
 
END COMPONENT; 
 
--------------------------------------------------------------------------
--------------------------------------------------------------------------
 
COMPONENT Odunze_ZEXT_generic 
 
generic(
        P: integer;
        M: integer
        );
 port(
         OP_A: in std_logic_vector (P-1 downto 0);
         OP_Q: out std_logic_vector(M-1 downto 0)
        );
 
END COMPONENT; 
 
--------------------------------------------------------------------------
--------------------------------------------------------------------------
 
COMPONENT Odunze_nzp_register 
 
port(
    Reset : in std_logic;
    CLK : in std_logic;
    EN: in std_logic;
    OP_A : in std_logic_vector(15 downto 0);
    OP_NZP : out std_logic_vector(2 downto 0)
);
 
END COMPONENT; 
 
---------------------------------------------------------------------------
----------------------------------------------------------------------------
COMPONENT Odunze_comb_adder 
 
 port(
        OP_A: in std_logic_vector(15 downto 0);
    	OP_B: in std_logic_vector(15 downto 0);
    	OP_Q: out std_logic_vector(15 downto 0)
   );
 
END COMPONENT; 
---------------------------------------------------------------------------
----------------------------------------------------------------------------
COMPONENT Odunze_alu 
 
port(
    	OP_A: in std_logic_vector(15 downto 0);
      	OP_B: in std_logic_vector(15 downto 0);
    	sel: in std_logic_vector(1 downto 0);
    	OP_Q: out std_logic_vector(15 downto 0)
    );
 
END COMPONENT;
----------------------------------------------------------------------------
---------------------------------------------------------------------------
COMPONENT Odunze_tri_state
 
port (
       	OP_A: in std_logic_vector (15 downto 0);
       	EN: in std_logic;
       	OP_Q: out std_logic_vector (15 downto 0)
     );
 
END COMPONENT;
----------------------------------------------------------------------------
----------------------------------------------------------------------------
COMPONENT Odunze_regarray
 
generic(P: integer:=16;
     W: integer:=3;
     E: natural:=8
    );
 port(CLK: in std_logic;
      Reset: in std_logic;
      LD_REG: in std_logic;
      DR: in std_logic_vector(W-1 downto 0);
      OP_A: in std_logic_vector(P-1 downto 0);
      SR1: in std_logic_vector(W-1 downto 0);
      SR2: in std_logic_vector(W-1 downto 0);
      OP_Q1: out std_logic_vector(P-1 downto 0);
      OP_Q2: out std_logic_vector(P-1 downto 0)
     );
 
END COMPONENT;
----------------------------------------------------------------------------
---------------------------------------------------------------------------
COMPONENT Odunze_pc_cnt
 
port( CLK: in std_logic;
        Reset: in std_logic;
        En: in std_logic;
        OP_A: in std_logic_vector(15 downto 0);
        OP_Q: out std_logic_vector(15 downto 0):= (others=>'0');
    	OP_R: out std_logic_vector(15 downto 0):= (others=>'0')
      );
 
END COMPONENT;
--------------------------------MEMORY COMPONENTS----------------------------
----------------------------------------------------------------------------
COMPONENT Odunze_mar
 
port (
     CLK: in std_logic;
     EN: in std_logic;
     Reset: in std_logic;
     BUS_IN: in std_logic_vector(15 downto 0);
     MAR_OUT: out std_logic_vector(8 downto 0)
     );
 
END COMPONENT;
----------------------------------------------------------------------------
----------------------------------------------------------------------------
COMPONENT  Odunze_mdr
 
port (
     CLK: in std_logic;
     EN: in std_logic;
     Reset: in std_logic;
     BUS_IN: in std_logic_vector(15 downto 0);
     MEM_IN: in std_logic_vector(15 downto 0);
     MDR_OUT: out std_logic_vector(15 downto 0)
     );
 
END COMPONENT;
-------------------------------------------------------------------------
COMPONENT Odunze_ram 
 
generic(add_width: integer:=9;
    width: integer:=16);
port(
CLK: in std_logic;
     MEM_ADDRESS: in std_logic_vector(add_width-1 downto 0);
     OP_A :in std_logic_vector(width-1 downto 0);
     RW_EN :in std_logic;
     OP_Q :out std_logic_vector(width-1 downto 0);
     MEM_EN: in std_logic
);
 
END COMPONENT;
-----------------------------MULTIPLEXER COMPONENTS------------------------
COMPONENT  Odunze_pcmux
 
port(
    BUS_IN, ADDER_IN, PC_PLUS: in std_logic_vector(15 downto 0);
    SEL: in std_logic_vector(1 downto 0);
    OP_Q: out std_logic_vector(15 downto 0)
    );
 
END COMPONENT;
---------------------------------------------------------------------------
COMPONENT Odunze_sr2mux
 
 port(SEXT_IN, SR2_IN: in std_logic_vector(15 downto 0);
    SEL: in std_logic;
    OP_Q: out std_logic_vector(15 downto 0));
 
END COMPONENT;
----------------------------------------------------------------------------
COMPONENT Odunze_addr2mux
 
port(SEXT11, SEXT9, SEXT6 : in std_logic_vector(15 downto 0);
    SEL: in std_logic_vector(1 downto 0);
    OP_Q: out std_logic_vector(15 downto 0));
 
END COMPONENT;
---------------------------------------------------------------------------
COMPONENT Odunze_addr1mux
 
port(SR1_IN, PC_PLUS: in std_logic_vector(15 downto 0);
    SEL: in std_logic;
    OP_Q: out std_logic_vector(15 downto 0));
 
END COMPONENT;
----------------------------------------------------------------------------
COMPONENT Odunze_marmux 
 
port(ZEXT_IN, ADDER_IN: in std_logic_vector(15 downto 0);
SEL: in std_logic;
OP_Q: out std_logic_vector(15 downto 0));
 
END COMPONENT;

----------------------------------------------------------------------------

component Odunze_LC3_FSM IS
port(
	CLK: IN STD_LOGIC;				--clk
	Reset:IN STD_LOGIC;				--reset
	OUT_FROM_IR: IN STD_LOGIC_VECTOR(15 DOWNTO 0);  --instruction
	FROM_NZP: IN STD_LOGIC_VECTOR(2 DOWNTO 0);		--nzp flag
	LD_IR: OUT STD_LOGIC;				
	LD_MARMUX: out std_logic; 
	LD_REG: out std_logic; 
	LD_PC: OUT STD_LOGIC;				
	LD_CC: OUT STD_LOGIC;
	LD_MAR: OUT STD_LOGIC;				
	LD_MDR: OUT STD_LOGIC;							
	LD_MEM: OUT STD_LOGIC;		
	GATE_PC: OUT STD_LOGIC;			
	GATE_MARMUX: OUT STD_LOGIC;			
	GATE_ALU: OUT STD_LOGIC;			
	GATE_MDR: OUT STD_LOGIC;		
	ADDR2_MUX_SEL: OUT STD_LOGIC_VECTOR(1 DOWNTO 0);
	ADDR1_MUX_SEL: OUT STD_LOGIC;			
	SR1_SEL: OUT STD_LOGIC_VECTOR(2 DOWNTO 0);	
	SR2_SEL: OUT STD_LOGIC_VECTOR(2 DOWNTO 0);	
	DR_SEL: OUT STD_LOGIC_VECTOR(2 DOWNTO 0);	
	SR2MUX_SEL: OUT STD_LOGIC;			
	PCMUX_SEL: OUT STD_LOGIC_VECTOR(1 DOWNTO 0);		
	ALU_SEL: OUT STD_LOGIC_VECTOR(1 DOWNTO 0);	
	MEM_EN: OUT STD_LOGIC;
	MEM_RW_EN: OUT STD_LOGIC				
);
end component;


----------------------------------------------------------------------------
----------------------------------------------------------------------------

signal GBUS: std_logic_vector(15 downto 0);
signal IR_OUT: std_logic_vector(15 downto 0);
signal OUT_SEXT5: std_logic_vector(15 downto 0);
signal OUT_SEXT6: std_logic_vector(15 downto 0);
signal OUT_SEXT9: std_logic_vector(15 downto 0);
signal OUT_SEXT11: std_logic_vector(15 downto 0);
signal OUT_SEXT8: std_logic_vector(15 downto 0);
signal ADDR1MUX_OUT: std_logic_vector(15 downto 0);
signal ADDR2MUX_OUT: std_logic_vector(15 downto 0);
signal SR1_OUT: std_logic_vector(15 downto 0);
signal SR2_OUT: std_logic_vector(15 downto 0);
signal SR2MUX_OUT: std_logic_vector(15 downto 0);
signal ALU_OUT: std_logic_vector(15 downto 0);
signal ADDER_OUT: std_logic_vector(15 downto 0);
signal PC_PLUS: std_logic_vector(15 downto 0);
signal PCMUX_OUT: std_logic_vector(15 downto 0);
signal PC_OUT: std_logic_vector(15 downto 0);
signal MARMUX_OUT: std_logic_vector(15 downto 0);
signal MAR_OUT: std_logic_vector(8 downto 0); 
signal MEM_OUT: std_logic_vector(15 downto 0);
signal MDR_OUT: std_logic_vector(15 downto 0);
signal NZP_IN: std_logic_vector(15 downto 0);
SIGNAL LD_PC : std_logic;
SIGNAL LD_REG: std_logic;
SIGNAL LD_IR : std_logic;
SIGNAL LD_MAR : std_logic;
SIGNAL LD_MEM : std_logic;
SIGNAL LD_MDR : std_logic;
SIGNAL LD_MARMUX : std_logic;
SIGNAL LD_CC : std_logic;
SIGNAL PCMUX_SEL : std_logic_vector (1 downto 0);
SIGNAL GATE_PC : std_logic;
SIGNAL GATE_MARMUX : std_logic;
SIGNAL GATE_MDR : std_logic;
SIGNAL GATE_ALU : std_logic;
SIGNAL MEM_EN : std_logic;
SIGNAL MEM_RW_EN : std_logic;
SIGNAL ADDR1_MUX_SEL : std_logic;
SIGNAL ADDR2_MUX_SEL : std_logic_vector (1 downto 0);
SIGNAL SR2MUX_SEL : std_logic; 
SIGNAL SR1_SEL: std_logic_vector (2 DOWNTO 0);	
SIGNAL SR2_SEL: std_logic_vector (2 DOWNTO 0);	
SIGNAL DR_SEL:  std_logic_vector (2 DOWNTO 0);		
SIGNAL ALU_SEL: std_logic_vector (1 DOWNTO 0);	
SIGNAL FROM_IR: std_logic_vector (15 DOWNTO 0);  --instruction
SIGNAL FROM_NZP: std_logic_vector (2 DOWNTO 0);	


begin
-----------------------------------
--- Odunze Henry
--- PC
-----------------------------------
PCMUX_TO_PC: Odunze_pcmux port map(GBUS, ADDER_OUT, PC_PLUS, PCMUX_SEL, PCMUX_OUT); 

-----------------------------------
--- Odunze Henry
--- PC Counter
-----------------------------------
PC_TO_GATE: Odunze_pc_cnt
        port map(CLK, Reset, LD_PC, PCMUX_OUT, PC_OUT, PC_PLUS);
GATE_TO_BUS: Odunze_tri_state port map(PC_OUT, GATE_PC, GBUS);
 
 
-----------------------------------
--- Odunze Henry
--- IR 
-----------------------------------
IR: Odunze_gen_reg generic map(P)
    port map(Reset, CLK, LD_IR, GBUS, IR_OUT);

-----------------------------------
-- SIGN EXTENDERS
-----------------------------------
TO_SEXT6: Odunze_sext_generic generic map(W+2, P) port map(IR_OUT(W+1 downto 0), OUT_SEXT6);
TO_SEXT9: Odunze_sext_generic generic map(2*W+1, P)port map(IR_OUT(2*W downto 0), OUT_SEXT9); 
TO_SEXT11: Odunze_sext_generic generic map(3*W-1, P) port map(IR_OUT(2*W+2 downto 0), OUT_SEXT11);
TO_SEXT5: Odunze_sext_generic generic map(W+1, P)  port map(IR_OUT(W downto 0), OUT_SEXT5);
TO_ZEXT8: Odunze_zext_generic generic map(E, P) port map(IR_OUT(2*W-1 downto 0), OUT_SEXT8); 
 
-----------------------------------
-- Register Array  
-----------------------------------
REGARRAY: Odunze_regarray generic map(P, W-1, E)
port map(CLK, Reset, LD_REG, DR_SEL(W-2 downto 0), GBUS, SR1_SEL(W-2 downto 0), SR2_SEL(W-2 downto 0), SR1_OUT, SR2_OUT);


-----------------------------------
-- ALU
-----------------------------------
SR2MUX_TO_ALU: Odunze_sr2mux port map(OUT_SEXT5, SR2_OUT, SR2MUX_SEL, SR2MUX_OUT);
ALU_TO_GATE: Odunze_alu port map(SR2MUX_OUT, SR1_OUT, ALU_SEL, ALU_OUT);
GATE_ALU_TO_BUS: Odunze_tri_state port map(ALU_OUT, GATE_ALU, GBUS);
 
-----------------------------------
-- COMB. ADDER
-----------------------------------
TO_ADDR2MUX: Odunze_addr2mux port map(OUT_SEXT6, OUT_SEXT9, OUT_SEXT11, ADDR2_MUX_SEL,ADDR2MUX_OUT);
TO_ADDR1MUX: Odunze_addr1mux port map(SR1_OUT, PC_PLUS, ADDR1_MUX_SEL, ADDR1MUX_OUT);
TO_COMB_ADDER: Odunze_comb_adder  port map(ADDR1MUX_OUT, ADDR2MUX_OUT, ADDER_OUT);

-----------------------------------
-- MARMUX
-----------------------------------
TO_MARMUX: Odunze_marmux port map(OUT_SEXT8, ADDER_OUT, LD_MEM, MARMUX_OUT);
TO_GATEMM: Odunze_tri_state port map(MARMUX_OUT, GATE_MARMUX, GBUS);
 
-----------------------------------
-- MEMORY
-----------------------------------

BUS_TO_MAR: Odunze_mar port map(CLK, LD_MAR, Reset, GBUS, MAR_OUT);
TO_RAM: Odunze_ram generic map(2*W+1, P)port map(CLK, MAR_OUT, MDR_OUT, MEM_RW_EN, MEM_OUT, MEM_EN);
TO_MDR: Odunze_mdr port map(CLK, LD_MDR, Reset, GBUS, MEM_OUT, MDR_OUT); 
TO_GATE_MDR: Odunze_tri_state port map(MDR_OUT, GATE_MDR, GBUS); 
 
NZP_TO_FSM : Odunze_nzp_register port map (CLK, Reset, LD_CC, NZP_IN, FROM_NZP);

FSM_TO_DP : Odunze_LC3_FSM 
port map (CLK, Reset, FROM_IR, FROM_NZP, LD_IR, LD_MARMUX, LD_REG, LD_PC, LD_CC, 
LD_MAR, LD_MDR, LD_MEM, GATE_PC, GATE_MARMUX,
GATE_ALU, GATE_MDR, ADDR2_MUX_SEL, ADDR1_MUX_SEL, SR1_SEL, SR2_SEL, DR_SEL, SR2MUX_SEL, 
PCMUX_SEL, ALU_SEL, MEM_EN, MEM_RW_EN );


END STRUCTURAL;


------------------------------------------------TESTBENCH-----------------------------------------------------------------------------
--File Name:		LC3_data_path_all_tb_amoo.vhd
--VHDL Source File:	A very simple test bench
-- 				Or, you can use the standard TB!
--Components: 		see lc3_parts_all_amoo.vhd
-- 			     requires lc3_datapath_all_amoo
--Comments: 		behavioral testbench description
--Odunze Henry
--ADvanced Digital Design
--Fall 217
-----------------------------------------------------------
library IEEE;
use IEEE.std_logic_1164.all;
use IEEE.std_logic_arith.all;
use IEEE.std_logic_unsigned.all;
use IEEE.NUMERIC_STD.all;

entity Odunze_DATAPATH_ALL_TB is
end Odunze_DATAPATH_ALL_TB;


architecture TB1 of Odunze_DATAPATH_ALL_TB is

constant P : integer := 16;
constant E : integer :=8;
constant W : integer := 4;

component Odunze_FSM_data_path is 
   generic(
	P: integer:= 16;
       	E: integer:= 8;
       	W: integer:= 4
    );
   port(
	CLK: IN STD_LOGIC;				--clk
	Reset:IN STD_LOGIC			--reset
	
);
end component Odunze_FSM_data_path;

signal	CLKtb		: std_logic; 				
signal	RSTntb	: std_logic;				

begin

CLK_GEN: process 
begin 
while now <= 600 ns loop 
CLKtb <='1'; wait for 5 ns; CLKtb <='0'; wait for 5 ns; 
end loop; 
wait; 
end process; 

Reset: process
begin
RSTntb  <='1', '0' after 10 ns;
wait;
end process;


--------------------------------------do not change-----------------------------------------------
datap: Odunze_FSM_data_path port map ( CLK=>CLKtb, Reset=>RSTntb);

end TB1;

configuration CFG_Odunze_DATAPATH_ALL_TB of Odunze_DATAPATH_ALL_TB is
	for TB1
	end for;
end CFG_Odunze_DATAPATH_ALL_TB;
