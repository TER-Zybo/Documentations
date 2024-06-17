# Projet VHDL : Contrôle de Compteur avec Encodeur Rotatif PMODENC et Affichage LED

## Introduction

Ce projet démontre comment concevoir un système basé sur VHDL en utilisant Xilinx Vivado pour contrôler un compteur avec l'encodeur rotatif Digilent PmodENC. La valeur du compteur est affichée sur 4 LEDs de la carte FPGA Zybo Z7-20.

Ce projet s'inspire du guide [Nexys 3 VHDL Example - ISE 14.2](https://digilent.com/reference/_media/reference/pmod/pmodenc/pmodenc_ise_demo_14-2.zip) de [Digilent](https://digilent.com/reference/pmod/pmodenc/start).

## Prérequis

- Xilinx Vivado Design Suite
- Carte FPGA Zybo Z7-20
- Encodeur rotatif Digilent PmodENC

## Code vhdl

Le projet utilise trois fichiers VHDL : pmodenc.vhd, encoder.vhd et debouncer.vhd. Ces fichiers définissent la logique de l'encodeur rotatif, le compteur et le décompteur, ainsi que la stabilisation des signaux d'entrée.

??? info "Code VHDL : `pmodenc.vhd`"
    Ce fichier contient l'entité principale qui connecte le décompteur et l'encodeur.

    ```vhdl
    library IEEE;
    use IEEE.STD_LOGIC_1164.ALL;
    use IEEE.STD_LOGIC_ARITH.ALL;
    use IEEE.STD_LOGIC_UNSIGNED.ALL;


    entity PmodENC is
        Port (
    		clk: in std_logic;
            pmod : in STD_LOGIC_VECTOR (3 downto 0);
    		leds: inout STD_LOGIC_VECTOR (3 downto 0)
    	);
    end PmodENC;

    architecture Behavioral of PmodENC is
        signal AO, BO: std_logic;

        begin
            C0: entity work.Debouncer port map ( 
                    clk=>clk,
                    Ain=>pmod(0),
                    Bin=>pmod(1),
                    Aout=> AO,
                    Bout=> BO
                );
                
            C1: entity work.Encoder port map ( 
                    clk=>clk,
                    A=>AO,
                    B=>BO,
                    BTN=>pmod(2),
                    LED=>leds
                );
    end Behavioral;
    ```

??? info "Code VHDL : `encoder.vhd`"
    Ce fichier gère l'état de l'encodeur rotatif et met à jour les LEDs en fonction des mouvements.

    ```vhdl
    library IEEE;
    use IEEE.STD_LOGIC_1164.ALL;
    use IEEE.STD_LOGIC_ARITH.ALL;
    use IEEE.STD_LOGIC_UNSIGNED.ALL;

    entity Encoder is
        Port (
            clk: in STD_LOGIC;
            -- signals from the pmod
            A : in  STD_LOGIC;	
            B : in  STD_LOGIC;
            BTN : in  STD_LOGIC;
            -- position of the shaft
            LED: inout STD_LOGIC_VECTOR (3 downto 0)
          );
    end Encoder;

    architecture Behavioral of Encoder is
        type stateType is ( idle, R1, R2, R3, L1, L2, L3, add, sub);
        signal curState, nextState: stateType;
    begin
    	clock: process (clk, BTN)
        begin
            if (BTN='1') then
                curState <= idle;
                LED <= "0000";
            elsif (clk'event and clk = '1') then
                if curState /= nextState then
                    if (curState = add) then
                        if LED < "1111" then 
                            LED <= LED+1;
                        else
                            LED <= "0000";
                        end if;
                    elsif (curState = sub) then
                        if LED > "0000" then 
                            LED <= LED-1;
                        else
                            LED <= "1111";
                        end if;
                    else
                        LED <= LED;
                    end if;
                else
                    LED <= LED;
                end if;
                curState <= nextState;
            end if;
        end process; 

        next_state: process (curState, A, B)
    	
        begin
            case curState is
                when idle =>
                    if B = '0' then
                        nextState <= R1;
                    elsif A = '0' then
                        nextState <= L1;
                    else
                        nextState <= idle;
                    end if;
                -- start of right cycle
                --R1
                when R1 =>
                    if B='1' then
                        nextState <= idle;
                    elsif A = '0' then
                        nextState <= R2;
                    else
                        nextState <= R1;
                    end if;
                --R2  					
                when R2 =>				
                    if A ='1' then
                        nextState <= R1;
                    elsif B = '1' then
                        nextState <= R3;
                    else
                        nextState <= R2;
                   end if;
                --R3	
                when R3 =>
                    if B ='0' then
                        nextState <= R2;
                    elsif A = '1' then
                        nextState <= add;
                    else
                        nextState <= R3;
                    end if;
                when add =>	
                    nextState <= idle;
                -- start of left cycle
                --L1 
                when L1 =>
                    if A ='1' then
                        nextState <= idle;
                    elsif B = '0' then
                        nextState <= L2;
                    else
                        nextState <= L1;
                    end if;
                --L2	
                when L2 =>
                    if B ='1' then
                       nextState <= L1;
                    elsif A = '1' then
                        nextState <= L3;
                    else
                        nextState <= L2;
                    end if;
                --L3
                when L3 =>
                    if A ='0' then
                        nextState <= L2;
                    elsif B = '1' then
                        nextState <= sub;
                    else
                        nextState <= L3;
                    end if;
                when sub =>	
                    nextState <= idle;
                when others =>
                    nextState <= idle;
            end case;
    	end process; 	

    end Behavioral;
    ```

