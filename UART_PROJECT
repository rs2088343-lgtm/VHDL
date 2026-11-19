library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity uart_controller_tb is
end uart_controller_tb;

architecture Behavioral of uart_controller_tb is

    constant CLOCK_PERIOD : time := 20 ns;

    signal clk       : std_logic := '0';
    signal reset     : std_logic := '1';

    signal tx_start  : std_logic := '0';
    signal tx_data   : std_logic_vector(7 downto 0) := (others => '0');
    signal tx_serial : std_logic;
    signal tx_busy   : std_logic;
    signal tx_done   : std_logic;

    signal rx_serial : std_logic;
    signal rx_data   : std_logic_vector(7 downto 0);
    signal rx_busy   : std_logic;
    signal rx_done   : std_logic;

    signal sim_done  : std_logic := '0';

begin

    clk <= not clk after CLOCK_PERIOD / 2 when sim_done = '0' else '0';

    rx_serial <= tx_serial;

    DUT : entity work.uart_controller
        generic map (
            CLOCK_FREQ => 50000000,
            BAUD_RATE  => 9600
        )
        port map (
            clk       => clk,
            reset     => reset,

            tx_start  => tx_start,
            tx_data   => tx_data,
            tx_serial => tx_serial,
            tx_busy   => tx_busy,
            tx_done   => tx_done,

            rx_serial => rx_serial,
            rx_data   => rx_data,
            rx_busy   => rx_busy,
            rx_done   => rx_done
        );


    STIMULUS : process

        variable tx_done_seen : boolean := false;
        variable rx_done_seen : boolean := false;

    begin

        reset <= '1';

        for i in 1 to 10 loop
            wait until rising_edge(clk);
        end loop;

        reset <= '0';

        wait until rising_edge(clk);

        tx_data  <= x"A5";
        tx_start <= '1';

        wait until rising_edge(clk);

        tx_start <= '0';

        for i in 0 to 600000 loop

            wait until rising_edge(clk);

            if tx_done = '1' then
                tx_done_seen := true;
            end if;

            if rx_done = '1' then
                rx_done_seen := true;
            end if;

            exit when tx_done_seen and rx_done_seen;

        end loop;

        assert tx_done_seen
            report "ERROR: TX transmission did not complete"
            severity error;

        assert rx_done_seen
            report "ERROR: RX reception did not complete"
            severity error;

        assert rx_data = x"A5"
            report "ERROR: Received data does not match transmitted data"
            severity error;

        if tx_done_seen and rx_done_seen and rx_data = x"A5" then

            report "SUCCESS: UART loopback test passed"
            severity note;

        end if;

        sim_done <= '1';

        wait;

    end process;

end Behavioral;
