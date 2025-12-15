# FPGA VGA Plinko Game (Nexys Board)

## Project Overview
This project implements a **VGA-based Plinko-style arcade game** on the **Digilent Nexys A7 FPGA board** using **VHDL** and **Vivado**. The game features gravity-driven ball motion through a fixed field of pegs and multiple scoring zones at the bottom of the screen. The system generates real-time VGA video, processes user input via onboard pushbuttons, and displays a running score on the onboard 7-segment display.

Here's the game we are basing our project on:

![Plinko](RealPlinko.jpg)

The design follows the modular structure used in the course labs, with separate components for:
- VGA timing and pixel scanning
- Game logic (ball physics, peg collisions, scoring)
- Clock generation
- Seven-segment display control
## Required Hardware
- **Digilent Nexys FPGA board**
<img src="image.jpg" alt="drawing" width="200"/>

- **VGA monitor** (800×600 @ 60 Hz)
<img src="monitor.png" alt="drawing" width="200"/>

- **VGA cable**
<img src="vga_cable.jpg" alt="drawing" width="200"/>

- **USB cable** (programming and power)
<img src="micro_usb_cable.png" alt="drawing" width="200"/>

--- 
## Diagrams
Some kind of high level block diagram showing how different parts of your program connect together and/or showing how what you have created might fit into a more complete system could be appropriate instead

## Add gif/vid/pics of gameplay
![Game](PlinkoGame.jpg)

---

## Inputs and Outputs
### Inputs
| Signal | Board Resource | Description |
|------|---------------|-------------|
| `clk_in` | 100 MHz clock | System clock |
| `btn0` | Pushbutton | Serve / start | 
| `btnl` | Pushbutton | Move bat left |
| `btnr` | Pushbutton | Move bat right |

### Outputs
| Signal | Board Resource | Description |
|-------|---------------|-------------|
| `VGA_red[3:0]` | VGA DAC | Red channel |
| `VGA_green[3:0]` | VGA DAC | Green channel |
| `VGA_blue[3:0]` | VGA DAC | Blue channel |
| `VGA_hsync` | VGA | Horizontal sync |
| `VGA_vsync` | VGA | Vertical sync |
| `SEG7_anode[7:0]` | 7-seg | Digit select |
| `Counter` | 7-seg | Segment data |
| `SEG7_seg[6:0]` | 7-seg | Segment data |
| `SEG7_seg[6:0]` | 7-seg | Segment data |