??? info "Code VHDL : `debouncer.vhd`"
    Ce fichier filtre les signaux d'entrée pour éliminer les rebonds (bruit) et fournit des signaux stables à l'encodeur.

    ```vhdl
    library IEEE;
    use IEEE.STD_LOGIC_1164.ALL;
    use IEEE.STD_LOGIC_ARITH.ALL;
    use IEEE.STD_LOGIC_UNSIGNED.ALL;


    entity Debouncer is
        Port ( 
            clk : in  STD_LOGIC;
            Ain : in  STD_LOGIC; 
            Bin : in  STD_LOGIC;
    		Aout: out STD_LOGIC;
    		Bout: out STD_LOGIC
    	);
    end Debouncer;

    architecture Behavioral of Debouncer is

    signal sclk: std_logic_vector (6 downto 0);
    signal sampledA, sampledB : std_logic;
    begin

    	process(clk)
    		begin 
    			if clk'event and clk = '1' then
    				sampledA <= Ain;
    				sampledB <= Bin;
    				-- clock is divided to 1MHz
    				-- samples every 1uS to check if the input is the same as the sample
    				-- if the signal is stable, the debouncer should output the signal
    				if sclk = "1100100" then
    					if sampledA = Ain then 
    						Aout <= Ain;
    					end if;
    					if sampledB = Bin then 
    						Bout <= Bin;
    					end if;
    					sclk <="0000000";
    				else
    					sclk <= sclk +1;
    				end if;
    			end if;
    	end process;
    	
    end Behavioral;
    ```

## Configuration matérielle

??? info "Contraintes XDC"
    Les contraintes XDC sont utilisées pour mapper les signaux du projet VHDL aux broches physiques de la carte FPGA.

    ```xdc
    create_clock -period 100.000 -name clk [get_ports clk]
    set_property PACKAGE_PIN K17 [get_ports clk]
    set_property IOSTANDARD LVCMOS33 [get_ports clk]

    ## LEDs
    set_property -dict { PACKAGE_PIN M14   IOSTANDARD LVCMOS33 } [get_ports {leds[0]}]
    set_property -dict { PACKAGE_PIN M15   IOSTANDARD LVCMOS33 } [get_ports {leds[1]}]
    set_property -dict { PACKAGE_PIN G14   IOSTANDARD LVCMOS33 } [get_ports {leds[2]}]
    set_property -dict { PACKAGE_PIN D18   IOSTANDARD LVCMOS33 } [get_ports {leds[3]}]

    ## JE
    set_property PACKAGE_PIN V12      [get_ports {pmod[0]}]					
    set_property IOSTANDARD LVCMOS33 [get_ports {pmod[0]}]
    set_property PACKAGE_PIN W16      [get_ports {pmod[1]}]					
    set_property IOSTANDARD LVCMOS33 [get_ports {pmod[1]}]
    set_property PACKAGE_PIN J15      [get_ports {pmod[2]}]					
    set_property IOSTANDARD LVCMOS33 [get_ports {pmod[2]}]
    set_property PACKAGE_PIN H15      [get_ports {pmod[3]}]					
    set_property IOSTANDARD LVCMOS33 [get_ports {pmod[3]}]
    ```


### Broches du connecteur PMOD JE pour le PMODENC

Nous avons utilisé les broches du connecteur PMOD JE pour connecter l'encodeur rotatif PMODENC à la carte Zybo Z7-20. Voici la configuration matérielle utilisée pour ce projet :

| Signal | Pin |
|--------|-----|
| A      | V12 |
| B      | W16 |
| BTN    | J15 |
| SWT    | H15 |


### Horloge

| Signal | Pin | Description    |
|--------|-----|----------------|
| clk    | K17 | sysclk du Zybo |

Pour plus d'informations sur les ports du Zybo Z7-20, consultez la [documentation officielle](https://github.com/Digilent/digilent-xdc/blob/master/Zybo-Z7-Master.xdc).

## Utilisation du Projet

### Ouverture du Projet dans Vivado

1.  **Lancer Vivado**:
    - Ouvrez Xilinx Vivado Design Suite sur votre ordinateur.

2.  **Ouvrir le Projet Existant**:
    - Allez dans `File > Open Project`.
    - Naviguez vers le répertoire du [repository](https://github.com/TER-Zybo/PmodENC_VHDL/tree/main) et sélectionnez le fichier de projet `ENC_demo.xpr` pour l'ouvrir.

### Construction du Projet

1.  **Générer le Bitstream**:
    - Cliquez sur `Generate Bitstream` pour compiler votre design en un fichier binaire exécutable sur la FPGA.

### Programmation de la FPGA

1.  **Connecter la Carte FPGA**:
    - Connectez la carte Zybo Z7-20 à votre ordinateur en utilisant un câble USB.

2.  **Programmer le Dispositif**:
    - Dans Vivado, allez dans `Open Hardware Manager` et cliquez sur `Open Target > Auto Connect`.
    - Cliquez sur `Program Device` et sélectionnez le fichier bitstream généré pour programmer la FPGA.

### Exécution de l'Application

1.  **Connecter le PMODENC**:
    - Attachez l'encodeur rotatif PMODENC au connecteur PMOD approprié sur la carte Zybo Z7-20.

2.  **Observer l'Affichage LED**:
    - Tournez l'encodeur rotatif PMODENC.
    - La valeur du compteur sera affichée sur les 4 LEDs de la carte Zybo Z7-20.