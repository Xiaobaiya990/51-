#include <STC89C5xRC.H>

#include <intrins.h>


// 1S = 1000MS = 1000000US 

sbit DJ_1 = P1^0;//电机的四个IO口
sbit DJ_2 = P1^1;
sbit DJ_3 = P1^2;
sbit DJ_4 = P1^3;
sbit PWM_1 = P1^6;
sbit PWM_2 = P1^7;


sbit D1 = P0^0;
sbit D2 = P0^1;
sbit D3 = P0^2;
sbit D4 = P0^3;
sbit D5 = P0^4;
sbit D6 = P0^5;
sbit D7 = P0^6;


unsigned char Counter_0,Compare_R;	//计数值和比较值，用于输出PWM_1
unsigned char Compare_L;	//计数值和比较值，用于输出PWM_2


void delay_ms(unsigned char ms)
{
     unsigned int i;
	 do{
	      i = 13000 / 13000;
		  while(--i)	;   //14T per loop
     }while(--ms);
}

void delay_500ms()
{
	delay_ms(250);delay_ms(250);
}

void delay_1s()
{
	delay_ms(250);delay_ms(250);delay_ms(250);delay_ms(250);
}

void Delay10ms()		//@33.000MHz
{
	unsigned char i, j;

	i = 54;
	j = 125;
	do
	{
		while (--j);
	} while (--i);
}




#define  Stop DJ_1 = 0;DJ_2 = 0;DJ_3 = 0; DJ_4 = 0 //停
#define  For  DJ_1 = 0;DJ_2 = 1;DJ_3 = 0; DJ_4 = 1 //前进
#define  Bof  DJ_1 = 1;DJ_2 = 0;DJ_3 = 1; DJ_4 = 0 //后退
#define  Left  DJ_1 = 0;DJ_2 = 1;DJ_3 = 0; DJ_4 = 0 //左转
#define  Right DJ_1 = 0;DJ_2 = 0;DJ_3 = 0; DJ_4 = 1 //右转
/**
  * @brief  定时器0初始化，100us@12.000MHz
  * @param  无
  * @retval 无
  */
void Timer0_Init(void)
{
	TMOD &= 0xF0;		//设置定时器模式
	TMOD |= 0x10;		//设置定时器模式
	TL0 = 0x9C;		//设置定时初值
	TH0 = 0xFF;		//设置定时初值
	TF0 = 0;		//清除TF0标志
	TR0 = 1;		//定时器0开始计时
	ET0=1;
	EA=1;
	PT0=0;
}


void Timer0_Routine() interrupt 1
{
	TL0 = 0x9C;		//设置定时初值
	TH0 = 0xFF;		//设置定时初值
	Counter_0++;
	Counter_0%=100;	//计数值变化范围限制在0~99
	if(Counter_0<Compare_R)	//计数值小于比较值
	{
		PWM_1 = 1;		//输出1
	}
	else				//计数值大于比较值
	{
		PWM_1 = 0;		//输出0
	}
	if(Counter_0<Compare_L)	//计数值小于比较值
	{
		PWM_2 = 1;		//输出1
	}
	else				//计数值大于比较值
	{
		PWM_2 = 0;		//输出0
	}
}

void motor(unsigned int Speed_R,unsigned int Speed_L)
{
	Compare_R=Speed_R;
	Compare_L=Speed_L;
}


void Dir_Cofig_V40()
{
	if(D4==0)//前进
	{
		motor(40,40);
		For;
	}
	else if(D3==0)
	{
		motor(40,40);
		For;
	}
	else if(D5==0)
	{
			motor(40,40);
			For;
	}
	else if(D1==0 && D2==1 && D3==1 && D4==1 && D5==1 && D6 == 1 && D7 == 1) //左转算法
	{
		motor(40,40);
		Left;
	}
	else if(D1==1 && D2==0 && D3==1 && D4==1 && D5==1 && D6 == 1 && D7 == 1)
	{
		motor(40,40);
		Left;
	}
	else if(D1==0 && D2==0 && D3==1 && D4==1 && D5==1 && D6 == 1 && D7 == 1)
	{
			motor(40,40);
			Left;
	}
	else if(D1==1 && D2==1 && D3==1 && D4==1 && D5==1 && D6 == 0 && D7 == 1)//右移算法
	{
		motor(40,40);
		Right;
	}
	else if(D1==1 && D2==1 && D3==1 && D4==1 && D5==1 && D6 == 1 && D7 == 0)
	{
		motor(40,40);
		Right;
	}
	else if(D1==1 && D2==1 && D3==1 && D4==1 && D5==1 && D6 == 0 && D7 == 0)
	{
		motor(40,40);
			Right;
	}
	else //自动矫正
	{
		motor(25,25);
		Bof;
	}
}


int main()
{
	delay_1s();
	Timer0_Init();
	while(1)
	{
		motor(50,50);
		Dir_Cofig_V40();
		Delay10ms();
	}
}


