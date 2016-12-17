# HAL_UART_ECHO
This project show HAL UART usage on simple example "echo"

Setup:
- NUCLEO-L053R8 board: http://www.st.com/en/evaluation-tools/nucleo-l053r8.html
- STM32 CubeMX: http://www.st.com/en/development-tools/stm32cubemx.html
- Drivers ST-Link Utility: http://www.st.com/en/embedded-software/stsw-link004.html
- System Workbench for STM32: http://www.openstm32.org/System+Workbench+for+STM32
- Hterm: http://www.der-hammer.info/terminal/ (best com-port terminal)

CubeMX:
In Win platform CubeMX don't install packages. Inslall it manualy: 
- dawnload pakege for STM32L0-series en.stm32cubel0.zip http://www.st.com/en/embedded-software/stm32cubel0.html
- Help >> Inslall New Libraries >> "From local" (left bottom button)

Click "Hew Project" in workspace of "File >> New project..."
"Board Selector" >> Type of board: Nucleo64 >> MCU Series: STM32L0 >> in "Boards list" chuse NUCLEO-L053R8.
On Pinout ribbon chuse USART2, mode Asynchronous.
On Clock Configuration ribbon check HCLK: put 32 and "Enter". Yes on Clock Wizard. On USART2 Source Mux check SYSCLK.
On Configuration ribbon click on NVIC button. Check enable USERT2 globla interrupt.
On Configuration ribbon click on USART2 button.Chuse Word Leght: 8 Bits (including Parity).
Click "Progect" in main menu "Project >> Settings..."
Put $your_project_name, chuse $your_folder and chuse Toolchain: SW4STM32. "OK"
Click "Progect" in main menu "Project >> Generate Code..."

System Workbench for STM32:
"File >> Import", On General click Existing Projects into Workspace, next, select $your_folder in Select root directory
Select $your_project_name in Projects, Finish

/* USER CODE BEGIN Includes */
#define RING_BUFFER_SIZE 256  // User can change this parametr
#define UART_BUFFER_SIZE 1    // User can change this parametr
/* USER CODE END Includes */

/* USER CODE BEGIN PV */
/* Private variables ---------------------------------------------------------*/
/* Uart2 states */
uint8_t rx_ready = 0;
uint8_t tx_ready = 1;  // Defalt state non zero!
/* Uart2 buffers */
uint8_t rx_buffer[UART_BUFFER_SIZE] = { 0 };
uint8_t tx_buffer[UART_BUFFER_SIZE] = { 0 };
/* Ring buffer apperanse */
uint8_t rb_in  = 0;
uint8_t rb_out = 0;
uint8_t ring_buffer[RING_BUFFER_SIZE] = {0};
/* USER CODE END PV */

/* USER CODE BEGIN PFP */
/* Private function prototypes -----------------------------------------------*/
uint8_t Ring_Buffer_Size(uint8_t rb_in, uint8_t rb_out) {
  if (rb_in >= rb_out)
    return (rb_in - rb_out);
  else
    return ((RING_BUFFER_SIZE - rb_out) + rb_in);
}
/* USER CODE END PFP */

  /* USER CODE BEGIN 2 */
  /* Enable receive IT and callback */
  __HAL_UNLOCK(&huart2);  // Hotfix
  HAL_UART_Receive_IT(&huart2, rx_buffer, sizeof(rx_buffer));
  /* USER CODE END 2 */
  
    /* USER CODE BEGIN 3 */
    /* If data has been reseived */
    if (rx_ready > 0) {
      rx_ready = 0;  // Reset flag
      /* Pull to ring buffer */
      for(uint8_t i = 0; i < sizeof(rx_buffer); i++) {
        ring_buffer[rb_in++] = rx_buffer[i];
      }
      /* Wait for receive new data */
      HAL_UART_Receive_IT(&huart2, rx_buffer, sizeof(rx_buffer));
    }
    /* If have some data to transmit and uart ready */
    if ((Ring_Buffer_Size(rb_in, rb_out) != 0)&(tx_ready > 0)) {
      tx_ready = 0;  // Reset flag
      /* Pullup in ring buffer */
      for(uint8_t i = 0; i < sizeof(tx_buffer); i++) {
        tx_buffer[i] = ring_buffer[rb_out++];
      }
      HAL_UART_Transmit_IT(&huart2, tx_buffer, sizeof(tx_buffer));
    }
  }
  /* USER CODE END 3 */
  
  /* USER CODE BEGIN 4 */
/* HAL_UART_Receive_IT() have been done */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart){
  /* Led indications */
  HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_SET);
  /* Data has been reseived */
  rx_ready = 1;
}
/* HAL_UART_Transmit_IT() have been done */
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart){
  /* Led indications */
  HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_RESET);
  /* Transmit has been done */
  tx_ready = 1;
}
/* USER CODE END 4 */

TODO take commit
