# -Ping-Pong. 程式碼概述 (Code Overview)
這個設計描述了一個名為 pingpong 的實體 (entity)，它使用狀態機 (FSM) 來控制遊戲邏輯，並使用時脈除頻器 (CLK_DIV) 來減慢 LED 移動的速度，使其能在實體電路板上被人眼觀察到。
2. 實體 (Entity) 定義
定義了電路板上的輸入與輸出腳位：
i_clk: 輸入主時脈訊號 (主頻)。
i_rst: 輸入重置訊號 (Reset)，低電平有效 ('0' 表示重置)。
i_swL: 輸入左側開關訊號 (左邊球員的拍子)。
i_swR: 輸入右側開關訊號 (右邊球員的拍子)。
o_led: 輸出到 8 個 LED 燈的訊號 (8 位元向量)。
3. 架構 (Architecture) - 主要訊號與狀態
定義了遊戲內部使用的狀態和訊號：
STATE_TYPE 定義了四種遊戲狀態：
MovingR: 球（LED 燈）正在向右移動。
MovingL: 球正在向左移動。
Lwin: 左邊玩家得分（贏得一分）。
Rwin: 右邊玩家得分。
state: 當前遊戲狀態。
prev_state: 前一個遊戲狀態，用於判斷得分的瞬間。
led_r: 控制 o_led 輸出的內部訊號（代表球的位置）。
scoreL / scoreR: 追蹤左右玩家得分的訊號（未使用於輸出腳位）。
slow_clk: 經除頻後較慢的時脈，用於控制球的移動速度。
4. 主要處理程序 (Processes) 解說
A. CLK_DIV 時脈除頻器
功能: 將高速的系統時脈 i_clk 轉換為慢速的 slow_clk，使 LED 的移動速度足夠慢，方便觀察。
原理: cnt 是一個 25 位元的計數器。當 cnt 計數到一半時，cnt(23) 位元會產生一個頻率慢很多倍的方波，即為 slow_clk。
B. FSM 狀態機 (Finite State Machine)
功能: 處理遊戲規則和狀態轉換，決定遊戲當下應該處於哪種模式（移動或得分）。
邏輯:
在 MovingR 狀態下，如果球到達最右邊 (led_r(0)='1') 且右邊開關 i_swR 被按下，則切換到 MovingL（成功反彈）。否則，如果球出界或玩家提前按鈕，切換到 Lwin。
MovingL 狀態邏輯類似，判斷左邊玩家的開關。
在 Lwin 或 Rwin 狀態下，等待玩家按下對應的開關來「發球」，重新開始移動狀態。
C. LED_PLED 控制器
功能: 根據 slow_clk 的節奏和當前狀態更新 LED 顯示。
邏輯:
MovingR: 將 led_r 訊號向右邏輯移位（>>），使亮的燈往右邊移動。
MovingL: 將 led_r 訊號向左邏輯移位（<<），使亮的燈往左邊移動。
Lwin/Rwin: 顯示特定的勝利圖案（例如四個燈全亮）作為得分提示。
D. score_L_p / score_R_p 計分板
功能: 追蹤比分。
邏輯: 僅在特定的狀態轉換瞬間（例如從 MovingR 變為 Lwin 的那一刻）增加對應玩家的分數。
總結
這是一個功能完整的 FPGA 乒乓球遊戲基礎設計。您需要將此程式碼載入 FPGA 開發軟體（如 Xilinx Vivado 或 Intel Quartus），將腳位對應到您的硬體，即可運行。


