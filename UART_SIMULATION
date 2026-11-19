library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity baud_generator is
    generic (
        CLOCK_FREQ : integer := 50000000;
        BAUD_RATE  : integer := 9600
    );
    port (
        clk       : in  std_logic;
        reset     : in  std_logic;
        baud_tick : out std_logic
    );
end baud_generator;

architecture Behavioral of baud_generator is

    constant DIVISOR : integer := CLOCK_FREQ / BAUD_RATE;

    signal counter : integer range 0 to DIVISOR - 1 := 0;

begin

    process(clk)
    begin
        if rising_edge(clk) then
            if reset = '1' then
                counter   <= 0;
                baud_tick <= '0';
            else
                if counter = DIVISOR - 1 then
                    counter   <= 0;
                    baud_tick <= '1';
                else
                    counter   <= counter + 1;
                    baud_tick <= '0';
                end if;
            end if;
        end if;
    end process;

end Behavioral;


library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity uart_tx is
    port (
        clk       : in  std_logic;
        reset     : in  std_logic;
        baud_tick : in  std_logic;
        tx_start  : in  std_logic;
        tx_data   : in  std_logic_vector(7 downto 0);

        tx_serial : out std_logic;
        tx_busy   : out std_logic;
        tx_done   : out std_logic
    );
end uart_tx;

architecture Behavioral of uart_tx is

    type state_type is (IDLE, START, DATA, STOP);

    signal current_state : state_type := IDLE;

    signal shift_reg : std_logic_vector(7 downto 0) := (others => '0');

    signal bit_count : integer range 0 to 7 := 0;

    signal stop_started : std_logic := '0';

    signal tx_serial_reg : std_logic := '1';
    signal tx_busy_reg   : std_logic := '0';
    signal tx_done_reg   : std_logic := '0';

begin

    tx_serial <= tx_serial_reg;
    tx_busy   <= tx_busy_reg;
    tx_done   <= tx_done_reg;

    process(clk)
    begin
        if rising_edge(clk) then

            if reset = '1' then

                current_state <= IDLE;
                shift_reg <= (others => '0');
                bit_count <= 0;
                stop_started <= '0';

                tx_serial_reg <= '1';
                tx_busy_reg   <= '0';
                tx_done_reg   <= '0';

            else

                tx_done_reg <= '0';

                case current_state is

                    when IDLE =>

                        tx_serial_reg <= '1';
                        tx_busy_reg   <= '0';

                        if tx_start = '1' then

                            shift_reg <= tx_data;
                            bit_count <= 0;
                            tx_busy_reg <= '1';

                            current_state <= START;

                        end if;

                    when START =>

                        if baud_tick = '1' then

                            tx_serial_reg <= '0';
                            current_state <= DATA;

                        end if;

                    when DATA =>

                        if baud_tick = '1' then

                            tx_serial_reg <= shift_reg(0);

                            shift_reg <= '0' & shift_reg(7 downto 1);

                            if bit_count = 7 then

                                stop_started <= '0';
                                current_state <= STOP;

                            else

                                bit_count <= bit_count + 1;

                            end if;

                        end if;

                    when STOP =>

                        if baud_tick = '1' then

                            if stop_started = '0' then

                                tx_serial_reg <= '1';
                                stop_started <= '1';

                            else

                                tx_busy_reg <= '0';
                                tx_done_reg <= '1';

                                stop_started <= '0';
                                current_state <= IDLE;

                            end if;

                        end if;

                end case;

            end if;
        end if;
    end process;

end Behavioral;


library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity uart_rx is
    generic (
        CLOCK_FREQ : integer := 50000000;
        BAUD_RATE  : integer := 9600
    );
    port (
        clk       : in  std_logic;
        reset     : in  std_logic;
        rx_serial : in  std_logic;

        rx_data   : out std_logic_vector(7 downto 0);
        rx_busy   : out std_logic;
        rx_done   : out std_logic
    );
end uart_rx;

architecture Behavioral of uart_rx is

    type state_type is (ST_IDLE, ST_START, ST_DATA, ST_STOP);

    signal current_state : state_type := ST_IDLE;

    constant CLKS_PER_BIT : integer := CLOCK_FREQ / BAUD_RATE;
    constant HALF_BIT     : integer := CLKS_PER_BIT / 2;

    signal clk_count : integer range 0 to CLKS_PER_BIT - 1 := 0;
    signal bit_count : integer range 0 to 7 := 0;

    signal rx_shift_reg : std_logic_vector(7 downto 0) := (others => '0');
    signal rx_data_reg  : std_logic_vector(7 downto 0) := (others => '0');

    signal rx_sync_1 : std_logic := '1';
    signal rx_sync_2 : std_logic := '1';

    signal rx_busy_reg : std_logic := '0';
    signal rx_done_reg : std_logic := '0';

