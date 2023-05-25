## 1. LCD

**Interfacing LPC2148 to LCD and display message on LCD**

```

#include<LPC214x.h>

void lcdcmd(unsigned int cmd);

void lcddata(unsigned int data);

void delay(unsigned int itime);

int main()

{

	unsigned int i = 0;

	unsigned char array[]= "MODERN COE, Shivajinagar, Pune-5";

	PINSEL0 = 0x00000000;

	PINSEL1 = 0x00000000;

	PINSEL2 = 0x00000000;//

	IODIR0 = 0x20000400;// P0.10 BC, P0.29 Enable output pins

	IODIR1 = 0x02FF0000;// P1.25-RS  and P1.16 to P1.23- D0 to D7 as outputs

	IOCLR0 = 0x00000400;

	lcdcmd(0x38);// 16x2 LCD

	delay(100);

	lcdcmd(0x06);// Increment Cursor and shift Right

	delay(100);

	lcdcmd(0x01);// Clear Display Screen

	delay(100);

	lcdcmd(0x0E);// Display ON cursor ON

	delay(100);

	lcdcmd(0x80);// First row, first position

	delay(100);

	

	

	for(i=0; i<17 ; i++)

	{

		lcddata(array[i]);

		delay(5000);

	}

	lcdcmd(0xC0);

	delay(1000);

	

	for(i=16; i<32 ; i++)

	{

		lcddata(array[i]);

		delay(5000);

	}

	lcdcmd(0xC0);

	delay(100);

	return 0;

}

void delay(unsigned int itime)

{

	int i,j;

	for(i=0; i<itime ; i++)

	for(j=0; j<200   ; j++);

}

void lcdcmd(unsigned int cmd)

{

	IOCLR1 = 0x00FF0000;

	cmd = cmd<<16; // Command to appear on P1.16-P1.22

	IOSET1 = cmd;

	IOCLR1 = 0x02000000;//RS=0 for Command Register 

	IOSET0 = 0x20000000;// Enable= 1

	delay(5000);

	IOCLR0 = 0x20000000;// Enable= 0

	delay(5000);

}

void lcddata(unsigned int data)

{

	IOCLR1 = 0x00FF0000;

	data = data<<16;//data to appear on P1.16 - P1.22

	IOSET1 = data;

	IOSET1 = 0x02000000;// RS= 1 for data Register 

	IOSET0 = 0x20000000;// Enable=1

	delay(5000);

	IOCLR0 = 0x20000000;//Enable=0

	delay(5000);

}

```

## 2. Seven Segment  

**Interfacing LPC 2148 to seven segment display and display a count from 0 to 9 with

suitable delay.**

```

#include<LPC214x.h>

unsigned int i;

void delay(unsigned int);

unsigned int arr[]={0x0000003F, 0x00000006, 0x0000005B, 0x000004F, 0x00000066, 0x0000006D, 0x00007D, 0x00000007, 0x0000007F, 0x00000009};

int main()

{

	IO1DIR = 0xFFFFFFF;

	while(1)

	{

		for(i=0;i<10;i++)

		{

			IO1SET = arr[i]<<16;

			delay(9000);

			IO1CLR = arr[i]<<16;

		}

	}

}

void delay(unsigned int k)

{

	int i, j;

	for(i=0; i<k;i++)

	for(j=0; j<=1000;j++);

}

```

## 3. ADC

**Interfacing LPC2148 for internal ADC and program to display digital value on serial port or on LCD**

```

#include<LPC214x.h>

#include<stdio.h>

void delay(void);

void delay(void)

{

	int i, j;

	for(i=0;i<2000;i++)

	for(j=0; j<2000; j++);

}

void Uart0Init(void)  //Initialize Serial Interface

	{

		PINSEL0 = 0x00000005; // Enable TxD0(P0.0) & RxD0(P0.1) 

		U0LCR = 0x83;  // 8 bits data, No parity, 1 Stop bit

		U0DLL = 97; // 9600 Baud rate @ 15MHz PCLK

		U0LCR = 0x03;

		U0DLM = 0;

	}

	

	unsigned char Uart0GetCh(void) //Read a character from RxD

	{

		while(!(U0LSR & 0x01));

		return(U0RBR);

	}

	void Uart0PutCh(unsigned char ch) // Write a character to TxD

	{

		U0THR = ch;

		while(!(U0LSR & 0x20));

	}

void Uart0PutS(unsigned char *str)

{

while(*str)

{

Uart0PutCh(*str++);

}

}

int main()

{

	int adcdata;

	float voltage;

	unsigned char volt[3];

	PINSEL0=0X00000000;

	PINSEL1 = 0x01000000;

	Uart0Init();

	AD0CR = 0x00210402;

  while(1)

	{

		if(AD0DR1 & 0x80000000)

		{			

		adcdata = (AD0DR1 & 0x0000FFC0);

		adcdata = adcdata>>6;

    voltage = ((adcdata/1023.0) * 3.3 );

			sprintf(volt, "%.1f",voltage);

			Uart0PutS("\n ADC o/p: ");

			Uart0PutS(volt);

			delay();

		}

	}

}

```

## 4. DAC

**Interfacing LPC2148 for internal DAC and program to generate waveform**

- DAC_Square

