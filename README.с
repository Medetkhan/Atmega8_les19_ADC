# Atmega8_19_Lesson

/*
 * Les_19_MAL.c
 *Изучение АЦП и прерывания в АЦП, подключение датчиков температуры TMP36 и вывод средней температуры с нескольких датчиков
 * Created: 11/6/2021 2:15:29 AM
 * Author : User
 */ 


#define F_CPU 1000000UL
#include <avr/io.h>
#include <util/delay.h>
#include <avr/interrupt.h>

#define  SEG7_DDR DDRD
#define  SEG7_PORT PORTD

#define VREF 5.0
#define coef_div 10.0




int r1_100 = 0;
int r2_10 = 0;
int r3_1 = 0;

int z = 0;

float voltage=0;
int volti=10;//for conversion equation
long ADC_result=0;
int zbl=0;



int digits [10] =
{
	0b00111111, //0
	0b00000110, //1
	0b01011011, //2
	0b01001111, //3
	0b01100110, //4
	0b01101101, //5
	0b01111101, //6
	0b00000111, //7
	0b01111111, //8
	0b01101111  //9
};

void chislo (unsigned int vse_chislo)
{
	r1_100 = vse_chislo/100;    // сотни
	r2_10  = vse_chislo%100/10; // десятки
	r3_1   = vse_chislo%10;		// единицы
}

void ADC_channel(int channel)
{
	switch(channel)
	{
		case 0: ADMUX &=~((1<<MUX3)|(1<<MUX2)|(1<<MUX1)|(1<<MUX0)); break;
		case 1: ADMUX &=~((1<<MUX3)|(1<<MUX2)|(1<<MUX1)); ADMUX|=(1<<MUX0); break;
		case 2: ADMUX &=~((1<<MUX3)|(1<<MUX2)|(1<<MUX0)); ADMUX|=(1<<MUX1); break;
		case 3: ADMUX &=~((1<<MUX3)|(1<<MUX2)); ADMUX|=(1<<MUX1)|(1<<MUX0); break;
	}
}


ISR(TIMER0_OVF_vect)
{
	
	//TCNT0 = 127;
	if(++z>6) z = 1;
	
	if (z == 2)
	{
		PORTB |= (1<<2) | (1<<1);	// Выключаем 2 и 3 разряды
		PORTB &= ~(1<<0);			// ВКлючаем 1 разряд
		SEG7_PORT = digits [r1_100];// Сотни
// 		if (voltage<10) PORTD|=(1<<7);
// 		else PORTD&=~(1<<7);
		
		
	} 
	else if (z == 4)
	{
		PORTB |= (1<<2) | (1<<0);	// Выключаем 1 и 3 разряды
		PORTB &= ~(1<<1);			// ВКлючаем 2 разряд
		SEG7_PORT = digits [r2_10];	// Десятки
// 		if (voltage>10) PORTD|=(1<<7);
// 		else PORTD&=~(1<<7);
	} 
	else if (z == 6)
	{
		PORTB |= (1<<1) | (1<<0);	// Выключаем 1 и 2 разряды
		PORTB &= ~(1<<2);			// ВКлючаем 3 разряд
		SEG7_PORT = digits [r3_1];	// Единицы
	}
	
	
}

ISR(ADC_vect)
{
	ADC_result+=ADC;
	zbl++;
	
	if (zbl>3)
	{
		chislo(((float)ADC_result/4*VREF/1023-0.5)*100);
		if ((((float)ADC_result/4*VREF/1023-0.5)*100)>30) PORTB&=~(1<<7);
		if ((((float)ADC_result/4*VREF/1023-0.5)*100)<25) PORTB|=(1<<7);
		zbl=0;
		ADC_result=0;		
	}
	ADC_channel(zbl);
	
	ADCSRA |= (1<<ADSC); // Запуск преобразования АЦП
}


void ADC_settings(void)
{
// 	DDRB |= (1<<1) | (1<<0);
// 	PORTB &= ~((1<<1) | (1<<0));

    
	
	DDRC &= ~((1<<3)|(1<<2)|(1<<1)|(1<<0)); // 4 е канала АЦП
	
	ADCSRA |= (1<<ADEN); // Включаем АЦП
	//ADCSRA |= (1<<ADFR); // Непрырывное измерение
	// Частота дискретизации 125кГц при Fмк=1МГц и коэф. дел. = 8
	ADCSRA &= ~(1<<ADPS2);
	ADCSRA |= (1<<ADPS1) | (1<<ADPS0);
	
	//ADMUX |= (1<<REFS1) | (1<<REFS0); // Внутренний ИОН 2,56 В
	//Внешний ИОН 5В
	ADMUX &= ~(1<<REFS1);
	ADMUX |= (1<<REFS0);
	
	ADCSRA|=(1<<ADIE);//Разрешаем прерыв от АЦП
	
	
	ADMUX &= ~(1<<ADLAR); // Правостороннее выравнивание результата
	
	// Задействуем третий канал АЦП (РС3)
	ADMUX &= ~((1<<3) | (1<<2));
	ADMUX |= (1<<1) | (1<<0);
	
	ADCSRA |= (1<<ADSC); // Запуск преобразования АЦП
}

int main(void)
{
	
	SEG7_DDR = 0xff;
	SEG7_PORT = 0x00;
	
	DDRB |= (1<<2) | (1<<1) | (1<<0);
	PORTB |= (1<<2) | (1<<1) | (1<<0);
	
	DDRB|=(1<<7);//LED
	PORTB &= ~(1<<7);
	
	TCCR0 = 0b00000010; // Делитель на 8 0-го ТС
	TCNT0 = 0;
	
	TIMSK = 0b00000001; // Разреш. прерыв при переполнении 0-го ТС
	
	ADC_settings();
	
	sei();
	
	while (1)
	{
		/*chislo(531);*/	
	}
}






