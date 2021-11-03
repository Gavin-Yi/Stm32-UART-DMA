# *Stm32-F407-UART-DMA SubProgram*

This program describes the UART transmission process in DMA mode between a Stm32-F407 board and PC using a USB2TTL connector.
The Stm32 also runs a FreeRTOS which is useful.

## How to use
1. Clion\MinGW\OpenOCD\arm-none-eabi-gcc
2. ST-link\Stm32F407
3. Download and run!

## Functionality
This subprogram implement such a functionality:
- The board receives a string of any length (<`BUFFERSIZE`)
- The board immediately returns the same string using the same UART port.

## Detail
At the beginning of the main program, the `SystemClock_Config()` function is called to initialize the clock to run at 8MHz.
The PLL configuration parameters are already set to make the Clock run at 84MHz, only need to change the `SYSCLKSource` to `RCC_SYSCLKSOURCE_PLLCLK`.
However, this subprogram does not need such high clock frequency, so 8MHz is enough. T^T.

The UART is configured as follows:
- PA9: Tx___PA10-Rx
- BaudRate = 115200 baud
- Word Length = 8 bits
- One Stop bit
- None parity
- Hardware flow control disabled
- Rx and Tx are enabled at the same time

The DMA is configured as follows； 
1. DMA2 - Stream2
    - Peripheral to Memory
    - Pripheral Inc disabled
    - Memory Inc enabled
    - Data alignment = Byte
    - Mode = Normal
    - No fifo **(Note: this means no fifo threshold level, still fifo is used)** 
2. DMA2 - Stream7
    - Memory to Peripheral
    - Pripheral Inc disabled
    - Memory Inc enabled
    - Data alignment = Byte
    - Mode = Normal
    - No fifo **(Note: this means no fifo threshold level, still fifo is used)**

### Take care
1. The `__HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);` must be added after the CubeMX automatically generates the code.
2. A `USER_UART_IRQHandler(UART_HandleTypeDef* huart)` should be defined and added to the `UART1_IRQHandler` to answer the UART_IDLE interrupt.
3. The `SerialBuf` should only be emptied when the transmission is done. Implemented in `HAL_UART_TxCpltCallback`.
4. Every time after a reception process, the DMA should be reinit to set the memory inc pointer to its original place. Or the data mat be displacement.