```

#include<LPC214x.h>

void delay()

{

int i;

for(i=0; i<80000;i++);

}

int main(void)

{

PINSEL1 = 0x80000;

IODIR0 = 0x80000;

delay();

while(1)

{

DACR = (0x3FF<<6);

delay();

DACR = (0X0000<<6);

delay();

}

delay();

return 0;

}

```

- DAC_triangular

```

// Triangular Waveform Generator

#include <LPC214x.h>

void delay()

{

	int i;

	for (i = 0; i < 5; i++)

		;

}

int main(void)

{

	int j;

	PINSEL1 = 0x80000;

	IODIR0 = 0x80000;

	delay();

	while (1)

	{

		for (j = 0x00; j <= 0x3FF; j++)

		{

			DACR = (j << 6);

			delay();

		};

		for (j = 0x3FF; j >= 0x00; j--)

		{

			DACR = (j << 6);

			delay();

		}

	}

}

```

## 5. LM35

**Interface LM 35 temperature Sensor with LPC 2148 and blink relay at port P1.24 if temperature exceeds 50 C.**

```

#include<LPC214x.h>

#include<stdio.h>

#include"serial.c"

#define BUZZ 4 

void delay(void);

void delay(void)

{

	int i, j;

	for(i=0;i<2000;i++)

	for(j=0; j<2000; j++);

}

int main()

{

	int adcdata;

	float voltage;

	unsigned char volt[3];

	PINSEL0=0X00000000;

	PINSEL1 = 0x01000000;

	Uart0Init();

	AD0CR = 0x00210402;

  while(1)

	{

		if(AD0DR1 & 0x80000000)

		{			

		adcdata = (AD0DR1 & 0x0000FFC0);

		adcdata = adcdata>>6;

            

			voltage = ((adcdata/1023.0) * 3.3 );

			sprintf(volt, "%.1f",voltage);

			Uart0PutS("\n ADC o/p: ");

			Uart0PutS(volt);

			Uart0PutS("\n");

			delay();

					

			if(adcdata >0x1FF)

				{

					IO0DIR = 1<<BUZZ;

					{

						IOCLR0 = 1<<BUZZ;// Buzzer ON

						delay();

						IOSET0 = 1<<BUZZ; // Buzzer OFF

						delay();

					}

				}

				else

				{

				IOSET0 = 1<<BUZZ; //Buzzer OFF

				}

		}

	}

}

```

## 6. Buzzer

**Interface buzzer with LPC2148 and turn on and off buzzer**	

```

#include <LPC214x.h>

#define BUZZ 4

void Delay(void);

void Wait(void);

void main()

{

	// PINSEL1 = 0x00;

	IO0DIR = 1 << BUZZ;

	// Configure Port0.4 as GPIO IODIR0 = 3 << BUZZ;

	// Configure Port0.4 as O/P pin

	while (1)

	{

		IOSET0 = 1 << BUZZ;

		Delay();

		IOCLR0 = 1 << BUZZ;

		Delay();

	}

}

void Delay()

{

	unsigned int i, j;

	for (i = 0; i < 1000; i++)

		for (j = 0; j < 7000; j++)

			;

}

```

## 7. UART

**LPC2148 UART Interfacing LPC2148 to display “Hello World” on terminal in embedded system**

```

#include <LPC214x.h>

void Uart0Init(void) // Initialize Serial Interface

{

	PINSEL0 = 0x00000005; // Enable TxD0(P0.0) & RxD0(P0.1)

	U0LCR = 0x83;		  // 8 bits data, No parity, 1 Stop bit

	U0DLL = 97;			  // 9600 Baud rate @ 15MHz PCLK

	U0LCR = 0x03;

	U0DLM = 0;

}

unsigned char Uart0GetCh(void) // Read a character from RxD

{

	while (!(U0LSR & 0x01))

		;

	return (U0RBR);

}

void Uart0PutCh(unsigned char ch) // Write a character to TxD

{

	U0THR = ch;

	while (!(U0LSR & 0x20))

		;

}

void Uart0PutS(unsigned char *str)

{

	while (*str)

	{

		Uart0PutCh(*str++);

	}

}

```

## 8. LED timer

**Write embedded C program to use timer block of LPC 2148 to generate suitable delay to toggle LEDs.**

```

#include<LPC214x.h>

#include"pll.h"

int main(void)

{

	IO1DIR = IO1DIR | (1<<24);//Make P0.0 an o/p pin

	pll();//Fosc=12MHz, CCLK=60MHz, PCLK=60MHz

	while(1)

	{

		IO1SET = IO1SET | (1<<24);

		delayms(500);//0.5s ON

		IO1CLR = IO1CLR | (1<<24);

		delayms(500);

	}

}

	

```

## 9. LED blink

**Write embedded C program to interface LED blink with LPC2148**

```

#include <lpc214x.h>

unsigned int delay;

int main(void)

{

  IO0DIR = (0xFF << 15); // Configure P0.15 to P0.22 as Output

  while (1)

  {

    IO0CLR = (0xFF << 15); // CLEAR P0.15-P0.22 to turn LED ON

    for (delay = 0; delay < 500000; delay++)

      ;                    // delay

    IO0SET = (0xFF << 15); // SET  P0.15-P0.22 to turn LED ON

    for (delay = 0; delay < 500000; delay++)

      ; // delay

  }

}

```