begin

    rx_data <= rx_data_reg;
    rx_busy <= rx_busy_reg;
    rx_done <= rx_done_reg;

    process(clk)
    begin
        if rising_edge(clk) then

            if reset = '1' then
                rx_sync_1 <= '1';
                rx_sync_2 <= '1';

            else
                rx_sync_1 <= rx_serial;
                rx_sync_2 <= rx_sync_1;

            end if;

        end if;
    end process;


    process(clk)
    begin
        if rising_edge(clk) then

            if reset = '1' then

                current_state <= ST_IDLE;
                clk_count <= 0;
                bit_count <= 0;

                rx_shift_reg <= (others => '0');
                rx_data_reg  <= (others => '0');

                rx_busy_reg <= '0';
                rx_done_reg <= '0';

            else

                rx_done_reg <= '0';

                case current_state is

                    when ST_IDLE =>

                        clk_count <= 0;
                        bit_count <= 0;
                        rx_busy_reg <= '0';

                        if rx_sync_2 = '0' then

                            rx_busy_reg <= '1';
                            rx_shift_reg <= (others => '0');
                            current_state <= ST_START;

                        end if;

                    when ST_START =>

                        if clk_count = HALF_BIT - 1 then

                            clk_count <= 0;

                            if rx_sync_2 = '0' then

                                bit_count <= 0;
                                current_state <= ST_DATA;

                            else

                                current_state <= ST_IDLE;
                                rx_busy_reg <= '0';

                            end if;

                        else

                            clk_count <= clk_count + 1;

                        end if;

                    when ST_DATA =>

                        if clk_count = CLKS_PER_BIT - 1 then

                            clk_count <= 0;

                            rx_shift_reg <= rx_sync_2 & rx_shift_reg(7 downto 1);

                            if bit_count = 7 then

                                bit_count <= 0;
                                current_state <= ST_STOP;

                            else

                                bit_count <= bit_count + 1;

                            end if;

                        else

                            clk_count <= clk_count + 1;

                        end if;

                    when ST_STOP =>

                        if clk_count = CLKS_PER_BIT - 1 then

                            clk_count <= 0;
                            current_state <= ST_IDLE;
                            rx_busy_reg <= '0';

                            if rx_sync_2 = '1' then

                                rx_data_reg <= rx_shift_reg;
                                rx_done_reg <= '1';

                            end if;

                        else

                            clk_count <= clk_count + 1;

                        end if;

                end case;

            end if;
        end if;
    end process;

end Behavioral;


library IEEE;
use IEEE.STD_LOGIC_1164.ALL;

entity uart_controller is
    generic (
        CLOCK_FREQ : integer := 50000000;
        BAUD_RATE  : integer := 9600
    );
    port (
        clk       : in  std_logic;
        reset     : in  std_logic;

        tx_start  : in  std_logic;
        tx_data   : in  std_logic_vector(7 downto 0);
        tx_serial : out std_logic;
        tx_busy   : out std_logic;
        tx_done   : out std_logic;

        rx_serial : in  std_logic;
        rx_data   : out std_logic_vector(7 downto 0);
        rx_busy   : out std_logic;
        rx_done   : out std_logic
    );
end uart_controller;

architecture Structural of uart_controller is

    signal baud_tick_internal : std_logic;

begin

    baud_inst : entity work.baud_generator
        generic map (
            CLOCK_FREQ => CLOCK_FREQ,
            BAUD_RATE  => BAUD_RATE
        )
        port map (
            clk       => clk,
            reset     => reset,
            baud_tick => baud_tick_internal
        );


    tx_inst : entity work.uart_tx
        port map (
            clk       => clk,
            reset     => reset,
            baud_tick => baud_tick_internal,
            tx_start  => tx_start,
            tx_data   => tx_data,
            tx_serial => tx_serial,
            tx_busy   => tx_busy,
            tx_done   => tx_done
        );


    rx_inst : entity work.uart_rx
        generic map (
            CLOCK_FREQ => CLOCK_FREQ,
            BAUD_RATE  => BAUD_RATE
        )
        port map (
            clk       => clk,
            reset     => reset,
            rx_serial => rx_serial,
            rx_data   => rx_data,
            rx_busy   => rx_busy,
            rx_done   => rx_done
        );

end Structural;
