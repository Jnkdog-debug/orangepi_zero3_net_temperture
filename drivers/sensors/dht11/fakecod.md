static unsigned char 
dht11_read8(void)
{
    int i = 0;      
    int j;       
    bool feedback = false;      
    unsigned char data = 0;        
    for(j = 0; j < 8; j++) {                		
        i = 0;  		/* 等待DHT11的引脚变为高电平 */		
        while(gpio_get_value(dht11_dev->dht11_pin) == LOW_LEVEL) {  	
            feedback = false;		
            udelay(10);  						
            if(i++ > 5)  				
             break;  		
        }  			
        udelay(30);              			
        i = 0;  		
        while(gpio_get_value(dht11_dev->dht11_pin) == HIGH_LEVEL) { 
            feedback = true; 			
            udelay(10);  						
            if(i++ > 6)  				
             break;  		
        }  		
        data <<= 1;  		
        data |= feedback;      
    }        
    
    return data;
}

从dht11 读取一个字节

    重复8次 一位一位给data赋值:
     {
        等待DHT11的引脚变为低电平
        如果为高电平则跳出循环
        否则此位为0
        等一会儿确定此信号为低位 至少50微秒

        等待引脚为高电平
        如果为低电平 
        则跳出循环
        否则此位为1
        等一会儿确定此信号为高位 至少60微秒

        给data一位赋值
        返回data
     }  