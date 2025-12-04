# ARM
//source code of ARM projects
#include <LPC21xx.h>

#define LCD_DATA 0xFF     // P0.0 – P0.7
#define RS (1 << 8)
#define EN (1 << 9)

int dice = 0;
int count = 0;
unsigned char dice_symbol[][8] =
{
    {0x00,0x00,0x00,0x04,0x00,0x00,0x00,0x00}, // 1
    {0x00,0x10,0x00,0x00,0x00,0x01,0x00,0x00}, // 2
    {0x00,0x10,0x00,0x04,0x00,0x01,0x00,0x00}, // 3
    {0x00,0x11,0x00,0x00,0x00,0x11,0x00,0x00}, // 4
    {0x00,0x11,0x00,0x04,0x00,0x11,0x00,0x00}, // 5
    {0x00,0x15,0x00,0x15,0x00,0x15,0x00,0x00}  // 6
};

void delay() {
		T0PR  = 14;
    T0TCR = 0x01;   // start timer
		while(T0TC<2000);
		T0TCR=0x03;
		T0TCR=0x00;
		
}

void lcd_cmd(unsigned char cmd) {
    IOCLR0 = LCD_DATA;
    IOSET0 = cmd;
    IOCLR0 = RS;

    IOSET0 = EN;
		delay();
    IOCLR0 = EN; 
}

void lcd_data(unsigned char data) {
    IOCLR0 = LCD_DATA;
    IOSET0 = data;
    IOSET0 = RS;

    IOSET0 = EN; 
		delay();
    IOCLR0 = EN;
}

void lcd_string(char *p) {
    while (*p) 
			lcd_data(*p++);
}

void lcd_print_num(int num) {
    lcd_data((num / 10) + '0');
    lcd_data((num % 10) + '0');
}

void lcd_init() {
	
    IODIR0  = LCD_DATA|RS|EN;

    lcd_cmd(0x38);
    lcd_cmd(0x0C);
    lcd_cmd(0x01);
    lcd_cmd(0x06);
}


void lcd_load_dice_symbols()
{
    int d, i;
    for (d = 0; d < 6; d++)
    {
        lcd_cmd(0x40 + (d * 8));    // CGRAM address
        for (i = 0; i < 8; i++)
            lcd_data(dice_symbol[d][i]);
    }
}

void EINT0_ISR(void) __irq
{


    EXTINT = 1 << 0;          // clear EINT0 flag
	    
    dice = (T0TC % 6) + 1;;   // Get random dice value

    count++;  // increment count
    VICVectAddr = 0;         
}


int main() {
		int index=0;
    lcd_init();
		lcd_load_dice_symbols(); 
    
		PINSEL1 = (1 << 0);      // P0.16 -> EINT0 (01)

    VICIntSelect = 0;        // As IRQ 
		VICVectCntl0 = (0x20) | 14;  // slot0, interrupt 14 = EINT0
    VICVectAddr0 = (unsigned long)EINT0_ISR;
   
	  EXTMODE = 1;      // Edge sensitive
		EXTPOLAR = 0;    // Falling edge

    VICIntEnable |= (1 << 14);   // Enable EINT0

    // Titles
    lcd_cmd(0x80);
		lcd_string("Digital Dice ");

    lcd_cmd(0xC0);
    lcd_string("Dice: ");
		lcd_string(" Count: ");
    while (1) {
			
 // show custom animated dice on 0x8E position     
			 lcd_cmd(0x8E);         
        lcd_data(index);      
        delay();

        index++;
        if(index >= 6) index = 0;
			
// Update display continuously based on ISR values
        lcd_cmd(0xC0 + 5);
        lcd_data(dice - 1);
			  lcd_data(' ');          // clear leftover

        lcd_cmd(0xC0 + 14);
        lcd_print_num(count);
    }
}
