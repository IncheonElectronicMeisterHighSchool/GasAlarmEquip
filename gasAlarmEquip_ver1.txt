#include <mega128.h>
#include <delay.h>

int cnt=0;
int TestingLed=1;
int Testing=0;
int TestCnt=0;
int DataCnt=0;
int GasTimeCnt=0;
int GasInputFlag=0;
int DataInput[10];
float DataAverage=0;
unsigned int resL;
unsigned int resH;

void GasInput() {
        GasInputFlag=1;        
}

void Control() {  
        int i;
        for (i=0;i<10;i++) {
                DataAverage+=DataInput[i];
        }                                 
        DataAverage=DataAverage/10;
        if (DataAverage>=255) {
                Testing=1;
        }
}

void main(void) {
        DDRA=0x01;
        DDRF=0x00;
        
        TCCR0=0x0C;
        OCR0=249;
        TIMSK=2;
        SREG=0x80;    
        
        ADMUX=0x00;
        ADCSRA=0x87;
                      
        while (1) {
                GasInput();              
                if (DataCnt>=11) {Control();DataCnt=0;}   
        }                
}

interrupt [TIM0_COMP] void TestingInterrupt(void) {
        if (Testing==1) {   
                cnt++;
                if (cnt>=100 && TestCnt<4) {
                        TestCnt++;
                        if (TestingLed==1) {
                                PORTA=0x01;
                                TestingLed=0;
                        }
                        else {
                                PORTA=0x00;
                                TestingLed=1;
                        }
                        cnt=0;
                }
                else if (TestCnt>=4) {
                        Testing=0;
                        TestCnt=0;
                }   
        }
        else {
        cnt=0;
        }     
        GasTimeCnt++;
        if (GasTimeCnt>=100 && GasInputFlag==1) {
                ADCSRA|=0x40;
                while ((ADCSRA&0x10)!=0x10);    
                resL = ADCL;
                resH = ADCH;
                DataInput[DataCnt]=(resH<<8)|resL;    
                DataCnt++; 
                GasTimeCnt=0;
                GasInputFlag=0;
        } 
}