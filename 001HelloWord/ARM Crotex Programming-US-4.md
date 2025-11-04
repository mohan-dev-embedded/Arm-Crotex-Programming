2025-11-01 15:40

Status: #Finished 

Tags: [[Basics]], [[Embedded Systems]], [[Programming]]

# Creating Project

- Create a workspace.
- Create STM32 project.
- Select target.
- Setup project with project type as `Empty`.
- `build` to compile the code.

# Serial Wire

- `printf` works over SWO pin(Serial Wire Output)
- Works for M3/M4/M7
![[Pasted image 20251101165157.png]]
- The Arm cortex M4 processor has a ITM unit(Instrumentation Trace Macrocell Unit) - it is optional though.
- The ITM helps to interface the SWD Debug connector. - Serial Wire Debug, 2 pin (debug), 1 pin(trace).
- SWD is a two-wire protocol which is a alternative to JTAG
- The Physical layer of SWD consists of two lines
	- SWDIO: a bidirectional data line
	- SWCLK: a clock driven by the host
- It can used for programming internal flash, access memory regions, add breakpoints, stop/run CPU.
- We can use the Serial Wire Viewer to view the print statements for debugging.
- Difference between SWD and JTAG
	- JTAG needs more pins and was used for older MCU's
	- SWD has only 3 pins and can use SWV
![[Pasted image 20251101170106.png]]

## Code Snippet to run SWV
```C

/////////////////////////////////////////////////////////////////////////////////////////////////////////
//					Implementation of printf like feature using ARM Cortex M3/M4/ ITM functionality
//					This function will not work for ARM Cortex M0/M0+
//					If you are using Cortex M0, then you can use semihosting feature of openOCD
/////////////////////////////////////////////////////////////////////////////////////////////////////////


//Debug Exception and Monitor Control Register base address
#define DEMCR        			*((volatile uint32_t*) 0xE000EDFCU )

/* ITM register addresses */
#define ITM_STIMULUS_PORT0   	*((volatile uint32_t*) 0xE0000000 )
#define ITM_TRACE_EN          	*((volatile uint32_t*) 0xE0000E00 )

void ITM_SendChar(uint8_t ch)
{

	//Enable TRCENA
	DEMCR |= ( 1 << 24);

	//enable stimulus port 0
	ITM_TRACE_EN |= ( 1 << 0);

	// read FIFO status in bit [0]:
	while(!(ITM_STIMULUS_PORT0 & 1));

	//Write to ITM stimulus port0
	ITM_STIMULUS_PORT0 = ch;
}
```
 - Need to add this code to `syscalls.c` to make it work.
 - In the `write` function comment out `__io_putchar(*ptr++);` and add `ITM_SendChar(*ptr++);` to print the output to the console.

# Printing Output

- Build the project.
- In debug project make sure the `SWV` is enabled and click apply/ok.
- This will upload the code to the board.
- Go to Window -> Show View -> SWV -> SWV ITM Data Console
- Click `Configure Trace` and select port `0`.
- Next start the `Trace`.
- Run the program and the output will be displayed.
![[screenshot-2025-11-04_18-07-19.png]]



### Reference

https://www.udemy.com/share/101Wdc3@RF-keeaUZyaQ9cnDGmOyOTex8xFAL-rgl6tqwObXjBAZD9lMJz2bFtf0IDTf6zrNCA==/

https://github.com/niekiran/Embedded-C/blob/master/All_source_codes/target/itm_send_data.c