### Bat and Ball VHDL File
```vhdl
LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;
USE IEEE.STD_LOGIC_ARITH.ALL;
USE IEEE.STD_LOGIC_UNSIGNED.ALL;

ENTITY bat_n_ball IS
    PORT (
        v_sync : IN STD_LOGIC;
        pixel_row : IN STD_LOGIC_VECTOR(10 DOWNTO 0);
        pixel_col : IN STD_LOGIC_VECTOR(10 DOWNTO 0);
        serve : IN STD_LOGIC; -- initiates serve
        red : OUT STD_LOGIC;
        green : OUT STD_LOGIC;
        blue : OUT STD_LOGIC;
        counter : OUT STD_LOGIC_VECTOR(6 DOWNTO 0);
        attempts : OUT STD_LOGIC_VECTOR(3 DOWNTO 0);
        ball_x : IN STD_LOGIC_VECTOR (10 DOWNTO 0) -- current ball x position
    );
END bat_n_ball;

ARCHITECTURE Behavioral OF bat_n_ball IS
    CONSTANT bsize : INTEGER := 8; -- ball size in pixels
    SIGNAL bat_w : INTEGER := 200; -- bat width in pixels
    CONSTANT bat_h : INTEGER := 10; -- bat height in pixels
    CONSTANT peg_row_gap : INTEGER := 40;
    CONSTANT peg_col_gap : INTEGER := 50;
    CONSTANT max_rows : INTEGER := 10; --Rows
    CONSTANT max_columns : INTEGER := 14; --Columns
    -- distance ball moves for each frame
    SIGNAL ball_speed : STD_LOGIC_VECTOR (10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR (3, 11);
    SIGNAL ball_on : STD_LOGIC; 
    SIGNAL ball_on1 : STD_LOGIC := '0'; 
    SIGNAL bat_on1 : STD_LOGIC; 
    SIGNAL bat_on2 : STD_LOGIC; 
    SIGNAL bat_on3 : STD_LOGIC; 
    SIGNAL bat_on4 : STD_LOGIC; 
    SIGNAL bat_on5 : STD_LOGIC; 
    SIGNAL bat_on6 : STD_LOGIC; 
    SIGNAL bat_on7 : STD_LOGIC; 
    SIGNAL bat_on8 : STD_LOGIC; 

    SIGNAL game_on : STD_LOGIC := '0'; -- indicates whether ball is in play
    --ball on top logic
    SIGNAL ball_x_1: STD_LOGIC_VECTOR(10 DOWNTO 0);
    SIGNAL ball_y: STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(50, 11);
    --peg logic
    SIGNAL ball_x1: STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(100, 11);
    SIGNAL ball_y1: STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(100, 11);
    
    -- bat vertical position
    CONSTANT bat_x1 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(45, 11);
    CONSTANT bat_y1 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x2 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(145, 11);
    CONSTANT bat_y2 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x3 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(245, 11);
    CONSTANT bat_y3 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x4 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(345, 11);
    CONSTANT bat_y4 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x5 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(445, 11);
    CONSTANT bat_y5 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x6 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(545, 11);
    CONSTANT bat_y6 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x7 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(645, 11);
    CONSTANT bat_y7 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);
    CONSTANT bat_x8 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(745, 11);
    CONSTANT bat_y8 : STD_LOGIC_VECTOR(10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(575, 11);

    -- current ball motion - initialized to (+ ball_speed) pixels/frame in both X and Y directions
    SIGNAL ball_x_motion, ball_y_motion : STD_LOGIC_VECTOR(10 DOWNTO 0) := ball_speed;
    SIGNAL counter1: STD_LOGIC_VECTOR(6 DOWNTO 0):= "0000000";
    SIGNAL attempts1 : STD_LOGIC_VECTOR(3 DOWNTO 0) := "0101";  -- Start with 5 attempts
    SIGNAL game_over : STD_LOGIC := '0';  -- Flag to indicate game is over
    
    type balls is array (0 to max_columns, 0 to max_rows) of integer;
    SIGNAL balls_x, balls_y: balls;
    
    -- Track which pegs are hit
    type peg_hit_array is array (0 to max_columns, 0 to max_rows) of STD_LOGIC;
    SIGNAL pegs_hit : peg_hit_array := (others => (others => '0'));
    
    -- Collision flag to prevent multiple hits per frame
    SIGNAL peg_collision : STD_LOGIC := '0';
    SIGNAL bat_collision : STD_LOGIC := '0';
    SIGNAL lockpoint : STD_LOGIC := '0';
    
    -- Random number generator for left/right bounce
    SIGNAL Random_Generator : STD_LOGIC_VECTOR(10 DOWNTO 0) := "10110100101";

BEGIN
    red <= NOT (bat_on1 or bat_on4 or bat_on6);
    green <= NOT (bat_on1 or bat_on2 or bat_on3 or bat_on5 or bat_on6 or bat_on8 or ball_on or ball_on1);
    blue <= NOT (bat_on3 or bat_on4 or bat_on5 or bat_on7 or bat_on8 or ball_on or ball_on1);
    
    -- Output counter and attempts values
    counter <= counter1;
    attempts <= attempts1;
    
    -- process to draw round ball
    balldraw1: PROCESS (pixel_row, pixel_col) IS
        VARIABLE vx, vy : STD_LOGIC_VECTOR (10 DOWNTO 0);
    BEGIN
        IF pixel_col <= ball_x_1 THEN
            vx := ball_x_1 - pixel_col;
        ELSE
            vx := pixel_col - ball_x_1;
        END IF;
        IF pixel_row <= ball_y THEN
            vy := ball_y - pixel_row;
        ELSE
            vy := pixel_row - ball_y;
        END IF;
        IF ((vx * vx) + (vy * vy)) < (bsize * bsize) THEN
            ball_on <= '1';  -- Ball always visible, not just when game_on = '1'
        ELSE
            ball_on <= '0';
        END IF;
    END PROCESS;
    
    -- process to draw pegs
    peg_proc : PROCESS(pixel_row, pixel_col)
        VARIABLE vx, vy: INTEGER;
        VARIABLE row_idx, col_idx : INTEGER;
        VARIABLE peg_x, peg_y : INTEGER;
    BEGIN
        ball_on1 <= '0';

        FOR row_idx IN 0 TO max_rows LOOP
            peg_y := row_idx * peg_row_gap + 100;

            FOR col_idx IN 0 TO max_columns LOOP
                IF (row_idx MOD 2 = 0) THEN
                    peg_x := col_idx * peg_col_gap + 50;
                ELSE
                    peg_x := col_idx * peg_col_gap + 25;
                END IF;
                balls_x(col_idx, row_idx) <= peg_x;
                balls_y(col_idx, row_idx) <= peg_y;

                vx := CONV_INTEGER(pixel_col);
                vy := CONV_INTEGER(pixel_row);

    -- Only draw peg if it hasn't been hit
                IF pegs_hit(col_idx, row_idx) = '0' THEN
                    IF (vx > peg_x - 4 AND vx < peg_x + 4 AND
                        vy > peg_y - 4 AND vy < peg_y + 4) THEN
                        ball_on1 <= '1';
                    END IF;
                END IF;
            END LOOP;
        END LOOP;
    END PROCESS;

    -- Bat drawing processes
    batdraw1: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x1 - bat_w) OR (bat_x1 <= bat_w)) AND
         pixel_col <= bat_x1 + bat_w AND
             pixel_row >= bat_y1 - bat_h AND
             pixel_row <= bat_y1 + bat_h THEN
                bat_on1 <= '1';
        ELSE
            bat_on1 <= '0';
        END IF;
    END PROCESS;
    
    batdraw2: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x2 - bat_w) OR (bat_x2 <= bat_w)) AND
         pixel_col <= bat_x2 + bat_w AND
             pixel_row >= bat_y2 - bat_h AND
             pixel_row <= bat_y2 + bat_h THEN
                bat_on2 <= '1';
        ELSE
            bat_on2 <= '0';
        END IF;
    END PROCESS;
    
    batdraw3: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x3 - bat_w) OR (bat_x3 <= bat_w)) AND
         pixel_col <= bat_x3 + bat_w AND
             pixel_row >= bat_y3 - bat_h AND
             pixel_row <= bat_y3 + bat_h THEN
                bat_on3 <= '1';
        ELSE
            bat_on3 <= '0';
        END IF;
    END PROCESS;
    
    batdraw4: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x4 - bat_w) OR (bat_x4 <= bat_w)) AND
         pixel_col <= bat_x4 + bat_w AND
             pixel_row >= bat_y4 - bat_h AND
             pixel_row <= bat_y4 + bat_h THEN
                bat_on4 <= '1';
        ELSE
            bat_on4 <= '0';
        END IF;
    END PROCESS;
    
    batdraw5: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x5 - bat_w) OR (bat_x5 <= bat_w)) AND
         pixel_col <= bat_x5 + bat_w AND
             pixel_row >= bat_y5 - bat_h AND
             pixel_row <= bat_y5 + bat_h THEN
                bat_on5 <= '1';
        ELSE
            bat_on5 <= '0';
        END IF;
    END PROCESS;
    
    batdraw6: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x6 - bat_w) OR (bat_x6 <= bat_w)) AND
         pixel_col <= bat_x6 + bat_w AND
             pixel_row >= bat_y6 - bat_h AND
             pixel_row <= bat_y6 + bat_h THEN
                bat_on6 <= '1';
        ELSE
            bat_on6 <= '0';
        END IF;
    END PROCESS;
    
    batdraw7: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x7 - bat_w) OR (bat_x7 <= bat_w)) AND
         pixel_col <= bat_x7 + bat_w AND
             pixel_row >= bat_y7 - bat_h AND
             pixel_row <= bat_y7 + bat_h THEN
                bat_on7 <= '1';
        ELSE
            bat_on7 <= '0';
        END IF;
    END PROCESS;
    
    batdraw8: PROCESS (pixel_row, pixel_col) IS
    BEGIN
        IF ((pixel_col >= bat_x8 - bat_w) OR (bat_x8 <= bat_w)) AND
         pixel_col <= bat_x8 + bat_w AND
             pixel_row >= bat_y8 - bat_h AND
             pixel_row <= bat_y8 + bat_h THEN
                bat_on8 <= '1';
        ELSE
            bat_on8 <= '0';
        END IF;
    END PROCESS;
    
    -- Main ball motion and collision detection process
    mball : PROCESS
        VARIABLE temp : STD_LOGIC_VECTOR (11 DOWNTO 0);
        VARIABLE row_idx, col_idx : INTEGER;
        VARIABLE peg_x, peg_y : INTEGER;

    BEGIN
        WAIT UNTIL rising_edge(v_sync);
        
        -- Serve: start new game
        IF serve = '1' AND game_on = '0' THEN
            ball_y <= CONV_STD_LOGIC_VECTOR(50, 11);  -- Start at top of screen
            game_on <= '1';
            ball_y_motion <= ball_speed;
            ball_x_motion <= (Others => '0');
            peg_collision <= '0';
            bat_collision <= '0';
            -- Reset game if attempts reached 0
            IF game_over = '1' THEN
                attempts1 <= "0101";  -- Reset to 5 attempts
                counter1 <= "0000000";  -- Reset score
                game_over <= '0';
            END IF;
        ELSIF game_on = '0' THEN
            -- When game is not active, ball follows the input ball_x position
            ball_x_1 <= ball_x;
        END IF;
        
        -- Bounce off top wall
        IF ball_y <= bsize THEN
            ball_y_motion <= ball_speed;
        -- Bounce off bottom wall - decrement attempts
        ELSIF ball_y + bsize >= 575 THEN
            ball_y_motion <= ball_speed;
            game_on <= '0';
            -- Decrement attempts when ball reaches bottom
            IF attempts1 > "0000" THEN
                attempts1 <= attempts1 - 1;
            ELSE
                attempts1 <= "0000";
                game_over <= '1';
            END IF;
        END IF;
        
        -- Bounce off left or right walls
        IF ball_x_1 + bsize >= 800 THEN
            ball_x_motion <= (NOT ball_speed) + 1;
        ELSIF ball_x_1 <= bsize THEN
            ball_x_motion <= ball_speed;
        END IF;
        
        -- Peg collision detection
        peg_collision <= '0';
        FOR row_idx IN 0 TO max_rows LOOP
            FOR col_idx IN 0 TO max_columns LOOP
                peg_x := balls_x(col_idx, row_idx);
                peg_y := balls_y(col_idx, row_idx);
                
                IF (ball_x_1 + bsize) >= (peg_x - 4) AND
                   (ball_x_1 - bsize) <= (peg_x + 4) AND
                   (ball_y + bsize) >= (peg_y - 4) AND
                   (ball_y - bsize) <= (peg_y + 4) THEN
                    
                    peg_collision <= '1';
                    ball_y_motion <= ball_speed;  -- Always bounce down
                    
                    -- Randomized movement based on whether the end bit is 0 or 1
                    IF Random_Generator(0) = '0' THEN
                        ball_x_motion <= ball_speed;  -- Go right
                    ELSE
                        ball_x_motion <= (NOT ball_speed) + 1;  -- Go left
                    END IF;
                    
                    -- Update the end bit for next collision
                    Random_Generator <= Random_Generator(9 DOWNTO 0) & (Random_Generator (10) XOR Random_Generator (9));
                END IF;
            END LOOP;
        END LOOP;
        
        -- Update ball position (only during gameplay)
        IF game_on = '1' THEN
            ball_x_1 <= ball_x_1 + ball_x_motion;
        END IF;
        
        -- Bat collision detection and decrement attempts
        bat_collision <= '0';
        IF (ball_x_1 + bsize) >= (bat_x1 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x1 + bat_w) AND
           (ball_y + bsize) >= (bat_y1 - bat_h) AND
           (ball_y - bsize) <= (bat_y1 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 1;  -- Bat 1: 1 point
                lockpoint <= '1';
                -- Decrement attempts after hitting bat
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 2 collision - Pink - 3 points
        ELSIF (ball_x_1 + bsize) >= (bat_x2 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x2 + bat_w) AND
           (ball_y + bsize) >= (bat_y2 - bat_h) AND
           (ball_y - bsize) <= (bat_y2 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 3;  -- Bat 2: 3 points
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 3 collision - Red - 0 points
        ELSIF (ball_x_1 + bsize) >= (bat_x3 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x3 + bat_w) AND
           (ball_y + bsize) >= (bat_y3 - bat_h) AND
           (ball_y - bsize) <= (bat_y3 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 0;  -- Bat 3: 0 points
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 4 collision - Green - 2 points
        ELSIF (ball_x_1 + bsize) >= (bat_x4 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x4 + bat_w) AND
           (ball_y + bsize) >= (bat_y4 - bat_h) AND
           (ball_y - bsize) <= (bat_y4 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 2;  -- Bat 4: 2 points
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 5 collision - Red - 0 points
        ELSIF (ball_x_1 + bsize) >= (bat_x5 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x5 + bat_w) AND
           (ball_y + bsize) >= (bat_y5 - bat_h) AND
           (ball_y - bsize) <= (bat_y5 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 0;  -- Bat 5: 0 points
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 6 collision - Blue - 1 point
        ELSIF (ball_x_1 + bsize) >= (bat_x6 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x6 + bat_w) AND
           (ball_y + bsize) >= (bat_y6 - bat_h) AND
           (ball_y - bsize) <= (bat_y6 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 1;  -- Bat 6: 1 point
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 7 collision - Gold - 5 points
        ELSIF (ball_x_1 + bsize) >= (bat_x7 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x7 + bat_w) AND
           (ball_y + bsize) >= (bat_y7 - bat_h) AND
           (ball_y - bsize) <= (bat_y7 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 5;  -- Bat 7: 5 points
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        -- Bat 8 collision - Red - 0 points
        ELSIF (ball_x_1 + bsize) >= (bat_x8 - bat_w) AND
           (ball_x_1 - bsize) <= (bat_x8 + bat_w) AND
           (ball_y + bsize) >= (bat_y8 - bat_h) AND
           (ball_y - bsize) <= (bat_y8 + bat_h) THEN
            IF lockpoint = '0' THEN
                bat_collision <= '1';
                counter1 <= counter1 + 0;  -- Bat 8: 0 points
                lockpoint <= '1';
                IF attempts1 > "0001" THEN
                    attempts1 <= attempts1 - 1;
                ELSIF attempts1 = "0001" THEN
                    attempts1 <= "0000";
                    game_over <= '1';
                END IF;
            END IF;
            game_on <= '0';
        ELSE
            lockpoint <= '0';
        END IF;
        
        -- Update ball vertical position
        temp := ('0' & ball_y) + (ball_y_motion(10) & ball_y_motion);
        IF game_on = '0' THEN
            ball_y <= CONV_STD_LOGIC_VECTOR(50, 11);
            bat_w <= 40;
            -- Reset pegs after each bat hit
            IF bat_collision = '1' THEN
                FOR row_idx IN 0 TO max_rows LOOP
                    FOR col_idx IN 0 TO max_columns LOOP
                        pegs_hit(col_idx, row_idx) <= '0';
                    END LOOP;
                END LOOP;
            END IF;
        ELSIF temp(11) = '1' THEN
            ball_y <= (OTHERS => '0');
        ELSE
            ball_y <= temp(10 DOWNTO 0);
        END IF;
    END PROCESS;
END Behavioral;
```
### Pong VHDL File
``` vhdl
LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;
USE IEEE.STD_LOGIC_ARITH.ALL;
USE IEEE.STD_LOGIC_UNSIGNED.ALL;

ENTITY pong IS
    PORT (
        clk_in : IN STD_LOGIC;
        VGA_red : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
        VGA_green : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
        VGA_blue : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
        VGA_hsync : OUT STD_LOGIC;
        VGA_vsync : OUT STD_LOGIC;
        btnl : IN STD_LOGIC;
        btnr : IN STD_LOGIC;
        btn0 : IN STD_LOGIC;
        SEG7_anode : OUT STD_LOGIC_VECTOR (7 DOWNTO 0);
        SEG7_seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
    ); 
END pong;

ARCHITECTURE Behavioral OF pong IS
    SIGNAL pxl_clk : STD_LOGIC := '0';
    SIGNAL S_red, S_green, S_blue : STD_LOGIC;
    SIGNAL S_vsync : STD_LOGIC;
    SIGNAL S_pixel_row, S_pixel_col : STD_LOGIC_VECTOR (10 DOWNTO 0);
    SIGNAL ball_x_pos : STD_LOGIC_VECTOR (10 DOWNTO 0) := CONV_STD_LOGIC_VECTOR(400, 11);
    SIGNAL count : STD_LOGIC_VECTOR (20 DOWNTO 0);
    SIGNAL display : std_logic_vector (19 DOWNTO 0);
    SIGNAL led_mpx : STD_LOGIC_VECTOR (2 DOWNTO 0);
    SIGNAL btnl_counter : STD_LOGIC_VECTOR (19 DOWNTO 0) := (OTHERS => '0');
    SIGNAL btnr_counter : STD_LOGIC_VECTOR (19 DOWNTO 0) := (OTHERS => '0');
    SIGNAL attempts_sig : STD_LOGIC_VECTOR(3 DOWNTO 0);
    
    COMPONENT bat_n_ball IS
        PORT (
            v_sync : IN STD_LOGIC;
            pixel_row : IN STD_LOGIC_VECTOR(10 DOWNTO 0);
            pixel_col : IN STD_LOGIC_VECTOR(10 DOWNTO 0);
            serve : IN STD_LOGIC;
            red : OUT STD_LOGIC;
            green : OUT STD_LOGIC;
            blue : OUT STD_LOGIC;
            counter : OUT STD_LOGIC_VECTOR(6 DOWNTO 0);
            attempts : OUT STD_LOGIC_VECTOR(3 DOWNTO 0);
            ball_x : IN STD_LOGIC_VECTOR (10 DOWNTO 0)
        );
    END COMPONENT;
    
    COMPONENT vga_sync IS
        PORT (
            pixel_clk : IN STD_LOGIC;
            red_in    : IN STD_LOGIC_VECTOR (3 DOWNTO 0);
            green_in  : IN STD_LOGIC_VECTOR (3 DOWNTO 0);
            blue_in   : IN STD_LOGIC_VECTOR (3 DOWNTO 0);
            red_out   : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
            green_out : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
            blue_out  : OUT STD_LOGIC_VECTOR (3 DOWNTO 0);
            hsync : OUT STD_LOGIC;
            vsync : OUT STD_LOGIC;
            pixel_row : OUT STD_LOGIC_VECTOR (10 DOWNTO 0);
            pixel_col : OUT STD_LOGIC_VECTOR (10 DOWNTO 0)
        );
    END COMPONENT;
    
    COMPONENT clk_wiz_0 is
        PORT (
            clk_in1  : in std_logic;
            clk_out1 : out std_logic
        );
    END COMPONENT;
    
    COMPONENT leddec16 IS
        PORT (
            dig : IN STD_LOGIC_VECTOR (2 DOWNTO 0);
            data : IN STD_LOGIC_VECTOR (19 DOWNTO 0);
            anode : OUT STD_LOGIC_VECTOR (7 DOWNTO 0);
            seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)
        );
    END COMPONENT;
    
BEGIN
    button_control : PROCESS (clk_in) is
    BEGIN
        if rising_edge(clk_in) then
            IF btnl = '1' THEN
                btnl_counter <= btnl_counter + 1;
                IF btnl_counter = 1000000 THEN
                    IF ball_x_pos > 20 THEN
                        ball_x_pos <= ball_x_pos - 5;
                    END IF;
                    btnl_counter <= (OTHERS => '0');
                END IF;
            ELSE
                btnl_counter <= (OTHERS => '0');
            END IF;
            
            IF btnr = '1' THEN
                btnr_counter <= btnr_counter + 1;
                IF btnr_counter = 1000000 THEN
                    IF ball_x_pos < 780 THEN
                        ball_x_pos <= ball_x_pos + 5;
                    END IF;
                    btnr_counter <= (OTHERS => '0');
                END IF;
            ELSE
                btnr_counter <= (OTHERS => '0');
            END IF;
        end if;
    END PROCESS;
    
    counter_display : PROCESS (clk_in)
    BEGIN
        if rising_edge(clk_in) then
            count <= count + 1;
        end if;
    END PROCESS;
    
    led_mpx <= count(19 DOWNTO 17);
    
    add_bb : bat_n_ball
    PORT MAP(
        v_sync => S_vsync, 
        pixel_row => S_pixel_row, 
        pixel_col => S_pixel_col, 
        serve => btn0, 
        red => S_red, 
        green => S_green, 
        blue => S_blue,
        counter => display(6 DOWNTO 0),
        attempts => attempts_sig,
        ball_x => ball_x_pos
    );
    
    display(19 DOWNTO 16) <= attempts_sig;
    
    vga_driver : vga_sync
    PORT MAP(
        pixel_clk => pxl_clk, 
        red_in => S_red & "000", 
        green_in => S_green & "000", 
        blue_in => S_blue & "000", 
        red_out => VGA_red, 
        green_out => VGA_green, 
        blue_out => VGA_blue, 
        pixel_row => S_pixel_row, 
        pixel_col => S_pixel_col, 
        hsync => VGA_hsync, 
        vsync => S_vsync
    );
    VGA_vsync <= S_vsync;
    
    clk_wiz_0_inst : clk_wiz_0
    port map (
        clk_in1 => clk_in,
        clk_out1 => pxl_clk
    );
    
    led1 : leddec16
    PORT MAP(
        dig => led_mpx, 
        data => display, 
        anode => SEG7_anode, 
        seg => SEG7_seg
    );
    
END Behavioral;
```
### Pong VHDL File
``` vhdl
LIBRARY IEEE;
USE IEEE.STD_LOGIC_1164.ALL;

ENTITY leddec16 IS
	PORT (
		dig : IN STD_LOGIC_VECTOR (2 DOWNTO 0); -- which digit to currently display
		data : IN STD_LOGIC_VECTOR (19 DOWNTO 0); -- 20-bit (5-digit) data
		anode : OUT STD_LOGIC_VECTOR (7 DOWNTO 0); -- which anode to turn on
		seg : OUT STD_LOGIC_VECTOR (6 DOWNTO 0)); -- segment code for current digit
END leddec16;

ARCHITECTURE Behavioral OF leddec16 IS
	SIGNAL data4 : STD_LOGIC_VECTOR (3 DOWNTO 0); -- binary value of current digit
BEGIN
	-- Select digit data to be displayed in this mpx period
	data4 <= data(3 DOWNTO 0) WHEN dig = "000" ELSE -- digit 0
	         data(7 DOWNTO 4) WHEN dig = "001" ELSE -- digit 1
	         data(11 DOWNTO 8) WHEN dig = "010" ELSE -- digit 2
	         data(15 DOWNTO 12) WHEN dig = "011" ELSE -- digit 3
	         data(19 DOWNTO 16); -- digit 4
	         
	-- Turn on segments corresponding to 4-bit data word
	seg <= "0000001" WHEN data4 = "0000" ELSE -- 0
	       "1001111" WHEN data4 = "0001" ELSE -- 1
	       "0010010" WHEN data4 = "0010" ELSE -- 2
	       "0000110" WHEN data4 = "0011" ELSE -- 3
	       "1001100" WHEN data4 = "0100" ELSE -- 4
	       "0100100" WHEN data4 = "0101" ELSE -- 5
	       "0100000" WHEN data4 = "0110" ELSE -- 6
	       "0001111" WHEN data4 = "0111" ELSE -- 7
	       "0000000" WHEN data4 = "1000" ELSE -- 8
	       "0000100" WHEN data4 = "1001" ELSE -- 9
	       "0001000" WHEN data4 = "1010" ELSE -- A
	       "1100000" WHEN data4 = "1011" ELSE -- B
	       "0110001" WHEN data4 = "1100" ELSE -- C
	       "1000010" WHEN data4 = "1101" ELSE -- D
	       "0110000" WHEN data4 = "1110" ELSE -- E
	       "0111000" WHEN data4 = "1111" ELSE -- F
	       "1111111";
	-- Turn on anode of 7-segment display addressed by 3-bit digit selector dig
	anode <= "11111110" WHEN dig = "000" ELSE -- 0
	         "11111101" WHEN dig = "001" ELSE -- 1
	         "11111011" WHEN dig = "010" ELSE -- 2
	         "11110111" WHEN dig = "011" ELSE -- 3
	         "11101111" WHEN dig = "100" ELSE -- 4
	         "11111111";
END Behavioral;
```


--- 
## Modifications
talk about how we used lab 6

--- 
## Project Setup Instructions
1. Download all VHDL and .xdc files from this repo
2. Create a new Vivado project for the Nexys A7-100T
3. Add Sources:
pong.vhd
4. Add Constraints File: .xdc
5. Run Synthesis
6. Run Implementation
7. Generate Bitstream
8. Open Hardware Manager and Program the Device

--- 
## Conclusion
In conclusion, our group has the wonderful opportunity We created the game Plinko by coding VHDL  in Vivado.
We got the opportunity to create a game and strengthen our skills in VHDL.

## Future Works
Future works include creating the Plinko with a betting system and implementing changes in location of the bats after hitting it.





