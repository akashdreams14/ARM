#include "lcd_headerfile.c"
#define C0   (IOPIN1&(1<<20))
#define C1   (IOPIN1&(1<<21))
#define C2   (IOPIN1&(1<<22))
#define C3   (IOPIN1&(1<<23))

#define R0    (1<<16)
#define R1    (1<<17)
#define R2    (1<<18)
#define R3    (1<<19)
unsigned int key_lut[4][4]={{1,2,3,4},{5,6,7,8},{9,10,11,12},{13,14,15,16}};
unsigned int KEY_SCAN();
int main(){
	int res;
  LCD_INIT();
	LCD_STR("4X4 KEYPAD");
	while(1){
		LCD_CMD(0XC0);
		res=KEY_SCAN();
		LCD_DATA((res/10)+48);
		LCD_DATA((res%10)+48);
		delay_ms(20);
		LCD_CMD(0X01);
}	
}
unsigned int KEY_SCAN(void){
unsigned char row_value,col_value;
	PINSEL2|=0X00000000;
	IODIR1|=R0|R1|R2|R3;
	while(1){
	IOCLR1|=R0|R1|R2|R3;
	IOSET1|=C0|C1|C2|C3;
	while((C0&&C1&&C2&&C3)==1);
		//TESTING FOR ROW 0
		IOCLR1|=R0;
		IOSET1|=R1|R2|R3;
		if((C0&&C1&&C2&&C3)==0){
    row_value=0;
			break;
}
//TESTING FOR ROW 1
		IOCLR1|=R1;
		IOSET1|=R0|R2|R3;
		if((C0&&C1&&C2&&C3)==0){
    row_value=1;
			break;
}
//TESTING FOR ROW 2
		IOCLR1|=R2;
		IOSET1|=R0|R1|R3;
		if((C0&&C1&&C2&&C3)==0){
    row_value=2;
			break;
}
//TESTING FOR ROW 3
		IOCLR1|=R3;
		IOSET1|=R0|R1|R2;
		if((C0&&C1&&C2&&C3)==0){
    row_value=3;
			break;
}
if(C0==0)
	col_value=0;
else if(C1==0)
	col_value=1;
else if(C2==0)
	col_value=2;
else 
	col_value=3;

delay_ms(200);
while((C0&&C1&&C2&&C3)==0);
return key_lut[row_value][col_value];
		
	}
}

